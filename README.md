# testvasundhara.github.io


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

       
