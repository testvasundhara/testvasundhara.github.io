# testvasundhara.github.io


// Subscription & Revanue Cat class

    implementation 'com.revenuecat.purchases:purchases:7.0.0'
    implementation "com.android.billingclient:billing-ktx:6.1.0"

      
       @Keep
       data class BillingProduct(
           var id : String = "",
           var title : String = "",
           var description : String = "",
           var packageType : MyPackageType? = null,
           var price : String = "",
           var amountmicros : Long = 0L,
           var period : String = "",
           var productType : String = BillingClient.ProductType.SUBS,
       )
       @Keep
       data class BillingINAPPProduct(
           var id : String = "",
           var title : String = "",
           var description : String = "",
           var packageType : MyPackageType? = null,
           var price : String = "",
           var amountmicros : Long = 0L,
           var period : String = "",
           var productType : String = BillingClient.ProductType.SUBS,
       )
       
       @Keep
       data class ProductBillingIDS(var id : String, var billingType : String)
       
              
       @Keep
       data class RevenueCatData(
           var id : String,
           var packageName:String,
           var packageType : PackageType,
           var price : String,
           var amountmicros : Long,
           var period : String,
           var freeTrial : FreeTrils,
           var product : Package
       )
       
       @Keep
       data class FreeTrils(var period : Int? = null, var TimeType : String, var i8 : String)
       @Keep
       data class MyOffer(var key : String="", var packages : List<Package> = listOf())



            ----  BillingHelper.class

              
              object BillingHelper {
                  private lateinit var billingClient: BillingClient
                  lateinit var productIds: List<ProductBillingIDS>
                  var myInAppProducts = ArrayList<BillingINAPPProduct>()
              
                  var myProducts = ArrayList<BillingProduct>()
                  var liveMyProducts = MutableLiveData<List<BillingProduct>>()
                  private var isConsumable: Boolean = false
                  var mySubProducts = ArrayList<BillingProduct>()
              
                  fun getBillingInit(
                      billContextAct: Activity,
                      billingKeys: List<ProductBillingIDS>,
                      productPurchaseListener: ProductPurchaseListener
                  ) {
                      StaticParam.productPurchaseListener = productPurchaseListener
                      productIds = billingKeys
                      billContext = billContextAct
                      getBillingHelp()
                  }
              
                  //    var productIds = Arrays.asList("body_editor_monthly", "body_editor_yearly")
              
                  fun getBillingHelp() {
                      val purchasesUpdatedListener = PurchasesUpdatedListener { billingResult, purchases ->
                          "Client Purchase Response : ${billingResult.responseCode}".bilLog()
                      }
                      billingClient = BillingClient.newBuilder(billContext).setListener(purchasesUpdatedListener)
                          .enablePendingPurchases().setListener { billingResult, purchases ->
                              when (billingResult.responseCode) {
                                  BillingClient.BillingResponseCode.OK -> purchases?.forEach {
                                      handlePurchase(it)
                                  }
              
                                  BillingClient.BillingResponseCode.ITEM_ALREADY_OWNED -> {
                                      ("Item Already Owned").log()
                                  }
              
                                  else -> {
                                      when (billingResult.responseCode) {
                                          BillingClient.BillingResponseCode.USER_CANCELED -> {
                                              toastMaker(
                                                  billContext, "You've cancelled the Google play billing process"
                                              )
                                              productPurchaseListener?.onPurchaseFail("You've cancelled the Google play billing process")
                                          }
                                      }
                                      if (billingResult.responseCode !in arrayOf(
                                              BillingClient.BillingResponseCode.USER_CANCELED,
                                              BillingClient.BillingResponseCode.OK
                                          )
                                      ) {
                                          toastMaker(
                                              billContext, "Item not found or Google play billing error"
                                          )
                                          productPurchaseListener?.onPurchaseFail("Item not found or Google play billing error")
                                      }
                                  }
                              }
                          }.build()
              
                      billingClient.startConnection(object : BillingClientStateListener {
                          override fun onBillingSetupFinished(billingResult: BillingResult) {
                              "Client Setup Response : ${billingResult.responseCode == BillingClient.BillingResponseCode.OK}".bilLog()
                              ("Connection ").log("FATZ")
                              billingClient.queryPurchasesAsync(
                                  QueryPurchasesParams.newBuilder().setProductType(BillingClient.ProductType.SUBS)
                                      .build()
                              ) { _, purchases ->
                                  purchases.size.toString().log()
                                  if (purchases.size > 0) {
                                      ("Item Purchase").log("FATZ")
                                      isSubscribedForAllGlobal(billContext, true)
                                      StaticParam.purchaseListener.postValue(true)
                                  } else {
                                      ("Item Not Purchase | Res Code: ${billingResult.responseCode}").log("FATZ")
                                      isSubscribedForAllGlobal(billContext, false)
              //                        isSubscribeNewModul.value = false
                                      if (billingResult.responseCode == BillingClient.BillingResponseCode.OK) {
                                          CoroutineScope(Dispatchers.IO).launch {
                                              subSkuPurchases(billingResult)
                                              inAppSkuPurchases(billingResult)
                                          }
                                      }
                                  }
                              }
                          }
              
                          override fun onBillingServiceDisconnected() {
                              "Client Disconnected ".bilLog()
                          }
                      })
                  }
                  private suspend fun subSkuPurchases(billingResult : BillingResult) {
                      billingClient.queryPurchasesAsync(
                          QueryPurchasesParams.newBuilder().setProductType(BillingClient.ProductType.SUBS).build()
                      ) { _, purchases ->
                          purchases.size.toString().log()
              
                          if(purchases.size > 0) {
                              ("Item Purchase").log("FATZ")
                          } else {
                              ("Item Not Purchase").log("FATZ")
                              if(billingResult.responseCode == BillingClient.BillingResponseCode.OK) {
                                  retriveMySKU()
                              }
                          }
                      }
                  }
              
                  fun getSkuProduct(
                      queryParam: QueryProductDetailsParams, listener: ProductDetailsResponseListener
                  ) {
                      billingClient.queryProductDetailsAsync(queryParam, listener)
              
                  }
              
                  fun makeProductPurchase(packageType: MyPackageType) {
                      if (StaticParam.subsSharedFile.isSubscriber) {
                          toastMaker(billContext, "Already Subscribed!")
                          return
                      }
                      if (packageType == MyPackageType.LIFETIME) {
                          val queryProductDetailsParams = newBuilder().setProductList(
                              listOf(
                                  Product.newBuilder().setProductId(productIds[3].id)
                                      .setProductType(productIds[3].billingType).build()
                              )
                          ).build()
                          getSkuProduct(queryProductDetailsParams) { billingResult, productDetails ->
                              var fetchedProduct = fetchProduct(productIds[3].id, productDetails)
                              fetchedProduct?.let {
                                  val productDetailsParamsList = listOf(
                                      BillingFlowParams.ProductDetailsParams.newBuilder()
                                          // retrieve a value for "productDetails" by calling queryProductDetailsAsync()
                                          .setProductDetails(it)
                                          // For One-time product, "setOfferToken" method shouldn't be called.
                                          // For subscriptions, to get an offer token, call ProductDetails.subscriptionOfferDetails()
                                          // for a list of offers that are available to the user
                                          .build()
                                  )
              
                                  val billingFlowParams = BillingFlowParams.newBuilder()
                                      .setProductDetailsParamsList(productDetailsParamsList).build()
                                  val resultBill = billingClient.launchBillingFlow(billContext, billingFlowParams)
              
                                  when (resultBill.responseCode) {
                                      BillingClient.BillingResponseCode.ITEM_ALREADY_OWNED -> {
                                          "Billing result: ITEM_ALREADY_OWNED".bilLog()
                                          productPurchaseListener?.onPurchase("Product Already Purchased!")
                                      }
              
                                      BillingClient.BillingResponseCode.OK -> {
                                          "Billing result: OK".bilLog()
                                          productPurchaseListener?.onPurchase("Product Available.")
                                      }
              
                                      else -> {
                                          "No Billing result".bilLog()
                                          productPurchaseListener?.onPurchaseFail("No Purchase Made!")
                                      }
                                  }
                              }
                          }
              
              // Launch the billing flow
                      } else {
              
                      }
                  }
              
                  private suspend fun inAppSkuPurchases(billingResult: BillingResult) {
                      billingClient.queryPurchasesAsync(
                          QueryPurchasesParams.newBuilder().setProductType(BillingClient.ProductType.INAPP)
                              .build()
                      ) { _, purchases ->
                          purchases.size.toString().log()
              
                          if (purchases.size > 0) {
                              ("Item Purchase {INAPP}").log("FATZ")
                          } else {
                              ("Item Not Purchase {INAPP}").log("FATZ")
                              if (billingResult.responseCode == BillingClient.BillingResponseCode.OK) {
                                  retriveMySkuInApp()
                              }
                          }
                      }
                  }
              
                  private fun retriveMySkuInApp() {
                      val queryProductDetailsParams = newBuilder().setProductList(
                          listOf(
              //                        Product.newBuilder().setProductId(productIds[0].id).setProductType(productIds[0].billingType).build(),
              //                        Product.newBuilder().setProductId(productIds[1].id).setProductType(productIds[1].billingType).build(),
              //                        Product.newBuilder().setProductId(productIds[2].id).setProductType(productIds[2].billingType).build(),
                              Product.newBuilder().setProductId(productIds[3].id)
                                  .setProductType(productIds[3].billingType).build()
                          )
                      ).build()
                      getSkuProduct(queryProductDetailsParams) { billingResult, productDetailsList ->
              //            Log.e("TAG", "retriveMySkuInApp: \n${productDetailsList.toGson()}\n")
                          productDetailsList.forEach { product ->
                              Log.e("TAG", "retriveMySkuInApp: \n${product.toGson()}\n")
              
                              val billingProduct = BillingINAPPProduct(product.productId)
                              billingProduct.id = product.productId
                              billingProduct.title = product.title
                              billingProduct.description = product.description
                              billingProduct.productType = product.productType
                              // app's Product is One time purchasable, mean it's one kind of LIFETIME For Lifetime Subscription and PRODUCT for single
                              billingProduct.packageType =
                                  if (product.title.contains("Lifetime")) MyPackageType.LIFETIME else MyPackageType.PRODUCT
                              billingProduct.price = product.oneTimePurchaseOfferDetails?.formattedPrice ?: ""
                              billingProduct.amountmicros =
                                  product.oneTimePurchaseOfferDetails?.priceAmountMicros ?: 0L
                              myInAppProducts.add(billingProduct)
                          }
                          SubscriptionSetter.liveMyPlan.postValue("Google Play Plan")
              
                          liveMyInAppProducts.postValue(myInAppProducts)
                          "myInAppProducts  :  $myInAppProducts".bilLog()
                      }
                  }
              
                  private fun retriveMySKU() {
              
                      val queryProductDetailsParams = newBuilder().setProductList(
                          listOf(
                              Product.newBuilder().setProductId(productIds[0].id)
                                  .setProductType(productIds[0].billingType).build(),
                              Product.newBuilder().setProductId(productIds[1].id)
                                  .setProductType(productIds[1].billingType).build(),
                              Product.newBuilder().setProductId(productIds[2].id)
                                  .setProductType(productIds[2].billingType).build(),
              //                        Product.newBuilder().setProductId(productIds[3].id).setProductType(productIds[3].billingType).build()
                          )
                      ).build()
              
                      getSkuProduct(queryProductDetailsParams) { billingResult, productDetailsList ->
                          Log.e("TAG", "retriveMySKU: \n${productDetailsList.toGson()}\n")
                          mySubProducts.clear()
                          productDetailsList.forEach {
                              val product = it
                              product.subscriptionOfferDetails?.forEach { subsDet ->
                                  subsDet.pricingPhases.pricingPhaseList.forEachIndexed { index, pricingPhase ->
                                      val billingProduct = BillingProduct(it.productId)
                                      pricingPhase.billingPeriod.messageLog()
                                      billingProduct.title = it.title
                                      if (pricingPhase.billingPeriod == "P1W") {
                                          billingProduct.packageType = MyPackageType.WEEK
                                      } else if (pricingPhase.billingPeriod == "P1Y") {
                                          billingProduct.packageType = MyPackageType.ANNUAL
                                      } else if (pricingPhase.billingPeriod == "P6M") {
                                          billingProduct.packageType = MyPackageType.SIX_MONTH
                                      } else if (pricingPhase.billingPeriod == "P1M") {
                                          billingProduct.packageType = MyPackageType.MONTH
                                      }
                                      Log.e("Purchase -->", "retriveMySKU: ${it.toGson()}\n")
              
                                      billingProduct.period = pricingPhase.billingPeriod
                                      billingProduct.price = pricingPhase.formattedPrice
                                      billingProduct.amountmicros = pricingPhase.priceAmountMicros
              //                        billingProduct.amountmicros = pricingPhase.priceAmountMicros
                                      mySubProducts.add(billingProduct)
                                  }
                              }
                          }
                          SubscriptionSetter.liveMyPlan.postValue("Google Play Plan")
                          livePackageProduct.postValue(mySubProducts)
                          "mySubProducts : $mySubProducts".bilLog()
                      }
                  }
              
                  fun makePurchase(type: MyPackageType) {
                      if (StaticParam.subsSharedFile.isSubscriber) {
                          toastMaker(billContext, "Already Subscribed!")
                          return
                      }
                      when (type) {
                          MyPackageType.WEEK -> {
                              purchaseMyProduct(productIds[0])
                          }
              
                          MyPackageType.SIX_MONTH -> {
                              purchaseMyProduct(productIds[1])
                          }
              
                          MyPackageType.ANNUAL -> {
                              purchaseMyProduct(productIds[2])
                          }
              
                          MyPackageType.LIFETIME -> {
                              purchaseMyProduct(productIds[3])
                          }
              
                          else -> {
              
                          }
                      }
                  }
              
                  private fun purchaseMyProduct(productIDS: ProductBillingIDS) {
                      val queryProductDetailsParams = newBuilder().setProductList(
                          listOf(
                              Product.newBuilder().setProductId(productIDS.id)
                                  .setProductType(productIDS.billingType).build()
                          )
                      ).build()
                      getSkuProduct(queryProductDetailsParams) { billingResult, productDetails ->
              //            Log.e("TAG", "purchaseMyProduct: ${productDetails.toGson()}")
                          var fetchedProduct = fetchProduct(productIDS.id, productDetails)
                          fetchedProduct?.let { product ->
                              Log.e(
                                  "TAG",
                                  "purchaseMyProduct: 1  ${product.oneTimePurchaseOfferDetails?.toGson()}  "
                              )
              
                              val offerToken = product.subscriptionOfferDetails?.get(0)?.offerToken
                              offerToken?.let { token ->
                                  Log.e("TAG", "purchaseMyProduct: 2 $token")
              
                                  var productDetailsParams = BillingFlowParams.ProductDetailsParams.newBuilder()
                                      .setProductDetails(product).setOfferToken(token).build()
              
                                  var billingFlowParams = BillingFlowParams.newBuilder()
                                      .setProductDetailsParamsList(mutableListOf(productDetailsParams)).build()
              
                                  var resultBill = billingClient.launchBillingFlow(billContext, billingFlowParams)
              
                                  when (resultBill.responseCode) {
                                      BillingClient.BillingResponseCode.ITEM_ALREADY_OWNED -> {
                                          "Billing result: ITEM_ALREADY_OWNED".bilLog()
                                          productPurchaseListener?.onPurchase("Product Already Purchased!")
                                      }
              
                                      BillingClient.BillingResponseCode.OK -> {
                                          "Billing result: OK".bilLog()
                                          productPurchaseListener?.onPurchase("Product Success Fully Purchased.")
                                      }
              
                                      else -> {
                                          "No Billing result".bilLog()
                                          productPurchaseListener?.onPurchaseFail("No Purchase Made!")
                                      }
                                  }
                              }
                          }
                      }
                  }
              
                  private fun fetchProduct(key: String, productDetails: List<ProductDetails>): ProductDetails? {
                      var product = productDetails.find { it.productId == key }
                      return product
                  }
              
                  private fun handlePurchase(purchase: Purchase) {
              
                      Log.e("TAG", "handlePurchase: ${purchase.toGson()}")
                      if (purchase.purchaseState == Purchase.PurchaseState.PURCHASED) {
                          for (product in purchase.products) {
                              productIds.forEach {
                                  if (it.id == product) {
                                      productPurchaseListener?.onPurchased(purchase)
                                  }
                              }
              
                          }
                      }
                      CoroutineScope(Dispatchers.IO).launch {
                          acknowledgePurchase(purchase = purchase)
                          if (isConsumable) {
                              consumePurchase(purchase = purchase)
                          }
                      }
                  }
              
                  private suspend fun acknowledgePurchase(purchase: Purchase) {
                      if (purchase.purchaseState == Purchase.PurchaseState.PURCHASED && !purchase.isAcknowledged) {
                          val acknowledgePurchaseParams =
                              AcknowledgePurchaseParams.newBuilder().setPurchaseToken(purchase.purchaseToken)
              
                          val ackPurchaseResult = withContext(Dispatchers.IO) {
                              billingClient?.acknowledgePurchase(acknowledgePurchaseParams.build(),
                                  object : AcknowledgePurchaseResponseListener {
                                      override fun onAcknowledgePurchaseResponse(acknowledgePurchaseResult: BillingResult) {
                                          "acknowledgePurchase: ${acknowledgePurchaseResult.responseCode}".bilLog()
                                      }
                                  })
                          }
              
                          if (ackPurchaseResult != null) {
                              "acknowledgePurchase: ".bilLog()
                          } else {
                              "acknowledgePurchase: =>> Not Found Any Purchase Result".bilLog()
                          }
                      }
                  }
              
                  fun consumePurchase(purchase: Purchase) {
                      val consumeParams =
                          ConsumeParams.newBuilder().setPurchaseToken(purchase.purchaseToken).build()
                      billingClient?.let {
                          it.consumeAsync(consumeParams, object : ConsumeResponseListener {
                              override fun onConsumeResponse(consumeResult: BillingResult, p1: String) {
                                  "consumePurchase: ${consumeResult.responseCode}".bilLog()
                              }
                          })
                      }
                  }
              }
              
              @Keep
              enum class MyPackageType {
                  ANNUAL, MONTH, SIX_MONTH, WEEK, LIFETIME, PRODUCT
              }
       
       interface ProductPurchaseListener {
           fun onPurchase(message : String)
           fun onPurchased(purchase : Purchase)
           fun onRevenueCatPurchased(purchases : CustomerInfo)
           fun onPurchaseFail(errorMessage : String)
       }


              ---------- RevenueCatHelper.kt 
                     
                     
                                   object RevenueCatHelper {
                         var currentSub : Map<String, EntitlementInfo>? = null
                     
                         var liverevenuecatedatalist = MutableLiveData<List<RevenueCatData>>()
                         var revenueCatDataList = ArrayList<RevenueCatData>()
                         var allAvailablePackages = ArrayList<MyOffer>()
                         var currentUserPackage = MyOffer()
                         fun revenueCateInit(context : Context, REVENUECAT_GOOGLE_API : String, abTestPackage : String, productPurchaseListener : ProductPurchaseListener) {
                             StaticParam.productPurchaseListener = productPurchaseListener
                     
                             Purchases.debugLogsEnabled = false
                     
                             Purchases.configure(PurchasesConfiguration.Builder(context, REVENUECAT_GOOGLE_API).build())
                     
                             Purchases.sharedInstance.updatedCustomerInfoListener = UpdatedCustomerInfoListener { info ->
                                 currentSub = info.entitlements.active
                                 productPurchaseListener.onRevenueCatPurchased(info)
                                 "$currentSub".log("User Purchased List-->")
                             }
                             Purchases.sharedInstance.getOfferings(object : ReceiveOfferingsCallback {
                                 override fun onError(error : PurchasesError) {
                                     "${error}".log("Purchase List error-->")
                                 }
                     
                                 override fun onReceived(offerings : com.revenuecat.purchases.Offerings) {
                                     offerings.all.forEach { (s, offering) ->
                                         var offers = MyOffer(s, offering.availablePackages)
                                         allAvailablePackages.add(offers)
                                     }
                                     allAvailablePackages.forEach { myoffer ->
                     
                                         myoffer.packages.forEach {
                                             var freeTrial = it.product.subscriptionOptions?.freeTrial?.let { it ->
                                                 it.freePhase?.billingPeriod
                                             }
                                             revenueCatDataList.add(
                                                     RevenueCatData(
                                                             it.product.id,
                                                             myoffer.key,
                                                             it.packageType,
                                                             it.product.price.formatted,
                                                             it.product.price.amountMicros,
                                                             it.product.period.toString(),
                                                             FreeTrils(freeTrial?.value, freeTrial?.unit.toString(), freeTrial?.iso8601.toString()),
                                                             it
                                                     )
                                             )
                                         }
                                     }
                     
                                     allAvailablePackages.find { it.key == abTestPackage }?.let {
                                         currentUserPackage = it
                                     }
                                     liveMyPlan.postValue(currentUserPackage.key)
                                     currentUserPackage?.packages?.forEach {
                                         var freeTrial = it.product.subscriptionOptions?.freeTrial?.let { it ->
                                             it.freePhase?.billingPeriod
                                         }
                                         revenueCatDataList.add(
                                                 RevenueCatData(
                                                         it.product.id,
                                                         currentUserPackage.key,
                                                         it.packageType,
                                                         it.product.price.formatted,
                                                         it.product.price.amountMicros,
                                                         it.product.period.toString(),
                                                         FreeTrils(
                                                                 freeTrial?.value, freeTrial?.unit.toString(), freeTrial?.iso8601.toString()
                                                         ),
                                                         it
                                                 )
                                         )
                                     }
                                     var myProducts = ArrayList<BillingProduct>()
                     
                                     revenueCatDataList.forEach {
                                         var billingProduct = BillingProduct()
                                         billingProduct.id = it.id
                                         billingProduct.title = it.packageName
                     
                                         when (it.packageType) {
                                             PackageType.MONTHLY -> {
                                                 billingProduct.packageType = MyPackageType.MONTH
                                             }
                     
                                             PackageType.ANNUAL -> {
                                                 billingProduct.packageType = MyPackageType.ANNUAL
                                             }
                     
                                             PackageType.WEEKLY -> {
                                                 billingProduct.packageType = MyPackageType.WEEK
                                             }
                     
                                             else -> {
                                                 billingProduct.packageType = MyPackageType.SIX_MONTH
                                             }
                     
                                         }
                                         billingProduct.price = it.price
                                         billingProduct.period = it.period
                                         billingProduct.amountmicros = it.amountmicros
                                         myProducts.add(billingProduct)
                                     }
                     
                                     SubscriptionSetter.livePackageProduct.postValue(myProducts)
                                     liverevenuecatedatalist.postValue(revenueCatDataList)
                                 }
                             })
                         }
                     
                         fun purchaseMonth(context : Activity, onSuccess : (StoreTransaction, CustomerInfo) -> Unit) {
                     //        var purchaseProduct = allAvailablePackages.find { it.key == key }
                             var product = revenueCatDataList.find { it.packageType == PackageType.MONTHLY }
                             if(product != null) {
                     //            var product = purchaseProduct.packages.find { it.packageType == PackageType.MONTHLY }
                                 product?.let {
                                     Purchases.sharedInstance.purchaseWith(PurchaseParams.Builder(context, product.product).build(),
                                             { error : PurchasesError, userCancelled : Boolean ->
                                                 Log.e("TAG", "initview:MonthPackage $error")
                                             },
                                             { purchase, customerInfo ->
                                                 purchase?.let { onSuccess.invoke(it, customerInfo) }
                                             })
                                 }
                             } else {
                                 Log.e("TAG", "initview:purchaseProduct null!")
                     
                             }
                         }
                     
                         fun purchaseOnTime(context : Activity,productkey:PackageType, onSuccess : (StoreTransaction, CustomerInfo) -> Unit) {
                     //        var purchaseProduct = allAvailablePackages.find { it.key == key }
                             var product = revenueCatDataList.find { it.packageType == productkey }
                             if(product != null) {
                                 product?.let {
                                     Purchases.sharedInstance.purchaseWith(PurchaseParams.Builder(context, product.product).build(),
                                             { error : PurchasesError, userCancelled : Boolean ->
                                                 Log.e("TAG", "initview:MonthPackage $error")
                                             },
                                             { purchase, customerInfo ->
                                                 purchase?.let { onSuccess.invoke(it, customerInfo) }
                                             })
                                 }
                             } else {
                                 Log.e("TAG", "initview:purchaseProduct null!")
                             }
                         }
                     }


          --------- SubscriptionSetter.kt class

                        object SubscriptionSetter {
                  var livePackageProduct = MutableLiveData<List<BillingProduct>>()
                  var liveMyInAppProducts = MutableLiveData<List<BillingINAPPProduct>>()
              
                  var liveMyPlan = MutableLiveData<String>()
              
                  fun makeMySubs(
                      activity: Activity,
                      packageType: MyPackageType,
                      onSuccess: (StoreTransaction, CustomerInfo) -> Unit
                  ) {
                      if (StaticParam.isRevenueCate) {
                          if (packageType == MyPackageType.MONTH) {
                              RevenueCatHelper.purchaseOnTime(activity, PackageType.MONTHLY, onSuccess)
                          } else if (packageType == MyPackageType.ANNUAL) {
                              RevenueCatHelper.purchaseOnTime(activity, PackageType.ANNUAL, onSuccess)
                          } else if (packageType == MyPackageType.SIX_MONTH) {
                              RevenueCatHelper.purchaseOnTime(activity, PackageType.SIX_MONTH, onSuccess)
                          } else if (packageType == MyPackageType.WEEK) {
                              RevenueCatHelper.purchaseOnTime(activity, PackageType.WEEKLY, onSuccess)
                          }
                      } else {
                          BillingHelper.makePurchase(packageType)
                      }
                  }
              
                  fun makeMyProduct(
                      activity: Activity, packageType: MyPackageType/* here packageType is Product's  */
                  ) {
                      if (StaticParam.isRevenueCate) {
              
                      } else {
                          BillingHelper.makeProductPurchase(packageType)
                      }
                  }
              }

       ------ SubsSharedFile.class
       
       class SubsSharedFile(var context : Context) {
           private var USER_OPEN = "user_open"
           private var IS_SUB = "is_sub"
           private var PURCHASED_PRODUCT = "purchased_product"
       
           val userOpen : SharedPreferences
               get() = context.getSharedPreferences(USER_OPEN, Context.MODE_PRIVATE)
       
           var isSubscriber : Boolean = false
       
           var purchaseProduct : String
               get() = userOpen.getString(PURCHASED_PRODUCT, "").toString()
               set(value) {
                   userOpen.edit().putString(IS_SUB, value).apply()
               }
       
       }

        ----- StaticParam.kt
        
              class StaticParam {
           companion object {
               var isRevenueCate = false
               lateinit var billContext: Activity
               var productPurchaseListener: ProductPurchaseListener? = null
               lateinit var subsSharedFile: SubsSharedFile
       
               var purchaseListener = MutableLiveData<Boolean>()
           }
       }


 -------- Splace Act

    var productIds = Arrays.asList(
        ProductBillingIDS("subscribid", BillingClient.ProductType.SUBS),
        ProductBillingIDS("subscribid", BillingClient.ProductType.SUBS),
        ProductBillingIDS("subscribe_yearly_kriadl", BillingClient.ProductType.SUBS),
        ProductBillingIDS("subscribid", BillingClient.ProductType.INAPP)
    )

        BillingHelper.getBillingInit(this, productIds, object : ProductPurchaseListener {
                   override fun onPurchase(message: String) {
                       ("onPurchase: $message").log()
                       message.messageLog()
                   }
       
                   override fun onRevenueCatPurchased(purchases: CustomerInfo) {
       
                   }
       
                   override fun onPurchased(purchase: Purchase) {
                       Log.wtf("Purchase -->", "-- purchase: " + purchase.purchaseState)
       //                subsSharedFile.isSubscriber = (purchase.purchaseState != -1)
       //                subsSharedFile.purchaseProduct = purchase.toGson()
       //                subsSharedFile.purchaseProduct.logPurchase()
                       val isSubscriber = (purchase.purchaseState != -1)
       
                       isSubscribeFun(isSubscriber)
                   }
       
                   override fun onPurchaseFail(errorMessage: String) {
       
                   }
               })
               

            subsSharedFile = SubsSharedFile(this)


             ----------- Subscription Act ---------

              Event to get a Observ is Subscribe or Not
              
                purchaseListener.observe(this@NewForYearSubAct, { purchase ->
                //finish
                ("Purchasing: $purchase").log()
                isSubscribedForAllGlobal(this@NewForYearSubAct, purchase)

                setResult(Activity.RESULT_OK)
                onBackPressed()
            })

              -- Life time btn click Sub

              SubscriptionSetter.makeMyProduct(
                    this@NewForYearSubAct,
                    MyPackageType.LIFETIME
                )

                  SubscriptionSetter.makeMySubs(this@NewForYearSubAct,
                            MyPackageType.ANNUAL,
                            { storeTransaction, customerInfo ->

                            })

                                     SubscriptionSetter.makeMySubs(this@NewForYearSubAct,
                            MyPackageType.ANNUAL,
                            { storeTransaction, customerInfo ->

                            })



                            ---- All Event to get a price and text and all

                                            SubscriptionSetter.liveMyInAppProducts.observe(this@NewForYearSubAct) { billingInAppProduct ->
                              billingInAppProduct?.let { productsList ->
                                  val lifetime =
                                      productsList.find { it.packageType == MyPackageType.LIFETIME && it.productType == BillingClient.ProductType.INAPP }
                                  lifetime?.let {
                                      ("Sucsess INAPP Data: ${it.toGson()}").log("FATZ")
                                      bind.lifetimebtn.visibility = View.VISIBLE
                                      bind.lifetimepriceid.text = lifetime?.price
                                      bind.lifetimetitalid.text = lifetime?.title
              
                                      bind.lifetimetitalid.setEllipsize(TextUtils.TruncateAt.MARQUEE);
                                      bind.lifetimetitalid.setSingleLine(true);
                                      bind.lifetimetitalid.setMarqueeRepeatLimit(-1); // -1 for infinite loop
                                      bind.lifetimetitalid.setSelected(true); // Required for the marquee effect to work
              
              //                       bind.tvpurid.text = lifetime?.description
              //                       bind.tvpurid.text = "Lifetime Purchase"
                                  }
                              }
                          }
              
                          SubscriptionSetter.livePackageProduct.observe(this@NewForYearSubAct,
                              object : Observer<List<BillingProduct>?> {
                                  override fun onChanged(billingProducts: List<BillingProduct>?) {
                                      billingProducts?.let { productsList ->
              
                                          val annualPlan =
                                              productsList.find { it.packageType == MyPackageType.ANNUAL }
                                          val weekPlan =
                                              productsList.find { it.packageType == MyPackageType.WEEK }
                                          val sixPlan =
                                              productsList.find { it.packageType == MyPackageType.SIX_MONTH }
              
              //                            bind.txtPurchaseMonth.text = monthPlan?.title
              //                            bind.priserid.text = weekPlan?.title
              //                            binding.thirdidyearly.text = sixPlan?.title
              
                                          bind.tvPriceyear.text = annualPlan?.price
                                          bind.priserid.text = weekPlan?.price
                                          bind.sixmonthjid.text = sixPlan?.price
              
                                          bind.weeklytvlongtvid.text =
                                              "• Weekly Subscription : ${weekPlan?.price}"
                                          bind.monthlytvlongtvid.text =
                                              "• Six Monthly Subscription : ${sixPlan?.price}"
                                          bind.yearlytvlongtvid.text =
                                              "• Yearly Subscription : ${annualPlan?.price}"
              
                                          bind.montbottomid.text = "per month \n ₹341/-"
                                          bind.montbottomid.text = "per month \n ₹341/-"
              
                                          var isFreeTrial = productsList.any { it.price == "Free" }
              
                                          if (isFreeTrial) {
                                              bind.startfreetrybtn.text = getSubscribeText(true)
                                          } else {
                                              bind.startfreetrybtn.text = getSubscribeText(false)
                                          }
                                      }
                                  }
                              })


        
