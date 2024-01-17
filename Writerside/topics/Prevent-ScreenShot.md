# Prevent ScreenShot Android
* INCLER 프로젝트에서 처음으로 생성됨
* 안드로이드에서 스크린샷을 방지하고자 할 때

<tabs>
<tab title="MainActivity">

```Kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    window.setFlags(WindowManager.LayoutParams.FLAG_SECURE, WindowManager.LayoutParams.FLAG_SECURE)
}
```
</tab>
</tabs>
<note>
<p>
쉽다!
</p>
</note>