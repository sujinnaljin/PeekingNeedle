# Needle 찍어먹기

> 블로그 글들 이리저리 긁어와서 내가 이해하기 쉬운 순서대로 재구성!

모듈 간의 종속성이 있는 경우, 서로를 바라보는 의존성 그래프가 그려집니다.

이를 즉 모듈 간의 순환 종속성 관계(Circular Dependency)를 가진다고 하는데요, 적절한 모듈화도 좋지만, 순환 종속성이 생기면 모듈화를 섣불리 하기 어렵습니다. ([[iOS][Swift] 모듈간의 관계를 Dependency Injection Container으로 풀어보자](https://minsone.github.io/programming/swift-solved-circular-dependency-from-dependency-injection-container))

우리는 이러한 문제점을 어떻게 해결 할 수 있을까요? 

바로 경험상 의존성 주입을 통해서 해결해야한다는 것을 알고 있습니다 (경험해보지 못하셨다면 유감!)

각 모듈에서는 자신이 필요한 기능을 정의해둔 protocol 을 정의해두고 사용합니다. 그리고 메인 프로젝트에서는 각 모듈을 알고 있으므로 실제 concrete 한 구현체는 이곳에서 주입을 해주거죠!

의존성 주입을 해줄때는 이렇게 보통 밖에서 인스턴스를 만들어서 주입해줍니다. 하지만 밖에서 인스턴스를 만들어서 주입해주는 곳은 앱에서 여러군데 입니다. 즉 인스턴스를 만드는 위치가 분산되어서 관리포인트가 많아지게 됩니다!

그렇다면

🤔 : 흠..  상자 (📦) 안에 

1. 내가 사용할 모든 인스턴스를 미리 다 만들어서 등록해두고, 
2. 필요한 시점에 이 상자 안에 등록해뒀던 인스턴스를 달라고 하면 

인스턴스 초기화는 이 상자 안에서 한번만 해주면 되겠네?! 라고 생각할 수 있습니다.  ([[DI] DI Container, IOC Container 개념과 예제](https://eunjin3786.tistory.com/233)) 

오호.. 상자의 필요성이 조금 다가옵니다. 이 상자 (📦) 의 필요성을 좀 더 이해하기위해, 상자가 없을 때의 다른 상황을 살펴보겠습니다. 

상자가 없다면 우리는 아래와 같이 앱 target 프로젝트에서 직접 Storage 라는 인스턴스를 주입할텐데요,

```
let manager = FilesystemManager(storage: Storage())
```

🤔 : FilesystemManager 에서는 어차피 protocol 로 인자를 받고 있을때고, concrete 한 구현체인 Storage 는 모르고 있으니까 상관없는거 아냐??

저도 그런것 같았는데..!! 앱 taget 에서 Storage의 생성과 FileSystemManager의 관계 설정을 해주고 있는 상황이 IoC 에는 위배된다고 하네요. 

IoC 란 Inversion of Control 의 약자로, 기존 구조적 설계와 비교해 프레임워크가 제어를 나누어 가져가되어 의존 관계가 방향이 달라지게 되는 것을 제어가 반전,역전되었다고 합니다. 

어쨌든 IoC에 의하면, 앱 taget 은 Storage를 알 필요가 없는 상황이고, 그냥 아까 말한 상자(📦)에서 객체를 관리하고 생성을 책임지고, 의존성을 관리하는 역할을 하면 되는거죠! ([[iOS] Dependency Injection, (IoC, DI, DIP)](https://lidium.tistory.com/34))

그리고 우리는 이 상자를 "DI Container" 혹은 "IoC Container" 라고 부르기로 합니다.

좋아여 그럼 여기까지 

1. 모듈간의 순환 종속성의 문제점을 해결하기 위해 의존성 주입이 필요하고, 
2. 의존성 주입의 관리 포인트를 줄이고, 의존 관계의 역전을 위해 DI / IoC Container 의 개념이 도입이 되었다! 를 알아봤습니다.

그럼 실제로 한번 구현을 해볼까요?

🤔 : ..........에??? 제가요.....?

뭐 여러 블로그 글들을 참고해서 차근차근 따라가보면 야 너두 할 수 있어! 겠지만....  의존성을 관리할 컨테이너를 직접 구현해야하고, 여기서 객체를 사용하기 위해서는 컨테이너를 생성할 때 모든 객체 생성자를 넣어줘야하고.. 등등의 불편함이 있겠죠? ([[Swift] Needle DI Tool - 의존성 라이브러리](https://nsios.tistory.com/186))

그래서 등장했습니다! DI Container 도구!! 바로 Swinject 나 Needle 같은 친구들이져!

iOS 진영에서 주로 사용하는 프레임워크로 [Swinject](https://github.com/Swinject/Swinject) 가 있습니다. 다만 얘는 컴파일 시점에 안전성을 확인하기 어렵다는 단점이 있는데요, 런타임에 dependency 를 등록하기 때문에 resolve 시점에 값이 없을 수도 있습니다. ([iOS) Needle 로 의존성 주입하기](https://okanghoon.medium.com/ios-needle-%EB%A1%9C-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85%ED%95%98%EA%B8%B0-f5019a4f2b92), [Swift Dependency Injection](https://www.gabrielle-earnshaw.com/posts/swift-dependency-injection/))

또한 의존성 수가 점점 많아질 수록, 수동으로 연결해야하는 수많은 객체가 생깁니다. 귀찮기도 할뿐더러 더더욱 안전성을 보장하기 어렵습니다.

그에 반해 Needle 의 차밍 포인트는 아래와 같습니다. ([모듈화하고 Needle 적용해보기](https://medium.com/daangn/%EB%AA%A8%EB%93%88%ED%99%94%ED%95%98%EA%B3%A0-needle-%EC%A0%81%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0-bd5e9f3c450b))

1. 컴파일 타임에서 잘못된 DI 계층을 지적해주기 때문에 컴파일 시점의 안정성 보장
2. 매번 새로운 객체를 추가할 때마다 register 해주는 코드를 작성할 필요 없이 자동으로 생성해줌
3. 컴파일과 동시에 계층적으로 그려진 DI 코드를 자동으로 생성해줌

그럼 이제 Needle 을 본격적으로 알아보러 가볼까요? ([iOS) Needle 로 의존성 주입하기](https://okanghoon.medium.com/ios-needle-%EB%A1%9C-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85%ED%95%98%EA%B8%B0-f5019a4f2b92))

Needle 에서 각 의존성의 범위는 `Component`로 정의하고, 그에 대한 의존성은 `protocol`로 캡슐화됩니다. 그리고 이 둘을 제네릭을 사용하여 연결합니다.

 🤔 : 뭐라고요.....?

ㅎㅎ 코드로 바로 가죠! 

먼저 필요한 의존성을 프로토콜로 정의합니다.

```
protocol MyDependency: Dependency {
  var chocolate: Food { get }
  var milk: Food { get }
}
```

그러니까 여기서는 chocolate 이랑 milk 를 상위로부터 의존성을 주입 받고 싶은 값이라는거겠져?

이렇게 정의한 의존성 프로토콜을 활용해 컴포넌트를 정의합니다. 

```
class MyComponent: Component<MyDependency> {
}
```

그러면 이 안에서는 `dependency.chocolate` , `dependency.milk` 처럼 값에 접근할 수 있게 됩니다.

만약 이번에는 hotChocolate 을 주입받고 싶고 싶은 애 (MyChildComponent) 가 있다?! 하면 똑같이 dependency  를 정의해주고, 컴포넌트로 연결해서 사용하면 되는데요

```
protocol MyChildDependency: Dependency {
  var hotChocolate: Drink { get }
}

class MyChildComponent: Component<MyChildDependency> {
  var veryHotChocolate: Drink {
    return VeryHotChocolate(dependency.hotChocolate)
  }
}
```

그럼 여기서 실제로 이 Component 의 의존성(MyDependency)은 어디서 획득하냐!! 라는 의문이 들 수 있습니다. 

그건 바로 상위 Scope 에서 정의를 해줍니다. 상위 Scope란, 해당 컴포넌트 생성에 사용하는 parent 를 의미합니다.

아까 예시로 들면 요렇게 실제로 MyChildComponent 를 생성하는 곳, 즉 parent scope 인 MyComponent 가 parent 가 되고, 이곳에서 child 에 필요한 의존성을 정의해두는거져

```
class MyComponent: Component<MyDependency> {

  // 새로운 객체인 hotChocolate을 의존성 그래프에 추가합니다.
  // 하위 Scope들에서 Dependency 프로토콜을 통해 이를 획득할 수 있습니다.
  var hotChocolate: Drink {
    return HotChocolate(dependency.chocolate, dependency.milk)
  }

  // 자식 Scope는 항상 부모 Scope에 의해 인스턴스화됩니다.
  var myChildComponent: MyChildComponent {
    return MyChildComponent(parent: self)
  }
}
```

🤔 음.. 대충 감은 잡은거 같은데.. Component 를 실제로 어떻게 이용하는데?? 실제 예시를 들어봐라!

먼저 최상위 컴포넌트를 정의합니다. 상위 Scope 이 없는 BootstrapComponent 를 활용합니다.

```
final class RootComponent: BootstrapComponent {}
```

이에 RootViewController 와 필요한 의존성들을 정의합니다. 로그인 화면과 로그아웃된 화면이 필요하기에 각각을 컴포넌트로 정의합니다.

**RootComponent 예시**

```
// RootComponent.swift

final class RootComponent: BootstrapComponent {
  var playersStream: PlayersStream {
    return mutablePlayersStream
  }

  // 해당 스코프에 객체가 하나로 유지되어야 하면 shared 를 활용해요
  // RootComponent 에서 활용하면 싱글톤 패턴으로 활용 가능해요
  var mutablePlayersStream: MutablePlayersStream {
    return shared { PlayersStreamImpl() }
  }

  var rootViewController: UIViewController {
    return RootViewController(
      loggedOutBuilder: loggedOutComponent,
      loggedInBuilder: loggedInComponent
    )    
  }

  var loggedOutComponent: LoggedOutComponent {
    return LoggedOutComponent(parent: self)
  }

  var loggedInComponent: LoggedInComponent {
    return LoggedInComponent(parent: self)
  }
}
```

**각 서브 컴포넌트 예시**

예를 들어 로그 아웃 화면이 필요하기 때문에 별도의 Component 로 정의를 하고, 여기서 필요한 의존성을 주입받아서 loggedOutViewController 를 생성해줄 수 있습니다.

```
protocol LoggedOutDependency: Dependency {
  var mutablePlayersStream: MutablePlayersStream { get }
}

final class LoggedOutComponent: Component<LoggedOutDependency>, LoggedOutBuilder {

  var loggedOutViewController: UIViewController {
    return LoggedOutViewController(
      mutablePlayersStream: mutablePlayersStream
    )
  }
}

// ViewController 를 지연 생성하기 위해 프로토콜과 computed property 활용
protocol LoggedOutBuilder {
  var loggedOutViewController: UIViewController { get }
}
```

(뭔가 볼수록 RIBs 같기도 하고..)

그럼 우리 프로젝트에서 적용을 해보겠습니다.

Needle 은 NeedleFoundation 프레임워크와 executable code generator 로 구성됩니다. Needle을 DI 시스템으로 사용하려면 두 부분 모두를 Swift 프로젝트에 통합해야 합니다.

1 . `NeedleFoundation` framework 설치하기

`NeedleFoundation` 프레임워크를 Swift 프로젝트와 통합하려면 표준 [Swift Package Manager 패키지 정의 프로세스](https://github.com/apple/swift-package-manager/blob/master/Documentation/Usage.md)를 통해 Needle을 의존성에 추가합니다.

```
dependencies: [
    .package(url: "https://github.com/uber/needle.git", .upToNextMajor(from: "VERSION_NUMBER")),
],
targets: [
    .target(
        name: "YOUR_MODULE",
        dependencies: [
            "NeedleFoundation",
        ]),
],
```

2. code generator 설치하기

```
brew install needle
```

needle code generator 는 개발자가 작성한 코드를 구문 분석해 Swift 소스 코드를 생성하는 커맨드라인 유틸리티입니다. 생성된 코드는 개발자가 작성하는 다양한 `Component` 서브클래스를 연결합니다. DI 그래프 구조에 따라 generator 는 각 `Component` 를 연결합니다. 생성된 코드는 앱에서 컴파일되며 완전한 DI 그래프를 제공합니다. needle code generator 의 대략적인 동작방식은 [iOS) Needle 로 의존성 주입하기](https://okanghoon.medium.com/ios-needle-%EB%A1%9C-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85%ED%95%98%EA%B8%B0-f5019a4f2b92) 을 참고해주세요

이렇게 설치한 code generator 를 실행할 수 있도록 build phase 에 스크립트를 작성해줘야하는데요, 저는 블로그에서 본 대로 제일 간단히 써봤습니다

```
if which needle; then
SOURCEKIT_LOGGING=0 && needle generate $SRCROOT/Sources/NeedleGenerated.swift $SRCROOT/.. / else echo "warning: Needle not installed, download from https: //github.com/uber/needle using Homebrew"
fi
```

이제 코드를 작성하고 런했을때 결과는!?!

ㅎㅎㅎ..  역시 니들 스크립트 실행할때부터 에러 터집니다.

```
Showing All Messages
SourceParsingFramework/FileEnumerator.swift:161: Fatal error: Failed to traverse file:///Library/Application%20Support/Apple/ParentalControls/Users with error Error Domain=NSCocoaErrorDomain Code=257 "The file “Users” couldn’t be opened because you don’t have permission to view it." UserInfo={NSURL=file:///Library/Application%20Support/Apple/ParentalControls/Users, NSFilePath=/Library/Application Support/Apple/ParentalControls/Users, NSUnderlyingError=0x600001bb4360 {Error Domain=NSPOSIXErrorDomain Code=13 "Permission denied"}}.
```

에러 로그를 보면.. 권한 실패요..?? Users 폴더에 읽기 권한 설정해줬는데도 해당 에러가 떴습니다. 

좀 더 구글링해서 [The file couldn’t be opened because you don’t have permission to view it.](https://joolib.tistory.com/11) 보고 엑코 자체에 전체 디스크 권한 주려고 했는데..  에? 왜 베타밖에 안떠요..?

Xcode 가 세개있는데, beta 한개밖에 안뜨길래,, 한개만 설정되는건가??? 해서 저 목록에서 베타를 삭제하고 다시 Xcode-13 을 클릭해도,, 베타가 뜹니다......? 엑코 괴담인가여..? 

beta 지우고 테스트해보고 싶다가도.. 다시 설치하려면 너무 한나절이라.. 이렇게 하면 될거라는 일단 믿음으로 스킵하겠읍니다...?

여기까지 얼레벌레 needle 찍먹하기 끝!

