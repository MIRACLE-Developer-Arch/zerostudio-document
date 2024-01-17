# CustomMVVM
* Compose Multiplatform 에서 제공해주는 여러 MVVM라이브러리들이 존재하지만, 외주의 상황과 요청에 따라 자유자재로 커스텀하기 위함
* 커스텀을 사용하는 만큼 자유도가 높기 때문에, 외주의 성격과 상황을 잘 고려해서 유동적으로 클래스 내부의 함수나 변수들은 변경될 수 있음
* 아래 예제는 Thanks_CarBon 의 Haimdall 프로젝트에서 사용된 커스텀 모델과 커스텀 뷰모델
<note>
<p>
정답은 없으니 개발해가며 연구하자!
</p>
</note>

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