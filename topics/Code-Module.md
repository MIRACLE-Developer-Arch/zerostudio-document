# Code Module

# Basic RULE
1. 띄워쓰기는 언더바로 표기 "_"
2. 주요 기능을 담당하는 타입(클래스, 이넘, 오브젝트), 함수의 경우 **대문자** 로 시작  
   ex)
```Kotlin
class ViewModel_Login
```
3. 함수의 인자(parameter)는 무조건 해당 함수나 클래스의 **소문자**로 표기  
   ex)
```Kotlin
@Composable
fun View_Login(viewModel: ViewModel_Login)
```
4. 클래스 내부에서 사용되는 **private**의 경우(MutableStateFlow) 언더바로 표기 "_"
5. 클래스 외부에서 사용되는 **public**의 경우(StateFlow) 언더바를 표기하지 않고 그냥 표기  
   ex)
````Kotlin
private val _loginState = MutableStateFlow<LoginState?>(null)
val loginState: StateFlow<LoginState?> get() = _loginState
````
6. 클래스 내부에서 public으로 선언되어야할 변수나 함수는 public 붙이지 말것
7. 하나의 파일 내부에 여러 함수나 클래스를 만들 시 최상단에 무조건 최상위 부모가 위치할 것  
   ex)
````Kotlin
@Composable
fun View_Login() {
}
@Composable
fun Button_Login() {}
````
8. MVVM 패턴 취급 시, 클래스나 함수를 생성할 때, 무조건 컴포넌트 명이 제일 앞으로 오도록 할 것  
   ex)
````Kotlin
class Model_Login() {}
class ViewModel_Login() {}
@Composable
fun View_Login() {}
````
9. 서버통신 클래스의 경우(Firebase) 결과 값을 가져올 때 항상 try-catch구문을 활용해 에러 핸들링 할 것  
   ex)
````Kotlin
fun getSomethingToFirebase() {
    return try {
        TODO()
    } catch (e: Exception) {
        TODO(Handling Error)
    }
}
````
10. 모든 클래스나 함수의 경우 **단일 책임 원칙**을 따르도록 설계할 것

