---
layout: single
title:  "Unreal-SubSystem"
categories: Unreal
tag : [Unreal]
toc : true
author_profile : false
sidebar:
    nav: "docs"

# search : false
---


언리얼 엔진 4의 서브시스템은 수명이 관리되는 자동 인스턴싱 클래스입니다. 이 클래스는 사용하기 쉬운 확장점을 제공하여, 프로그래머는 블루프린트 및 Python에 바로 노출시킴과 동시에 복잡한 엔진 클래스 수정 또는 오버라이드를 피할 수 있습니다.

 

**1. 서브시스템**

| 서브시스템   | 상속 원본                     |
| ------------ | ----------------------------- |
| Engine       | UEngineSubsystem 클래스       |
| Editor       | UEditorSubsystem 클래스       |
| GameInstance | UGameInstanceSubsystem 클래스 |
| LocalPlayer  | ULocalPlayerSubsystem 클래스  |

위 표는 현재 지원되는 서브시스템들입니다.



여기서 GameInstance 서브시스템을 커스터마이징했을때, 커스터마이징한 서브시스템의 수명에 대해 설명드리겠습니다.

```
class UMyGameSubsystem : public UGameInstanceSubsystem
```

위 예제 코드처럼 베이스 클래스에서 파생된 클래스를 생성하면 다음 스텝으로 수명이 관리됩니다.

- UGameInstance 생성 이후, UMyGameSubsystem 인스턴스 역시 생성됩니다.
- UGameInstance 초기화시, UMyGameSubsystem 인스턴스에서 Initialize()가 호출됩니다.
- UGameInstance 종료시, UMyGameSubsystem 인스턴스에서 Deinitialize()가 호출됩니다.
- Deinitialize()가 호출된 시점에서 UMyGameSubsystem 인스턴스에 대한 참조가 삭제되고 더이상 참조가 없으면 UMyGameSubsystem 인스턴스는 가비지 컬렉션됩니다.

 

 

**2. 서브시스템을 사용하는 이유**

프로그래밍 서브시스템을 사용하는데는 다음과 같은 몇가지 이유가 존재합니다.

- 프로그래밍 시간이 절약됩니다.
- 엔진 클래스 오버라이드를 피할 수 있습니다.
- 이미 복잡한 클래스에 API 추가를 피할 수 있습니다.
- 사용자에게 친숙한 유형의 노드를 통해 블루프린트로 액세스할 수 있습니다.
- 에디터 스크립팅이나 테스트 코드 작성을 위해 Python 스크립트에서 액세스할 수 있습니다.
- 코드베이스의 모듈성과 일관성을 제공합니다.

서브시스템은 플러그인을 만들때 특히 유용합니다. 플러그인 작동에 필요한 코드 관련 지침이 없어도 됩니다.

사용자는 플러그인을 게임에 추가하기만 하면, 플러그인이 언제 인스턴싱 및 초기화될지 정확히 알 수 있습니다.

따라서 언리얼 엔진 4에서 제공되는 API 및 기능을 사용하는데만 중점을 둘 수 있습니다.

 

 

**3. 블루프린트로 서브시스템 액세스**

서브시스템은 블루프린트에 자동 노출되며, 컨텍스트를 이해하는 스마트 노드가 추가되므로 형변환이 필요없습니다.

표준 UFUNCTION() 마크업 및 규칙으로 블루프린트에서 사용할 수 있는 API를 제어할 수 있습니다.

 



![img](https://blog.kakaocdn.net/dn/bNrTOI/btraZNbHz59/xFR1ekRnvNwSZx6qfMq6K0/img.png)



블루프린트 그래프에 우클릭해서 컨텍스트 메뉴를 표시하고 subsystems를 검색하면 위와 같이 표시됩니다.

위 사진은 특정 서브시스템에 대한 주요 유형 및 개별 항목 각각에 대한 카테고리입니다.

 



![img](https://blog.kakaocdn.net/dn/dyx1YX/btraQrIhdrJ/wMk9W4pVk9ESvPbdkhTH41/img.png)



위 사진처럼 서브시스템 노드를 추가하면, 서브시스템의 API를 사용할 수 있습니다.

 

 

**4. Python으로 서브시스템 액세스**

```
my_engine_subsystem = unreal.get_engine_subsystem(unreal.MyEngineSubsystem)
my_editor_subsystem = unreal.get_editor_subsystem(unreal.MyEditorSubsystem)
```

Python을 사용하는 경우, 위 예제와 같이 내장된 접근자를 사용해서 서브시스템에 액세스할 수 있습니다.

 

 

**5. 각각의 서브시스템 수명**

- Engine Subsystem

```
class UMyEngineSubsystem : public UEngineSubsystem { ... };
```

Engine Subsystem의 모듈이 로드되면, 서브시스템은 모듈의 Startup() 함수 반환 이후 Initialize()를 호출하고, 모듈의 Shutdown() 함수 반환 이후 Deinitialize()를 호출합니다.

 

```
UMyEngineSubsystem* MySubsystem = GEngine->GetEngineSubsystem<UMyEngineSubsystem>();
```

Engine Subsystem은 위 예제와 같이 GEngine을 통해 액세스할 수 있습니다.

 

- Editor Subsystem

```
class UMyEditorSubsystem : public UEditorSubsystem { ... };
```

Editor Subsystem도 마찬가지로 모듈이 로드되면, 서브시스템은 모듈의 Startup() 함수 반환 이후 Initialize()하고, 모듈의 Shutdown() 함수 반환 이후 Deinitialize()합니다.

 

```
UMyEditorSubsystem* MySubsystem = GEditor->GetEditorSubsystem<UMyEditorSubsystem>();
```

Editor Subsystem은 위 예제와 같이 GEditor를 통해 액세스할 수 있습니다.

 

- GameInstance Subsystem

```
class UMyGameSubsystem : public UGameInstanceSubsystem { ... };
```

GameInstance Subsystem 또한 모듈 로드시 Startup() => Initialize(), Shutdown() => Deinitialize() 순으로 진행됩니다.

 

```
UGameInstance* GameInstance = GetGameInstance();
UMyGameSubsystem* MySubsystem = GameInstance->GetSubsystem<UMyGameSubsystem>();
```

GameInstance Subsystem은 위 예제와 같이 UGameInstance를 통해 액세스할 수 있습니다.

 

- LocalPlayer Subsystem

```
class UMyPlayerSubsystem : public ULocalPlayerSubsystem { ... };
```

LocalPlayer Subsystem도 모듈 로드시 Startup() => Initialize(), Shutdown() => Deinitialize() 순으로 진행됩니다.

 

```
UGameInstance* GameInstance = GetGameInstance();
ULocalPlayer* LocalPlayer = GameInstance->GetFirstGamePlayer();;
UMyPlayerSubsystem* MySubsystem = LocalPlayer->GetSubsystem<UMyPlayerSubsystem>();
```

LocalPlayer Subsystem은 위 예제와 같이 ULocalPlayer를 통해 액세스할 수 있습니다.



[참조 사이트]https://jhtop0419.tistory.com/67?category=944635