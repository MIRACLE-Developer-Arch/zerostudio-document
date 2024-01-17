# InApp Purchase
* INCLER 프로젝트에서 처음 생성됨
* 구글 / 앱스토어 인앱 결제가 필요했음


{ collapsible = true default-state = collapsed }
How To Start Android
:
1. 앱 수준의 build.gradle 세팅
```text
implementation("com.android.billingclient:billing-ktx:6.1.0")
```
2. Manifest 추가
```text
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="com.android.vending.BILLING"/>
```


How To Start IOS
:
추가바람


<tabs>
<tab title="Shared">

```Kotlin
data class CustomProductDetails(
    val productId: String,
    val name: String,
    val description: String,
    val price: String,
)

expect abstract class PurchaseHelper {
    val coroutineScope: CoroutineScope
    val availableProducts: StateFlow<List<CustomProductDetails>>
    val productName: StateFlow<String>
    val buyEnabled: StateFlow<Boolean>
    val consumeEnabled: StateFlow<Boolean>
    val statusText: StateFlow<String>

    val purchaseCompleted: SharedFlow<PurchaseSaveInfo>
    val clientPurchaseProgress: StateFlow<Boolean>

    abstract fun billingSetup(productIds: List<String>)
    abstract fun queryProduct(productIds: List<String>)
    abstract fun makePurchase(selectedProductId: String)
    abstract fun consumePurchase()
    abstract fun resetAll()
}
```
</tab>
</tabs>
<tabs>
<tab title="Android">

```Kotlin
actual abstract class PurchaseHelper {
    actual val coroutineScope: CoroutineScope
        get() = CoroutineScope(Dispatchers.IO)

    protected open val _availableProducts = MutableStateFlow<List<CustomProductDetails>>(emptyList())
    protected open val _productName = MutableStateFlow("Searching")
    protected open val _buyEnabled = MutableStateFlow(false)
    protected open val _statusText = MutableStateFlow("Initializing...")
    protected open val _consumeEnabled = MutableStateFlow(false)
    protected open val _purchaseCompleted = MutableSharedFlow<PurchaseSaveInfo>(replay = 0)
    protected open val _clientPurchaseProgress = MutableStateFlow(false)
    actual val availableProducts: StateFlow<List<CustomProductDetails>>
        get() = _availableProducts
    actual val productName: StateFlow<String>
        get() = _productName
    actual val buyEnabled: StateFlow<Boolean>
        get() = _buyEnabled
    actual val consumeEnabled: StateFlow<Boolean>
        get() = _consumeEnabled
    actual val statusText: StateFlow<String>
        get() = _statusText
    actual val purchaseCompleted: SharedFlow<PurchaseSaveInfo>
        get() = _purchaseCompleted
    actual val clientPurchaseProgress: StateFlow<Boolean>
        get() = _clientPurchaseProgress

    actual abstract fun billingSetup(productIds: List<String>)
    actual abstract fun queryProduct(productIds: List<String>)
    actual abstract fun makePurchase(selectedProductId: String)
    actual abstract fun consumePurchase()
    actual abstract fun resetAll()
}
```
{ collapsible="true" default-state="collapsed" }
</tab>
<tab title="IOS">

```Kotlin
actual abstract class PurchaseHelper {
    actual val coroutineScope: CoroutineScope
        get() = CoroutineScope(Dispatchers.IO)

    protected open val _availableProducts = MutableStateFlow<List<CustomProductDetails>>(emptyList())
    protected open val _productName = MutableStateFlow("Searching")
    protected open val _buyEnabled = MutableStateFlow(false)
    protected open val _statusText = MutableStateFlow("Initializing...")
    protected open val _consumeEnabled = MutableStateFlow(false)
    protected open val _purchaseCompleted = MutableSharedFlow<PurchaseSaveInfo>(replay = 0)
    protected open val _clientPurchaseProgress = MutableStateFlow(false)
    actual val availableProducts: StateFlow<List<CustomProductDetails>>
        get() = _availableProducts
    actual val productName: StateFlow<String>
        get() = _productName
    actual val buyEnabled: StateFlow<Boolean>
        get() = _buyEnabled
    actual val consumeEnabled: StateFlow<Boolean>
        get() = _consumeEnabled
    actual val statusText: StateFlow<String>
        get() = _statusText
    actual val purchaseCompleted: SharedFlow<PurchaseSaveInfo>
        get() = _purchaseCompleted
    actual val clientPurchaseProgress: StateFlow<Boolean>
        get() = _clientPurchaseProgress

    actual abstract fun billingSetup(productIds: List<String>)
    actual abstract fun queryProduct(productIds: List<String>)
    actual abstract fun makePurchase(selectedProductId: String)
    actual abstract fun consumePurchase()
    actual abstract fun resetAll()
}
```
{ collapsible="true" default-state="collapsed" }
</tab>
</tabs>
<tabs>
<tab title="AndroidProject">

