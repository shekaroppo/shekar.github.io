The alarm mechanism allows events, specifically broadcast intents, to be triggered at a specific time or interval.
Alarms can be one time or recurrent.
Alarms can either wake the device up or not.
Alarms can be scheduled based on the time since the device last booted up (good for short duration alarms) or based on the wall clock


MainActivity
  SampleAlarmReceiver
    setAlarm
      init AlarmManager
      init PendingIntent
      init Calendar
      Sets a repeating alarm
        AlarmManager.setInexactRepeating
          -You need to use ELAPSED_REALTIME_WAKEUP if you simply want your alarm to fire every 60 minutes
          -You need to use RTC_WAKEUP if you want your alarm to fire at a particular date/time
        Enable BootReceiver to automatically restart the alarm when the device is rebooted.
          PackageManager.setComponentEnabledSetting(ComponentName,  PackageManager.COMPONENT_ENABLED_STATE_ENABLED, PackageManager.DONT_KILL_APP)
    onReceive
      Create a new intent to deliver to the service.
        Intent service = new Intent(context, SampleSchedulingService.class);
        startWakefulService(context, service);

   SampleSchedulingService
      onHandleIntent
        sendNotification
   SampleBootReceiver
      onReceive
        SampleAlarmReceiver.setAlarm(context);

Main Points

- ## alarmManager.setRepeating( )
  - This method is more resource consuming, we do not recommend this approach unless it is really necessary.
- ## alarmManager.setInexactRepeating( )
  - Android will synchronize multiple in-exact alarms at run time to save resources
  - As of API 19, all repeating alarms will be in-exact and subject to batching with other alarms regardless of their stated repeat interval.
- ## ELAPSED_REALTIME_WAKEUP: This is the preferred choice if your alarm is based on elapsed time--for example, if you simply want your alarm to fire every 60 minutes.
- ## RTC_WAKEUP: If you want your alarm to fire at a particular date/time
  http://stackoverflow.com/questions/5102073/android-alarm-what-is-the-difference-between-four-types-of-alarm-that-alarmmanag
  http://stackoverflow.com/questions/5938213/android-alarmmanager-rtc-wakeup-vs-elapsed-realtime-wakeup
- Check if alarm is already set
    http://stackoverflow.com/questions/4556670/how-to-check-if-alarmmanager-already-has-an-alarm-set
- Alarm with setInExact not firing at right time, firing with some delay
    http://stackoverflow.com/questions/21461191/alarmmanager-fires-alarms-at-wrong-time
