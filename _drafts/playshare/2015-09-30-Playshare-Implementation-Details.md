---
title: Playshare Implementation Details
date: 2015-09-30
published: false
---


#### Login
  *

  MusicPlayerActivity
  onStart
    MediaBrowser
      MusicService
      MediaBrowser.ConnectionCallback
        onConnected
          mMediaBrowser.subscribe(mMediaBrowser.getRoot(), mSubscriptionCallback);
            mSubscriptionCallback
              onChildrenLoaded
          MediaController(mMediaBrowser.getSessionToken())
          registerCallback(mMediaControllerCallback);
            mMediaControllerCallback
              onMetadataChanged
              onPlaybackStateChanged
              onSessionDestroyed
           setMediaController(mediaController);