// TextView Horizently AutoScroll TextView SCroll View

       bind.lifetimetitalid.setEllipsize(TextUtils.TruncateAt.MARQUEE);
                             bind.lifetimetitalid.setSingleLine(true);
                             bind.lifetimetitalid.setMarqueeRepeatLimit(-1); // -1 for infinite loop
                             bind.lifetimetitalid.setSelected(true); // Required for the marquee effect to work
     

                    <TextView
                        android:id="@+id/lifetimetitalid"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_gravity="start"
                        android:layout_marginTop="@dimen/_8sdp"
                        android:fontFamily="@font/poppinsmedium"
                        android:gravity="start"
                        android:includeFontPadding="false"
                        android:maxLines="1"
                        android:ellipsize="marquee"
                        android:layout_marginHorizontal="@dimen/_10sdp"
                        android:scrollHorizontally="true"
                        android:text="Purchase once, delight indefinitely"
                        android:textColor="#1C1818"
                        android:textSize="@dimen/_12sdp"
                        app:layout_constraintBottom_toBottomOf="parent"
                        app:layout_constraintEnd_toStartOf="@+id/view"
                        app:layout_constraintStart_toStartOf="parent"
                        app:layout_constraintTop_toTopOf="parent"
                        app:layout_goneMarginStart="@dimen/_10sdp" />

