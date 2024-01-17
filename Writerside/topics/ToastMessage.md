# ToastMessage

<tabs>
<tab title="Shared">

````Kotlin
expect fun toastMessage(context: Any?, message: String)
````
</tab>
</tabs>
<tabs>
<tab title="Android">

````Kotlin
actual fun toastMessage(context: Any?, message: String) {
    if (context != null) {
        val actualContext = when (context) {
            is Context -> context
            else -> throw IllegalArgumentException("Invalid type for context")
        }
        Toast.makeText(actualContext, message, Toast.LENGTH_SHORT).show()
    }
}
````
{ collapsible="true" default-state="expanded" }
</tab>
<tab title="IOS">

````Kotlin
actual fun toastMessage(context: Any?, message: String) {
    val toastLabel = UILabel()
    toastLabel.backgroundColor = UIColor.clearColor
    toastLabel.textColor = UIColor.blackColor()
    toastLabel.textAlignment = NSTextAlignmentCenter
    toastLabel.text = message
    toastLabel.alpha = 1.0
    toastLabel.layer.cornerRadius = 10.0
    toastLabel.clipsToBounds = true
    toastLabel.numberOfLines = 0

    toastLabel.sizeToFit()

    val maxWidth = UIScreen.mainScreen.bounds.useContents { size.width } * 0.8
    if (toastLabel.frame.useContents { size.width } > maxWidth) {
        toastLabel.frame.useContents {
            size.width = maxWidth
        }
    }

    val window = UIApplication.sharedApplication.keyWindow
    window?.addSubview(toastLabel)
    toastLabel.center = window!!.center

    UIView.animateWithDuration(
        4.0,
        animations = { toastLabel.alpha = 0.0 },
        completion = { _ -> toastLabel.removeFromSuperview() }
    )
}
````
{collapsible="true" default-state="expanded" }
</tab>
</tabs>