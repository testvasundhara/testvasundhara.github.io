# testvasundhara.github.io


// Make a eassy way to make a bundle

    org.gradle.jvmargs=-Xmx4g -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8


// Ffmpeg Demo & Image to background ML Kit


    //ML KIT
    implementation(libs.segmentation.selfie)
    implementation ("com.google.mlkit:image-labeling:17.0.8")
         segmentation-selfie = { module = "com.google.mlkit:segmentation-selfie", version.ref = "segmentation-selfie" }segmentation-selfie = "16.0.0-beta5"
         
         //ffmpeg
             implementation("com.arthenica:ffmpeg-kit-full-gpl:6.0-2")



             // Ffmpege main command Class
                  
                               
                  fun mp4toAvi(input : String, output : String) {
                      val command = "-i \"$input\" \"$output\""
                      FFmpegKit.executeAsync(command) {
                          if(it.returnCode.value == ReturnCode.SUCCESS) {
                              Log.d("FFmpeg", "Command execution succeeded.")
                          } else if(it.returnCode.value == ReturnCode.CANCEL) {
                              Log.d("FFmpeg", "Command execution cancelled.")
                          } else {
                              Log.d("FFmpeg", "Command execution failed with code: ${it.returnCode.value}")
                          }
                      }
                  }
                  
                  fun Context.makeVideoFromImages(listOfImages : ArrayList<String>, output : String, dur : Int = 9, onGoPlay : (String) -> Unit) {
                      var duration = dur
                  
                      if(dur < listOfImages.size * 2) {
                          duration = (listOfImages.size * 2)
                          Log.e("TAG", "hdbvhusbcvhjudsvb: $duration")
                      }
                      duration += listOfImages.size - 1
                  //    if(listOfImages.size % 2 == 1) {
                  //        for (i in 1..listOfImages.size - 1) {
                  //            duration += 1
                  //        }
                  //    }
                      var eachImageDur = duration.toFloat().div(listOfImages.size)
                  
                      Log.e("KP FFMPEG", "makeSingleImageVideo: $listOfImages")
                      val folder = File(cacheDir, "FFMPEG Demo/temp")
                      val scope = CoroutineScope(Dispatchers.IO)
                      val countDownLatch = CountDownLatch(listOfImages.size)
                      var progressDialog = ProgressDialog(this)
                      progressDialog.apply {
                          setTitle("Making Video")
                          setCancelable(false)
                          setMessage("0% Progressing")
                      }
                      progressDialog.show()
                      var tmpTargetPercentage = 80
                      var targetDuration = duration * 1000f
                      var progressFile = 0f
                      var mainProgress = 0f
                      if(! folder.exists()) {
                          if(folder.mkdirs()) {
                              Log.e("KP FFMPEG", "1 makeSingleImageVideo: ${folder.absolutePath}")
                  
                              scope.launch {
                                  listOfImages.forEachIndexed { index, path ->
                                      launch {
                                          makeSingleImageVideo(eachImageDur, path, index, folder, {
                  
                                              (this@makeVideoFromImages as Activity).runOnUiThread {
                                                  if(! it) {
                                                      toast("have an Issue!")
                                                  }
                                              }
                  
                                              countDownLatch.countDown() // Signal completion of this task
                                          }, {
                                              progressFile += it
                                              mainProgress = (tmpTargetPercentage * progressFile) / targetDuration
                                              (this@makeVideoFromImages as Activity).runOnUiThread {
                                                  progressDialog.setMessage("${String.format("%.2f", mainProgress)} Progressing")
                                              }
                                          })
                                      }
                                  }
                                  countDownLatch.await() // Wait for all tasks to complete
                                  // After all makeSingleImageVideo calls are done
                                  var tmpTargetPercentageFinal = 20
                                  var targetDuration = duration * 1000.0f
                                  var lastProgress = 0.0f
                                  Log.e("TAG", "makeVideoFromImages: ------------------->lastProgress:$lastProgress")
                                  makeVideo(listOfImages, folder, output, duration, { result, path ->
                                      progressDialog.dismiss()
                                      if(result) {
                                          onGoPlay(path)
                                      }
                                  }, {
                                      lastProgress += it
                                      var progress = mainProgress + (tmpTargetPercentageFinal * lastProgress).div(targetDuration)
                                      progressDialog.setMessage("${String.format("%.2f", progress)} Progressing")
                  
                                  })
                              }
                  
                          }
                          Log.e("KP FFMPEG", "exists makeSingleImageVideo: ${folder.exists()}")
                  
                      } else {
                          Log.e("KP FFMPEG", "2 makeSingleImageVideo: ${folder.absolutePath}")
                          scope.launch {
                              listOfImages.forEachIndexed { index, path ->
                                  launch {
                                      makeSingleImageVideo(eachImageDur, path, index, folder, {
                                          (this@makeVideoFromImages as Activity).runOnUiThread {
                                              if(! it) {
                                                  toast("have an Issue!")
                                              }
                                          }
                                          countDownLatch.countDown() // Signal completion of this task
                                      }, {
                                          progressFile += it
                                          mainProgress = (tmpTargetPercentage * progressFile) / targetDuration
                                          (this@makeVideoFromImages as Activity).runOnUiThread {
                                              progressDialog.setMessage("${String.format("%.2f", mainProgress)} Progressing")
                                          }
                                      })
                                  }
                              }
                              countDownLatch.await() // Wait for all tasks to complete
                              // After all makeSingleImageVideo calls are done
                  
                              var tmpTargetPercentageFinal = 20
                              var targetDuration = duration * 1000.0f
                              var lastProgress = 0.0f
                              makeVideo(listOfImages, folder, output, duration, { result, path ->
                                  progressDialog.dismiss()
                                  if(result) {
                                      onGoPlay(path)
                                  }
                              }, {
                                  lastProgress += it
                                  Log.e("TAG", "makeVideoFromImages: ---------targetDuration :$targetDuration ---------->lastProgress:$lastProgress")
                  
                                  var progress = mainProgress + (tmpTargetPercentageFinal * lastProgress).div(targetDuration)
                                  progressDialog.setMessage("${String.format("%.2f", progress)} Progressing")
                  
                              })
                          }
                  
                      }
                  }
                  
                  
                  /*fun Context.makeVideo(listOfVideos : List<String>, folder : File, output : String) {
                      val fileListBuilder = StringBuilder()
                  
                      listOfVideos.forEachIndexed { index, s ->
                          fileListBuilder.append("file '").append(File(folder, "vd$index.mp4").absolutePath).append("'\n")
                      }
                      // Write the list to a file
                      val listFile = File(File(getExternalFilesDir(null), "FFMPEG Demo"), "file_list.txt")
                      try {
                          FileWriter(listFile).use { writer -> writer.write(fileListBuilder.toString()) }
                      } catch (e : IOException) {
                          e.printStackTrace()
                          return
                      }
                      // Build the complete FFmpeg command
                      val command = "-f concat -safe 0 -i \"${listFile.absolutePath}\" -c copy \"$output\""
                  
                      // Log the command for debugging
                      Log.e("ffmpeg-kit ", "makeVideo: $command")
                  
                      // Execute the command asynchronously
                      FFmpeg.executeAsync(command) { executionId, returnCode ->
                          deleteImagesData()
                  
                          listOfVideos.forEachIndexed { index, _ ->
                              val tempFile = File(folder, "vd$index.mp4")
                              if(tempFile.exists()) {
                                  tempFile.delete()
                              }
                          }
                          Log.e("KP FFMPEG", "listFile: ${listFile.absolutePath} returnCode: $returnCode")
                  
                          if(returnCode == RETURN_CODE_SUCCESS) {
                              val outputFile = File(output)
                              if(outputFile.exists()) {
                                  Log.d("FFmpeg KP", "Video created successfully at ${outputFile.absolutePath}")
                              } else {
                                  Log.d("FFmpeg KP", "Video creation failed.")
                              }
                              Log.d("FFmpeg KP", "Command execution succeeded.")
                          } else if(returnCode == RETURN_CODE_CANCEL) {
                              Log.d("FFmpeg KP", "Command execution cancelled.")
                          } else {
                              Log.d("FFmpeg KP", "Command execution failed with code: $returnCode")
                          }
                      }
                  }*/
                  
                  
                  /*
                  fun Context.makeVideo(listOfImages : List<String>, folder : File, output : String) {
                      // Build input files string
                      val listPath = listOfImages.indices.joinToString(separator = " ") { index ->
                          "-i \"${File(folder, "vd$index.mp4").absolutePath}\""
                      }
                  
                  // Build filter_complex string
                      val filterComplex = listOfImages.indices.joinToString(separator = ";") { index ->
                          "[${index}:v]scale=720:1280,setdar=16/9[v$index]"
                      } + "; " +
                              listOfImages.indices.joinToString(separator = ";") { index ->
                                  "[v$index]"
                              } +
                              "concat=n=${listOfImages.size}:v=1:a=0[v]"
                  
                  // Escape output path
                      val quotedOutputPath = "\"${output}\""
                  
                  // Build the complete FFmpeg command
                      val command = "$listPath -filter_complex \"$filterComplex\" -map \"[v]\" -r 25 -c:v libx264 -pix_fmt yuv420p -preset ultrafast $quotedOutputPath"
                  
                  // Log the command for debugging
                      Log.e("ffmpeg-kit ", "makeVideo: $command")
                  
                      // Execute the command asynchronously
                      FFmpeg.executeAsync(command) { executionId, returnCode ->
                          deleteImagesData()
                  
                          listOfImages.forEachIndexed { index, s ->
                              var tempFile = File(folder, "vd$index.mp4")
                              tempFile.delete()
                          }
                          Log.e("KP FFMPEG", "3 listPath  returnCode: $returnCode  $listPath ")
                  
                          if(returnCode == RETURN_CODE_SUCCESS) {
                              val outputFile = File(output)
                              if(outputFile.exists()) {
                                  Log.d("FFmpeg KP", "Video created successfully at ${outputFile.absolutePath}")
                              } else {
                                  Log.d("FFmpeg KP", "Video creation failed.")
                              }
                              Log.d("FFmpeg KP", "Command execution succeeded.")
                          } else if(returnCode == RETURN_CODE_CANCEL) {
                              Log.d("FFmpeg KP", "Command execution cancelled.")
                          } else {
                              Log.d("FFmpeg KP", "Command execution failed with code: $returnCode")
                          }
                      }
                  }
                  */
                  
                  fun Context.makeVideo1(listOfImages : List<String>, folder : File, output : String) {
                      // Create a text file with paths of input videos
                      val fileListPath = File(cacheDir, "filelist.txt")
                      fileListPath.writeText(listOfImages.joinToString(separator = "\n") { index ->
                          "file '$${File(folder, "vd$index.mp4").absolutePath}'"
                      })
                      // Build the FFmpeg command to concatenate videos using the file list
                      val command = "ffmpeg -f concat -safe 0 -i ${fileListPath.absolutePath} -c copy \"$output\""
                  
                  // Execute the command asynchronously
                      FFmpegKit.executeAsync(command) { session ->
                          when (session.returnCode.value) {
                              ReturnCode.SUCCESS -> {
                                  Log.d("FFmpeg_", "Merge successful: ${session.output}")
                              }
                  
                              ReturnCode.CANCEL -> {
                                  Log.d("FFmpeg_", "Merge canceled")
                              }
                  
                              else -> {
                                  Log.e("FFmpeg_", "Merge failed: ${session.statistics}")
                              }
                          }
                      }
                  
                  
                  }
                  
                  fun Context.makeVideo(
                      listOfImages : List<String>, folder : File, output : String, duration : Int, onComplate : (Boolean, String) -> Unit, progress : (Int) -> Unit
                  ) {
                      val transitionDuration = 1
                      var pr = 0.0
                      val inputOptions = listOfImages.indices.joinToString(separator = " ") { index ->
                          "-y -i \"${File(folder, "vd$index.mp4").absolutePath}\""
                      }
                      var lastOffsetValue = 0
                      val filerComplex = buildString {
                          // Task 3 : Prepare FilterComplex For Video Ratio and bitrate/ fps etc
                          if(listOfImages.size > 1) {
                              for (i in 0 until listOfImages.size) {
                                  append("[$i:v]settb=AVTB,setdar=720/1280,fps=25[v${i}];")
                              }
                              var totalVideoLength = 0
                  
                              // Task 4 : Prepare FilterComplex For Each Temp Video's Transitions
                              for (i in 0 until listOfImages.size - 1) {
                                  val currentVideoLength = duration.div(listOfImages.size)
                                  totalVideoLength += currentVideoLength
                                  if(i == 0) {
                                      if(listOfImages.size - 1 == 1) {
                                          lastOffsetValue = totalVideoLength - transitionDuration
                                          val transaction =
                                              "[v0][v1]xfade=transition=${transitionList.get((0..transitionList.size - 1).random())}:duration=$transitionDuration:offset=${lastOffsetValue}"
                                          append(transaction)
                                          Log.e("allVideosList", "makeVideo: idfhnn 1")
                                      } else {
                                          lastOffsetValue = totalVideoLength - transitionDuration
                                          val transaction =
                                              "[v0][v1]xfade=transition=${transitionList.get((0..transitionList.size - 1).random())}:duration=$transitionDuration:offset=${lastOffsetValue}[f0]; "
                                          append(transaction)
                                          Log.e("allVideosList", "makeVideo: idfhnn 2")
                                      }
                                  } else if(i == listOfImages.lastIndex - 1) {
                                      lastOffsetValue = totalVideoLength - ((i + 1) * transitionDuration)
                                      val transaction =
                                          "[f${i - 1}][v${i + 1}]xfade=transition=${transitionList.get((0..transitionList.size - 1).random())}:duration=$transitionDuration:offset=${lastOffsetValue}"
                                      append(transaction)
                                      Log.e("allVideosList", "makeVideo: idfhnn 3")
                                  } else {
                                      lastOffsetValue = totalVideoLength - ((i + 1) * transitionDuration)
                                      val transaction =
                                          "[f${i - 1}][v${i + 1}]xfade=transition=${transitionList.get((0..transitionList.size - 1).random())}:duration=$transitionDuration:offset=${lastOffsetValue}[f$i]; "
                                      append(transaction)
                                      Log.e("allVideosList", "makeVideo: idfhnn 4")
                                  }
                                  Log.e("TAG", "makeVideo: lastOffsetValue---> $lastOffsetValue  $currentVideoLength $totalVideoLength")
                              }
                          }
                      }
                      val quotedOutputPath = "\"${output}\""
                  
                      var command = ""
                      if(filerComplex.isEmpty()) {
                          command = "-y $inputOptions -c:v copy $quotedOutputPath"
                      } else {
                          command =
                              "$inputOptions" + " -filter_complex" + " \"${filerComplex}\"" + " -r 25" + " -c:v libx264" + " -pix_fmt yuv420p" + " -preset ultrafast" + " $quotedOutputPath"
                      }
                      Log.e("ffmpeg-kit ", "makeVideo: $command")
                  
                      /**
                       *     val command = "-y -i \"/data/user/0/com.kp.demo.bgremoval/cache/FFMPEG Demo/temp/vd0.mp4\" -i \"/data/user/0/com.kp.demo.bgremoval/cache/FFMPEG Demo/temp/vd1.mp4\" -filter_complex \"[0:v]scale=720:1280[v0];[1:v]scale=720:1280[v1];[v0][v1]xfade=transition=fade:duration=0.5:offset=0.0[vout]\" -map \"[vout]\" \"/storage/emulated/0/Pictures/FFMPEG Demo/30_07_2024__16_54_57.mp4\""
                       * */
                  // Execute the command asynchronously
                      FFmpegKit.executeAsync(command, {
                          deleteImagesData()
                  //        listOfImages.forEachIndexed { index, s ->
                  //            var tempFile = File(folder, "vd$index.mp4")
                  //            tempFile.delete()
                  //        }
                          Log.e("KP FFMPEG", "3 listPath  returnCode: ${it.returnCode}  ")
                  
                  
                          if(it.returnCode.value == ReturnCode.SUCCESS) {
                              val outputFile = File(output)
                              this.runOnUi {
                                  onComplate(true, outputFile.absolutePath)
                                  toast("Video created successfully")
                              }
                              if(outputFile.exists()) {
                  
                                  Log.d("FFmpeg KP", "Video created successfully at ${outputFile.absolutePath}")
                              } else {
                                  Log.d("FFmpeg KP", "Video creation failed.")
                              }
                              Log.d("FFmpeg KP", "Command execution succeeded.")
                          } else if(it.returnCode.value == ReturnCode.CANCEL) {
                              this.runOnUi {
                                  onComplate(false, "")
                                  toast("Video creation cancelled.")
                              }
                              Log.d("FFmpeg KP", "Command execution cancelled.")
                          } else {
                              this.runOnUi {
                                  onComplate(false, "")
                                  toast("Video creation failed.")
                              }
                              Log.d("FFmpeg KP", "Command execution failed with code: ${it.returnCode.value}")
                          }
                      }, {
                  
                      }, {
                          var progressTime = it.time - pr
                          pr = it.time
                          this.runOnUi {
                              progress(progressTime.toInt())
                          }
                          Log.e("FFmpeg KP", "makeVideo: Progress -> ${it.toJson}")
                      })
                  }
                  
                  suspend fun makeSingleImageVideo(
                      eachImageDur : Float, imagePath : String, index : Int, folder : File, onComplete : (Boolean) -> Unit, progress : (Int) -> Unit
                  ) {
                      var pr = 0.0
                      var resizeImagePath = File(folder, "tmp_image_$index.jpg")
                      // Task 1 : Making temp Images for next Temp Video Operations
                      var resizeCommand =
                          "-y" + " -i \"${imagePath}\"" + " -vf \"[0:v]split=2[blur][vid];" + " [blur]scale=720:1280:force_original_aspect_ratio=increase,crop=720:1280,boxblur=luma_radius=min(h\\,w)/5:luma_power=1:chroma_radius=min(cw\\,ch)/5:chroma_power=1[bg];" + " [vid]scale=720:1280:force_original_aspect_ratio=decrease[ov];" + " [bg][ov]overlay='(main_w-overlay_w)/2:(main_h-overlay_h)/2'\"" + " -preset ultrafast" + " \"${resizeImagePath}\""
                  
                      FFmpegKit.executeAsync(resizeCommand, {
                          if(ReturnCode.isSuccess(it.returnCode)) {
                              var tempFile = File(folder, "vd$index.mp4")
                  
                              // Task 2 : Making temp Video for Final Video
                              var command =
                  //        "-y" + " -framerate 30" + " -i \"$imagePath\"" + " -t ${eachImageDur.toString()} -c:v libx264 -pix_fmt yuv420p -preset ultrafast \"${tempFile.absolutePath}\""
                                  "-y -loop 1" + " -t $eachImageDur" + " -framerate 24" + " -i \"${resizeImagePath}\"" + " -c:v libx264" + " -pix_fmt yuv420p" + " -preset ultrafast" + " \"${tempFile.absolutePath}\""
                  
                              Log.e("KP FFMPEG", "FFMPEG Command: $command")
                  
                              FFmpegKit.executeAsync(command, {
                                  Log.e("ffmpeg-kit ", "makeSingleImageVideo:${tempFile.absolutePath}  ${it.returnCode.value == ReturnCode.SUCCESS} --> ${it.returnCode.value}")
                                  onComplete(ReturnCode.isSuccess(it.returnCode))
                              }, {
                  
                              }, {
                                  var progressTime = it.time - pr
                                  pr = it.time
                                  progress(progressTime.toInt())
                                  Log.d("TAG", "makeSingleImageVideo: progress -->$index ${it.toJson}")
                              })
                          } else {
                              Log.d("TAG", "resize Image :FAILED $$resizeImagePath ====>")
                  
                          }
                      }, {}, {
                          Log.e("TAG", "resize Image : status --> ${it.toJson}")
                      })
                  
                  
                  }
                  
                  fun Context.executeFFmpegCommand(input : File, output : String, durationPerImage : Int = 5) {
                      // Wrap paths with quotes
                      val quotedInputPath = "\"${input.absolutePath}\""
                      val quotedOutputPath = "\"${output}\""
                  
                      // Construct the command string
                  //    val command = "-f concat -safe 0 -i $quotedInputPath -vf fps=1/$durationPerImage -c:v libx264 -pix_fmt yuv420p -r 1/$durationPerImage $quotedOutputPath"
                  //    val command = "-f concat -safe 0 -i $quotedInputPath -vf \"scale=ceil(iw/2)*2:ceil(ih/2)*2\" -c:v libx264 -r 30 -pix_fmt yuv420p $quotedOutputPath"
                  
                  //        val command = "-f concat -safe 0 -i $quotedInputPath -vf \"scale=720:1280,setsar=1\" -framerate 1 -t 00:00:05 -c:v libx264 -r 24 -pix_fmt yuv420p $quotedOutputPath"
                      val command = "-f concat -safe 0 -i $quotedInputPath -vf \"scale=720:1280,setsar=1\" -t 5 -framerate 25 -c:v libx264 -pix_fmt yuv420p $quotedOutputPath"
                  
                  
                      Log.d("FFmpeg", "Command: $command")
                  
                      // Execute the command asynchronously
                      FFmpegKit.executeAsync(command) {
                          deleteImagesData()
                  
                          if(it.returnCode.value == ReturnCode.SUCCESS) {
                              val outputFile = File(output)
                              if(outputFile.exists()) {
                                  Log.d("FFmpeg KP", "Video created successfully at ${outputFile.absolutePath}")
                              } else {
                                  Log.d("FFmpeg KP", "Video creation failed.")
                              }
                              Log.d("FFmpeg KP", "Command execution succeeded.")
                          } else if(it.returnCode.value == ReturnCode.CANCEL) {
                              Log.d("FFmpeg KP", "Command execution cancelled.")
                          } else {
                              Log.d("FFmpeg KP", "Command execution failed with code: ${it.returnCode.value}")
                          }
                      }
                  }


                  --- appUtils.kt
                      
    fun Context.toast(st : String) {
        Toast.makeText(this, st, Toast.LENGTH_SHORT).show()
    }
    
    fun Context.checkPermissions() : Boolean {
        return if(Build.VERSION.SDK_INT >= 33) {
            PermissionChecker.checkSelfPermission(
                    this, Manifest.permission.READ_MEDIA_IMAGES
            ) == PermissionChecker.PERMISSION_GRANTED
        } else {
            PermissionChecker.checkSelfPermission(
                    this, Manifest.permission.READ_EXTERNAL_STORAGE
            ) == PermissionChecker.PERMISSION_GRANTED && PermissionChecker.checkSelfPermission(
                    this, Manifest.permission.WRITE_EXTERNAL_STORAGE
            ) == PermissionChecker.PERMISSION_GRANTED
        }
    }
    
    fun Context.runOnUi(action:Runnable){
        (this as Activity).runOnUiThread(action)
    }
    
    val Any.toJson : String
        get() = Gson().toJson(this)

          --- utils.kt
            
            
                      
            
            fun View.click(action : (View) -> Unit) = setOnClickListener {
                action(it)
            }
            
            fun processImageFromUri(context : Context, imageUri : Uri?) : Bitmap? {
                try {
                    // Get the content resolver
                    val contentResolver : ContentResolver = context.contentResolver
            
                    // Open an input stream from the URI
                    val inputStream = contentResolver.openInputStream(imageUri !!)
            
                    // Decode the stream into a bitmap
                    val originalBitmap = BitmapFactory.decodeStream(inputStream)
            
                    // Close the input stream
                    inputStream !!.close()
            
                    // Process the bitmap with segmentation
                    return originalBitmap
                } catch (e : IOException) {
                    // Handle potential exceptions (e.g., file not found, permission issues)
                    Log.e("TAG", "Error loading image from URI: " + e.message)
                }
            
                return null
            }
            //fun getMutableBitmapFromUri(context : Context, uri : Uri, config : Bitmap.Config) : Bitmap? {
            //    var inputStream : InputStream? = null
            //    return try {
            //        inputStream = context.contentResolver.openInputStream(uri)
            //        val options = BitmapFactory.Options()
            //        options.inMutable = true
            //        val bitmap = BitmapFactory.decodeStream(inputStream, null, options)
            //        if(bitmap != null && bitmap.config != config) {
            //            val mutableBitmap = bitmap.copy(config, true)
            //            bitmap.recycle() // Recycle the original bitmap
            //            mutableBitmap // Return the mutable bitmap with the desired configuration
            //        } else {
            //            bitmap // Return the original mutable bitmap
            //        }
            //    } catch (e : Exception) {
            //        e.printStackTrace()
            //        null
            //    } finally {
            //        inputStream?.close()
            //    }
            //}
            
            // Function to export the bitmap
            fun View.exportBitmap(bitmap : Bitmap, quality : Int, filename : String) {
                // Get the directory for the app's private pictures directory.
            
                var path = File(
                        Environment.getExternalStoragePublicDirectory(
                                Environment.DIRECTORY_PICTURES
                        ).absolutePath + "/Bg Removal"
                )
                if(! path.exists()) {
                    path.mkdir()
                }
                val file = File(path, filename)
            //    if(! file.exists()) {
            //        file.createNewFile()
            //    }
            
                // Initialize a FileOutputStream object
            //    val outputStream : OutputStream = FileOutputStream(file)
            
                tryWithLogging {
                    // Compress the bitmap into the OutputStream
            //        bitmap.compress(Bitmap.CompressFormat.PNG, quality, outputStream)
            //        // Flushes the stream
            //        outputStream.flush()
            //        // Closes the stream
            //        outputStream.close()
                    saveBitmapWithTransparency(bitmap, file)
                }
            }
            
            suspend fun getSelfieSegmentBitmap(
                source : Any,
                context : Context,
                response : (Bitmap, SEGMENT_RESULT) -> Unit,
                process : (Int) -> Unit
            ) {
                var originalBitmap : Bitmap? = null
                if(source is Bitmap) {
                    originalBitmap = source
                } else if(source is Uri) {
                    originalBitmap = processImageFromUri(context, source)
                } else {
            
                }
                val options = SelfieSegmenterOptions.Builder().setDetectorMode(SelfieSegmenterOptions.SINGLE_IMAGE_MODE).build()
                val segmenter = Segmentation.getClient(options)
                segmenter.process(originalBitmap !!, 0).addOnSuccessListener { results ->
                    // Task completed successfully
                    if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
                        response.invoke(startImageBuffer(results, originalBitmap, process) !!, SEGMENT_RESULT.SUCCESS)
                    }
            
                }.addOnFailureListener { e ->
                    response.invoke(originalBitmap, SEGMENT_RESULT.FAIL)
                }
            }
            
            private fun startImageBuffer(segmentationMask : SegmentationMask, bitmap : Bitmap, process : (Int) -> Unit) : Bitmap? {
                val mask = segmentationMask.getBuffer()
                val width = segmentationMask.getWidth()
                val height = segmentationMask.getHeight()
            
            //        var segmentedImage = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888)
                var res = height * width
                var s=res/6
                var p = 0
                var per = 0
                var segmentedImage = bitmap !!.copy(bitmap !!.config, true)
                for (y in 0 until height) {
                    for (x in 0 until width) {
                        val foregroundConfidence : Float = mask.float
                        p ++
                        if(foregroundConfidence < 0.7f) {// < for Bg & > for person
            
                            var s = if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
                                var c = bitmap.getColor(x, y)
                                Color.rgb(c !!.red(), c !!.green(), c !!.blue())
                            } else {
                                Color.argb(((1.0 - foregroundConfidence) * 255.0).toInt(), 0, 0, 0)
                            }
                            segmentedImage.setPixel(x, y, s)
                        } else {
                            segmentedImage.setPixel(x, y, Color.TRANSPARENT)
                        }
                    }
            //        if((y*width)%s==0) {
            //            if(((100 * p) / res) != per) {
            //                per = ((100 * p) / res)
            //                CoroutineScope(Dispatchers.Main).launch {
            //                    process.invoke(per)
            //                }
            //            }
            //        }
                }
                mask.rewind()
                return segmentedImage
            }
            
            
            fun saveBitmapWithTransparency(erasedBitmap : Bitmap, outputFile : File) {
                // Create a new bitmap with an alpha channel
                val resultBitmap = Bitmap.createBitmap(erasedBitmap.width, erasedBitmap.height, Bitmap.Config.ARGB_8888)
            
                // Create a canvas with the result bitmap
                val canvas = Canvas(resultBitmap)
            
                // Draw the erased bitmap onto the result bitmap's canvas
                canvas.drawBitmap(erasedBitmap, 0f, 0f, null)
            
                // Save the result bitmap to a file
                try {
                    FileOutputStream(outputFile).use { out ->
                        resultBitmap.compress(Bitmap.CompressFormat.PNG, 100, out)
                    }
                    Log.d("SaveBitmap", "Bitmap saved successfully")
                } catch (e : IOException) {
                    e.printStackTrace()
                    Log.e("SaveBitmap", "Error saving bitmap: ${e.message}")
                }
            }
            
            inline fun <T> tryWithLogging(block : () -> T) : T? {
                return try {
                    block()
                } catch (e : Exception) {
                    // Log the exception here
                    Log.e("TAG", "tryWithLogging: Exception occurred: ${e.message}")
                    null
                }
            }
            
            fun cropBitmap(bitmap : Bitmap, targetWidth : Int, targetHeight : Int) : Bitmap? {
                // Calculate the dimensions for cropping
                val width = bitmap.width
                val height = bitmap.height
            
                // Calculate the final crop dimensions
                val cropWidth = if(width >= targetWidth) targetWidth else width
                val cropHeight = if(height >= targetHeight) targetHeight else height
            
                // Calculate the starting position for cropping
                val x = (width - cropWidth) / 2
                val y = (height - cropHeight) / 2
            
                // Create a cropped bitmap
                return try {
                    Bitmap.createBitmap(bitmap, x, y, cropWidth, cropHeight)
                } catch (e : Exception) {
                    // Handle any exceptions, such as OutOfMemoryError
                    e.printStackTrace()
                    null
                }
            }

                  
         ---  fileUtils.kt
                  
                           
                  fun getFilePathFromUri(context : Context, uri : Uri) : String? {
                      var filePath : String? = null
                  
                      // DocumentProvider
                      if(DocumentsContract.isDocumentUri(context, uri)) {
                          when {
                              isExternalStorageDocument(uri) -> {
                                  val docId = DocumentsContract.getDocumentId(uri)
                                  val split = docId.split(":").toTypedArray()
                                  val type = split[0]
                                  if("primary".equals(type, ignoreCase = true)) {
                                      filePath = Environment.getExternalStorageDirectory().toString() + "/" + split[1]
                                  } else {
                                      // Handle non-primary volumes
                                      val externalMediaDirs = context.externalMediaDirs
                                      for (file in externalMediaDirs) {
                                          if(file.absolutePath.contains(type)) {
                                              filePath = file.absolutePath.substringBefore("/Android") + "/" + split[1]
                                              break
                                          }
                                      }
                                  }
                              }
                  
                              isDownloadsDocument(uri) -> {
                                  val id = DocumentsContract.getDocumentId(uri)
                                  if(id.startsWith("raw:")) {
                                      filePath = id.removePrefix("raw:")
                                  } else {
                                      val contentUri = ContentUris.withAppendedId(
                                              Uri.parse("content://downloads/public_downloads"), id.toLongOrNull() ?: return null
                                      )
                                      filePath = getDataColumn(context, contentUri, null, null)
                                  }
                              }
                  
                              isMediaDocument(uri) -> {
                                  val docId = DocumentsContract.getDocumentId(uri)
                                  val split = docId.split(":").toTypedArray()
                                  val type = split[0]
                                  val contentUri : Uri = when (type) {
                                      "image" -> MediaStore.Images.Media.EXTERNAL_CONTENT_URI
                                      "video" -> MediaStore.Video.Media.EXTERNAL_CONTENT_URI
                                      "audio" -> MediaStore.Audio.Media.EXTERNAL_CONTENT_URI
                                      else -> return null
                                  }
                                  val selection = "_id=?"
                                  val selectionArgs = arrayOf(split[1])
                                  filePath = getDataColumn(context, contentUri, selection, selectionArgs)
                              }
                          }
                      }
                      // MediaStore (and general)
                      else if("content".equals(uri.scheme, ignoreCase = true)) {
                          filePath = getDataColumn(context, uri, null, null)
                      }
                      // File
                      else if("file".equals(uri.scheme, ignoreCase = true)) {
                          filePath = uri.path
                      }
                  
                      return filePath
                  }
                  
                  private fun isExternalStorageDocument(uri : Uri) : Boolean {
                      return "com.android.externalstorage.documents" == uri.authority
                  }
                  
                  private fun isDownloadsDocument(uri : Uri) : Boolean {
                      return "com.android.providers.downloads.documents" == uri.authority
                  }
                  
                  private fun isMediaDocument(uri : Uri) : Boolean {
                      return "com.android.providers.media.documents" == uri.authority
                  }
                  
                  fun Context.deleteImagesData() {
                      File(File(getExternalFilesDir(null), "FFMPEG Demo"), "images.txt").delete()
                      File(File(getExternalFilesDir(null), "FFMPEG Demo"), "file_list.txt").delete()
                      File(filesDir, "filelist.txt").delete()
                  }
                  
                  private fun getDataColumn(context : Context, uri : Uri, selection : String?, selectionArgs : Array<String>?) : String? {
                      var cursor : Cursor? = null
                      val column = "_data"
                      val projection = arrayOf(column)
                      try {
                          cursor = context.contentResolver.query(uri, projection, selection, selectionArgs, null)
                          if(cursor != null && cursor.moveToFirst()) {
                              val columnIndex = cursor.getColumnIndexOrThrow(column)
                              return cursor.getString(columnIndex)
                          }
                      } finally {
                          cursor?.close()
                      }
                      return null
                  }
                  
                  val getRandomName : String
                      get() = SimpleDateFormat("dd_MM_yyyy__HH_mm_ss", Locale.getDefault()).format(Date())
                  
                  //fun getTmpImagePath() : String {
                  //    var name= (20..5000).random().toString()+".jpg"
                  //File()
                  //
                  //}
                  
                  fun prepareFileList(filePaths : List<String>, context : Context, folder : File) : File {
                      val fileListContent = filePaths.indices.joinToString("\n") { index ->
                          "file '${File(folder, "vd$index.mp4").absolutePath}'"
                      }
                      val fileList = File(context.filesDir, "filelist.txt")
                      fileList.writeText(fileListContent)
                      return fileList
                  }
                  
                  fun Context.getBitmapFromUri( uri: Uri): Bitmap? {
                      return try {
                          val inputStream = contentResolver.openInputStream(uri)
                          inputStream?.use {
                              BitmapFactory.Options().run {
                                  inPreferredConfig = Bitmap.Config.ARGB_8888 // Ensure high-quality bitmap
                                  BitmapFactory.decodeStream(it, null, this)
                              }
                          } ?: run {
                              // Log or handle the case where inputStream is null
                              Log.e("TAG", "getBitmapFromUri: ee", )
                              null
                          }
                      } catch (e: Exception) {
                          Log.e("TAG", "getBitmapFromUri: $e", )
                          e.printStackTrace() // Log error
                          null
                      }
                  }

         --- MLKiitUtil.kt
                           
                  fun Activity.bgDetection(uri : Uri, info : (String) -> Unit) {
                  //        binding.imgState.setImageURI(uri)
                      var progressDialog = ProgressDialog(this)
                      progressDialog.apply {
                          setTitle("Objects Find")
                          setCancelable(false)
                          setMessage("0% Progressing")
                      }
                      if(! isFinishing) {
                          progressDialog.show()
                      }
                      var process = - 1
                  
                  
                      CoroutineScope(Dispatchers.IO).launch {
                          var mainProgress = 0
                          getSelfieSegmentBitmap(uri, this@bgDetection, { selfie, segmentResult ->
                              takeLabels(selfie) { process, info ->
                                  mainProgress = process
                                  CoroutineScope(Dispatchers.Main).launch {
                                      progressDialog.setMessage("$mainProgress% Progressing")
                                      info(info)
                                      Handler(Looper.getMainLooper()).postDelayed({
                                          if (!isFinishing) {
                                              progressDialog.dismiss()
                                          }
                                      }, 1200)
                                  }
                              }
                          }, { it ->
                              if (it != process) {
                                  process = it
                                  Log.e("TAG", "bgDetection: -progress-> $process")
                                  CoroutineScope(Dispatchers.Main).launch {
                                      progressDialog.setMessage("$process% Progressing")
                                  }
                              }
                          })
                      }
                  
                  }
                  
                  fun Activity.takeLabels(bitmap : Bitmap?, process : (Int, String) -> Unit = { _, o -> }) {
                      val image : InputImage
                      try {
                          image = InputImage.fromBitmap(bitmap !!, 0)
                          val labeler = ImageLabeling.getClient(ImageLabelerOptions.DEFAULT_OPTIONS)
                  
                  
                          labeler.process(image).addOnSuccessListener {
                              Log.e("ML KIT Scope", "takeLabels: Success!")
                              var info = ""
                              it.forEachIndexed { index, information ->
                                  info += "${index + 1}:\t${information.text}\n"
                                  Log.e("ML KIT Scope", "takeLabels: lebel ${information.index}:\n\t${information.text}\n\t${information.confidence}")
                              }
                              process(99, info)
                  
                          }.addOnFailureListener {
                              Log.e("ML KIT Scope", "takeLabels: Fail :${it.message}")
                          }
                  
                      } catch (e : IOException) {
                          e.printStackTrace()
                      }
                  
                  
                  }

              ---- const.kt

                                
                  object Constant {
                  
                      var VIDEO_PATH = "video_path"
                  
                  
                      var transitionList = listOf(
                              "fade",
                              "fadeblack",
                              "fadewhite",
                              "distance",
                              "wipeleft",
                              "wiperight",
                              "wipeup",
                              "wipedown",
                              "slideleft",
                              "slideright",
                              "slidedown",
                              "smoothleft",
                              "smoothright",
                              "smoothup",
                              "smoothdown",
                              "circlecrop",
                              "rectcrop",
                              "circleclose",
                              "circleopen",
                              "horzclose",
                              "horzopen",
                              "vertclose",
                              "vertopen",
                              "diagbl",
                              "diagbr",
                              "diagtl",
                              "diagtr",
                              "hlslice",
                              "hrslice",
                              "vuslice",
                              "vdslice",
                              "dissolve",
                              "pixelize",
                              "radial",
                              "hblur",
                              "wipetl",
                              "wipetr",
                              "wipebl",
                              "wipebr",
                              "squeezev",
                              "squeezeh",
                              "zoomin",
                      )
                  }