// Load Reward Ad

     
     interface RewardAdsCallback {
         fun onAdsDismissClick()
         fun onFullShowAds()
     }
     
     fun loadRewardAds(activity: Activity, onRewardCallBack: RewardAdsCallback) {
         if (isSubScribe || !isShowRewardAds || !isNetworkConnected(activity)) onRewardCallBack.onAdsDismissClick()
         else {
             isRewardedShow = true
             RewardAds(activity) {
                 it.log()
                 isRewardedShow = false
                 when (it) {
                     "onAdsFullShow" -> {
                         onRewardCallBack.onFullShowAds()
                     }
     
                     "onAddDismiss" -> {
                         onRewardCallBack.onAdsDismissClick()
                     }
     
                     "onAddFail" -> {
                         onRewardCallBack.onAdsDismissClick()
                     }
                 }
             }
         }
     }

       // reward class

          
          const val COUNTER_TIME = 10L
          
          class RewardAds(
              var activity: Activity, var onAdsClick: (String) -> Unit
          ) {
              private var isEarnReward: Boolean = false
              private val AD_UNIT_ID = REWARD_ADS
              var progressBar: RewardAdsDialog = RewardAdsDialog(activity)
              var timeOver = false
          
              private var countdownTimer: CountDownTimer? = null
              private var isLoading = false
              private var rewardedAd: RewardedAd? = null
              private var timeRemaining: Long = 0L
          
              init {
                  if (isSubScribe) {
                      onAdsClick("onAddDismiss")
                  } else {
                      if (!activity.isFinishing) {
                          progressBar.show()
                          createTimer(COUNTER_TIME)
                          loadRewardedAd()
                      }
                  }
              }
          
              private fun loadRewardedAd() {
                  if (rewardedAd == null) {
                      ("Reward Ads Loading ID: $AD_UNIT_ID").log()
                      isLoading = true
                      val adRequest = AdManagerAdRequest.Builder().build()
          
                      RewardedAd.load(activity, AD_UNIT_ID, adRequest, object : RewardedAdLoadCallback() {
                          override fun onAdFailedToLoad(adError: LoadAdError) {
                              ("Reward Error: " + adError.message + " |Time Over: $timeOver").log()
                              isLoading = false
                              rewardedAd = null
                              timeOver = true
                              countdownTimer!!.cancel()
                              progressBar.onDismiss()
                              isRewardedShow = false
                              (activity.getString(R.string.please_try_again)).tos(activity)
          
          //                    timeOver = true
          //                    if (progressBar.isShowingDialog()) {
          //                        onAdsClick("onAddFail")
          //                    }
                          }
          
                          override fun onAdLoaded(ad: RewardedAd) {
                              ("Ad was loaded.: $timeOver").log()
                              rewardedAd = ad
                              isLoading = false
                              countdownTimer!!.cancel()
                              if (!timeOver && !activity.isFinishing) {
                                  showRewardedVideo()
                              }
                          }
                      })
                  }
              }
          
              private fun createTimer(time: Long) {
                  countdownTimer?.cancel()
                  countdownTimer = object : CountDownTimer(time * 1000, 50) {
                      override fun onTick(millisUnitFinished: Long) {
                          timeRemaining = millisUnitFinished / 1000 + 1
                      }
          
                      override fun onFinish() {
                          ("Reward OnFinish ").log()
                          if (!timeOver && !activity.isFinishing) {
                              showRewardedVideo()
                          }
                      }
                  }
                  countdownTimer?.start()
              }
          
              private fun showRewardedVideo() {
                  progressBar.onDismiss()
                  timeOver = true
                  if (rewardedAd != null) {
                      rewardedAd?.fullScreenContentCallback = object : FullScreenContentCallback() {
                          override fun onAdDismissedFullScreenContent() {
                              ("Reward Ad was dismissed.").log()
                              rewardedAd = null
                              if (isEarnReward) onAdsClick("onAddDismiss")
                              else {
                                  isRewardedShow = false
                                  (activity.getString(R.string.please_try_again)).tos(activity)
                              }
                          }
          
                          override fun onAdFailedToShowFullScreenContent(adError: AdError) {
                              ("Ad failed to show.").log()
                              rewardedAd = null
                              onAdsClick("onAddFail")
                          }
          
                          override fun onAdShowedFullScreenContent() {
                              ("Ad showed fullscreen content.").log()
                          }
                      }
          
                      rewardedAd?.show(activity) {
                          isEarnReward = true
                          ("User earned the reward.").log()
                      }
                  } else {
                      isRewardedShow = false
                      (activity.getString(R.string.please_try_again)).tos(activity)
          //            onAdsClick("onAddDismiss")
                  }
              }
          }


