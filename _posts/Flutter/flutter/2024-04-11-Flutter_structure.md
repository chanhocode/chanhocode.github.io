---
title: "[Flutter] Flutter란 무엇이고 기본 프로젝트 생성을 통한 구조 이해"
writer: chanho
date: 2024-04-11 17:10:00 +0900
categories: [Flutter, flutter]
tags: [flutter]
---

Android, ios에 모두 대응할 수 있는 개발 방식으로 모바일 크로스 플랫폼 방식이 있다. 크로스 플랫폼 개발 프레임워크로는 현 시점에서는 크게 'React-native' 와 'flutter'로 나뉘었다고 본다. 그 중 우리는 'Flutter' 무엇인지에 대해 알아보고자 한다.

![image](https://github.com/chanhocode/chanhocode.github.io/assets/105937460/58c9440f-11de-46d0-9a27-e29c6930587c)
[출처](https://codigee.com/blog/react-native-vs-flutter-what-technology-is-the-best-for-fast-mobile-mvp)

# 크로스 플랫폼 이란?
> 크로스 플랫폼이란 여러 운영체제 또는 플랫폼과 호환되는 소프트웨어, 응용 프로그램 또는 도구를 의미한다. 앱 개발의 맥락에서 이는 단일 코드 베이스를 사용하여 다양한 운영체제에서 작동하는 애플리케이션을 개발하는 능력을 의미한다.

## 장점
- 개발 시간 및 비용 절감: 여러 플랫폼에 대해 코드를 한 번 작성하는 것이 별도의 코드 베이스를 유지 관리하는 것보다 더 효율적이다.
- 더 쉬워진 유지 관리 및 배포: 업데이트 및 버그 수정은 하나의 코드베이스에서만 수행되어야 한다.
- 더 넓은 시장 도달 범위: 여러 플랫폼에 동시에 배포할 수 있다는 것은 더 넓은 고객에게 다가갈 수 있음을 의미한다.

# Flutter이란 무엇인가?
> 'Flutter'이란 'google'에서 만든 오픈 소스 UI 소프트웨어 개발 키트 이다. 단일 코드베이스에서 "android, ios, linux, max, windows, google fuchsia 및 웹용 크로스 플랫폼" 어플리케이션을 개발하는 데 사용된다.

`💡 Android Meterial & IOS Cupertino`
- Meterial Widget: Android의 기본 화면 구성 요소를 구현한 플러터 위젯이다.
- Cupertino Widget: IOS의 기본 화면 구성 요소를 구현한 플러터 위젯이다.

# Flutter 기본 프로젝트 구조
> 'Flutter'의 기본 프로젝트를 생성하면 구성되는 폴더와 파일에 대한 간략한 소개를 하고자 한다.

![image](https://github.com/chanhocode/chanhocode.github.io/assets/105937460/ac84ea81-05e8-4738-bfad-083a2d38cbc5)

플러터 프로젝트를 생성시 위 이미지와 같은 폴더 구조 및 파일을 가지게 된다.

## Directory
|폴더명|소개|
|---|----|
|.idea|개발 도구 설정|
|android|안드로이드 네이티브 코드를 작성|
|ios|IOS 네이티브 코드를 작성|
|build|빌드시 생성되는 파일|
|`lib`|`dart 코드를 작성하는 부분`|
|test|테스트 코드를 작성|
|web|WEB 프로젝트 폴더|
|window|Window 프로젝트 폴더|

## File
|파일명|소개|
|----|----|
|.gitignore|git에서 버전 관리시 무시한 파일에 대한 정보를 작성|
|.metadata|프로젝트가 관리하는 파일로 임의 수정 금지|
|.packages|패키지에 대한 정보로 임의 수정 금지|
|`analysis_options.yaml`|해당 프로젝트에 대한 규칙을 정의|
|flutter_app_name.iml|개발 도구에 필요한 설정 파일로 임의 수정 금지|
|pubspec.lock|패키지 매니저가 이용하는 파일로 임의 수정 금지|
|`pubspec.yaml`|`패키지 매니저가 이용하는 파일`|
|README.md|해당 프로젝트에 대한 설명을 작성|

# Flutter 기본 프로젝트 실행
![image](https://github.com/chanhocode/chanhocode.github.io/assets/105937460/6174c235-7dc9-460d-9fe9-d14b12d5fe5c)

> 'Flutter'프로젝트를 생성 후 기본으로 생성된 프로젝트를 구동하면 볼 수 있는 화면이다. 해당 프로젝트는 단순 카운터 어플리케이션으로 에뮬레이터 우측 하단의 '+' 버튼을 클릭하면 중앙의 숫자가 올라가는 카운터 어플리케이션이다.

```dart
import 'package:flutter/material.dart';
```
> 해당 구문을 사용하여 머티리얼 패키지를 불러올 수 있다. 해당 패키지는 버튼, 레이아웃 구조 및 Android, ios에서 일관된 디자인으로 시각적으로 매력적인 앱을 구축하는 데 도움이 되는 기타 많은 UI 구성 요소를 가지고 있다.

```dart
void main() {
  runApp(const MyApp());
}
```
> 앱의 시작 지점이다. 'runApp'을 통해 해단 어플리케이션의 'MyApp' 인스턴스를 전달한다.

```dart
class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}
```
> StatelessWidget 클래스는 상태가 없는 위젯을 정의하는 기본 클래스이다. 상태를 가지지 않는다는 것은 한 번 그려진 후 다시 그리지 않는 경우를 의미한다. 해당 클래스는 프로퍼티로 변수를 가지지 않지만 상수는 가질수 있다.

```dart
class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key, required this.title}) : super(key: key);
  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.headline4,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ),
    );
  }
}
```
> StatefulWidget 클래스는 상태가 있는 위젯을 정의할 때 사용한다. 'MyHomePage'클래스는 'StatefulWidget'을 상속받아 앱이 실행되는 동안 상태를 변경하여 UI가 변경될 수 있다. 'createState'메서드를 통해 'MyHomePage'를 해당 상태 클래스와 연결하는 '_MyHomePageState'의 인스턴스를 반환한다.