// Crop and resize bitmap images resizebitmap cropbitmap

         fun cropAndResizeBitmap(
             src: Bitmap?,
             startX: Int,
             startY: Int,
             targetWidth: Int,
             targetHeight: Int
         ): Bitmap? {
             var startX = startX
             var startY = startY
             var targetWidth = targetWidth
             var targetHeight = targetHeight
             if (src == null || targetWidth <= 0 || targetHeight <= 0 || startX < 0 || startY < 0) {
                 return null
             }
             val srcWidth = src.width
             val srcHeight = src.height
     
             // Ensure crop area stays within source image bounds (avoid out-of-bounds issues)
             startX = Math.max(0, startX)
             startY = Math.max(0, startY)
     
             // Calculate the maximum possible crop width and height within the source image
             val maxWidth = srcWidth - startX
             val maxHeight = srcHeight - startY
     
             // Adjust target dimensions if they exceed source image boundaries
             targetWidth = Math.min(targetWidth, maxWidth)
             targetHeight = Math.min(targetHeight, maxHeight)
     
             // Calculate center point within the crop area
             val centerX = startX + targetWidth / 2
             val centerY = startY + targetHeight / 2
     
             // Calculate adjusted crop coordinates to maintain center based on target dimensions
             var cropX = centerX - targetWidth / 2
             var cropY = centerY - targetHeight / 2
     
             // Ensure final crop area stays within source image bounds
             cropX = Math.max(0, cropX)
             cropY = Math.max(0, cropY)
     
             // Calculate final crop width and height (might be smaller if source is smaller)
             val cropWidth = Math.min(targetWidth, srcWidth - cropX)
             val cropHeight = Math.min(targetHeight, srcHeight - cropY)
     
             // Create a new cropped bitmap
             val croppedBitmap: Bitmap
             croppedBitmap = try {
                 Bitmap.createBitmap(src, cropX, cropY, cropWidth, cropHeight)
             } catch (e: OutOfMemoryError) {
                 Log.e("CropAndResizeBitmap", "Error cropping image: OutOfMemoryError", e)
                 return null
             }
     
             // Resize the cropped bitmap (optional)
             var resizedBitmap: Bitmap? = null
             if (targetWidth != cropWidth || targetHeight != cropHeight) {
                 resizedBitmap = resizeBitmap(croppedBitmap, targetWidth, targetHeight)
                 croppedBitmap.recycle() // Recycle croppedBitmap if resized version is needed
             } else {
                 resizedBitmap = croppedBitmap // No resizing needed, use the cropped bitmap directly
             }
             return resizedBitmap
         }
     
         fun resizeBitmap(src: Bitmap?, targetWidth: Int, targetHeight: Int): Bitmap? {
             return if (src == null || targetWidth <= 0 || targetHeight <= 0) {
                 null
             } else try {
                 // Create a new resized bitmap with desired dimensions
                 Bitmap.createScaledBitmap(src, targetWidth, targetHeight, true)
             } catch (e: OutOfMemoryError) {
                 Log.e("ResizeBitmap", "Error scaling image: OutOfMemoryError", e)
                 null
             }
         }


                val srcBitmap = BitmapFactory.decodeResource(
                 resources,
                 com.example.testproject.R.drawable.testforbigimg
             )
     
             val targetWidth = 1440
             val targetHeight = 2020
     
     // Resize and center crop the image
             srcBitmap.width.log()
     
             val croppedBitmap = cropAndResizeBitmap(
                 srcBitmap,
                 srcBitmap.width / 2,
                 srcBitmap.height / 2,
                 targetWidth,
                 targetHeight
             )
             