// Load Singal ID Intern Ad

     const val GAME_LENGTH_MILLISECONDS: Long = 5000
     
     class InternAds(var activity: Activity, var AD_UNIT_ID: String, var onNext: () -> Unit) {
         private var interstitialAd: AdManagerInterstitialAd? = null
         private var countDownTimer: CountDownTimer? = null
         private var gamePaused: Boolean = false
         private var gameOver: Boolean = false
         private var adIsLoading: Boolean = false
         private var timerMilliseconds: Long = 0
         var progressBar: RewardAdsDialog = RewardAdsDialog(activity)
     
         init {
             loadAd()
             startGame()
         }
     
         private fun startGame() {
             createTimer(GAME_LENGTH_MILLISECONDS)
             gamePaused = false
             gameOver = false
         }
     
         private fun createTimer(milliseconds: Long) {
             countDownTimer?.cancel()
             countDownTimer = object : CountDownTimer(milliseconds, 50) {
                 override fun onTick(millisUntilFinished: Long) {
                     timerMilliseconds = millisUntilFinished
                 }
     
                 override fun onFinish() {
                     if (!activity.isFinishing) {
                         gameOver = true
                         progressBar.onDismiss()
                         showInterstitial()
                     }
                 }
             }
             if (!activity.isFinishing) {
                 progressBar.show()
                 countDownTimer?.start()
             }
         }
     
         private fun showInterstitial() {
             // Show the ad if it's ready. Otherwise restart the game.
             if (interstitialAd != null) {
                 interstitialAd?.fullScreenContentCallback = object : FullScreenContentCallback() {
                     override fun onAdDismissedFullScreenContent() {
                         interstitialAd = null
                         ("Ad was dismissed.").log()
                         isRewardedShow = false
                         onNext()
                     }
     
                     override fun onAdFailedToShowFullScreenContent(adError: AdError) {
                         interstitialAd = null
                         ("Ad failed to show.").log()
                         isRewardedShow = false
                         onNext()
                     }
     
                     override fun onAdShowedFullScreenContent() {
                         ("Ad showed fullscreen content.").log()
                     }
                 }
     //            if (activity is TrailActivity) activity.onBackPressed()
                 interstitialAd?.show(activity)
             } else {
                 onNext()
     //            loadAd()
             }
         }
     
         private fun loadAd() {
             // Request a new ad if one isn't already loaded.
             if (adIsLoading || interstitialAd != null) {
                 return
             }
             adIsLoading = true
             val adRequest = AdManagerAdRequest.Builder().build()
     
             AdManagerInterstitialAd.load(
                 activity,
                 AD_UNIT_ID,
                 adRequest,
                 object : AdManagerInterstitialAdLoadCallback() {
                     override fun onAdFailedToLoad(adError: LoadAdError) {
                         (adError.message).log()
                         interstitialAd = null
                         adIsLoading = false
                         val error =
                             "domain: ${adError.domain}, code: ${adError.code}, " + "message: ${adError.message}"
                         error.log()
     //                    Toast.makeText(
     //                        activity, "onAdFailedToLoad() with error $error", Toast.LENGTH_SHORT
     //                    ).show()
                     }
     
                     override fun onAdLoaded(interstitialAd: AdManagerInterstitialAd) {
                         ("Intern Ad was loaded ID: $AD_UNIT_ID").log()
                         this@InternAds.interstitialAd = interstitialAd
                         adIsLoading = false
     //                    Toast.makeText(activity, "onAdLoaded()", Toast.LENGTH_SHORT).show()
                     }
                 })
         }
     }


