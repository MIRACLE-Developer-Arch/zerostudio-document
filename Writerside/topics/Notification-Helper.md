# Notification Helper
* Thanks Carbon 프로젝트에서 처음 생성됨
* 각 유저별로 알림을 보낼 수 있었어야 했음

{ collapsible = true default-state = collapsed }
How To Start Android
: 추가바람

How To Start IOS
: 추가바람


<tabs>
<tab title="Shared">

```Kotlin
expect interface NotificationHelper {
    fun areNotificationsEnabled(): Boolean
    fun openNotificationSettings()
    fun isFirstRun(): Boolean
}
```
</tab>
</tabs>

<tabs>
<tab title = "Android">

```Kotlin
actual interface NotificationHelper {
    actual fun areNotificationsEnabled(): Boolean
    actual fun openNotificationSettings()
    actual fun isFirstRun(): Boolean
}
```
</tab>
<tab title="IOS">

```Kotlin
actual interface NotificationHelper {
    actual fun areNotificationsEnabled(): Boolean
    actual fun openNotificationSettings()
    actual fun isFirstRun(): Boolean
}
```
</tab>
</tabs>
<tabs>
<tab title="AndroidManifest">

```text
        <service
            android:name= ".MyFirebaseMessagingService"
            android:exported= "false" >
            <intent-filter>
                <action android:name= "com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>
```
</tab>
<tab title="AndroidProject">

```Kotlin
class MyFirebaseMessagingService : FirebaseMessagingService() {
    override fun onMessageReceived(remoteMessage: RemoteMessage) {
        super.onMessageReceived(remoteMessage)

        //receive the notification
        if (remoteMessage.notification != null) {
            sendNotification(
                remoteMessage.notification!!.title!!,
                remoteMessage.notification!!.body!!
            )
        }
    }

    private fun sendNotification(title: String, body: String) {
        val intent = Intent(this, MainActivity::class.java)
        intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP)
        val pendingIntent = PendingIntent.getActivity(
            this, 0, intent,
            PendingIntent.FLAG_IMMUTABLE
        )

        val channelId = "default_channel"
        val defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION)
        val notificationBuilder = NotificationCompat.Builder(this, channelId)
            .setSmallIcon(R.drawable.notification_icon)
            .setContentTitle(title)
            .setContentText(body)
            .setAutoCancel(true)
            .setSound(defaultSoundUri)
            .setContentIntent(pendingIntent)

        val notificationManager =
            getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

        // Oreo 이상
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                channelId,
                "Channel human readable title",
                NotificationManager.IMPORTANCE_DEFAULT
            )
            notificationManager.createNotificationChannel(channel)
        }

        notificationManager.notify(0, notificationBuilder.build())
    }
}
```
{ collapsible="true" default-state="collapsed" }
```Kotlin
class AOSNotificationHelper(private val context: Context) : NotificationHelper {
    override fun areNotificationsEnabled(): Boolean {
        return NotificationManagerCompat.from(context).areNotificationsEnabled()
    }

    override fun openNotificationSettings() {
        val intent = Intent().apply {
            action = Settings.ACTION_APP_NOTIFICATION_SETTINGS
            putExtra(Settings.EXTRA_APP_PACKAGE, context.packageName)
        }
        context.startActivity(intent)
    }

    override fun isFirstRun(): Boolean {
        val sharedPreferences = context.getSharedPreferences("MyAppPreferences", Context.MODE_PRIVATE)
        if (sharedPreferences.getBoolean("isFirstRun", true)) {
            sharedPreferences.edit().putBoolean("isFirstRun", false).apply()
            return true
        }
        return false
    }

}
```
{ collapsible="true" default-state="collapsed" }
</tab>
</tabs>

<tabs>
<tab title="IOS">
추가바람
</tab>
<tab title="Swift">
추가바람
</tab>
</tabs>