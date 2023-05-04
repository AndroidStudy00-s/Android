# MVC 패턴

**MVC 패턴이란?**

MVC 패턴은 소프트웨어 아키텍처 디자인 패턴으로 소프트웨어를 구성하는 요소를 `Model`, `View`, `Controller` 세 가지 요소로 나눠서
서로 분리되어 동작하는 방식이다. 주로 웹에서 사용하며, 가장 널리 사용되 구조이다.

## Model
- 비즈니스 로직을 담당하고 `View`에 보여질 데이터를 나타낸다.
- 데이터에 대한 로직을 정의(DB, Api Call 등)하고 해당 데이터를 핸들링 하는 메서드(함수)를 제공한다.
- 데이터 변경시 `Controller`에게 알려야한다.

## View
- 사용자 인터페이스(UI)를 나타낸다.
- `View`는 `Model`을 조작하지 않으며, 독립적으로 작동해야함.

## Controller
- 사용자의 입력을 수신한다.
- `Model`의 데이터를 업데이트하거나 `View`를 업데이트한다.

# 안드로이드에서는 어떻게 사용할까?
위 내용에선 보통 일반적으로 MVC패턴을 사용하며, 주로 웹에서 사용한다고 설명했다.

안드로이드에선 어떻게 사용되는지 알아보자.
<img width="1184" alt="image" src="https://user-images.githubusercontent.com/70135188/236113578-517ca659-5041-4e4b-bcc7-52b09257e322.png">

왼쪽이 기본적으로 사용되는 MVC패턴, 오른쪽이 안드로이드에서 사용되는 MVC패턴이다.

유저의 입력을 받는 부분과, 결과를 나타내는 UI가 Activity or Fragment로 나타낸다.

따라서 오른쪽 그림은 `View`와 `Controller`가 혼합되어 있다. 

## MVC 패턴을 구현하는 안드로이드 코틀린 코드
```kotlin
// Activitiy (View, Controller)
binding.userUpdateButton.setOnclickListener {
    val userResponse = UserModel().updateUser()
    
    if(userResponse.isSuccess) {
        // 업데이트된 데이터 매핑
        binding.nickname.text = userResponse.nickname
        ...    
    } else {
        // toast 등 띄우기
    }
}


// Model
class UserModel() {
    fun updateUser(): UpdateUserResponse {
        val response = ... // 업데이트 해당하는 api call
        return response
    }
}
```
간략하게 표현하면 위 코드같은 형태일 것이다. 음 그런데 안드로이드로 MVC 패턴을 작성해보니 한가지 문제점이 나온다.

## 문제점이 무엇일까?
MVC 패턴은 각 요소를 나눠서 서로 분리되는 동작을 해야한다. 그런데, 안드로이드에선 `Controller`와 `View` 가 묶여있어서 비즈니스
로직을 처리하는 `Model`은 위 2가지 요소에 의존성이 높다.

**앱의 크기가 커질수록 다음과 같은 결과를 가지고 올 수 있다.**
- 코드의 라인수와 복잡도 증가.
- `View`와 `Model` 사이의 결합도가 높아 테스트에 용이하지 않음.
- 요소들의 분리가 되어있지 않아 유지보수가 어렵다.

**MVC 디자인 패턴은 언제 사용하는게 좋을까?**

다른 디자인 패턴보다 설계가 쉬워 짧은 프로젝트에 잘 사용될 것 같다. Activity, Fragment에서 모든 처리 가능.


# 1분 답변


MVC 디자인 패턴은 `Model`, `View`, `Controller` 세 가지 요소로 나눠 분리되어 동작하는 방식입니다. 유저의 이밴트를 `Contoller`에서 받고
이벤트에 따라 필요시 `Model`에 업데이트 요청을 하여 `View`에 나타냅니다. 안드로이드에선 `Model`은 `View`에 종속적이며 요소들의 분리가 되어있지 않아
유지보수가 어렵고, 복잡도가 증가하는 단점이 있습니다.

# 출처
https://medium.com/@jang.wangsu/%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4-mvc-%ED%8C%A8%ED%84%B4%EC%9D%B4%EB%9E%80-1d74fac6e256

https://thdev.tech/androiddev/2016/10/23/Android-MVC-Architecture/



