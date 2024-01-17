# Basic Module
**[KMM(Kotlin-MultiPlatform-Mobile)](https://kotlinlang.org/docs/multiplatform.html)** 의 가장 큰 장점은 코틀린 언어로 Android, IOS, Desktop, Wearable 등 여러 플랫폼을 단일 코드로 개발이 가능하다는 것이다.

아래는 **[ZeroStudio](https://zerostudio.co.kr/)** 에서 사용중인 KMM의 가장 강력한 기능인 [expect/actual](https://kotlinlang.org/docs/multiplatform-expect-actual.html)을 사용하여 만든 모듈이며, 
모든 제로스튜디오 일원들의 생산성 향상을 목적으로 제작되었음.


## KMM expect/actual
### 토스트메시지

<tabs>
<tab title="Shared">

````Kotlin
expect fun toastMessage(context: Any?, message: String)
````
</tab>
</tabs>
<tabs>
<tab title="Android">

````Kotlin
actual fun toastMessage(context: Any?, message: String) {
    if (context != null) {
        val actualContext = when (context) {
            is Context -> context
            else -> throw IllegalArgumentException("Invalid type for context")
        }
        Toast.makeText(actualContext, message, Toast.LENGTH_SHORT).show()
    }
}
````
{ collapsible="true" default-state="expanded" }
</tab>
<tab title="IOS">

````Kotlin
actual fun toastMessage(context: Any?, message: String) {
    val toastLabel = UILabel()
    toastLabel.backgroundColor = UIColor.clearColor
    toastLabel.textColor = UIColor.blackColor()
    toastLabel.textAlignment = NSTextAlignmentCenter
    toastLabel.text = message
    toastLabel.alpha = 1.0
    toastLabel.layer.cornerRadius = 10.0
    toastLabel.clipsToBounds = true
    toastLabel.numberOfLines = 0

    toastLabel.sizeToFit()

    val maxWidth = UIScreen.mainScreen.bounds.useContents { size.width } * 0.8
    if (toastLabel.frame.useContents { size.width } > maxWidth) {
        toastLabel.frame.useContents {
            size.width = maxWidth
        }
    }

    val window = UIApplication.sharedApplication.keyWindow
    window?.addSubview(toastLabel)
    toastLabel.center = window!!.center

    UIView.animateWithDuration(
        4.0,
        animations = { toastLabel.alpha = 0.0 },
        completion = { _ -> toastLabel.removeFromSuperview() }
    )
}
````
{collapsible="true" default-state="expanded" }
</tab>
</tabs>


### PDF View
<tabs>
<tab title="Shared">

````Kotlin
@Composable
expect fun PDFViewer(
    modifier: Modifier = Modifier,
    url: String
)
````
</tab>
</tabs>
<tabs>
<tab title="Android">

````Kotlin
@Composable
actual fun PDFViewer(
    modifier: Modifier,
    url: String
) {
    val context = LocalContext.current
    var pdfUri by remember { mutableStateOf<Uri?>(null) }
    var tempFile by remember { mutableStateOf<File?>(null) }
    var isLoading by remember { mutableStateOf(true) }
    var loadError by remember { mutableStateOf(false) }

    LaunchedEffect(url) {
        isLoading = true
        loadError = false
        try {
            val result = downloadPdfFromUrl(context, url)
            pdfUri = result.first
            tempFile = result.second
        } catch (e: Exception) {
            loadError = true
        } finally {
            isLoading = false
        }
    }

    DisposableEffect(Unit) {
        onDispose {
            tempFile?.delete()
        }
    }

    Column(modifier) {
        if (isLoading) {
            Text("Loading PDF...")
        } else if (loadError) {
            Text("Error loading PDF.")
        } else if (pdfUri != null) {
            AndroidView(
                factory = { context ->
                    PDFView(context, null).apply {
                        fromUri(pdfUri)
                            .enableDoubletap(true)
                            .spacing(10)
                            .load()
                    }
                },
                modifier = modifier
            )
        }
    }
}

suspend fun downloadPdfFromUrl(context: Context, pdfUrl: String): Pair<Uri, File> {
    return withContext(Dispatchers.IO) {
        val url = URL(pdfUrl)
        val connection = url.openConnection()
        val inputStream = BufferedInputStream(connection.getInputStream())
        val outputFile = File(context.cacheDir, "downloaded_pdf.pdf")
        val outputStream = FileOutputStream(outputFile)

        inputStream.copyTo(outputStream)

        inputStream.close()
        outputStream.close()

        Pair(Uri.fromFile(outputFile), outputFile)
    }
}
````
{ collapsible="true" collapsed-title="actual fun PDFViewer()"}
</tab>
<tab title="IOS">

````Kotlin
@OptIn(ExperimentalForeignApi::class)
@Composable
actual fun PDFViewer(
    modifier: Modifier,
    url: String
) {
    Column {
        if (url == "") {
            Text("Loading PDF...")
        } else {
            UIKitView(
                factory = {
                    val pdfView = PDFView().apply {
                        // URL을 NSURL 객체로 변환
                        val nsURL = NSURL.URLWithString(url)
                        // PDFDocument를 생성하고 PDFView에 할당
                        document = PDFDocument(nsURL!!)
                        // PDFView가 PDF 문서에 자동으로 맞춰지도록 설정
                        autoScales = true
                    }
                    pdfView
                },
                modifier = modifier,
                update = {},
            )
        }
    }
}
````
{ collapsible="true" collapsed-title="actual fun PDFViewer()"}
</tab>
</tabs>

### 디바이스에 설정된 폰트크기에 상관없이 글자 크기 DP로 고정 함수
<tabs>
<tab title="Shared">

````Kotlin
    @Composable
    fun fixedFontSize(dpSize: Dp): TextUnit {
        return with(LocalDensity.current) { dpSize.toSp() }
    }
````
</tab>
</tabs>

* 해당 함수는 SharedUtil 내부에 포함되어있음


## Custom MVVM
* Compose Multiplatform 에서 제공해주는 여러 MVVM라이브러리들이 존재하지만, 외주의 상황과 요청에 따라 자유자재로 커스텀하기 위함
* 커스텀을 사용하는 만큼 자유도가 높기 때문에, 외주의 성격과 상황을 잘 고려해서 유동적으로 클래스 내부의 함수나 변수들은 변경될 수 있음
* 아래 예제는 Thanks_CarBon 의 Haimdall 프로젝트에서 사용된 커스텀 모델과 커스텀 뷰모델  

<tabs>
<tab title="Model">

```Kotlin
interface Model {
    fun onCleared()
    fun getScreenName(): Screen
}
sealed class DataResult {
    data class MOVE(val screen: Screen): DataResult()
    object STAY: DataResult()
    object EXIT: DataResult()
}
sealed class DataState {
    object Idle : DataState()
    object Loading: DataState()
    data class Success(val result: DataResult): DataState()
    data class ERROR(val message: String): DataState()
}
abstract class BaseModel: Model, CoroutineScope {
    private val modelJob = SupervisorJob()

    protected val _dataState = MutableStateFlow<DataState>(DataState.Idle)
    val dataState: StateFlow<DataState> get() = _dataState

    override val coroutineContext: CoroutineContext
        get() = Dispatchers.IO + modelJob

    override fun onCleared() {
        modelJob.cancel()
    }
}
```
{ collapsible="true" collapsed-title="abstract class BaseModel: Model, CoroutineScope" default-state="expanded"}
</tab>

<tab title="ViewModel">

```Kotlin
interface ViewModel {
    fun init()
    fun onCleared()
    fun getScreenName(): Screen
}
abstract class BaseViewModel(val model: BaseModel?): ViewModel, CoroutineScope {
    private val viewModelJob = SupervisorJob()
    protected open val _viewState = MutableStateFlow<Screen?>(null) //각 뷰모델에서 다른 뷰로 이동 추적을 담당
    protected open val _closeCurrentView = MutableStateFlow<Boolean>(false)
    protected val _isLoading = MutableStateFlow<Boolean>(false)// 각 뷰에서 로딩 다이얼로그 상태를 담당
    protected val _toastMessage = MutableSharedFlow<String?>(replay = 0)
    val viewState: StateFlow<Screen?> get() = _viewState
    val isLoading: StateFlow<Boolean> get() = _isLoading
    val message: SharedFlow<String?> get() = _toastMessage
    val closeCurrentView: StateFlow<Boolean> get() = _closeCurrentView

    override val coroutineContext: CoroutineContext
        get() = Dispatchers.Main + viewModelJob

    override fun onCleared() {
        _viewState.value = null
        _isLoading.value = false
        launch {
            _toastMessage.emit(null)
        }
        viewModelJob.cancel()
        model?.onCleared()
    }

    override fun getScreenName(): Screen {
        return model?.getScreenName()!!
    }

    fun updateLoadingState(isLoading: Boolean) {
        _isLoading.value = isLoading
    }
    fun startNewView(screen: Screen) {
        _viewState.value = screen
    }
    fun closeCurrentView() {
        _closeCurrentView.value = true
    }

    fun handleToastMessage(message: String) {
        launch {
            _toastMessage.emit(message)
        }
    }
    fun handleError(message: String) {
        _isLoading.value = false
        launch {
            _toastMessage.emit(message)
        }

    }
}
```
{ collapsible="true" collapsed-title="abstract class BaseViewModel(val model: BaseModel?)" default-state="expanded"}
</tab>
</tabs>


<tabs>
<tab title="ModelFactory">

```Kotlin
interface ModelFactory {
    fun createModel(screen: Screen): Model?
    fun clearModel()
}
class ConcreteModelFactory(
    private val firebaseOperations: FirebaseOperations,
    private val appleLoginClient: PlatformAppleLoginClient?,
    private val notificationHelper: NotificationHelper,
    private val locationInterface: PlatformLocationManager
): ModelFactory {
    private val sharedFirebaseController = SharedFirebaseController(firebaseOperations)
    override fun createModel(screen: Screen): Model? {
        return when (screen) {
            Screen.SPLASH -> Model_Splash(sharedFirebaseController)
            Screen.LOGIN -> Model_Login(sharedFirebaseController, appleLoginClient)
            Screen.MAIN -> Model_Main(sharedFirebaseController, locationInterface, notificationHelper)
            Screen.AGREEMENT -> Model_Agreement(sharedFirebaseController)
            Screen.USER_REGISTER_FIND_BY_NAME -> Model_UserRegister_FindByName(sharedFirebaseController)
            Screen.USER_REGISTER -> Model_UserRegister(sharedFirebaseController)
            is Screen.PLANTING_RICE_INFO -> {
                (if (screen.selectedFarmLandIdx != null) screen.selectedFarmLandIdx else null)?.let {
                    Model_PlantingRiceInfo(sharedFirebaseController, it)
                }
            }
            Screen.FARM_INFO -> Model_FarmInfo(sharedFirebaseController)
            Screen.ADD_FARMLAND -> Model_AddFarmLand(sharedFirebaseController)
            Screen.MY_PROFILE -> Model_MyProfile(sharedFirebaseController, notificationHelper)
            is Screen.PDF -> { Model_PDF(sharedFirebaseController, screen.type) }
            is Screen.CHANGE_WATERING_DATE -> {
                (if (screen.selectedFarmLandIdx != null) screen.selectedFarmLandIdx else null)?.let {
                    Model_ChangeWateringDate(sharedFirebaseController, it)
                }
            }
        }
    }

    override fun clearModel() {}
}
```
{ collapsible="true" collapsed-title="class ConcreteModelFactory()" default-state="expanded"}

</tab>

<tab title="ViewModelFactory">

```Kotlin
class ViewModelFactory(
    private val modelFactory: ModelFactory,
    private val mediaPickerController: MediaPickerController?,
    private val platformMapImplementation: PlatformMapImplementation?
) {
    private val viewModelCache = mutableMapOf<Screen, BaseViewModel?>()

    fun create(screen: Screen): BaseViewModel? {
        return viewModelCache[screen] ?: createViewModel(screen).also { baseViewModel ->
            viewModelCache[screen] = baseViewModel
        }
    }

    fun clearViewModel(screen: Screen) {
        viewModelCache[screen]?.onCleared()
        viewModelCache.remove(screen)
    }

    fun clearAllViewModel() {
        if (viewModelCache != null) {
            viewModelCache.values.forEach { baseViewModel ->
                baseViewModel?.onCleared()
            }
            viewModelCache.clear()
            modelFactory.clearModel()
        }
    }

    private fun createViewModel(screen: Screen) : BaseViewModel? {
        val model = modelFactory.createModel(screen)

        return when (screen) {
            Screen.SPLASH -> ViewModel_Splash(model as Model_Splash)
            Screen.LOGIN -> ViewModel_Login(model as Model_Login, mediaPickerController)
            Screen.MAIN -> ViewModel_Main(model as Model_Main, mediaPickerController!!)
            Screen.AGREEMENT -> ViewModel_Agreement(model as Model_Agreement)
            Screen.USER_REGISTER_FIND_BY_NAME -> ViewModel_UserRegister_FindByName(model as Model_UserRegister_FindByName)
            Screen.USER_REGISTER -> ViewModel_UserRegister(model as Model_UserRegister, platformMapImplementation)
            is Screen.PLANTING_RICE_INFO -> ViewModel_PlantingRiceInfo(model as Model_PlantingRiceInfo)
            Screen.FARM_INFO -> ViewModel_FarmInfo(model as Model_FarmInfo, platformMapImplementation!!)
            Screen.ADD_FARMLAND -> ViewModel_AddFarmLand(model as Model_AddFarmLand, platformMapImplementation!!)
            Screen.MY_PROFILE -> ViewModel_MyProfile(model as Model_MyProfile)
            is Screen.PDF -> ViewModel_PDF(model as Model_PDF)
            is Screen.CHANGE_WATERING_DATE -> ViewModel_ChangeWateringDate(model as Model_ChangeWateringDate)
        }
    }
}
```
{ collapsible="true" default-state="expanded"}
</tab>
</tabs>



### KMM 프로젝트에서 공유모듈에서 이미지랑 폰트를 불러와서 사용하는 방법  


Start
:
1. [MOKO-resources](https://github.com/icerockdev/moko-resources) 라이브러리를 필요로 함
2. sharedModule root 디렉터리에 resources폴더를 추가로 생성해줘야함
3. resources폴더 내부 MR폴더 생성, 필요한 타입의 폴더를 생성해줌
4. 각 폴더에 필요한 파일을 두고 IDE에서 Sync를 수행하면 자동으로 임포트됨

<note>
  <p>
    찾을 수 없다고 빨간줄이 뜰텐데, Sync를 수행했으면 실행은 제대로 되니 겁먹지 말자
  </p>
</note>


폴더 계층 구조는 다음과 같다

```text
shared
  │
  ├── src
  │    └── commonMain
  │           ├── kotlin
  │           └── resources
  │                 └── MR
  │                     ├── fonts/ <- .ttf 파일들이 위치함
  │                     └── images/ <- .svg 또는 .png 파일들이 위치함
```
<tabs>
<tab title="Image">

```Kotlin
@Composable
fun loadImageResource(imageFile: ImageFile) : Painter {
    val resId = when (imageFile) {
        ImageFile.GOOGLE_LOGIN -> MR.images.android_light_sq_SI
        ImageFile.APPLE_LOGIN -> MR.images.appleid_button
        ImageFile.ADD -> MR.images.add
        ImageFile.CANCEL -> MR.images.cancel
        ImageFile.HOME -> MR.images.home
        ImageFile.LOCATION -> MR.images.location
        ImageFile.PERSON -> MR.images.person
        ImageFile.RIGHT_ARROW -> MR.images.rightArrow
        ImageFile.CAMERA -> MR.images.addPhoto
        ImageFile.GALLERY -> MR.images.photoLibrary
    }

    return painterResource(resId)
}
```
</tab>
<tab title="Font">

```Kotlin
@Composable
fun loadFontResource(fontStyle: FontStyle) : FontFamily {
    val fontId = when (fontStyle) {
        FontStyle.EXTRA_BOLD -> MR.fonts.Pretendard.extraBold
        FontStyle.BOLD -> MR.fonts.Pretendard.bold
        FontStyle.LIGHT -> MR.fonts.Pretendard.light
        FontStyle.REGULAR -> MR.fonts.Pretendard.regular
        FontStyle.SEMI -> MR.fonts.Pretendard.semiBold
        FontStyle.MEDIUM -> MR.fonts.Pretendard.medium
    }
    return fontFamilyResource(fontId)
}
```
</tab>
</tabs>