// Lassi For Pick Images & Videos

     https://github.com/Mindinventory/Lassi-Android


      if (checkStoragePermission()) {
                    val intent = Lassi(this@HomeActivity)
                        .with(LassiOption.GALLERY)
                        .setMaxCount(100)
                        .setGridSize(3)
                        .setMinTime(3)
                        .setMaxTime(300)
                        .setMinFileSize(0)
                        .setMaxFileSize(100000)
                        .setMediaType(MediaType.VIDEO)
                        .setPlaceHolder(com.lassi.R.drawable.ic_image_placeholder)
                        .setErrorDrawable(com.lassi.R.drawable.ic_image_placeholder)
                        .build()
                    inAppVideoSelectionLauncher.launch(intent)
                } else {
                    toastLong(resources.getString(R.string.please_allow_permission_first_after_you_can_choose_images_or_videos))
                }

// WhatsApp Sticker add to Whatsapp


     Android Menifest

       <queries>

        <!-- Explicit apps you know in advance about: -->
        <package android:name="com.whatsapp" />
        <package android:name="com.whatsapp.w4b" />
    </queries>



      <provider
            android:name=".foldername.whatsappsticker.StickerContentProvider"
            android:authorities="${contentProviderAuthority}"
            android:exported="true"
            android:readPermission="com.whatsapp.sticker.READ"
            android:grantUriPermissions="true">
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <category android:name="android.intent.category.DEFAULT"/>
            </intent-filter>
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/provider_path" />
        </provider>


        provider_path.xml
        
    <paths xmlns:android="http://schemas.android.com/apk/res/android">
        <external-path name="external_files" path="Android/data/com.name.photo.birthday.cake.quotes.frame.editor/files/StickerPacks/"/>
    </paths>


        ---- First Download a file path and give file path and identifire 
        
      public fun Activity.addStickerPackToWhatsAppKP(stickerPackIdentifier : String?, stickerPackName : String?) {
            Log.e(TAG, "addStickerPackToWhatsAppKP: $stickerPackIdentifier")
            val intent = Intent()
            intent.setAction("com.whatsapp.intent.action.ENABLE_STICKER_PACK")
            intent.putExtra("sticker_pack_id", stickerPackIdentifier)
            intent.putExtra("sticker_pack_authority", BuildConfig.CONTENT_PROVIDER_AUTHORITY)
            intent.putExtra("sticker_pack_name", stickerPackName)
            try {
                startActivityForResult(intent, ADD_PACK)
            } catch (e : ActivityNotFoundException) {
                Toast.makeText(this, "WhatsApp not installed.", Toast.LENGTH_SHORT).show()
            }
        }


