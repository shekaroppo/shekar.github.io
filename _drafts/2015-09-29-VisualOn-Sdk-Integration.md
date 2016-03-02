---
title: VisualOn Sdk Integration
date: 2015-09-03
published: false
---

  * OnStream MediaPlayer+ (OSMP+) is a multimedia player development kit enabling cross-platform content delivery and playback on connected devices including mobile handsets, tablets, desktops, set-top boxes, and smart TVs.
  * OSMP+ builds upon market-proven technology that currently powers over 200 million mobile devices.
  * The OSMP+ Player SDK for Android is a development kit to create a media player on Android-based devices. It supports various functions   such as:
    * Playback of multiple video/audio formats to devices with or without native support.
    * Playback of live or VOD streaming, progressive download, and local media sources.
    * Easy integration with Digital Rights Management (DRM) decryption engines.
    * Ad insertion and in-stream Ad playback.
    * Audio post-processing for 3rd party audio effect plug-ins and equalizers.
    * Video post-processing for Closed Captions (CC) and subtitles rendering.

#### V-Nova
  * A London-based video compression solutions.
  * The company’s technology team includes developers of the first generation codecs, including MPEG and JPEG.
  * V-Nova claims that its new video compression software *PERSEUS* delivers HD quality video in SD bit rates.

#### What makes PERSEUS so special?
  * Offering UHD quality at HD bitrates, HD at SD bitrates, and SD video at audio bitrates.
  * It’s more than three times more efficient than other compression platforms.
  * More than five times more efficient with broadcast transmissions.
  * More than six times more efficient than other compression systems with on-demand.
  * This new method of data compression could see UHD/ 4K being streamed to TVs and other devices using around 50% of the bandwidth currently needed
  * It provides seamless, adaptive streaming from a single encode
  * Make continuous, frame-by-frame and “on the fly” bitrate changes.
  * HD video can be live-encoded below 500 kbit/s and without loss to the picture or viewing continuity.
  * Over-the-air software update
  * V-Nova describe PERSEUS as “an advanced signal processing technology based on the principles underlying human vision”.

#### OnStream MediaPlayer+ Player SDK Integration Guide
  * Access to the SDK is provided through the VOCommonPlayer interface
  * Basic Integration
  ![](/public/images/visualonsdkflow.png){: .center-image }

  **INITIALIZING THE CLIENT**
Surface Identification and Configuration(Initializing Client)
The drawing surface must be identified and configured for use with the SDK.
Android SurfaceView and its SurfaceHolder provide the drawing surface for the SDK
The SurfaceHolder must be configured to the RGB32 pixel format
And its callbacks must be initiated to notify the SDK of any surface changes.
Sample code:
public void onCreate(Bundle savedInstanceState) {

...
// Find SurfaceView and Surface holder from the layout                                                                         m_svMain = (SurfaceView) findViewById(R.id.svMain);
// Add a Callback for this holder                                                               m_svMain.getHolder().addCallback(this);
// Set pixel format to RGB32                                                  m_svMain.getHolder().setFormat(PixelFormat.RGBA_8888); ...

}
SDK File Transfer(Loading DRM library)
The SDK requires a license file (provided by VisualOn, Inc.) that must be locally installed with the SDK client application on the device.
The SDK may also leverage a local device capability file, which optimizes playback by limiting bit rates and resolutions to those supported by the particular device/processor.
Copy the asset files to the package directory on the device.
public void onCreate(Bundle savedInstanceState) {

...
// Copy license file and device capability file                                                                   copyfile(this, "voVidDec.dat", "voVidDec.dat");                                                                   copyfile(this, "cap.xml", "cap.xml");

}
INITIALIZING THE SDK
Implements the SDK initialization and configuration tasks in surfaceCreated()callback method, which is called when the surface is created,
 Create the SDK Instance
// Create the SDK
VO_OSMP_RETURN_CODE nRet;                                                                                                                                                                          m_sdkPlayer = new VOCommonPlayerImpl();                                                          m_svMain.getHolder().setType(SurfaceHolder.SURFACE_TYPE_NORMAL);
Set media framework to VO_OSMP_VOME2_PLAYER.                                                                                                                                                                                                      // SDK player engine type
VO_OSMP_PLAYER_ENGINE eEngineType = VO_OSMP_PLAYER_ENGINE.VO_OSMP_VOME2_PLAYER;
Load DRM library
// Retrieve location of libraries
String apkPath = getUserPath(this) + "/lib/";
String cfgPath = getUserPath(this) + "/";
VOOSMPInitParam init = new VOOSMPInitParam();
init.setContext(this);
init.setLibraryPath(apkPath);
// Initialize SDK player
         nRet = m_sdkPlayer.init(eEngineType, init);
