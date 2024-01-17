# Basic
**[KMM(Kotlin-MultiPlatform-Mobile)](https://kotlinlang.org/docs/multiplatform.html)** 의 가장 큰 장점은 코틀린 언어로 Android, IOS, Desktop, Wearable 등 여러 플랫폼을 단일 코드로 개발이 가능하다는 것이다.

아래는 **[ZeroStudio](https://zerostudio.co.kr/)** 에서 사용중인 KMM의 가장 강력한 기능인 [expect/actual](https://kotlinlang.org/docs/multiplatform-expect-actual.html)을 사용하여 만든 모듈이며, 
모든 제로스튜디오 일원들의 생산성 향상을 목적으로 제작되었음.



### 글자 크기 DP로 고정 함수
* 디바이스 세팅에 따라 글자 크기가 변하는 것을 막고자 함.
* 해당 함수는 SharedUtil 내부에 포함되어있음
<tabs>
<tab title="Shared">

````Kotlin
    @Composable
    fun fixedFontSize(dpSize: Dp): TextUnit {
        return with(LocalDensity.current) { dpSize.toSp() }
    }
````
</tab>
</tabs>



### Load external resource to Common
How To Start
:
1. [MOKO-resources](https://github.com/icerockdev/moko-resources) 라이브러리를 필요로 함
2. sharedModule root 디렉터리에 resources폴더를 추가로 생성해줘야함
3. resources폴더 내부 MR폴더 생성, 필요한 타입의 폴더를 생성해줌
4. 각 폴더에 필요한 파일을 두고 IDE에서 Sync를 수행하면 자동으로 임포트됨

<note>
  <p>
    찾을 수 없다고 빨간줄이 뜰텐데, Sync를 수행했으면 실행은 제대로 되니 겁먹지 말자
  </p>
</note>


폴더 계층 구조는 다음과 같다

```text
shared
  │
  ├── src
  │    └── commonMain
  │           ├── kotlin
  │           └── resources
  │                 └── MR
  │                     ├── fonts/ <- .ttf 파일들이 위치함
  │                     └── images/ <- .svg 또는 .png 파일들이 위치함
```
<tabs>
<tab title="Image">

```Kotlin
@Composable
fun loadImageResource(imageFile: ImageFile) : Painter {
    val resId = when (imageFile) {
        ImageFile.GOOGLE_LOGIN -> MR.images.android_light_sq_SI
        ImageFile.APPLE_LOGIN -> MR.images.appleid_button
        ImageFile.ADD -> MR.images.add
        ImageFile.CANCEL -> MR.images.cancel
        ImageFile.HOME -> MR.images.home
        ImageFile.LOCATION -> MR.images.location
        ImageFile.PERSON -> MR.images.person
        ImageFile.RIGHT_ARROW -> MR.images.rightArrow
        ImageFile.CAMERA -> MR.images.addPhoto
        ImageFile.GALLERY -> MR.images.photoLibrary
    }

    return painterResource(resId)
}
```
</tab>
<tab title="Font">

```Kotlin
@Composable
fun loadFontResource(fontStyle: FontStyle) : FontFamily {
    val fontId = when (fontStyle) {
        FontStyle.EXTRA_BOLD -> MR.fonts.Pretendard.extraBold
        FontStyle.BOLD -> MR.fonts.Pretendard.bold
        FontStyle.LIGHT -> MR.fonts.Pretendard.light
        FontStyle.REGULAR -> MR.fonts.Pretendard.regular
        FontStyle.SEMI -> MR.fonts.Pretendard.semiBold
        FontStyle.MEDIUM -> MR.fonts.Pretendard.medium
    }
    return fontFamilyResource(fontId)
}
```
</tab>
</tabs>
