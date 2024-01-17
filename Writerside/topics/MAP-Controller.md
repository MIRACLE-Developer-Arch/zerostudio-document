# MAP Controller
* Thanks Carbon 프로젝트에서 처음 제작됨
* 구글 지도를 이용한 유저별 자신의 농지 위치를 확인할 수 있었어야 했음

{ collapsible = true default-state = collapsed }
How To Start Android
:
1. [Google Cloud 프로젝트 설정](https://developers.google.com/maps/documentation/android-sdk/start?hl=ko#get-key)
2. [앱에 API키 추가](https://developers.google.com/maps/documentation/android-sdk/start?hl=ko#add-key)
3. [모듈 Gradle 파일](https://developers.google.com/maps/documentation/android-sdk/start?hl=ko#module_gradle_file)
4. MainActivity에 아래 내용 추가
```Kotlin
override fun onCreate(savedInstanceState: Bundle?) {
        Places.initializeWithNewPlacesApiEnabled(this, apiKey)
        MapsInitializer.initialize(applicationContext)
}
```

How To Start IOS
:
1. [Maps SDK 설치 - CocoaPods 사용](https://developers.google.com/maps/documentation/ios-sdk/config?hl=ko#download-sdk)
2.
3.



<tabs>
<tab title="Shared">

```Kotlin
interface MapInterface {
    @Composable
    fun RenderMap(latFlow: StateFlow<Double?>, lngFlow: StateFlow<Double?>, color: Color)
    // 기타 필요한 메서드
    fun autoCompleteAddress(query: String)
    fun validateAddressAndRenderMap(address: String)
}


expect abstract class PlatformMapImplementation() : MapInterface {
    val autoCompletePrediction: StateFlow<List<MyAutocompletePrediction>>
}
```
</tab>
</tabs>

<tabs>
<tab title="Android">

```Kotlin
actual abstract class PlatformMapImplementation: MapInterface {
    protected val _autoCompletedPrediction = MutableStateFlow<List<MyAutocompletePrediction>>(emptyList())
    actual val autoCompletePrediction: StateFlow<List<MyAutocompletePrediction>>
        get() = _autoCompletedPrediction
}
```
</tab>
<tab title="IOS">

* IOS의 경우 IOS모듈에서만 설정해주면 됨
```Kotlin
class MapController(): PlatformMapImplementation() {
    @Composable
    override fun RenderMap(latFlow: StateFlow<Double?>, lngFlow: StateFlow<Double?>, color: Color) {
        renderMapByGEO(
            modifier = Modifier.fillMaxSize(),
            latFlow = latFlow,
            lngFlow = lngFlow,
            color = color
        )
    }

    @OptIn(ExperimentalForeignApi::class)
    override fun autoCompleteAddress(query: String) {
        val placesClient = GMSPlacesClient.sharedClient()

        val filter = GMSAutocompleteFilter().apply {
            type = GMSPlacesAutocompleteTypeFilter.kGMSPlacesAutocompleteTypeFilterAddress
        }

        placesClient.findAutocompletePredictionsFromQuery(query, filter, null) { predictions, error ->
            if (error != null) {
                println("Autocomplete error: ${error.localizedDescription}")
                return@findAutocompletePredictionsFromQuery
            }

            val myPredictions = predictions?.map { prediction ->
                convertToMyAutocompletePrediction(prediction as GMSAutocompletePrediction)
            }
            if (myPredictions != null) {
                _autoCompletedPrediction.value = myPredictions
            }
        }
    }

    override fun validateAddressAndRenderMap(address: String) {
        TODO("Not yet implemented")
    }

    @OptIn(ExperimentalForeignApi::class)
    private fun convertToMyAutocompletePrediction(prediction: GMSAutocompletePrediction): MyAutocompletePrediction {
        return MyAutocompletePrediction(
            placeId = prediction.placeID,
            primaryText = prediction.attributedPrimaryText.string,
            secondaryText = prediction.attributedSecondaryText?.string ?: ""
        )
    }
}

@OptIn(ExperimentalForeignApi::class)
@Composable
fun renderMapByGEO(
    modifier: Modifier,
    latFlow: StateFlow<Double?>,
    lngFlow: StateFlow<Double?>,
    color: Color
) {
    val lat by latFlow.collectAsState()
    val lng by lngFlow.collectAsState()

    var isMapRedrawTriggered by remember { mutableStateOf(true) }

    var currentMarker by remember { mutableStateOf<GMSMarker?>(null) }

    val cameraPos = latFlow.value?.let {
        lngFlow.value?.let { it1 ->
            CLLocationCoordinate2DMake(
                latitude = it,
                longitude = it1
            )
        }
    }

    LaunchedEffect(lat, lng) {
        if (lat != null && lng != null) {
            isMapRedrawTriggered = true
        }
    }

    val googleMapView = remember(isMapRedrawTriggered) {
        GMSMapView().apply {
            setMyLocationEnabled(true)
            settings.setScrollGestures(true)
            settings.setZoomGestures(true)
            settings.setCompassButton(false)
            this.setMapStyle(
                GMSMapStyle.styleWithJSONString(
                    mapStyle1(),
                    error = null
                )
            )
        }
    }

    val delegate = remember { object : NSObject(), GMSMapViewDelegateProtocol {
        override fun mapView(
            mapView: GMSMapView,
            @Suppress("PARAMETER_NAME_CHANGED_ON_OVERRIDE")
            didChangeCameraPosition: GMSCameraPosition
        ) {

        }

    }}

    UIKitView(
        modifier = modifier,
        interactive = true,
        factory = {
            googleMapView.apply {
                setDelegate(delegate)
                this.selectedMarker = selectedMarker
                this.setMyLocationEnabled(true)
            }

            googleMapView
        },
        update = { view ->
            if (isMapRedrawTriggered) {
                if (lat != null && lng != null) {
                    currentMarker?.map = null
                    currentMarker = GMSMarker().apply {
                        position = CLLocationCoordinate2DMake(lat!!, lng!!)
                        title = "Farm"
                        snippet = "Lat: $lat, Lng: $lng"
                        icon = markerImageWithColor(color.toUIColor())
                        map = view
                    }

                    view.animateToCameraPosition(
                        GMSCameraPosition.cameraWithLatitude(
                            latitude = lat!!,
                            longitude = lng!!,
                            zoom = 17.0f
                        )
                    )
                }

                isMapRedrawTriggered = false
            }
        }
    )
}
```
{ collapsible="true" default-state="collapsed" }
</tab>
</tabs>

<tabs>
<tab title="Android Project">

```Kotlin
class MapController(val applicationContext: Context): PlatformMapImplementation() {

    private val TAG = "MapController"
    private val placesClient by lazy {
        Places.createClient(applicationContext)
    }



    override fun autoCompleteAddress(query: String) {
        val request = FindAutocompletePredictionsRequest.builder()
            .setQuery(query)
            .build()

        placesClient.findAutocompletePredictions(request)
            .addOnSuccessListener { response ->
                Log.i(TAG, "response is $response")
                val myPredictions = response.autocompletePredictions.map { autocompletePrediction ->
                    convertToMyAutocompletePrediction(autocompletePrediction)
                }
                _autoCompletedPrediction.value = myPredictions
            }
            .addOnFailureListener { response ->
                Log.i(TAG, "response is $response")
            }
    }

    override fun validateAddressAndRenderMap(address: String) {
        TODO("Not yet implemented")
    }

    @SuppressLint("StateFlowValueCalledInComposition")
    @Composable
    override fun RenderMap(latFlow: StateFlow<Double?>, lngFlow: StateFlow<Double?>, color: Color) {
        Log.i(TAG, "color is ${color}")
//        val icon: BitmapDescriptor = remember { bitmapDescriptorFromVector(applicationContext, color.toArgb()) }
        renderMapByGEO(latFlow = latFlow, lngFlow = lngFlow, color, applicationContext)
    }

    private fun convertToMyAutocompletePrediction(prediction: AutocompletePrediction): MyAutocompletePrediction {
        return MyAutocompletePrediction(
            placeId = prediction.placeId,
            primaryText = prediction.getPrimaryText(null).toString(),
            secondaryText = prediction.getSecondaryText(null).toString()
        )
    }
}

@Composable
fun renderMapByGEO(latFlow: StateFlow<Double?>, lngFlow: StateFlow<Double?>, color: Color, context: Context) {
    // Flow에서 방출하는 값을 State로 변환
    val lat by latFlow.collectAsState()
    val lng by lngFlow.collectAsState()
//    val color by colorRGB.collectAsState()

    val cameraPositionState = rememberCameraPositionState()
    val markerState = rememberMarkerState()

    var icon = bitmapDescriptorFromVector(context, color.toArgb())

    SharedUtil.Log("colorRGC is $color")


    LaunchedEffect(lat, lng) {
        if (lat != null && lng != null) {
            val newPosition = CameraPosition.fromLatLngZoom(LatLng(lat!!, lng!!), SharedUtil.cameraZoom)
            cameraPositionState.move(CameraUpdateFactory.newCameraPosition(newPosition))
            markerState.position = LatLng(lat!!, lng!!)
        }
    }
    LaunchedEffect(color) {
        icon = bitmapDescriptorFromVector(context, color.toArgb())
    }

    if (lat != null && lng != null && icon != null) {
        GoogleMap(
            modifier = Modifier.fillMaxSize(),
            cameraPositionState = cameraPositionState
        ) {
            Marker(
                state = markerState,
                title = "CityPosition",
                snippet = "Marker in CintyPosition",
                icon = icon,
                alpha = 1.0f,
                visible = true
            )

        }
    }
}
```
{ collapsible="true" default-state="collapsed" }
</tab>
</tabs>

### Vector 이미지를 BitmapDescriptor 로 변환하는 함수
```Kotlin
fun bitmapDescriptorFromVector(context: Context, tintColor: Int? = null): BitmapDescriptor {
    val vectorDrawable = ContextCompat.getDrawable(context, MR.images.location.drawableResId) as VectorDrawable
    if (tintColor != null) {
        vectorDrawable.apply {
            setTint(tintColor)
            setTintMode(PorterDuff.Mode.SRC_IN)
        }
    }
    vectorDrawable.setBounds(0, 0, vectorDrawable.intrinsicWidth, vectorDrawable.intrinsicHeight)
    val bitmap = Bitmap.createBitmap(SharedUtil.locationIconSize, SharedUtil.locationIconSize, Bitmap.Config.ARGB_8888)
    val canvas = Canvas(bitmap)

    vectorDrawable.setBounds(0, 0, canvas.width, canvas.height)
    vectorDrawable.draw(canvas)


    return BitmapDescriptorFactory.fromBitmap(bitmap)
}
```
{ collapsible="true" default-state="expanded" }