// Revanue Cat

            --Splace Screen 
            
           RevenueCatHelper.revenueCateInit(this,
                     BuildConfig.REVANUECAT_ID,
                     "Current Plan",
                     object : ProductPurchaseListener {
                         override fun onPurchase(message: String) {
                             message.messageLog()
                         }
         
                         override fun onPurchased(purchase: Purchase) {
                         }
         
                         override fun onRevenueCatPurchased(purchases: CustomerInfo) {
         //                        if(purchases.activeSubscriptions)
                             Log.e("TAG", "onRevenueCatPurchased: ${purchases.activeSubscriptions}")
                             Log.e("TAG", "onRevenueCatPurchased: ${purchases.entitlements.active}")
         
                             val data = purchases.entitlements.active.toString()
                             val isActive: Boolean = extractBooleanValue(data, "isActive")
                             ("isActive: $isActive").log("onRevenueCatPurchased")
                             SubsSharedFile(this@SplashActivity).isSubscriber =
                                 purchases.activeSubscriptions.isNotEmpty()
                             subsSharedFile?.purchaseProduct = purchases.toGson()
                             subsSharedFile?.purchaseProduct?.logPurchase()
                         }
         
                         override fun onPurchaseFail(errorMessage: String) {
         
                         }
                     }) {
                     runOnUiThread {
                         if (isFirstTime) {
                             isFirstTime = false
                             ("Time 2222222").log("TIMECHECK")
                             gotoLoadApp()
                         }
                     }
                 }



           ---- Revanue Cat Class
                  
                                                 object RevenueCatHelper {
                      var currentSub: Map<String, EntitlementInfo>? = null
                      var isSplash = true
                  
                      var liverevenuecatedatalist = MutableLiveData<List<RevenueCatData>>()
                      var revenueCatDataList = ArrayList<RevenueCatData>()
                      var allAvailablePackages = ArrayList<MyOffer>()
                      var currentUserPackage = MyOffer()
                  
                      private fun saveJsonToFile(jsonString: String) {
                          val path = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOCUMENTS)
                          val file = File(path, "ravanuecat.json")
                          try {
                              path.mkdirs()
                              val fos = FileOutputStream(file)
                              fos.write(jsonString.toByteArray())
                              fos.close()
                  //                Toast.makeText(this, "JSON saved to " + file.path, Toast.LENGTH_LONG).show()
                          } catch (e: IOException) {
                              e.printStackTrace()
                  //                Toast.makeText(this, "Error saving JSON", Toast.LENGTH_LONG).show()
                          }
                      }
                  
                      fun revenueCateInit(
                          context: Activity,
                          REVENUECAT_GOOGLE_API: String = BuildConfig.REVANUECAT_ID,
                          abTestPackage: String,
                          productPurchaseListener: ProductPurchaseListener,
                          openNextScreen: () -> Unit
                      ) {
                          StaticParam.isRevenueCate = true
                          StaticParam.productPurchaseListener = productPurchaseListener
                  
                          var isSubscribe = false
                  
                          Purchases.debugLogsEnabled = false
                          Purchases.configure(
                              PurchasesConfiguration.Builder(context, REVENUECAT_GOOGLE_API).appUserID(null).build()
                          )
                          ("Heree innerrr").log("onRevenueCatPurchased")
                  
                          Purchases.sharedInstance.getCustomerInfo(
                              CacheFetchPolicy.NOT_STALE_CACHED_OR_CURRENT,
                              object : ReceiveCustomerInfoCallback {
                                  override fun onError(error: PurchasesError) {
                                      ("on Received on Error : ${error.message}").log("onRevenueCatPurchased")
                                  }
                  
                                  override fun onReceived(customerInfo: CustomerInfo) {
                                      ("on Received Subscription: ${customerInfo.activeSubscriptions} || inApp : ${customerInfo.entitlements.active}").log(
                                          "onRevenueCatPurchased"
                                      )
                  
                                 if (customerInfo.activeSubscriptions.isNotEmpty()) {
                        isSubscribe = true
                        isSubscribedForAllGlobal(context, true)
                    }

                    if (App.getBoolean("isForRevalueFirstTime") == false) {
                        App.putBoolean("isForRevalueFirstTime", true)
                        ("Life Time Success: ---  $isSubscribe").log(
                            "onRevenueCatPurchased"
                        )
                        if (!context.isDestroyed && isSubscribe) {
                            productPurchaseListener.onRevenueCatPurchased(customerInfo)
                            openNextScreen.invoke()
                        } else {
                            App.putBoolean("isForRevalueFirstTime", false)
                        }
                    }
                                      }
                                  }
                              })
                  
                          Purchases.sharedInstance.syncPurchases(object : SyncPurchasesCallback {
                              override fun onError(error: PurchasesError) {
                                  ("is syncPurchases syncPurchases Time Plan Error: --- ${error.message}").log(
                                      "onRevenueCatPurchased"
                                  )
                  
                                  if (!isSubscribe) isSubscribedForAllGlobal(context, false)

                if (App.getBoolean("isForRevalueFirstTime") == false) {
                    App.putBoolean("isForRevalueFirstTime", true)
                    if (!context.isDestroyed) {
                        openNextScreen.invoke()
                    }
                }
                              }
                  
                              override fun onSuccess(customerInfo: CustomerInfo) {
                                  ("syncPurchases onSuccess : ${customerInfo.entitlements.active} \n Active Sub: ${customerInfo.activeSubscriptions.toGson()} ").log(
                                      "onRevenueCatPurchased"
                                  )
                                  if (customerInfo.activeSubscriptions.isNotEmpty()) {
                                      isSubscribedForAllGlobal(context, true)
                                  } else if (customerInfo.entitlements.active.isNotEmpty()) {
                                      isSubscribedForAllGlobal(context, true)
                                  } else {
                                      isSubscribedForAllGlobal(context, false)
                                  }
                  
                                  if (App.getBoolean("isForRevalueFirstTime") == false) {
                                      App.putBoolean("isForRevalueFirstTime", true)
                  //                    if (!context.isDestroyed) {
                                      ("Life Time Success: ---  ").log(
                                          "onRevenueCatPurchased"
                                      )
                                      productPurchaseListener.onRevenueCatPurchased(customerInfo)
                                      if (!context.isDestroyed)
                                          openNextScreen.invoke()
                  //                    }
                                  }
                              }
                          })
                  
                          Purchases.sharedInstance.getOfferings(object : ReceiveOfferingsCallback {
                              override fun onError(error: PurchasesError) {
                                  "${error}".log("Purchase List error-->")
                              }
                  
                              override fun onReceived(offerings: com.revenuecat.purchases.Offerings) {
                                  (offerings.all.toGson()).log("REVANUEPLAN")
                  
                                  if (BuildConfig.DEBUG) saveJsonToFile(Gson().toJson(offerings))
                  
                                  offerings.all.forEach { (s, offering) ->
                                      val offers = MyOffer(s, offering.availablePackages)
                                      allAvailablePackages.add(offers)
                                  }
                  
                                  allAvailablePackages.forEach { myoffer ->
                                      myoffer.packages.forEach {
                                          val freeTrial = it.product.subscriptionOptions?.freeTrial?.let { it ->
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
                                      val freeTrial = it.product.subscriptionOptions?.freeTrial?.let { it ->
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
                                              it
                                          )
                                      )
                                  }
                  
                                  val myProducts = ArrayList<BillingProduct>()
                  
                                  revenueCatDataList.forEach {
                                      val billingProduct = BillingProduct()
                                      billingProduct.id = it.id
                                      billingProduct.title = it.packageName
                  
                                      it.product.product.let { q ->
                                          billingProduct.productType =
                                              if (q.type == ProductType.INAPP) INAPP else SUBS
                                      }
                  
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
                  
                                          PackageType.LIFETIME -> {
                                              billingProduct.packageType = MyPackageType.LIFETIME
                                          }
                  
                                          else -> {
                                              billingProduct.packageType = MyPackageType.SIX_MONTH
                                          }
                                      }
                                      billingProduct.price = it.price
                                      billingProduct.period = it.period
                                      billingProduct.amountmicros = it.amountmicros
                                      billingProduct.freeTrials = it.freeTrial
                  
                                      myProducts.add(billingProduct)
                                  }
                                  SubscriptionSetter.livePackageProduct.postValue(myProducts)
                                  liverevenuecatedatalist.postValue(revenueCatDataList)
                              }
                          })
                  
                          if (App.getBoolean("isForRevalueFirstTime") == true && !context.isDestroyed) openNextScreen.invoke()
                      }
                  
                      fun RestorePurchase(activity: Activity, onNext: () -> Unit) {
                  
                          if (!isOnline(activity)) {
                              Toast.makeText(
                                  activity, activity.getString(R.string.please_connect_internet), Toast.LENGTH_SHORT
                              ).show()
                              return
                          }
                  
                          if (Purchases.isConfigured && activity.window != null && activity.windowManager != null) {
                              Purchases.sharedInstance.restorePurchases(object : ReceiveCustomerInfoCallback {
                                  override fun onError(error: PurchasesError) {
                                      "onRevenueCatPurchased".log("Error--> $error")
                                      activity.runOnUiThread {
                                          Snackbar.make(
                                              activity.findViewById(android.R.id.content),
                                              activity.getString(R.string.nothing_to_restore),
                                              Snackbar.LENGTH_SHORT
                                          ).apply {
                                              setActionTextColor(Color.WHITE)
                                              show()
                                          }
                                      }
                                      onNext()
                                  }
                  
                                  override fun onReceived(customerInfo: CustomerInfo) {
                                      ("restorePurchases $customerInfo").log("onRevenueCatPurchased")
                                      if (customerInfo?.activeSubscriptions?.isNotEmpty() == true) {
                                          isSubscribedForAllGlobal(activity, true)
                                          onNext()
                                          activity.setResult(Activity.RESULT_OK)
                                          activity.onBackPressed()
                                      } else {
                                          if (customerInfo.entitlements?.active?.isNotEmpty() == true) {
                                              isSubscribedForAllGlobal(activity, true)
                                              onNext()
                                              activity.setResult(Activity.RESULT_OK)
                                              activity.onBackPressed()
                                          } else {
                                              onNext()
                                              activity.runOnUiThread {
                                                  Snackbar.make(
                                                      activity.findViewById(android.R.id.content),
                                                      activity.getString(R.string.nothing_to_restore),
                                                      Snackbar.LENGTH_SHORT
                                                  ).apply {
                                                      setActionTextColor(Color.WHITE)
                                                      show()
                                                  }
                                              }
                                          }
                                      }
                                  }
                              })
                          } else {
                              Purchases.configure(
                                  PurchasesConfiguration.Builder(activity, BuildConfig.REVANUECAT_ID).build()
                              )
                          }
                      }
                  
                      fun purchaseMonth(context: Activity, onSuccess: (StoreTransaction, CustomerInfo) -> Unit) {
                  //        var purchaseProduct = allAvailablePackages.find { it.key == key }
                          var product = revenueCatDataList.find { it.packageType == PackageType.MONTHLY }
                          if (product != null) {
                  //            var product = purchaseProduct.packages.find { it.packageType == PackageType.MONTHLY }
                              product?.let {
                                  Purchases.sharedInstance.purchaseWith(PurchaseParams.Builder(
                                      context, product.product
                                  ).build(), { error: PurchasesError, userCancelled: Boolean ->
                                      Log.e("TAG", "initview:MonthPackage $error")
                                  }, { purchase, customerInfo ->
                                      purchase?.let { onSuccess.invoke(it, customerInfo) }
                                  })
                              }
                          } else {
                              Log.e("TAG", "initview:purchaseProduct null!")
                  
                          }
                      }
                  
                      fun purchaseOnTime(
                          context: Activity,
                          productkey: PackageType,
                          onSuccess: (StoreTransaction, CustomerInfo) -> Unit
                      ) {
                  //        var purchaseProduct = allAvailablePackages.find { it.key == key }
                          val product = revenueCatDataList.find { it.packageType == productkey }
                          if (product != null) {
                              product?.let {
                                  Purchases.sharedInstance.purchaseWith(PurchaseParams.Builder(
                                      context, product.product
                                  ).build(), { error: PurchasesError, userCancelled: Boolean ->
                                      Log.e("TAG", "initview:MonthPackage $error")
                                  }, { purchase, customerInfo ->
                                      purchase?.let { onSuccess.invoke(it, customerInfo) }
                                  })
                              }
                          } else {
                              Log.e("TAG", "initview:purchaseProduct null!")
                          }
                      }
                  }
               
         @Keep
         data class BillingProduct(
             var id: String = "",
             var title: String = "",
             var description: String = "",
             var packageType: MyPackageType? = null,
             var price: String = "",
             var amountmicros: Long = 0L,
             var amountmicrostoPriceInt: Int = 0,
             var period: String = "",
             var freeTrials: FreeTrils? = null,
             var productType: String = BillingClient.ProductType.SUBS,
         )
         
         @Keep
         data class BillingINAPPProduct(
             var id: String = "",
             var title: String = "",
             var description: String = "",
             var packageType: MyPackageType? = null,
             var price: String = "",
             var amountmicros: Long = 0L,
             var amountmicrostoPriceInt: Int = 0,
             var period: String = "",
             var productType: String = BillingClient.ProductType.SUBS,
         )
         
         @Keep
         data class ProductBillingIDS(var id: String, var billingType: String)

     
                  @Keep
                  data class RevenueCatData(
                      var id : String,
                      var packageName:String,
                      var packageType : PackageType,
                      var price : String,
                      var amountmicros : Long,
                      var period : String,
                      var product : Package,
                      var freeTrial : FreeTrils = FreeTrils()
                  )
                  
                  @Keep
                  data class FreeTrils(var period : Int? = null, var TimeType : String = "", var i8 : String = "")
                  @Keep
                  data class MyOffer(var key : String="", var packages : List<Package> = listOf())

         ----- SubscriptionSetter.clas

                         
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
                            when (packageType) {
                                MyPackageType.MONTH -> {
                                    RevenueCatHelper.purchaseOnTime(activity, PackageType.MONTHLY, onSuccess)
                                }
                
                                MyPackageType.ANNUAL -> {
                                    RevenueCatHelper.purchaseOnTime(activity, PackageType.ANNUAL, onSuccess)
                                }
                
                                MyPackageType.SIX_MONTH -> {
                                    RevenueCatHelper.purchaseOnTime(activity, PackageType.SIX_MONTH, onSuccess)
                                }
                
                                MyPackageType.WEEK -> {
                                    RevenueCatHelper.purchaseOnTime(activity, PackageType.WEEKLY, onSuccess)
                                }
                
                                else -> {
                                    RevenueCatHelper.purchaseOnTime(activity, PackageType.SIX_MONTH, onSuccess)
                                }
                            }
                        } else {
                            BillingHelper.makePurchase(packageType)
                        }
                    }
                
                    fun makeMyProduct(
                        activity: Activity,
                        packageType: MyPackageType/* here packageType is Product's  */,
                        onSuccess: (StoreTransaction, CustomerInfo) -> Unit
                    ) {
                        if (StaticParam.isRevenueCate) {
                            RevenueCatHelper.purchaseOnTime(activity, getRevenuePackage(packageType), onSuccess)
                        } else {
                            BillingHelper.makeProductPurchase(packageType)
                        }
                    }
                    fun getRevenuePackage(packageType : MyPackageType) : PackageType {
                        if(packageType == MyPackageType.MONTH) {
                            return PackageType.MONTHLY
                        } else if(packageType == MyPackageType.ANNUAL) {
                            return PackageType.ANNUAL
                        } else if(packageType == MyPackageType.SIX_MONTH) {
                            return PackageType.SIX_MONTH
                        } else if(packageType == MyPackageType.WEEK) {
                            return PackageType.WEEKLY
                        } else if(packageType == MyPackageType.LIFETIME) {
                            return PackageType.LIFETIME
                        } else {
                            return PackageType.UNKNOWN
                        }
                    }
                }


             
            --- Subscription Act

               -- For LifeTime ------

                 SubscriptionSetter.makeMyProduct(
                        this@NewForYearSubAct,
                        MyPackageType.LIFETIME,
                        { storeTransaction, customerInfo -> })

               --- For Plan Subscription ----
               
            SubscriptionSetter.makeMySubs(
                    this@NewForYearSubAct, MyPackageType.WEEK
                ) { storeTransaction, customerInfo ->
                    isSubscribeFun(this@NewForYearSubAct, true)
                    ("onSuccess WEEK").log("onRevenueCatPurchased")
                }


                   SubscriptionSetter.makeMyProduct(
                        this@NewForYearSubAct, MyPackageType.LIFETIME
                    ) { storeTransaction, customerInfo ->

                        ("onSuccess LifeTime").log("onRevenueCatPurchased")

                        isSubscribeFun(this@NewForYearSubAct, true)

                    }

                      purchaseListener.postValue(false)
    
                purchaseListener.observe(this@NewForYearSubAct) { purchase ->
                    //finish
                    ("Purchasing: $purchase").log()
    
                    if (purchase) {
                        setResult(Activity.RESULT_OK)
                        isSubscribedForAllGlobal(this@NewForYearSubAct, purchase)
                        onBackPressed()
                    }
                }



            ---- Live All Produt Detail 

                                 SubscriptionSetter.livePackageProduct.observe(
                    this@NewForYearSubAct,
                    object : Observer<List<BillingProduct>?> {
                        @SuppressLint("SetTextI18n")
                        override fun onChanged(billingProducts: List<BillingProduct>?) {
                            billingProducts?.let { productsList ->
                                try {
                                    if (!isOnline(this@NewForYearSubAct)) return
    
                                    val annualPlan =
                                        productsList.find { it.packageType == MyPackageType.ANNUAL }
                                    val weekPlan =
                                        productsList.find { it.packageType == MyPackageType.WEEK }
                                    val sixPlan =
                                        productsList.find { it.packageType == MyPackageType.SIX_MONTH }
    
                                    val lifetime =
                                        productsList.find { it.packageType == MyPackageType.LIFETIME && it.productType == BillingClient.ProductType.INAPP }
    
                                    lifetime?.let {
                                        ("Sucsess INAPP Data: ${it.toGson()}").log("FATZ")
                                        bind.lifetimebtn.visibility = View.VISIBLE
                                        bind.lifetimepriceid.text =
                                            removeTrailingZeros(lifetime?.price!!)
    
                                        if (!isOnline(this@NewForYearSubAct)) {
                                            bind.lifetimepriceid.text = "₹2,000"
                                        }
                                    }
    
                                    bind.tvPriceyear.text = removeTrailingZeros(annualPlan?.price!!)
                                    bind.priserid.text = removeTrailingZeros(weekPlan?.price!!)
                                    bind.sixmonthjid.text = removeTrailingZeros(sixPlan?.price!!)
    
                                    ("Micro Annual: " + annualPlan!!.amountmicros + " | Week: " + weekPlan!!.amountmicrostoPriceInt + " | 6 Month: " + sixPlan!!.amountmicrostoPriceInt).log(
                                        "FATZ"
                                    )
    
                                    //todo: Revanue Cat Change divided
                                    val annulemicro = (annualPlan?.amountmicros?.div(1000000))
                                    val weekmicro = (weekPlan?.amountmicros?.div(1000000))
                                    val sixmonthmicro = (sixPlan?.amountmicros?.div(1000000))
    
                                    bind.weeklytvlongtvid.text =
                                        "• ${getString(R.string.weeksubtv)} : ${
                                            removeTrailingZeros(
                                                weekPlan?.price!!
                                            )
                                        }"
                                    bind.monthlytvlongtvid.text =
                                        "• ${getString(R.string.sixmonthsubtv)} : ${
                                            removeTrailingZeros(
                                                sixPlan?.price!!
                                            )
                                        }"
                                    bind.yearlytvlongtvid.text =
                                        "• ${getString(R.string.yearsubtv)} : ${
                                            removeTrailingZeros(
                                                annualPlan?.price!!
                                            )
                                        }"
    
                                    if (annulemicro != null) {
                                        bind.montbottomid.text =
                                            "₹${annulemicro / 52}/ ${getString(R.string.perweek)}"
                                    }
                                    if (sixmonthmicro != null) {
                                        bind.sixmonthbottom.text =
                                            "₹${sixmonthmicro / 26}/ ${getString(R.string.perweek)}"
                                    }
    
                                    if (weekmicro != null) {
                                        val numberFormat: NumberFormat = NumberFormat.getInstance()
                                        val formattedNumber: String =
                                            numberFormat.format((weekmicro * 26))
                                        bind.linabc.text = "₹${formattedNumber}"
                                        bind.linabcde.text =
                                            "₹${numberFormat.format((weekmicro?.times(52)))}"
                                        bind.tvlifetimestrik.text =
                                            "₹${numberFormat.format((weekmicro?.times(52 * 2)))}"
                                    }
    
                                    if (sixmonthmicro != null) {
                                        val yearperchantage =
                                            ((26 * weekmicro!!) - sixmonthmicro).toDouble()
                                        val sec = (yearperchantage * 100 / (26 * weekmicro)).toDouble()
    
                                        ("Six: " + sec).log()
    
                                        bind.sixmonthtvbg.text = "${(sec).roundToInt()}${
                                            getString(
                                                R.string.off
                                            )
                                        }"
                                    }
    
                                    if (annulemicro != null) {
    
                                        val yearperchantage =
                                            (52 * weekmicro!!) - annulemicro.toDouble()
                                        val sec = yearperchantage * 100 / (52 * weekmicro).toDouble()
    
                                        ("Annul: " + sec).log()
    
                                        bind.monthtvdid.text = "${(sec).roundToInt()}${
                                            getString(
                                                R.string.off
                                            )
                                        }"
                                    }
    
                                    if (annulemicro != null) {
                                        ("Name: " + ((annulemicro * 100) / (annulemicro * 48))).log()
                                    }
    
                                    val isFreeTrial = productsList.any { it.price == "Free" }
                                    ("isFreeTrial $isFreeTrial").log()
    
                                    if (isFreeTrial) {
                                        App.putBoolean("isFreeTry", true)
                                        bind.startfreetrybtn.text =
                                            getSubscribeText(this@NewForYearSubAct, true)
                                    } else {
                                        App.putBoolean("isFreeTry", false)
                                        bind.startfreetrybtn.text =
                                            getSubscribeText(this@NewForYearSubAct, false)
                                    }
                                } catch (e: Exception) {
                                    e.printStackTrace()
                                }
                            }
                        }
                    })
        
        
                fun removeTrailingZeros(text: String): String {
            return if (text.endsWith(".00")) {
                text.substring(0, text.length - 3)
            } else {
                text
            }
        }
        
          @SuppressLint("StringFormatMatches")
            private fun getSubTrial(trial: String): String? {
                //Parse I8
                return try {
                    val size = trial.length
                    val period = trial.substring(1, size - 1)
                    val str = trial.substring(size - 1, size)
                    Log.d("TAG", "getSubTrial <------------------>: ${size} $period - $str")
                    when (str) {
                        "D" -> resources.getString(R.string.days, period)
                        "M" -> resources.getString(R.string.months, period)
                        "Y" -> resources.getString(R.string.months, (period.toInt() * 12))
                        "W" -> resources.getString(R.string._7_day)
                        else -> resources.getString(R.string.months, period)
                    }
                } catch (e: Exception) {
                    resources.getString(R.string._12_months)
                }
            }
            