// hide status bar and set color

     fun hideBottomNavigationBar() {
            window.statusBarColor = ContextCompat.getColor(this, R.color.status_bar_color)
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
                // For devices running Android 11 (API level 30) and above
                window.setDecorFitsSystemWindows(false)
                window.insetsController?.hide(WindowInsets.Type.navigationBars())
            } else {
                // For devices running below Android 11
                window.decorView.systemUiVisibility =
                    (View.SYSTEM_UI_FLAG_HIDE_NAVIGATION or View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY)
            }
        }
        

// Custom Tab for privacy policy

    implementation 'com.android.support:customtabs:28.0.0'

         val privacyPolicyUrl =
                          "https://imagecropnwallpaperchanger.blogspot.com/2023/07/privacy-policy.html"
                      val builder = CustomTabsIntent.Builder()
                      builder.setToolbarColor(
                          ContextCompat.getColor(
                              this@MainActivity,
                              android.R.color.black
                          )
                      )
                      val customTabsIntent = builder.build()
                      customTabsIntent.launchUrl(this@MainActivity, Uri.parse(privacyPolicyUrl))
                      

// Custom native Ad

      https://github.com/DevHamza090/Admob-AppLovin/tree/master/app/src/main/res/layout
             https://github.com/googleads/googleads-mobile-android-native-templates/tree/main/nativetemplates/src/main/res/layout


// Facebook Shimmer layout

      implementation 'com.facebook.shimmer:shimmer:0.5.0'

          <com.facebook.shimmer.ShimmerFrameLayout
                                android:layout_width="@dimen/_120sdp"
                                android:layout_height="@dimen/_30sdp"
                                app:shimmer_auto_start="true"
                                app:shimmer_duration="2000"
                                app:shimmer_fixed_width="@dimen/_100sdp"
                                app:shimmer_highlight_color="@color/white">

                                <LinearLayout
                                    android:layout_width="match_parent"
                                    android:layout_height="match_parent"
                                    android:background="@drawable/shimmer_layout">

                                </LinearLayout>

                            </com.facebook.shimmer.ShimmerFrameLayout>

                           // shimmer_layout.xml
                           
                                    <shape
            android:shape="rectangle" xmlns:android="http://schemas.android.com/apk/res/android">
        
            <solid
                android:color="#61FFFFFF">
        
            </solid>
        
            <corners
                android:radius="@dimen/_12sdp">
        
            </corners>
        
        </shape>



// When Change the Language in beta

    fun setAppLocale(activity: Activity, languageCode: String) {
        App.putString(LANGUAGE_KEY, languageCode)
        val locale = Locale(languageCode)
        Locale.setDefault(locale)
        val configuration = Configuration()
        configuration.setLocale(locale)
        activity.resources.updateConfiguration(configuration, activity.resources.displayMetrics)
    }

    build.gradl file

     bundle {
            language {
                enableSplit = false
            }
        }

            list.add(LanguageModel("Arabic", false, "Ar", "عربي", "#CEE3F7"))
        list.add(LanguageModel("English", false, "en", "English", "#DBF1DB"))
        list.add(LanguageModel("French", false, "fr", "Français", "#CFF6E6"))
        list.add(LanguageModel("German", false, "de", "Deutsch", "#F7DBD2"))
        list.add(LanguageModel("Japanese", false, "ja", "日本語", "#FDF2D2"))
        list.add(LanguageModel("Korean", false, "ko", "한국인", "#D7F0F4"))
        list.add(LanguageModel("Portuguese", false, "pt", "Português", "#E4DBDB"))
        list.add(LanguageModel("Swedish", false, "sv", "svenska", "#E8D2FD"))
        list.add(LanguageModel("Hindi", false, "hi", "हिन्दी", "#F7DBD2"))

         // for Arabic in menifest file

           tools:replace="android:supportsRtl"
           android:supportsRtl="false"

         

// Firebase Event Debug

    adb shell setprop debug.firebase.analytics.app package_name
    adb shell setprop debug.firebase.analytics.app       
    usage: setprop NAME VALUE


         val bundle = Bundle()
        bundle.putString("Kriadl_${javaClass.simpleName}", "Kriadl_${javaClass.simpleName}")
        val mFirebaseAnalytics = FirebaseAnalytics.getInstance(this)
        mFirebaseAnalytics.logEvent("Kriadl_${javaClass.simpleName}", bundle)

// Test AD ID ADX
    
    Sample ad unit ID	
    App Open	/6499/example/app-open
    Banner	/6499/example/banner
    Interstitial	/6499/example/interstitial
    Rewarded	/6499/example/rewarded
    Rewarded Interstitial	/21775744923/example/rewarded_interstitial
    Native	/6499/example/native
    Native Video	/6499/example/native-video

// Simple Important lib

    implementation 'com.github.SimpleMobileTools:Simple-Commons:c31d0fda56'

