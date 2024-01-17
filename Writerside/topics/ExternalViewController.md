# ExternalView Controller
* INCLER 프로젝트에서 처음 생성됨
* 이메일 보내기 / 카카오 친구추가 등 외부 링크를 사용했어야 했음


<tabs>
<tab title="Shared">

```Kotlin
enum class MailType {
    FAQ,
    REQUEST
}
expect interface PlatformExternalViewController {
    fun showEmailView(type: MailType)
    fun showKakaoPlusFriend()
}
```
</tab>
</tabs>

<tabs>
<tab title="Android">

```Kotlin
actual interface PlatformExternalViewController {
    actual fun showEmailView(type: MailType)
    actual fun showKakaoPlusFriend()
}
```
</tab>
<tab title="IOS">

```Kotlin
actual interface PlatformExternalViewController {
    actual fun showEmailView(type: MailType)
    actual fun showKakaoPlusFriend()
}
```
</tab>
</tabs>

<tabs>
<tab title="AndroidProject">

```Kotlin
class AOSExternalViewController(private var context: Context): PlatformExternalViewController {
    override fun showEmailView(type: MailType) {
        val email = Intent(Intent.ACTION_SEND)
        email.type = "plain/text"
        val address = arrayOf("test@gmail.com")
        email.putExtra(Intent.EXTRA_EMAIL, address)

        if (type == MailType.REQUEST) {
            email.putExtra(Intent.EXTRA_TEXT, "이메일 형식의 내용 입력")
        }

        context.startActivity(email)
    }

    override fun showKakaoPlusFriend() {
        val url = "http://pf.kakao.com/KakaoLink"
        val view = Intent(Intent.ACTION_VIEW)
        view.data = Uri.parse(url)
        context.startActivity(view)
    }
}
```
{ collapsible="true" default-state="collapsed" }
</tab>
<tab title="SwiftProject">

```Swift
추가바람
```
</tab>
</tabs>