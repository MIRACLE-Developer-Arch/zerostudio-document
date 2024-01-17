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
import MessageUI

class IOSExternalViewController: PlatformExternalViewController {
    func showEmailView(type: MailType) {
        let email = "cs@incler.kr"
        var url = URL(string: "mailto:\(email)")
        
        if type == MailType.request {
            url = URL(string: "mailto:\(email)?subject= &body=" +
                             "콘텐츠 신청 폼\n" +
                             "•신청 콘텐츠는 일정 신청 수 이상일 경우\n" +
                             "콘텐츠가 발행됩니다\n" +
                             "\n" +
                             "필수 기입*\n" +
                             "기업명:\n" +
                             "업종:\n" +
                             "시장: K-OTC, 코넥스, 비상장 중 택 1\n" +
                             "\n" +
                             "선택 기입\n" +
                      "기타 요청사항:")
        }
        UIApplication.shared.open(url!)
        //mailto:[수신자 이메일 주소]?subject=[제목]&body=[본문]&cc=[참조]&bcc=[숨은 참조]
        //mailto:example@example.com?subject=Meeting Request&body=Hi there,%0D%0A%0D%0APlease let me know your availability for a meeting next week.%0D%0AThank you!&cc=cc@example.com&bcc=bcc@example.com
    }
    
    func showKakaoPlusFriend() {
        if let url = URL(string: "http://pf.kakao.com/_xbxnZxjG") {
            UIApplication.shared.open(url)
        }
    }
}
```
</tab>
</tabs>