// Firebase Notification

    implementation 'com.google.firebase:firebase-messaging:23.1.1'

    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

     <service
            android:name=".MyFirebaseMessagingService"
            android:exported="true">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>

      
      val flags = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
          PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
      } else PendingIntent.FLAG_UPDATE_CURRENT
        
      class MyFirebaseMessagingService : FirebaseMessagingService() {
          var fcm_vaule = ""
          override fun onNewToken(token: String) {
              super.onNewToken(token)
              ("onNewToken: ====>$token").log()
          }
      
          override fun onMessageReceived(remoteMessage: RemoteMessage) {
              ("on Message Recive From Firebase").log()
              if (remoteMessage.data.isNotEmpty()) {
                  val data = remoteMessage.data
                  data.forEach {
                      val key: String = it.key
                      fcm_vaule = it.value
                      CAT_VAULE = it.value
                      ("Firebase FCM: Key: $key Value: $fcm_vaule").log()
                  }
              }
      
              // Handle notification payload if present
              if (remoteMessage.notification != null) {
                  // Handle notification message
                  val notificationTitle = remoteMessage.notification!!.title
                  val notificationBody = remoteMessage.notification!!.body
                  // Process notification message
                  ("Title: $notificationTitle Body: $notificationBody").log()
              }
      
              remoteMessage.notification?.let {
                  sendNotification(it.title, it.body, remoteMessage)
              }
          }
      
          private fun sendNotification(
              title: String?, messageBody: String?, remoteMessage: RemoteMessage
          ) {
              val intent = Intent(this, SearchDataActivity::class.java).putExtra("keyword", fcm_vaule)
                  .putExtra("isFromNotify", true)
      
              intent.flags =
                  Intent.FLAG_ACTIVITY_REORDER_TO_FRONT or Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_NEW_TASK
              intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK or Intent.FLAG_ACTIVITY_NO_HISTORY)
              intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP)
      
              val pendingIntent = PendingIntent.getActivity(
                  this, 0, intent, flags
              )
      
              val channelId = "default_channel_id"
              val defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION)
              val notificationBuilder =
                  NotificationCompat.Builder(this, channelId).setSmallIcon(R.mipmap.ic_launcher)
                      .setContentTitle(title).setContentText(messageBody).setAutoCancel(true).setVibrate(
                          longArrayOf(
                              1000, 1000, 1000, 1000, 1000
                          )
                      ).setSound(defaultSoundUri).setOnlyAlertOnce(true).setContentIntent(pendingIntent)
      
              val notificationManager =
                  getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
      
              if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                  val channel = NotificationChannel(
                      channelId, "Important Notification", NotificationManager.IMPORTANCE_HIGH
                  )
                  channel.vibrationPattern = longArrayOf(1000, 1000, 1000, 1000, 1000)
                  notificationManager.createNotificationChannel(channel)
              }
              notificationManager.notify(0, notificationBuilder.build())
          }
      }

    in App Background.
    SplaceAct
     if (intent.extras != null) {
            ("22222").log()
            for (firebase_key in intent.extras!!.keySet()) {
                ("33333").log()
                firebase_value = intent.extras!![firebase_key].toString()
                Log.d("FATZ", "----- Fuirebase --------- Key: $firebase_key Value: $firebase_value")
                if (firebase_key!! == "FIREBASE_KEY") {
                    isOpenSearchAct = true
                    break
                }
            }
        }