// Connection Live Data
        
        class ConnectionLiveData(val context: Context) : LiveData<Boolean>() {
        
            private var connectivityManager: ConnectivityManager = context.getSystemService(CONNECTIVITY_SERVICE) as ConnectivityManager
        
            private lateinit var connectivityManagerCallback: ConnectivityManager.NetworkCallback
        
            private lateinit var  networkRequestBuilder: NetworkRequest.Builder
            
            @SuppressLint("MissingPermission")
            override fun onActive() {
                super.onActive()
                updateConnection()
                when {
                    Build.VERSION.SDK_INT >= Build.VERSION_CODES.N -> connectivityManager.registerDefaultNetworkCallback(
                        getConnectivityMarshmallowManagerCallback()
                    )
                    Build.VERSION.SDK_INT >= Build.VERSION_CODES.M -> marshmallowNetworkAvailableRequest()
                    else -> lollipopNetworkAvailableRequest()
                }
            }
        
            override fun onInactive() {
                super.onInactive()
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                    try { connectivityManager.unregisterNetworkCallback(connectivityManagerCallback) }catch (_: Exception){}
                }
            }
        
            @SuppressLint("MissingPermission")
            private fun lollipopNetworkAvailableRequest() {
                if(!::networkRequestBuilder.isInitialized) {
                    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                        networkRequestBuilder = NetworkRequest.Builder()
                            .addTransportType(NetworkCapabilities.TRANSPORT_CELLULAR)
                            .addTransportType(NetworkCapabilities.TRANSPORT_WIFI)
                    }
                }
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                    connectivityManager.registerNetworkCallback(
                        networkRequestBuilder.build(),
                        getConnectivityLollipopManagerCallback()
                    )
                }
            }
        
            @SuppressLint("MissingPermission")
            @TargetApi(Build.VERSION_CODES.M)
            private fun marshmallowNetworkAvailableRequest() {
                if(!::networkRequestBuilder.isInitialized) {
                    networkRequestBuilder = NetworkRequest.Builder()
                        .addTransportType(NetworkCapabilities.TRANSPORT_CELLULAR)
                        .addTransportType(NetworkCapabilities.TRANSPORT_WIFI)
                }
                connectivityManager.registerNetworkCallback(
                    networkRequestBuilder.build(),
                    getConnectivityMarshmallowManagerCallback()
                )
            }
        
            private fun getConnectivityLollipopManagerCallback(): ConnectivityManager.NetworkCallback {
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                    connectivityManagerCallback = object : ConnectivityManager.NetworkCallback() {
                        override fun onAvailable(network: Network) {
                            postValue(true)
                        }
        
                        override fun onLost(network: Network) {
                            postValue(false)
                        }
                    }
                }
                return connectivityManagerCallback
            }
        
            private fun getConnectivityMarshmallowManagerCallback(): ConnectivityManager.NetworkCallback {
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                    connectivityManagerCallback = object : ConnectivityManager.NetworkCallback() {
                        override fun onCapabilitiesChanged(
                            network: Network,
                            networkCapabilities: NetworkCapabilities
                        ) {
                            networkCapabilities.let { capabilities ->
                                if (capabilities.hasCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET) && capabilities.hasCapability(
                                        NetworkCapabilities.NET_CAPABILITY_VALIDATED
                                    )
                                ) {
                                    postValue(true)
                                }
                            }
                        }
        
                        override fun onLost(network: Network) {
                            postValue(false)
                        }
                    }
                    return connectivityManagerCallback
                } else {
                    throw IllegalAccessError("Accessing wrong API version")
                }
            }
        
            private val networkReceiver = object : BroadcastReceiver() {
                override fun onReceive(context: Context, intent: Intent) {
                    updateConnection()
                }
            }
        
            @SuppressLint("MissingPermission")
            private fun updateConnection() {
                val activeNetwork: NetworkInfo? = connectivityManager.activeNetworkInfo
                postValue(activeNetwork?.isConnected == true)
            }
        }

    
    // Activity class 
    
    
     connectionLiveData = ConnectionLiveData(requireActivity())
            connectionLiveData?.observe(requireActivity()) { isConnected ->
                isConnected?.let {
                    if (it) {
                        Handler(Looper.myLooper()!!).postDelayed({
                            loadNative()
                        }, 100)
                    } else {
                        binding.apply{
                            noAdsImg.visible()
                            FBAdContainer.gone()
                            mframlayout.gone()
                        }
                    }
                }
            }
            

