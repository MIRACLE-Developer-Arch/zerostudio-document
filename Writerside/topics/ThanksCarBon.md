# ThanksCarbon
* 해당 프로젝트의 경우, 유저가 빠르게 파이어베이스의 문서에 접근을 해야하다 보니, Firebase SDK와 클라우드 함수 호출을 유동적으로 사용

## MAP Controller
* 해당 프로젝트의 경우 구글 지도를 이용한 농부별 자신의 농지 위치를 확인할 수 있었어야 했음.
* 따라서 expect/actual 키워드를 통해 구글 지도에 관련된 클래스와 인터페이스를 구축 및 개발 진행하였음

SharedModule)
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
AndroidModule)
```Kotlin
actual abstract class PlatformMapImplementation: MapInterface {
    protected val _autoCompletedPrediction = MutableStateFlow<List<MyAutocompletePrediction>>(emptyList())
    actual val autoCompletePrediction: StateFlow<List<MyAutocompletePrediction>>
        get() = _autoCompletedPrediction
}
```
AndroidProject)
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
{ collapsible="true" }
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

IOSModule)
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
{ collapsible="true" }

SwiftProject)
```Swift
추가바람
```

## Notification Helper
* 해당 프로젝트의 경우 알림을 보낼 수 있었어야 했음.
* 역시 expect/actual 을 통해 알림을 각 플랫폼에 맞게 구축 및 개발 진행하였음  
  SharedModule)
```Kotlin
expect interface NotificationHelper {
    fun areNotificationsEnabled(): Boolean
    fun openNotificationSettings()
    fun isFirstRun(): Boolean
}
```
AndroidModule)
```Kotlin
actual interface NotificationHelper {
    actual fun areNotificationsEnabled(): Boolean
    actual fun openNotificationSettings()
    actual fun isFirstRun(): Boolean
}
```
AndroidProject)
```Kotlin
class AOSNotificationHelper(private val context: Context) : NotificationHelper {
    override fun areNotificationsEnabled(): Boolean {
        return NotificationManagerCompat.from(context).areNotificationsEnabled()
    }

    override fun openNotificationSettings() {
        val intent = Intent().apply {
            action = Settings.ACTION_APP_NOTIFICATION_SETTINGS
            putExtra(Settings.EXTRA_APP_PACKAGE, context.packageName)
        }
        context.startActivity(intent)
    }

    override fun isFirstRun(): Boolean {
        val sharedPreferences = context.getSharedPreferences("MyAppPreferences", Context.MODE_PRIVATE)
        if (sharedPreferences.getBoolean("isFirstRun", true)) {
            sharedPreferences.edit().putBoolean("isFirstRun", false).apply()
            return true
        }
        return false
    }

}
```
IOSModule)
```Kotlin
actual interface NotificationHelper {
    actual fun areNotificationsEnabled(): Boolean
    actual fun openNotificationSettings()
    actual fun isFirstRun(): Boolean
}
```
SwiftProject)
```Swift
추가바람
```

## LocationInterface
* 해당 프로젝트의 경우 사진을 촬영했을 때, 사진을 촬영한 위치의 정보가 또 필요했음.
* 역시 expect/actual 을 통해 각 플랫폼에 맞게 구축 및 개발 진행  
  SharedModule)
```Kotlin
interface LocationInterface {
    fun getCurrentLocation()
}
expect abstract class PlatformLocationManager(): LocationInterface {
    val lat: StateFlow<Double>
    val long: StateFlow<Double>
}
```
AndroidModule)
```Kotlin
actual abstract class PlatformLocationManager: LocationInterface {
    protected val _lat = MutableStateFlow<Double>(0.0)
    protected val _long = MutableStateFlow<Double>(0.0)
    actual val lat: StateFlow<Double>
        get() = _lat
    actual val long: StateFlow<Double>
        get() = _long

}
```
AndroidProject)
```Kotlin
class LocationManger(private val context: Context): PlatformLocationManager() {
    private lateinit var fusedLocationClient: FusedLocationProviderClient

    fun init() {
        fusedLocationClient = LocationServices.getFusedLocationProviderClient(context)
    }

    override fun getCurrentLocation() {
        if (ContextCompat.checkSelfPermission(context, Manifest.permission.ACCESS_FINE_LOCATION)
            != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(context as Activity, arrayOf(Manifest.permission.ACCESS_FINE_LOCATION), 1)
        } else {
            val task: Task<Location> = fusedLocationClient.lastLocation
            task.addOnSuccessListener { location: Location? ->
                if (location != null) {
                    // 위치가 업데이트될 때 호출.
                    val latitude = location.latitude
                    val longitude = location.longitude
                    // 여기서 위도와 경도를 사용.

                    _lat.value = latitude
                    _long.value = longitude
                    Log.i("LocationManager", "location is $latitude, $longitude")
                }

            }
            task.addOnFailureListener {
                // 위치 정보 가져오기 실패 처리
            }
        }
    }
}
```