// Safty Net

      // play intigrity
          coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.0.3")
         implementation("com.google.apis:google-api-services-playintegrity:v1-rev20220211-1.32.1")
         implementation("com.google.apis:google-api-services-playintegrity:v1-rev20231109-2.0.0")
         //  Google Authentication
         implementation("com.google.api-client:google-api-client-jackson2:1.20.0")
         implementation("com.google.auth:google-auth-library-credentials:1.20.0")
         implementation 'com.google.auth:google-auth-library-credentials:1.20.0'
         implementation("com.google.auth:google-auth-library-oauth2-http:1.20.0")
         implementation("com.google.android.play:integrity:1.3.0")
         implementation 'com.scottyab:rootbeer-lib:0.1.0'

         in asset -> addJson File 
         in Build.gradl File ->     
         buildConfigField("String", "CLOUD_PROJECT_NUM", "100200")

       buildTypes {
             release {
                 minifyEnabled true
                 shrinkResources true
     //            signingConfig signingConfigs.release
                 proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
                 firebaseCrashlytics {
                     nativeSymbolUploadEnabled true
                 }
                 signingConfig signingConfigs.debug
                 buildConfigField("String", "CLOUD_PROJECT_NUM", "\"00000\"")
     //            signingConfig signingConfigs.debug
                 ndk {
                     abiFilters 'x86', 'x86_64', 'armeabi-v7a', 'arm64-v8a'
                 }
             }
     
             debug {
                 buildConfigField("String", "CLOUD_PROJECT_NUM", "\"00000\"")
                 minifyEnabled false
                 shrinkResources false
                 proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
                 firebaseCrashlytics {
                     nativeSymbolUploadEnabled false
                 }
                 ndk {
                     abiFilters 'armeabi-v7a', 'arm64-v8a', 'x86'
                 }
     //            ndk {
     //                abiFilters 'x86', 'x86_64', 'armeabi-v7a', 'arm64-v8a'
     //            }
             }
         }

               packagingOptions {
             exclude("META-INF/DEPENDENCIES")
             exclude("META-INF/LICENSE")
             exclude("META-INF/LICENSE.txt")
             exclude("META-INF/license.txt")
             exclude("META-INF/NOTICE")
             exclude("META-INF/NOTICE.txt")
             exclude("META-INF/notice.txt")
             exclude("META-INF/ASL2.0")
             exclude("META-INF/*.kotlin_module")
         }

      // PlaySafety.kt class
      
                class PlaySafety(
              var activity: Activity, var cloudProjectNumber: Long, private var onNext: () -> Unit
          ) {
              private var interpin: StandardIntegrityManager.StandardIntegrityToken? = null
          
              fun cxroot() {
                  val rootBeer = RootBeer(activity)
                  if (rootBeer.isRooted || rootBeer.isRootedWithBusyBoxCheck) {
                      PlayInitDialog(activity)
                  } else {
                      onNext.invoke()
                  }
              }
          
              fun sft() {
                  val standardIntegrityManager: StandardIntegrityManager =
                      IntegrityManagerFactory.createStandard(activity);
                  var integrityTokenProvider: StandardIntegrityManager.StandardIntegrityTokenProvider;
                  val cloudProjectNumber = cloudProjectNumber
          
                  // Prepare integrity token. Can be called once in a while to keep internal
                  // state fresh.
          
                  standardIntegrityManager.prepareIntegrityToken(
                      StandardIntegrityManager.PrepareIntegrityTokenRequest.builder()
                          .setCloudProjectNumber(cloudProjectNumber).build()
                  ).addOnSuccessListener {
                      integrityTokenProvider = it
                      val integrityTokenResponse: Task<StandardIntegrityManager.StandardIntegrityToken>? =
                          integrityTokenProvider.request(
                              StandardIntegrityManager.StandardIntegrityTokenRequest.builder()
                                  .setRequestHash(generateNonce()).build()
                          )
                      integrityTokenResponse!!.addOnSuccessListener { response ->
                          interpin = response
                          decryptToken(response.token())
                          "check token addOnSuccessListener:${response.token()}".log()
                      }.addOnFailureListener { exception ->
                          "check token addOnFailureListener:${exception.message}".log()
                      }
                  }.addOnFailureListener(
                      activity
                  ) { p0 ->
                      "check token addOnFailureListener:1:${p0.message}".log()
          
                      val rootBeer = RootBeer(activity)
                      if (rootBeer.isRooted || rootBeer.isRootedWithBusyBoxCheck) {
                          CoroutineScope(Dispatchers.Main).launch {
                              PlayInitDialog(activity)
                          }
                      } else {
                          activity.runOnUiThread {
                              onNext.invoke()
                          }
                      }
                      "check token addOnFailureListener:2:${p0.message}".log()
                  }
              }
          
              private fun decryptToken(token: String) {
                  try {
                      "check the response is the:decryptToken:$token".log()
                      val requestObj = DecodeIntegrityTokenRequest()
                      requestObj.integrityToken = token
                      val credentials = GoogleCredentials.fromStream(activity.assets.open("credentials.json"))
                      val requestInitializer: HttpRequestInitializer = HttpCredentialsAdapter(credentials)
                      val initialiser: GoogleClientRequestInitializer = PlayIntegrityRequestInitializer()
                      val playIntegrity =
                          PlayIntegrity.Builder(NetHttpTransport(), JacksonFactory(), requestInitializer)
                              .setApplicationName(
                                  activity.resources.getString(
                                      R.string.app_name
                                  )
                              ).setGoogleClientRequestInitializer(initialiser)
                      val play = playIntegrity.build()
          
                      CoroutineScope(Dispatchers.IO).launch {
                          "check the package name:${activity.packageName}".log()
                          try {
                              val response =
                                  play.v1().decodeIntegrityToken(activity.packageName, requestObj).execute()
                              val rootBeer = RootBeer(activity)
                              if (rootBeer.isRooted || rootBeer.isRootedWithBusyBoxCheck) {
                                  CoroutineScope(Dispatchers.Main).launch {
                                      playdialog(1)
                                  }
                              } else {
                                  isRecognized(response)
                              }
                              "check the response is the:123:response ${isRecognized(response)} and $response".log()
          //                "check the response is the:123:response ${response["tokenPayloadExternal"]}".log()
                          } catch (e: Exception) {
                              "check the response is the:123:catch".log()
                              val rootBeer = RootBeer(activity)
                              if (rootBeer.isRooted || rootBeer.isRootedWithBusyBoxCheck) {
                                  CoroutineScope(Dispatchers.Main).launch {
                                      "check the response is the:123:catch:root device".log()
                                      playdialog(1)
                                  }
                              } else {
                                  "check the response is the:123:catch:not root device".log()
                                  activity.runOnUiThread {
                                      onNext.invoke()
                                  }
                              }
                          }
                      }
                  } catch (e: Exception) {
                      activity.runOnUiThread {
                          onNext.invoke()
                      }
                  }
              }
          
              @SuppressLint("SuspiciousIndentation")
              fun isRecognized(payload: DecodeIntegrityTokenResponse) {
                  val appLicensingVerdict = payload.tokenPayloadExternal.accountDetails.appLicensingVerdict
          
                  val appRecognitionVerdict = payload.tokenPayloadExternal.appIntegrity.appRecognitionVerdict
          
                  val deviceRecognitionVerdict =
                      payload.tokenPayloadExternal.deviceIntegrity.deviceRecognitionVerdict
          
          //        val isSafeLast = payload.tokenPayloadExternal.accountDetails.appLicensingVerdict.isNotEmpty() && (appLicensingVerdict == "LICENSED") && (appRecognitionVerdict == "PLAY_RECOGNIZED") && (deviceRecognitionVerdict[0] == "MEETS_DEVICE_INTEGRITY")
                  val isSafeLast =
                      payload.tokenPayloadExternal.accountDetails.appLicensingVerdict.isNotEmpty() && appLicensingVerdict.contains(
                          "LICENSED"
                      ) && appRecognitionVerdict.contains("PLAY_RECOGNIZED") && deviceRecognitionVerdict.contains(
                          "MEETS_DEVICE_INTEGRITY"
                      )
                  ("check the device is the device:${appLicensingVerdict.contains("LICENSED")} " + "and ${
                      appRecognitionVerdict.contains(
                          "PLAY_RECOGNIZED"
                      )
                  } and " + "${deviceRecognitionVerdict.contains("MEETS_DEVICE_INTEGRITY")} and" + " ${payload.tokenPayloadExternal.accountDetails.appLicensingVerdict.isNotEmpty()}").log()
          
                  "check the device is the device:isSafeLast:$isSafeLast".log()
          
                  if (isSafeLast) {
                      "check the device is the device:isSafeLast:$isSafeLast".log()
                      activity.runOnUiThread {
                          onNext.invoke()
                      }
                  } else {
                      "check the device is the device:else".log()
                      activity.runOnUiThread {
                          playdialog(1)
                      }
                  }
              }
          
              private fun playdialog(showDialogType: Int) {
                  val integrityDialogResponseCode: Task<Int> = interpin!!.showDialog(activity, showDialogType)
                  integrityDialogResponseCode.addOnSuccessListener {
                      "playdialog: Success----> $it".log()
                      if (it == 2) {
                          activity.finishAffinity()
                      } else if (it == 3) {
                          try {
                              activity.startActivity(
                                  Intent(
                                      Intent.ACTION_VIEW,
                                      Uri.parse("market://details?id=${activity.packageName}")
                                  )
                              )
                              activity.finishAffinity()
                          } catch (e: ActivityNotFoundException) {
                              activity.startActivity(
                                  Intent(
                                      Intent.ACTION_VIEW,
                                      Uri.parse("https://play.google.com/store/apps/details?id=${activity.packageName}")
                                  )
                              )
                              activity.finishAffinity()
                          }
                      }
                  }.addOnFailureListener {
                      "playdialog: Failure----> $it".log()
                  }
              }
          
              private fun generateNonce(): String {
                  val length = 50
                  var nonce = ""
                  val allowed = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789"
                  for (i in 0 until length) {
                      nonce += allowed[Math.floor(Math.random() * allowed.length).toInt()].toString()
                  }
                  return nonce
              }
          }
          
          
          //todo: Need to Add Below Lib
          //implementation("com.google.apis:google-api-services-playintegrity:v1-rev20220211-1.32.1")
          //    implementation("com.google.apis:google-api-services-playintegrity:v1-rev20231109-2.0.0")
          //    //  Google Authentication
          //    implementation("com.google.api-client:google-api-client-jackson2:1.20.0")
          ////    implementation ("com.google.auth:google-auth-library-credentials:1.20.0")
          //    implementation 'com.google.auth:google-auth-library-credentials:1.20.0'
          //    implementation("com.google.auth:google-auth-library-oauth2-http:1.20.0")
          //implementation("com.google.android.play:integrity:1.3.0")
          //
          //configurations {
          //    all {
          //        resolutionStrategy {
          //            // Exclude specific modules causing conflicts
          //            exclude group: 'com.google.android.play', module: 'core'
          //        }
          //    }
          //}
          //implementation 'com.google.android.play:app-update:2.1.0'
          
          //-keep class com.google.api.services.playintegrity.** { *; } #REQUIRED
          //-keep class com.google.api.client.** { *; } #REQUIRED
          
          
          //  PlaySafetyNetByFenil(this, 787207734962) {
          //            //(Your Next Code)
          //        }.sft()
          
          
          // Asset-> add json credentials.json

     PlayInitDialog.kt
     
     class PlayInitDialog {
         constructor(activity: Activity) {
     
     //        init {
             var dialog: Dialog = Dialog(activity)
             var bind: PlayinitLayoutBinding = PlayinitLayoutBinding.inflate(activity.layoutInflater)
             dialog.setContentView(bind.root)
             dialog.window!!.setBackgroundDrawableResource(android.R.color.transparent)
             dialog.setCancelable(false)
             dialog.show()
     
             bind.apply {
                 btnExit.click {
                     activity.finishAffinity()
                 }
                 btnInstallNow.click {
                     try {
                         activity.startActivity(
                             Intent(
                                 Intent.ACTION_VIEW,
                                 Uri.parse("market://details?id=${activity.packageName}")
                             )
                         )
                     } catch (e: ActivityNotFoundException) {
                         activity.startActivity(
                             Intent(
                                 Intent.ACTION_VIEW,
                                 Uri.parse("https://play.google.com/store/apps/details?id=${activity.packageName}")
                             )
                         )
                     }
     //                }
                 }
             }
         }
     }

     playinit_layout.xml

          <androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
         xmlns:cardview="http://schemas.android.com/apk/res-auto"
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         android:layout_gravity="top|bottom|left|right|center_vertical|fill_vertical|center_horizontal|fill_horizontal|center|fill|start|end"
         android:layout_marginLeft="10dp"
         android:layout_marginRight="10dp"
         android:elevation="5dp"
         android:gravity="top|bottom|left|right|center_vertical|fill_vertical|center_horizontal|fill_horizontal|center|fill|start|end"
         android:layoutDirection="ltr"
         android:orientation="vertical"
         cardview:cardCornerRadius="20dp"
         cardview:cardElevation="5dp"
         cardview:cardUseCompatPadding="true">
     
         <LinearLayout
             android:layout_width="match_parent"
             android:layout_height="wrap_content"
             android:background="@color/white"
             android:orientation="vertical">
     
             <LinearLayout
                 android:layout_width="match_parent"
                 android:layout_height="wrap_content"
                 android:layout_gravity="top|bottom|left|right|center_vertical|fill_vertical|center_horizontal|fill_horizontal|center|fill|start|end"
                 android:gravity="top|bottom|left|right|center_vertical|fill_vertical|center_horizontal|fill_horizontal|center|fill|start|end"
                 android:orientation="vertical">
     
                 <TextView
                     android:id="@+id/dialogTitle"
                     android:layout_width="match_parent"
                     android:layout_height="wrap_content"
                     android:layout_marginTop="10dp"
                     android:layout_marginBottom="10dp"
                     android:fontFamily="@font/poppins"
                     android:gravity="center"
                     android:text="Alert"
                     android:textColor="#FF0000"
                     android:textSize="25sp" />
     
                 <TextView
                     android:id="@+id/dialogMessage"
                     android:layout_width="match_parent"
                     android:layout_height="wrap_content"
                     android:layout_marginHorizontal="10dp"
                     android:layout_marginBottom="10dp"
                     android:fontFamily="@font/poppins"
                     android:gravity="center|left"
                     android:justificationMode="inter_word"
                     android:text="This app is not licensed or legally available on the Play Store. We kindly ask you to uninstall the current app. Please search for on the Play Store and install the official app to ensure a legal and authorized experience."
                     android:textColor="@color/black"
                     android:textSize="15sp" />
     
                 <LinearLayout
                     android:layout_width="match_parent"
                     android:layout_height="wrap_content"
                     android:layout_marginStart="15dp"
                     android:layout_marginTop="5dp"
                     android:layout_marginEnd="15dp"
                     android:layout_marginBottom="15dp"
                     android:orientation="horizontal">
     
                     <TextView
                         android:id="@+id/btnExit"
                         android:layout_width="match_parent"
                         android:layout_height="wrap_content"
                         android:layout_marginLeft="10dp"
                         android:layout_marginRight="10dp"
                         android:layout_weight="1"
                         android:background="@drawable/edround"
                         android:backgroundTint="#FF0000"
                         android:fontFamily="@font/poppins"
                         android:gravity="center"
                         android:padding="7dp"
                         android:text="Exit"
                         android:textAllCaps="true"
                         android:textColor="@color/white"
                         android:textSize="13sp"
                         android:textStyle="bold" />
     
                     <TextView
                         android:id="@+id/btnInstallNow"
                         android:layout_width="match_parent"
                         android:layout_height="wrap_content"
                         android:layout_marginLeft="10dp"
                         android:layout_marginRight="10dp"
                         android:layout_weight="1"
                         android:background="@drawable/edround"
                         android:backgroundTint="#F44336"
                         android:ellipsize="end"
                         android:fontFamily="@font/poppins"
                         android:gravity="center"
                         android:padding="7dp"
                         android:singleLine="true"
                         android:text="Install Now"
                         android:textAllCaps="true"
                         android:textColor="@color/white"
                         android:textSize="13sp"
                         android:textStyle="bold" />
     
                 </LinearLayout>
     
             </LinearLayout>
     
         </LinearLayout>
     
     </androidx.cardview.widget.CardView>

    // Now in SplaceScreen

      if (NetworkHelper.isOnline(this)) {
                        PlaySafety(this, BuildConfig.CLOUD_PROJECT_NUM.toLong()) {
                            if (isOpenSearchAct) {
                                openActivity(SearchDataActivity::class.java) {
                                    putString("keyword", firebase_value)
                                    putBoolean("isFromNotify", true)
                                }
                                finish()
                            } else {
                                //            openActivity(MainActivity::class.java)
                                if (isSubScribe) openActivity(MainActivity::class.java)
                                else openSubscription()
                            }
                            overridePendingTransition(0, 0)
                        }.sft()
                    } else {
                        PlaySafety(this, BuildConfig.CLOUD_PROJECT_NUM.toLong()) {

                            if (isOpenSearchAct) {
                                openActivity(SearchDataActivity::class.java) {
                                    putString("keyword", firebase_value)
                                    putBoolean("isFromNotify", true)
                                }
                                finish()
                            } else {
                                //            openActivity(MainActivity::class.java)
                                if (isSubScribe) openActivity(MainActivity::class.java)
                                else openSubscription()
                            }
                            overridePendingTransition(0, 0)
                        }.cxroot()
                    }
                    
