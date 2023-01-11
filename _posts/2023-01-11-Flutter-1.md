---
title : "[Flutter] StatefulWidget & LifeCycle"
excerpt : "StatefulWidget LifeCycle 정리"
categories :
  - Flutter
tags:
  - Flutter
  - Widget
  - LifeCycle
last_modified_at : 2023-01-11
---

## StatefulWidget
> A widget that has mutable state

StatefulWidget은 변경 가능한 (mutable) state를 가진 widget을 말한다.<br>

> State is information that (1) can be read synchronously when the widget is built and (2) might change during the lifetime of the widget.

이때, state(상태)란 다음과 같이 정의될 수 있다.
1. widget이 build 되었을 때 동기적(synchronously)으로 읽을 수 있는 정보
2. widget의 lifetime 동안 변경될 수 있는 정보

이와 같은 State를 가지는 StatefulWidget은 State의 변화에 따라 UI의 변경이 필요할 때 사용될 수 있다. (즉, State가 변경될 때마다, build가 수행된다.)


## LifeCycle
<center><img src="/assets/images/flutter/StatefulWidge_LifeCycle.png" width="70%"></center>

#### 1. createState()
- 플러터가 StatefulWidget을 빌드하도록 지시할 시 호출된다.
- 해당 메소드는 StatefulWidget 내에 반드시 존재해야 한다.
- 위젯 트리에 상태를 만들기 위해 호출된다.

```dart
class App extends StatefulWidget {
  @override
  _App createState() => new _AppState();
}
```
#### 2. mounted == true

createState()가 State 클래스를 생성시 `buildContext`가 State에 할당된다.
(`buildContext`는 위젯이 배치된 위젯트리의 위치를 단순화 한 것)

모든 위젯은 `bool` 형식의 `this.mounted` 속성을 가지고 있다. `buildContext`가 할당 시 `true`를 return 한다. 위젯이 `unmounted` 상태일 시 `setState`를 호출하면 error가 발생한다.

#### 3. initState()

위젯이 생성될때 (클래스 생성자 다음으로) 처음으로 호출되는 메서드 이다.
initState는 최초 한번만 호출되며, 반드시 내부에서 super.initState()를 호출해야 한다. 
(주로 초기화 하는 역할을 담당한다.)

다음 사항들을 주로 initState에서 수행한다.
1. 생성된 위젯 인스턴스의 BuildContext에 의존적인 것들의 데이터 초기화.
2. 동일한 위젯트리 내에 부모 위젯에 의존하는 속성 초기화.
3. Stream 구독, 알림 변경, 또는 위젯의 데이터를 변경할 수 있는 다른 객체 핸들링.

#### 4. didChangeDependencies()

- initState() 다음으로 바로 호출된다.
- State 객체의 종속성이 변경될 때 호출된다.
    - 위젯이 의존하는 데이터의 객체가 호출될 때마다 호출된다.<br>(ex. 업데이트 되는 위젯을 상속한 경우)
    - 상속한 위젯이 업데이트 될때 네트워크 호출이 필요한 경우 유용. <br>(또는 다른 비용이 큰 액션 API 호출 등)

#### 5.build()  
- 재정의(@override)대상이고 반드시 존재해야 한다. 
- 위젯으로 만든 UI를 구축한다. (Widget을 return한다.)
- 다양한 곳에서 반복적으로 호출된다.
- 변경된 부분 트리를 감지하고 대체한다.

#### 6. didUpdateWidget()
- 위젯의 구성이 변경되거나, 부모 위젯이 변경되어 해당 위젯을 다시 그려야 하는 경우 호출된다.
- oldWidget 인수를 취득해 비교한다.
- Flutter는 항상 해당 메서드를 수행한 후, build()를 호출하므로, setState() 이후 추가적인 (직접적) 호출은 불필요하다. 
- 기본적으로 위젯의 상태와 관련된 위젯을 재구성해야 하는 경우 initState()를 대체한다.

```dart
@override
void didUpdateWidget(Widget oldWidget) {
  if (oldWidget.importantProperty != widget.importantProperty) {
    _init();
  }
}
```

#### 7. setState()

- state가 변경된 사실을 프레임워크에 알리는데 사용된다.
- `buildContext`의 위젯을 다시 빌드하게 한다.
- async가 아닌 callback을 사용한다. (callback으로 비동기를 사용할 수 없다.)



#### 8. deactivate()

- 거의 사용되지 않는다.
- tree에서 State가 제거될때 호출된다. 


#### 9. dispose()

- 객체가 트리에서 완전히 삭제되고 두 번 다시 빌드되지 않을 시 호출된다.
- 해당 메소드는 모든 animation, streams 등등을 unsubscribe하거나 cancel한다.

#### 10. mounted == false
- `state`객체가 다시는 remount되지 않는다.
- setState()가 호출 시 error을 발생시킨다.