// Subscription & Revanue Cat class

    implementation 'com.revenuecat.purchases:purchases:7.0.0'
    implementation "com.android.billingclient:billing-ktx:6.1.0"

     ------ Splace Screen -------

       var productIds = Arrays.asList(
        ProductBillingIDS("subscribe_weekly_kriadl", BillingClient.ProductType.SUBS),
        ProductBillingIDS("subscribe_sixly_kriadl", BillingClient.ProductType.SUBS),
        ProductBillingIDS("subscribe_yearly_kriadl", BillingClient.ProductType.SUBS),
        ProductBillingIDS("subscribe_lifetime_kriadl", BillingClient.ProductType.INAPP)
    )

    ---------- call onCreate() -------
    
           BillingHelper.getBillingInit(this, productIds, object : ProductPurchaseListener {
                override fun onPurchase(message: String) {
                    ("onPurchase: $message").log()
    //                message.messageLog()
                }
    
                override fun onRevenueCatPurchased(purchases: CustomerInfo) {
    
                }
    
                override fun onPurchased(purchase: Purchase) {
                    Log.wtf("FATZ", "-- purchase: " + purchase.purchaseState)
    //                subsSharedFile.isSubscriber = (purchase.purchaseState != -1)
    //                subsSharedFile.purchaseProduct = purchase.toGson()
    //                subsSharedFile.purchaseProduct.logPurchase()
                    val isSubscriber = (purchase.purchaseState != -1)
                    isSubscribeFun(isSubscriber)
                }
    
                override fun onPurchaseFail(errorMessage: String) {
    
                }
            }) { isSubscribe ->
                ("is Here Initialized is Subscribe: $isSubscribe").log()
                runOnUiThread {
                    if (isFirstTime) {
                        isFirstTime = false
                        gotoLoadApp()
                    }
                }
            }

    
     private fun isSubscribeFun(activity: Activity, isSubscribe: Boolean) {
        StaticParam.purchaseListener.postValue(true)
        isSubscribedForAllGlobal(activity, true)
    }



            --------- BillingHelper Class -------
            
                            object BillingHelper {
                var billingClient: BillingClient? = null
                var productIds: List<ProductBillingIDS> = listOf(
                    ProductBillingIDS("subscribe_weekly_kriadl", BillingClient.ProductType.SUBS),
                    ProductBillingIDS("subscribe_sixly_kriadl", BillingClient.ProductType.SUBS),
                    ProductBillingIDS("subscribe_yearly_kriadl", BillingClient.ProductType.SUBS),
                    ProductBillingIDS("subscribe_lifetime_kriadl", BillingClient.ProductType.INAPP)
                )
            
                var myInAppProducts = ArrayList<BillingINAPPProduct>()
            
                var myProducts = ArrayList<BillingProduct>()
                var liveMyProducts = MutableLiveData<List<BillingProduct>>()
                private var isConsumable: Boolean = false
                var mySubProducts = ArrayList<BillingProduct>()
            
                fun getBillingInit(
                    billContextAct: Activity,
                    billingKeys: List<ProductBillingIDS>,
                    productPurchaseListener: ProductPurchaseListener,
                    onNextAct: (Boolean) -> Unit
                ) {
                    StaticParam.productPurchaseListener = productPurchaseListener
                    productIds = billingKeys
                    billContext = billContextAct
                    getBillingHelp(onNextAct)
                }
            
                //    var productIds = Arrays.asList("body_editor_monthly", "body_editor_yearly")
            
                fun getBillingHelp(onNextAct: (Boolean) -> Unit) {
                    val purchasesUpdatedListener = PurchasesUpdatedListener { billingResult, purchases ->
                        "Client Purchase Response : ${billingResult.responseCode}".bilLog()
                    }
                    billingClient =
                        BillingClient.newBuilder(billContext!!).setListener(purchasesUpdatedListener)
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
                                                    billContext!!,
                                                    "You've cancelled the Google play billing process"
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
                                                billContext!!, "Item not found or Google play billing error"
                                            )
                                            productPurchaseListener?.onPurchaseFail("Item not found or Google play billing error")
                                        }
                                    }
                                }
                            }.build()
            
                    billingClient?.startConnection(object : BillingClientStateListener {
                        override fun onBillingSetupFinished(billingResult: BillingResult) {
            
                            "Client Setup Response : ${billingResult.responseCode == BillingClient.BillingResponseCode.OK}".bilLog()
                            ("Connection ").log("FATZ")
            
                            if (billingResult.responseCode == BillingClient.BillingResponseCode.OK) {
            
                                CoroutineScope(Dispatchers.IO).launch {
                                    subSkuPurchases(billingResult)
                                    inAppSkuPurchases(billingResult)
                                }
            
                                billingClient?.queryPurchasesAsync(
                                    QueryPurchasesParams.newBuilder()
                                        .setProductType(BillingClient.ProductType.SUBS).build()
                                ) { _, purchases ->
                                    purchases.size.toString().log()
                                    if (purchases.size > 0) {
                                        ("Item Purchase").log("FATZ")
                                        isSubscribedForAllGlobal(billContext!!, true)
                                        StaticParam.purchaseListener.postValue(true)
                                        onNextAct.invoke(true)
                                    } else {
                                        ("11 Item Not Purchase | Res Code: ${billingResult.responseCode}").log("FATZ")
            //                        isSubscribedForAllGlobal(billContext!!, false)
            //                        isSubscribeNewModul.value = false
                                        billingClient?.queryPurchasesAsync(
                                            QueryPurchasesParams.newBuilder()
                                                .setProductType(BillingClient.ProductType.INAPP).build()
                                        ) { _, purchases ->
                                            purchases.size.toString().log()
                                            if (purchases.size > 0) {
                                                ("11 Item Purchas in App").log("FATZ")
                                                isSubscribedForAllGlobal(billContext!!, true)
                                                StaticParam.purchaseListener.postValue(true)
                                                onNextAct.invoke(true)
                                            } else {
                                                ("11 Item Not Purchase in APP | Res Code: ${billingResult.responseCode}").log(
                                                    "FATZ"
                                                )
                                                isSubscribedForAllGlobal(billContext!!, false)
                                                onNextAct.invoke(false)
                                            }
                                        }
                                    }
                                }
            //                    onNextAct.invoke(isSubScribe)
                            } else {
                                isSubscribedForAllGlobal(billContext!!, false)
                                onNextAct.invoke(false)
                            }
                        }
            
                        override fun onBillingServiceDisconnected() {
                            "Client Disconnected ".bilLog()
                            CoroutineScope(Dispatchers.Main).launch {
                                isSubscribedForAllGlobal(billContext!!, false)
                                onNextAct.invoke(false)
                            }
                        }
                    })
                }
            
            //    fun checkRestore(isRestoreOrNot:(Boolean) -> Unit) {
            //        billingClient.startConnection(object : BillingClientStateListener {
            //            override fun onBillingSetupFinished(billingResult: BillingResult) {
            ////                "Client Setup Response : ${billingResult.responseCode == BillingClient.BillingResponseCode.OK}".bilLog()
            //                ("Check Restore Connection ").log("FATZ")
            //                billingClient.queryPurchasesAsync(
            //                    QueryPurchasesParams.newBuilder().setProductType(BillingClient.ProductType.SUBS)
            //                        .build()
            //                ) { _, purchases ->
            //                    purchases.size.toString().log()
            //                    if (purchases.size > 0) {
            //                        ("Item Purchase").log("FATZ")
            //                        isSubscribedForAllGlobal(billContext!!, true)
            //                        StaticParam.purchaseListener.postValue(true)
            //                    } else {
            //                        ("Item Not Purchase | Res Code: ${billingResult.responseCode}").log("FATZ")
            //                        isSubscribedForAllGlobal(billContext!!, false)
            ////                        isSubscribeNewModul.value = false
            //                        if (billingResult.responseCode == BillingClient.BillingResponseCode.OK) {
            //                            CoroutineScope(Dispatchers.IO).launch {
            //                                subSkuPurchases(billingResult)
            //                                inAppSkuPurchases(billingResult)
            //                            }
            //                        }
            //                    }
            //                }
            //            }
            //
            //            override fun onBillingServiceDisconnected() {
            //                "Client Disconnected ".bilLog()
            //            }
            //        })
            //    }
            
                private suspend fun subSkuPurchases(billingResult: BillingResult) {
                    billingClient?.queryPurchasesAsync(
                        QueryPurchasesParams.newBuilder().setProductType(BillingClient.ProductType.SUBS).build()
                    ) { _, purchases ->
                        purchases.size.toString().log()
            
                        if (purchases.size > 0) {
                            ("Item Subscription Purchase").log("FATZ")
                        } else {
                            ("Item Subscription Not Purchase").log("FATZ")
                            if (billingResult.responseCode == BillingClient.BillingResponseCode.OK) {
                                retriveMySKU()
                            }
                        }
                    }
                }
            
                fun getSkuProduct(
                    queryParam: QueryProductDetailsParams, listener: ProductDetailsResponseListener
                ) {
                    billingClient?.queryProductDetailsAsync(queryParam, listener)
                }
            
                fun makeProductPurchase(packageType: MyPackageType) {
                    if (StaticParam.subsSharedFile?.isSubscriber == true) {
                        toastMaker(billContext!!, "Already Subscribed!")
                        return
                    }
            
                    if (productIds.isEmpty()) return
            
                    if (packageType == MyPackageType.LIFETIME) {
                        val queryProductDetailsParams = newBuilder().setProductList(
                            listOf(
                                Product.newBuilder().setProductId(productIds[3].id)
                                    .setProductType(productIds[3].billingType).build()
                            )
                        ).build()
                        getSkuProduct(queryProductDetailsParams) { billingResult, productDetails ->
                            val fetchedProduct = fetchProduct(productIds[3].id, productDetails)
                            fetchedProduct?.let {
                                val productDetailsParamsList = listOf(
                                    BillingFlowParams.ProductDetailsParams.newBuilder().setProductDetails(it)
                                        .build()
                                )
            
                                val billingFlowParams = BillingFlowParams.newBuilder()
                                    .setProductDetailsParamsList(productDetailsParamsList).build()
                                val resultBill =
                                    billingClient?.launchBillingFlow(billContext!!, billingFlowParams)
            
                                when (resultBill?.responseCode) {
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
                    billingClient?.queryPurchasesAsync(
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
                                    billingProduct.amountmicrostoPriceInt =
                                        pricingPhase.priceAmountMicros.div(1000000.0).toInt()
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
                    if (StaticParam.subsSharedFile?.isSubscriber == true) {
                        toastMaker(billContext!!, "Already Subscribed!")
                        return
                    }
            
                    if (productIds.isNotEmpty()) when (type) {
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
                        val fetchedProduct = fetchProduct(productIDS.id, productDetails)
                        fetchedProduct?.let { product ->
                            Log.e(
                                "TAG",
                                "purchaseMyProduct: 1  ${product.oneTimePurchaseOfferDetails?.toGson()}  "
                            )
            
                            val offerToken = product.subscriptionOfferDetails?.get(0)?.offerToken
                            offerToken?.let { token ->
                                Log.e("TAG", "purchaseMyProduct: 2 $token")
            
                                val productDetailsParams = BillingFlowParams.ProductDetailsParams.newBuilder()
                                    .setProductDetails(product).setOfferToken(token).build()
            
                                val billingFlowParams = BillingFlowParams.newBuilder()
                                    .setProductDetailsParamsList(mutableListOf(productDetailsParams)).build()
            
                                val resultBill =
                                    billingClient?.launchBillingFlow(billContext!!, billingFlowParams)
            
                                when (resultBill?.responseCode) {
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


        ----- SubscriptionSetter.kt
        
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
                    when (packageType) {
                        MyPackageType.MONTH -> {
                            RevenueCatHelper.purchaseOnTime(activity, PackageType.MONTHLY, onSuccess)
                        }
        
                        MyPackageType.ANNUAL -> {
                            RevenueCatHelper.purchaseOnTime(activity, PackageType.ANNUAL, onSuccess)
                        }
        
                        MyPackageType.SIX_MONTH -> {
                            RevenueCatHelper.purchaseOnTime(activity, PackageType.SIX_MONTH, onSuccess)
                        }
        
                        MyPackageType.WEEK -> {
                            RevenueCatHelper.purchaseOnTime(activity, PackageType.WEEKLY, onSuccess)
                        }
        
                        else -> {
                            RevenueCatHelper.purchaseOnTime(activity, PackageType.SIX_MONTH, onSuccess)
                        }
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

      ------ in Subscription Act --------

        purchaseListener.postValue(false)

            purchaseListener.observe(this@NewForYearSubAct) { purchase ->
                //finish
                ("Purchasing: $purchase").log()

    //                isSubscribedForAllGlobal(this@NewForYearSubAct, purchase)

                if (purchase) {
                    setResult(Activity.RESULT_OK)
                    onBackPressed()
                }
            }
            
        
        
           SubscriptionSetter.liveMyInAppProducts.observe(this@NewForYearSubAct) { billingInAppProduct ->
                        billingInAppProduct?.let { productsList ->
        
                            if (!NetworkHelper.isOnline(this@NewForYearSubAct)) return@observe
        
                            val lifetime =
                                productsList.find { it.packageType == MyPackageType.LIFETIME && it.productType == BillingClient.ProductType.INAPP }
                            lifetime?.let {
                                ("Sucsess INAPP Data: ${it.toGson()}").log("FATZ")
                                bind.lifetimebtn.visibility = View.VISIBLE
                                bind.lifetimepriceid.text = lifetime?.price
        
                                if (!NetworkHelper.isOnline(this@NewForYearSubAct)) {
                                    bind.lifetimepriceid.text = "₹2,000/-"
                                }
        
        //                        bind.lifetimetitalid.text = lifetime?.title
        
        //                        bind.lifetimetitalid.setEllipsize(TextUtils.TruncateAt.MARQUEE);
        //                        bind.lifetimetitalid.setSingleLine(true);
        //                        bind.lifetimetitalid.setMarqueeRepeatLimit(-1); // -1 for infinite loop
        //                        bind.lifetimetitalid.setSelected(true); // Required for the marquee effect to work
        
        //                       bind.tvpurid.text = lifetime?.description
        //                       bind.tvpurid.text = "Lifetime Purchase"
                            }
                        }
                    }
        
                    SubscriptionSetter.livePackageProduct.observe(this@NewForYearSubAct,
                        object : Observer<List<BillingProduct>?> {
                            @SuppressLint("SetTextI18n")
                            override fun onChanged(billingProducts: List<BillingProduct>?) {
                                billingProducts?.let { productsList ->
                                    try {
                                        if (!NetworkHelper.isOnline(this@NewForYearSubAct)) return
        
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
        
        //                            ("Micro Annual: " + annualPlan!!.amountmicrostoPriceInt + " | Week: " + weekPlan!!.amountmicrostoPriceInt + " | 6 Month: " + sixPlan!!.amountmicrostoPriceInt).log(
        //                                "FATZ"
        //                            )
        
                                        val annulemicro = annualPlan?.amountmicrostoPriceInt
                                        val weekmicro = weekPlan?.amountmicrostoPriceInt
                                        val sixmonthmicro = sixPlan?.amountmicrostoPriceInt
        
                                        bind.weeklytvlongtvid.text =
                                            "• ${getString(R.string.weeksubtv)} : ${weekPlan?.price}"
                                        bind.monthlytvlongtvid.text =
                                            "• ${getString(R.string.sixmonthsubtv)} : ${sixPlan?.price}"
                                        bind.yearlytvlongtvid.text =
                                            "• ${getString(R.string.yearsubtv)} : ${annualPlan?.price}"
        
                                        if (annulemicro != null) {
                                            bind.montbottomid.text =
                                                "${getString(R.string.permonth)} \n ₹${annulemicro / 12}/-"
                                        }
                                        if (sixmonthmicro != null) {
                                            bind.sixmonthbottom.text =
                                                "${getString(R.string.perweek)} \n ₹${sixmonthmicro / 24}/-"
                                        }
        
                                        if (weekmicro != null) {
                                            bind.linabc.text = "₹${(weekmicro * 24)}/-"
                                        }
                                        if (annulemicro != null) {
                                            bind.linabcde.text = "₹${(annulemicro * 48)}/-"
                                        }
        
                                        if (sixmonthmicro != null) {
                                            val yearperchantage =
                                                ((24 * weekmicro!!) - sixmonthmicro).toDouble()
                                            val sec = (yearperchantage * 100 / (24 * weekmicro)).toDouble()
        
                                            ("Six: " + sec).log()
        
                                            bind.sixmonthtvbg.text = "${(sec).roundToInt()}${
                                                getString(
                                                    R.string.off
                                                )
                                            }"
                                        }
        
                                        if (annulemicro != null) {
        
                                            val yearperchantage =
                                                (52 * weekmicro!!) - annulemicro.toDouble()
                                            val sec = yearperchantage * 100 / (52 * weekmicro).toDouble()
        
                                            ("Annul: " + sec).log()
        
                                            bind.monthtvdid.text = "${(sec).roundToInt()}${
                                                getString(
                                                    R.string.off
                                                )
                                            }"
                                        }
        
                                        if (annulemicro != null) {
                                            ("Name: " + ((annulemicro * 100) / (annulemicro * 48))).log()
                                        }
        
                                        val isFreeTrial = productsList.any { it.price == "Free" }
        
                                        if (isFreeTrial) {
                                            App.putBoolean("isFreeTry", true)
                                            bind.startfreetrybtn.text =
                                                getSubscribeText(this@NewForYearSubAct, true)
                                        } else {
                                            App.putBoolean("isFreeTry", false)
                                            bind.startfreetrybtn.text =
                                                getSubscribeText(this@NewForYearSubAct, false)
                                        }
                                    } catch (e: Exception) {
                                        e.printStackTrace()
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

            
        /*
        
        
        -keep class com.google.api.services.playintegrity.** { *; } #REQUIRED
        -keep class com.google.api.client.** { *; } #REQUIRED
        
        
        private fun playSafety() {
                Protectivity(this, BuildConfig.CLOUD_PROJECT_NUM.toLong(), {
                    initview()
                }, {
                    startActivity(Intent(Intent.ACTION_VIEW, Uri.parse("https://play.google.com/store/apps/details?id=$packageName")))
                })
            }
        
                buildConfigField("String", "CLOUD_PROJECT_NUM", "\"787207734962\"")
        
                coreLibraryDesugaringEnabled = true
        
        *  //Play integrity
            implementation("com.google.android.play:integrity:1.3.0")
        
            implementation 'com.google.android.gms:play-services-auth:20.7.0'
            implementation("com.google.apis:google-api-services-playintegrity:v1-rev20231109-2.0.0") {}
            //  Google Authentication and api Access
            implementation("com.google.api-client:google-api-client-jackson2:1.20.0")
            implementation 'com.google.auth:google-auth-library-credentials:1.20.0'
            implementation("com.google.auth:google-auth-library-oauth2-http:1.20.0")
        
            //desugar
            coreLibraryDesugaring 'com.android.tools:desugar_jdk_libs:2.0.4'
        * */
        
        class Protectivity {
            private var onNext : () -> Unit
            private var onReCheck : () -> Unit
            var activity : Activity
            private lateinit var interPin : StandardIntegrityManager.StandardIntegrityToken
        
            constructor(activity : Activity, cloudProjectNumber : Long, onNext : () -> Unit, onReCheck : () -> Unit) {
                this.onNext = onNext
                this.onReCheck = onReCheck
                this.activity = activity
        
                if(activity.isOnlineed) {
                    val standardIntegrityManager : StandardIntegrityManager = IntegrityManagerFactory.createStandard(activity)
                    var integrityTokenProvider : StandardIntegrityManager.StandardIntegrityTokenProvider
                    val standardIntegrityResponse = standardIntegrityManager.prepareIntegrityToken(
                            StandardIntegrityManager.PrepareIntegrityTokenRequest.builder().setCloudProjectNumber(cloudProjectNumber).build()
                    )
                    standardIntegrityResponse.addOnSuccessListener {
                        integrityTokenProvider = it
                        val integrityTokenResponse : Task<StandardIntegrityManager.StandardIntegrityToken>? = integrityTokenProvider.request(
                                StandardIntegrityManager.StandardIntegrityTokenRequest.builder().setRequestHash(generateNonce()).build()
                        )
                        integrityTokenResponse?.addOnSuccessListener { response ->
                            interPin = response
                            decryptToken(response.token())
                        }?.addOnFailureListener {
                            rootHunter(activity)
                        }
                    }.addOnFailureListener {
                        "Token Exception $it".logPlay()
                        rootHunter(activity)
                    }
        
                } else {
                    rootHunter(activity)
                }
            }
        
            private fun decryptToken(token : String) {
        
                val requestObj = DecodeIntegrityTokenRequest()
                requestObj.integrityToken = token
                val credentials = GoogleCredentials.fromStream(activity.assets.open("credentials.json"))
                val requestInitializer : HttpRequestInitializer = HttpCredentialsAdapter(credentials)
                val mHttpTransport = NetHttpTransport()
                val mJacksonFactory = JacksonFactory()
                val mInitializer : GoogleClientRequestInitializer = PlayIntegrityRequestInitializer()
        
        
                val playIntegrity =
                    PlayIntegrity.Builder(mHttpTransport, mJacksonFactory, requestInitializer).setApplicationName(activity.resources.getString(
                        R.string.app_name))
                        .setGoogleClientRequestInitializer(mInitializer)
                val play = playIntegrity.build()
        
                CoroutineScope(Dispatchers.IO).launch {
                    try {
                        val response = play.v1().decodeIntegrityToken(activity.packageName, requestObj).execute()
                        "response $response".logPlay()
                        recognizer(response)
                    } catch (e : Exception) {
                        "response Exception :$e".logPlay()
                        rootHunter(activity)
                    }
                }
            }
        
            private fun recognizer(response : DecodeIntegrityTokenResponse) {
        
                val appLicensingVerdict = response.tokenPayloadExternal.accountDetails.appLicensingVerdict
        
                val appRecognitionVerdict = response.tokenPayloadExternal.appIntegrity.appRecognitionVerdict
        
                val deviceRecognitionVerdict = response.tokenPayloadExternal.deviceIntegrity.deviceRecognitionVerdict
                val isSafeLast =
                    response.tokenPayloadExternal.accountDetails.appLicensingVerdict.isNotEmpty() && appLicensingVerdict.contains("LICENSED") && appRecognitionVerdict.contains(
                            "PLAY_RECOGNIZED"
                    ) && deviceRecognitionVerdict.contains("MEETS_DEVICE_INTEGRITY")
                if(isSafeLast) {
                    CoroutineScope(Dispatchers.Main).launch {
                        onNext.invoke()
                    }
        
                } else {
                    CoroutineScope(Dispatchers.Main).launch {
                        this@Protectivity.playDialog(isLICENSED = getDialogTypeCode(appLicensingVerdict))
                    }
                }
            }
        
            private fun generateNonce() : String {
                val length = 50
                var nonce = ""
                val allowed = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789"
                for (i in 0 until length) {
                    nonce += allowed[floor(Math.random() * allowed.length).toInt()].toString()
                }
                return nonce
            }
        
            private fun rootHunter(activity : Activity, respond : () -> Unit = onNext) {
                val rootBeer = RootBeer(activity)
                if(! rootBeer.isRooted || ! rootBeer.isRootedWithBusyBoxCheck) {
                    CoroutineScope(Dispatchers.Main).launch {
                        respond.invoke()
                    }
                } else {
                    CoroutineScope(Dispatchers.Main).launch {
                        if(! activity.isFinishing) {
                            PlayInitDialog(activity)
                        }
                    }
                }
            }
        
            private fun playDialog(isLICENSED : Int) {
                interPin.showDialog(activity, /*isLICENSED*/ GET_LICENSED).addOnCanceledListener {
                    "Dialog closed".logPlay()
                }.addOnSuccessListener {
                    "==> ${it}".logPlay()
                }.addOnCompleteListener {
                    if(it.result == DIALOG_SUCCESSFUL) {
                        "DIALOG_SUCCESSFUL".logPlay()
                        onReCheck.invoke()
                    }
                    if(it.result == DIALOG_CANCELLED) {
                        "DIALOG_CANCELLED".logPlay()
                        activity.finishAffinity()
                        exitProcess(0)
                    }
                    if(it.result == DIALOG_FAILED) {
                        "DIALOG_FAILED".logPlay()
                    }
                    if(it.result == DIALOG_UNAVAILABLE) {
                        "DIALOG_UNAVAILABLE".logPlay()
                    }
                }.addOnFailureListener {
                    "dialog Exception = ${it.message}".logPlay()
                }
            }
        
            private fun getDialogTypeCode(appLicensingVerdict : String) : Int {
                return if(appLicensingVerdict == "UNLICENSED") {
                    GET_LICENSED
                } else 0
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

       