// WrapHeightViewPager

     public class WrapHeightViewPager extends ViewPager {
         public WrapHeightViewPager(Context context) {
             super(context);
         }
     
         public WrapHeightViewPager(Context context, AttributeSet attrs) {
             super(context, attrs);
         }
     
         @Override
         protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
             int height = 0;
             View view = null;
             for (int i = 0; i < getChildCount(); i++) {
                 view = getChildAt(i);
                 view.measure(widthMeasureSpec, MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED));
                 int h = view.getMeasuredHeight();
                 if (h > height) height = h;
             }
     
             if (height != 0) {
                 heightMeasureSpec = MeasureSpec.makeMeasureSpec(height, MeasureSpec.EXACTLY);
             }
             super.onMeasure(widthMeasureSpec, heightMeasureSpec);
             setMeasuredDimension(getMeasuredWidth(), measureHeight(heightMeasureSpec, view));
         }
     
         private int measureHeight(int measureSpec, View view) {
             int result = 0;
             int specMode = MeasureSpec.getMode(measureSpec);
             int specSize = MeasureSpec.getSize(measureSpec);
     
             if (specMode == MeasureSpec.EXACTLY) {
                 result = specSize;
             } else {
                 // set the height from the base view if available
                 if (view != null) {
                     result = view.getMeasuredHeight();
                 }
                 if (specMode == MeasureSpec.AT_MOST) {
                     result = Math.min(result, specSize);
                 }
             }
             return result;
         }
     }

// InApp Update

     Lib
            configurations {
           all {
               resolutionStrategy {
                   // Exclude specific modules causing conflicts
                   exclude group: 'com.google.android.play', module: 'core'
               }
           }
       }

    implementation 'com.google.android.play:app-update:2.1.0'

       class InAppUpdate(activity: Activity) : InstallStateUpdatedListener {
       
           private var appUpdateManager: AppUpdateManager
           private val MY_REQUEST_CODE = 500
           private var parentActivity: Activity = activity
       
           private var currentType = AppUpdateType.FLEXIBLE
       
           init {
               appUpdateManager = AppUpdateManagerFactory.create(parentActivity)
               appUpdateManager.appUpdateInfo.addOnSuccessListener { info ->
       //            ("Isupdatecheck $info").log()
                   // Check if update is available
                   if (info.updateAvailability() == UpdateAvailability.UPDATE_AVAILABLE) {
                       // UPDATE IS AVAILABLE
                       if (info.availableVersionCode() > BuildConfig.VERSION_CODE) {
                           startUpdate(info, AppUpdateType.FLEXIBLE)
                       }
       
       //                if (info.updatePriority() == 5) { // Priority: 5 (Immediate update flow)
       //                    if (info.isUpdateTypeAllowed(AppUpdateType.IMMEDIATE)) {
       //                        startUpdate(info, AppUpdateType.IMMEDIATE)
       //                    }
       //                } else if (info.updatePriority() == 4) { // Priority: 4
       //                    val clientVersionStalenessDays = info.clientVersionStalenessDays()
       //                    if (clientVersionStalenessDays != null && clientVersionStalenessDays >= 5 && info.isUpdateTypeAllowed(AppUpdateType.IMMEDIATE)) {
       //                        // Trigger IMMEDIATE flow
       //                        startUpdate(info, AppUpdateType.IMMEDIATE)
       //                    } else if (clientVersionStalenessDays != null && clientVersionStalenessDays >= 3 && info.isUpdateTypeAllowed(AppUpdateType.FLEXIBLE)) {
       //                        // Trigger FLEXIBLE flow
       //                        startUpdate(info, AppUpdateType.FLEXIBLE)
       //                    }
       //                } else if (info.updatePriority() == 3) { // Priority: 3
       //                    val clientVersionStalenessDays = info.clientVersionStalenessDays()
       //                    if (clientVersionStalenessDays != null && clientVersionStalenessDays >= 30 && info.isUpdateTypeAllowed(AppUpdateType.IMMEDIATE)) {
       //                        // Trigger IMMEDIATE flow
       //                        startUpdate(info, AppUpdateType.IMMEDIATE)
       //                    } else if (clientVersionStalenessDays != null && clientVersionStalenessDays >= 15 && info.isUpdateTypeAllowed(AppUpdateType.FLEXIBLE)) {
       //                        // Trigger FLEXIBLE flow
       //                        startUpdate(info, AppUpdateType.FLEXIBLE)
       //                    }
       //                } else if (info.updatePriority() == 2) { // Priority: 2
       //                    val clientVersionStalenessDays = info.clientVersionStalenessDays()
       //                    if (clientVersionStalenessDays != null && clientVersionStalenessDays >= 90 && info.isUpdateTypeAllowed(AppUpdateType.IMMEDIATE)) {
       //                        // Trigger IMMEDIATE flow
       //                        startUpdate(info, AppUpdateType.IMMEDIATE)
       //                    } else if (clientVersionStalenessDays != null && clientVersionStalenessDays >= 30 && info.isUpdateTypeAllowed(AppUpdateType.FLEXIBLE)) {
       //                        // Trigger FLEXIBLE flow
       //                        startUpdate(info, AppUpdateType.FLEXIBLE)
       //                    }
       //                } else if (info.updatePriority() == 1) { // Priority: 1
       //                    // Trigger FLEXIBLE flow
       //                    startUpdate(info, AppUpdateType.FLEXIBLE)
       //                } else { // Priority: 0
       //                    // Do not show in-app update
       //                }
                   } else {
                       // UPDATE IS NOT AVAILABLE
                   }
               }
               appUpdateManager.registerListener(this)
           }
       
           private fun startUpdate(info: AppUpdateInfo, type: Int) {
               appUpdateManager.startUpdateFlowForResult(info, type, parentActivity, MY_REQUEST_CODE)
               currentType = type
           }
       
           fun onResume() {
               appUpdateManager.appUpdateInfo.addOnSuccessListener { info ->
                   if (currentType == AppUpdateType.FLEXIBLE) {
                       // If the update is downloaded but not installed, notify the user to complete the update.
                       if (info.installStatus() == InstallStatus.DOWNLOADED) flexibleUpdateDownloadCompleted()
                   } else if (currentType == AppUpdateType.IMMEDIATE) {
                       // for AppUpdateType.IMMEDIATE only, already executing updater
                       if (info.updateAvailability() == UpdateAvailability.DEVELOPER_TRIGGERED_UPDATE_IN_PROGRESS) {
                           startUpdate(info, AppUpdateType.IMMEDIATE)
                       }
                   }
               }
           }
       
           fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
               if (requestCode == MY_REQUEST_CODE) {
                   if (resultCode != AppCompatActivity.RESULT_OK) {
                       // If the update is cancelled or fails, you can request to start the update again.
                       Log.e("ERROR", "Update flow failed! Result code: $resultCode")
                   }
               }
           }
       
           private fun flexibleUpdateDownloadCompleted() {
               Snackbar.make(
                   parentActivity.findViewById(android.R.id.content),
                   "An update has just been downloaded.",
                   Snackbar.LENGTH_INDEFINITE
               ).apply {
                   setAction("RESTART") { appUpdateManager.completeUpdate() }
                   setActionTextColor(Color.WHITE)
                   show()
               }
           }
       
           fun onDestroy() {
               appUpdateManager.unregisterListener(this)
           }
       
           override fun onStateUpdate(state: InstallState) {
               if (state.installStatus() == InstallStatus.DOWNLOADED) {
                   flexibleUpdateDownloadCompleted()
               }
           }
       }

       in MainAct
           private var inAppUpdate: InAppUpdate? = null


        inAppUpdate = InAppUpdate(this)

        inAppUpdate?.onActivityResult(requestCode, resultCode, data)
        inAppUpdate?.onResume()
        inAppUpdate?.onDestroy()

       
