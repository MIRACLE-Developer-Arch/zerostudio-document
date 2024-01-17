# Location Interface
* Thanks Carbon 프로젝트에서 처음 생성됨
* 각 유저별로 사진을 찍었을 그때의 유저 위치를 알아왔어야 했음

{ collapsible = true default-state = collapsed }
How To Start Android
:
1. Manifest에 추가
```text
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
```

How To Start IOS
: 추가바람

<tabs>
<tab title="Shared">

```Kotlin
interface LocationInterface {
    fun getCurrentLocation()
}
expect abstract class PlatformLocationManager(): LocationInterface {
    val lat: StateFlow<Double>
    val long: StateFlow<Double>
}
```
</tab>
</tabs>


<tabs>
<tab title="Android">

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
</tab>
<tab title="IOS">

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
</tab>
</tabs>

<tabs>
<tab title="AndroidProject">

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
{ collapsible="true" default-state="collapsed" }
</tab>
</tabs>