Configure SDK View Settings
 //Set view: Provide SDK with the current SurfaceView.
m_sdkPlayer.setView(m_svMain);
// Set surface view size:  Provide SDK with the display size, based on the default DisplayMetrics.
DisplayMetrics dm = new DisplayMetrics();
getWindowManager().getDefaultDisplay().getMetrics(dm);
m_sdkPlayer.setViewSize(dm.widthPixels, dm.heightPixels);
Register SDK Event Listener
Register SDK event listener callback to manage SDK events.
m_sdkPlayer.setOnEventListener(m_listenerEvent);
Configure Device Capability Settings
// Set device capability file location
String capFile = cfgPath + "cap.xml";
m_sdkPlayer.setDeviceCapabilityByFile(capFile);
Configure License Settings
// Set license file location
m_sdkPlayer.setLicenseFilePath(cfgPath);
Media Source Identification (Open Media Source)
The VOCommonPlayer.open() method opens the source and prepares the media pipeline for playback. The VOCommonPlayer.open() method must be provided with:
The source location (URL or file path) as a String* type.
The SDK will require a media source (e.g., URL or file path) for playback.                                                  public void onCreate(Bundle savedInstanceState) {          ...
         // Define your playback URL/local media file here
         m_strVideoPath ="http://devimages.apple.com/iphone/samples/bipbop/bipbopall.m3u8" ; ... }
Source flags (currently, VO_OSMP_SRC_FLAG.VO_OSMP_FLAG_SRC_OPEN_SYNC(Opening the media source in sync mode causes the client application to block until the API call to VOCommonPlayer.open() method completes/returns.) and VO_OSMP_SRC_FLAG.VO_OSMP_FLAG_SRC_OPEN_ASYNC(API call returns immediately, and the client application is not blocked waiting for the media source to be opened.)are supported.
The source format (H.264, MP4, etc., or auto-detect) as a VO_OSMP_SRC_FORMAT type.
Initialization parameters/flags (not currently used).
public void surfaceCreated(SurfaceHolder surfaceholder) { // Initialize the player
// Load DRM library
...


         // Start playing the video
         // First open the media source
         // Auto-detect source format
         VO_OSMP_SRC_FORMAT format =

VO_OSMP_SRC_FORMAT.VO_OSMP_SRC_AUTO_DETECT;
// Set source flag to asynchronous
VO_OSMP_SRC_FLAG eSourceFlag;
eSourceFlag = VO_OSMP_SRC_FLAG.VO_OSMP_FLAG_SRC_OPEN_ASYNC;

VOOSMPOpenParam openParam = new VOOSMPOpenParam(); openParam.setDecoderType(VO_OSMP_DECODER_TYPE.VP_OSMP_DEC_VIDEO_SW.g

etValue() | VO_OSMP_DECODER_TYPE.VO_OSMP_DEC_AUDIO_SW.getValue());
//Open media source
if (mRet = m_sdkPlayer.open(m_strVideoPath, eSourceFlag, format,

openParam);

... }

... }
Basic SDK Event Handling

Requirements and Recommendations
The SDK client shall:
Find the SurfaceView and its SurfaceHolder.
Initiate the SurfaceHolder callback and set the pixel format to RGB32.
Copy the voVidDec.dat license file to the local package directory.
Identify a media source for playback.
The SDK client should:
Copy the cap.xml configuration file to the local package directory.




#### Understanding bitrates in video files
 * Video data rates are given in bits per second.
 * The data rate for a video file is the bitrate.
 * So a data rate specification for video content that runs at 1 megabyte per second would be given as a bitrate of 8 megabits per second (8 mbps), as 1 megabyte equals 8 megabits.
   * LD 240p 3G Mobile @ H.264 baseline profile 350 kbps (3 MB/minute)
   * LD 360p 4G Mobile @ H.264 main profile 700 kbps (6 MB/minute)
   * SD 480p WiFi @ H.264 main profile 1200 kbps (10 MB/minute)
   * HD 720p @ H.264 high profile 2500 kbps (20 MB/minute)
   * HD 1080p @ H.264 high profile 5000 kbps (35 MB/minute)
 * A higher bitrate will accommodate higher image quality in the video output.
 * At the same bitrate, video in a newer codec such as H.264 will look substantially better than an older codec like H.263.
 * Variable bitrate (VBR) encoding will produce better image quality than constant bitrate (CBR) in most applications.