```Kotlin
data class AOSPurchaseHelper(val activity: Activity): PurchaseHelper() {

    private lateinit var billingClient: BillingClient
    private lateinit var purchase:Purchase

    private val originalProductDetailsMap = mutableMapOf<String, ProductDetails>()


    private val purchasesUpdateListener = PurchasesUpdatedListener { billingResult, purchases ->
        if (billingResult.responseCode == BillingClient.BillingResponseCode.OK && purchases != null) {
            for (purchase in purchases) {
                completePurchase(purchase)
            }
        } else if (billingResult.responseCode == BillingClient.BillingResponseCode.USER_CANCELED) {
            _statusText.value = "Purchase Canceled"
        } else {
            _statusText.value = "Purchase Error"
        }
    }
    private val purchasesListener = PurchasesResponseListener { billingResult, purchases ->
        if (purchases.isNotEmpty()) {
            purchase = purchases.first()
            _buyEnabled.value = false
            _consumeEnabled.value = true
            _statusText.value = "Previous Purchase Found"
        } else {
            _buyEnabled.value = true
            _consumeEnabled.value = false
        }
    }

    @OptIn(ExperimentalCoroutinesApi::class)
    override fun resetAll() {
        _availableProducts.value = emptyList()
        _purchaseCompleted.resetReplayCache()
        billingClient.endConnection()
    }
    override fun makePurchase(selectedProductId: String) {
        val selectedProductDetails = originalProductDetailsMap[selectedProductId]

        selectedProductDetails?.let { productDetails ->
            val billingFlowParams = BillingFlowParams.newBuilder()
                .setProductDetailsParamsList(
                    immutableListOf(
                        BillingFlowParams.ProductDetailsParams.newBuilder()
                            .setProductDetails(productDetails)
                            .build()
                    )
                )
                .build()
            billingClient.launchBillingFlow(activity, billingFlowParams)
        } ?: run {
            _statusText.value = "Selected product not found"
        }
    }

    override fun consumePurchase() {
        val consumeParams = ConsumeParams.newBuilder()
            .setPurchaseToken(purchase.purchaseToken)
            .build()

        coroutineScope.launch {
            val result = billingClient.consumePurchase(consumeParams)

            if (result.billingResult.responseCode == BillingClient.BillingResponseCode.OK) {
                _statusText.value = "Purchase Consumed"
                _buyEnabled.value = true
                _consumeEnabled.value = false
            }
        }
    }

    private fun completePurchase(item: Purchase) {
        purchase = item
        if (purchase.purchaseState == Purchase.PurchaseState.PURCHASED) {
            _buyEnabled.value = false
            _consumeEnabled.value = true
            Log.i("PURCHASE HELPER", "purchase Completed! ${purchase.products.size}")
            Log.i("PURCHASE HELPER", "purchase Completed! ${purchase.orderId}")
            Log.i("PURCHASE HELPER", "purchase Completed! ${purchase.purchaseState}")
            Log.i("PURCHASE HELPER", "purchase Completed! ${purchase.purchaseTime}")
            Log.i("PURCHASE HELPER", "purchase Completed! ${purchase.purchaseToken}")
            coroutineScope.launch {
                if (purchase.products.size < 2) {
                    _purchaseCompleted.emit(
                        PurchaseSaveInfo(
                            ProductID = purchase.products[0],
                            OrderID = purchase.orderId,
                            PurchaseState = purchase.purchaseState,
                            PurchaseTime = purchase.purchaseTime,
                            PurchaseToken = purchase.purchaseToken,
                        )
                    )
                }
            }
            _statusText.value = "Purchase Completed"
        }
    }


    override fun billingSetup(productIds: List<String>) {
        billingClient = BillingClient.newBuilder(activity)
            .setListener(purchasesUpdateListener)
            .enablePendingPurchases()
            .build()

        billingClient.startConnection(object : BillingClientStateListener {
            override fun onBillingServiceDisconnected() {
                _statusText.value = "Billing Client Connection Lost"
            }

            override fun onBillingSetupFinished(billingResult: BillingResult) {
                if (billingResult.responseCode == BillingClient.BillingResponseCode.OK) {
                    _statusText.value = "Billing Client Connected"
//                    val demoProductIds = listOf(demoProductId1, demoProductId2, demoProductId3, demoProductId4)
                    queryProduct(productIds)
                    reloadPurchase()
                } else {
                    _statusText.value = "Billing Client Connection Failure"
                }
            }

        })
    }

    override fun queryProduct(productIds: List<String>) {
        val productList = productIds.map { productId ->
            QueryProductDetailsParams.Product.newBuilder()
                .setProductId(productId)
                .setProductType(BillingClient.ProductType.INAPP)
                .build()
        }
        val queryProductDetailsParams = QueryProductDetailsParams.newBuilder()
            .setProductList(productList)
            .build()

        billingClient.queryProductDetailsAsync(queryProductDetailsParams) { billingResult, productDetailList ->
            if (productDetailList.isNotEmpty()) {
                productDetailList.forEach { productDetails ->
                    originalProductDetailsMap[productDetails.productId] = productDetails
                }

                val customProductDetailsList = productDetailList.map { productDetails ->

                    CustomProductDetails(
                        productId = productDetails.productId,
                        name = productDetails.name,
                        description = productDetails.description,
                        price = productDetails.oneTimePurchaseOfferDetails?.formattedPrice ?: ""
                    )
                }
                _availableProducts.value = customProductDetailsList
                _productName.value = "Select a Product"
                _buyEnabled.value = true
            } else {
                _statusText.value = "No Matching Products Found"
                _buyEnabled.value = false
            }
        }
    }
    
    private fun reloadPurchase() {
        val queryPurchasesParams = QueryPurchasesParams.newBuilder()
            .setProductType(BillingClient.ProductType.INAPP)
            .build()

        billingClient.queryPurchasesAsync(queryPurchasesParams, purchasesListener)
    }
}
```
{ collapsible="true" default-state="collapsed" }
</tab>
<tab title="SwiftProject">
추가바람
</tab>
</tabs>