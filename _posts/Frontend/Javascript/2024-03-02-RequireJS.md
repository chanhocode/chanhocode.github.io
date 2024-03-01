---
title: "[Library] RequireJS 기본 사용방법 및 AMD(Asynchronous Module Definition)에 대해서"
writer: chanho
date: 2024-03-02 00:08:00 +0900
categories: [Frontend, JavaScript]
tags: [frontend, javascript, library]
---

`RequireJS`는 'JavaScript' 파일과 모듈을 관리하고 로드하기 위해 사용되는 'JavaScript Library'이다. 주요 목적으로 페이지 성능을 향상시키고 코드의 유지 관리를 쉽게 하기위해 사용된다. 'RequireJS'를 사용하는 주된 이유는 모듈화된 개발 접근 방식을 채택하여 대규모 'JavaScript' 프로젝트의 복잡성을 관리하고, 성능을 최적화 하며, 유지보수를 용이하게 만들수 있기에 사용된다.

# 주요 기능

1. `모듈 정의 및 비동기 로딩`: 'RequireJS'는 'AMD(`Asynchronous Module Definition`)'규격을 사용하여 모듈을 정의하고 비동기적으로 로드한다. 이것은 페이지 로딩 시간을 단축시키고, 필요할 때만 특정 스크립트를 로드할 수 있게 한다.
2. `코드 조직 및 관리`: 크고 복잡한 'JavaScript 어플리케이션'에서 코드를 더 잘 조직하고 관리 할 수 있게 도와준다. 모듈은 서로의 의존성을 명시적으로 선언할 수 있으며, 이는 코드의 가독성과 유지 보수성을 향상시킨다.
3. `의존성 관리`: 'RequireJS'는 모듈 간의 의존성을 관리하고, 의존하는 모듈이 로드되고 초기화 된 후에만 모듈을 실행한다. 이는 코드의 `실행 순서를 보장`하고, 의존성 문제를 방지한다.
4. `플러그인 지원`: 텍스트, 이미지, JSON 데이터 등 다양한 유형의 리소스를 로드하기 위한 플러그인을 지원한다. 이를 통해 개발자는 다양한 리소스를 쉽게 통합하고 사용할 수 있다.
5. `최적화 도구`: 애플리케이션을 위한 최적화 도구가 포함되어 있어, 모듈을 하나의 파일로 결합하고 최조화하여 배포할 수 있다. 이는 네트워크 요청을 줄이고 로딩시간을 개선한다.

# RequireJS가 필요한 이유

> 'RequireJS'를 사용하지 않았을 떄 발생할 수 있는 문제를 예시로 들어 해당 라이브러리가 왜 필요한지 이해하고자 한다.

## 문제 상황: 의존성 관리 없이 모듈을 로딩한 상태

> 자바스크립트에서 여러 모듈(파일)을 사용할 때, `각 모듈은 특정 순서로 로드되어야 할 수 있다.` 만약, 이 순서가 보장되지 않는다면, `아직 정의 되지 않은 함수나 객체에 접근하려고 할 때 에러가 발생`할 수 있다.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Sample</title>
    <script src="module1.js"></script>
    <!-- 의존: module2.js -->
    <script src="module2.js"></script>
  </head>
  <body>
    <script>
      // module1과 module2를 사용
      module1Function();
      module2Function();
    </script>
  </body>
</html>
```

```javascript
// module1.js : 현재, module2.js에 정의된 함수를 사용한다.
function module1Function() {
  module2Function();
  console.log("Module 1 function called.");
}
```

```javascript
// module2.js
function module2Function() {
  console.log("Module 2 function called.");
}
```

해당 상황은 'module1.js'는 'module2.js'에 정의된 'module2Function'에 `의존`하고있다. 그러나 'module1.js'가 'module2.ja' `보다 먼저 로드`되면 'module1Function'이 'module2Function'을 호출할 때 'module2Function'은 아직 정의되지 않았기 때문에 에러가 발생하게 된다. (html 파일에서 script 순서에 의해 로드순서가 정의됨)

## RequireJS를 사용하여 해결

> RequireJS를 사용하면, 각 모듈이 필요로하는 `의존성을 명시적으로 선언할 수 있으며`, 이러한 의존성을 해결하여 모듈이 올바른 순서로 로드 되도록한다.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Sample Page</title>
    <script data-main="app" src="require.js"></script>
  </head>
  <body></body>
</html>
```

- 'script' 태그에서 ' src="require.js" '는 RequireJS라이브러리를 로드한다.
- ' data-main="app" ' 속성은 RequireJS가 초기에 로드할 JavaScript 파일을 지정한다. 여기서는 'app.js'파일을 의미한다.

```javascript
// app.js
require(["module1"], function (module1) {
  module1.module1Function();
});
```

- 'require'함수는 RequireJS에서 모듈을 로드하는 데 사용된다.
- ['module1']는 'require'함수에게 'module1' 모듈을 로드하라고 지시한다.
- 'function(module1) {...}'는 'module1'모듈이 로드되면 실행 될 콜백함수 이다. 이 함수 내에서 'module1.module1Function()'를 호출하여 'module1'의 함수를 실행한다.

```javascript
// module2.js
define([], function () {
  return {
    module2Function: function () {
      console.log("Module 2 function called.");
    }
  };
});
```

```javascript
// module1.js
define(['module2'], function(module2) {
  return {
     module1Function: function() {
      module2.module2Function();
      console.log("console.log("Module 1 function called.");
     }
  };
});
```

- 'define'함수는 RequireJS에서 모듈을 정의하는 데 사용된다.
- ['module2']는 'module1' 모듈이 'module2' 모듈에 의존한다는 것을 나타낸다.
- 'function(module2) {...}'는 'module2'가 로드되었을 때 실행될 콜백함수이다.
- 이 함수는 'module1Function'을 객체로 반환한다. 이 함수 내에서는 먼저 'module2.module2Function()'을 호출한 다음, "Module 1 function called."를 콘솔에 출력한다.

# 설정 : Requirejs.config()

## 주요 설정

- `baseUrl`: `모든 모듈의 기준 URL을 설정`한다. 'baseUrl'이 설정되면, 이 경로를 기준으로 모든 모듈의 경로가 계산된다. 예를 들어, 모듈 파일들이 '/js/lib/' 디렉토리에 있다면, "baseUrl: '/js/lib/'"로 설정할 수 있다.
- `paths`: `모듈 이름과 해당 모듈의 실제 파일 경로 간의 매핑을 정의`한다. 이를 통해 모듈 이름을 단순화 시키거나, CDN과 같은 외부 경로에서 모듈을 로드할 때 유용하다. 예를 들어 `'jquery': 'https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min'`와 같이 설정하여 'define'에서 'jquery'를 불러오게 되면, 스크립트 태그에서는 설정한 경로를 읽어오게 된다. 추가적으로 `'efg': 'abcd/efg'` 가 있다면 'define'에서 'efg/module'를 불러온다면 'src="abcd/efg/module.js'로 잡는다. 이때, `'/' 또는 'http'등으로 시작하지 않는다면, 기본적으로는 'baseUrl'에 상대경로로 설정`이 된다. 또한 'path 매핑 코드'는 자동적으로 '.js' 확장자를 붙여 모듈명을 매핑한다.
- `shim`: `AMD 형식을 지원하지 않는 라이브러리를 RequireJS와 함께 사용`하기 위한 설정이다. 'shim'을 사용하면, 특정 라이브러리의 의존성과 초기화 코드를 정의할 수 있다.
- `map`: 특정 모듈에 대해 다른 모듈을 사용하도록 매핑할 수 있다. 이는 같은 API를 가진 두 모듈을 서로 대체할 때 유용하다.
- `config`: 모듈별로 추가 설정을 제공할 수 있다. 이는 특정 모듈이 구성 데이터를 필요로 할 때 사용된다.
- `deps`: `애플리케이션 초기 로드 시에 포함할 모듈 목록`이다. 이 설정은 'require'함수를 직접 호출하지 않고도 모듈을 로드할 수 있게 한다.
- `callback`: 'deps'를 통해 로드된 모듈들이 준비되었을 때 실행할 콜백 함수이다. 이 함수는 모든 의존성이 로드되고 난 후에 실행된다.
- `waitSeconds`: 모듈 로드 시 대기할 최대 시간(초)을 설정한다. 이 시간이 지나면 로드가 실패한 것으로 간주하고 오류가 발생한다.

```javascript
requirejs.config({
  baseUrl: "js/lib",
  paths: {
    jquery: "https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min"
  },
  shim: {
    "jquery.alpha": ["jquery"],
    "jquery.beta": ["jquery"]
  },
  deps: ["app"],
  callback: function (app) {
    // 모든 의존성이 로드된 후 실행될 코드
  },
  waitSeconds: 15
});

require(["module1", "module2"], function (module1, module2) {
  // 모듈 로드 후 실행될 코드
});
```

# AMD (Asynchronous Module Definition)

> 자바스크립트에서 `비동기적으로 모듈을 정의하고 사용`하기 위한 표준이다. 이 형식은 특히 브라우저 환경에서 자바스크립트 모듈과 라이브러리를 비동기적으로 로드하고 관리하는 데 유용하다.

## 주요 특징

- `비동기 로딩`: AMD는 모듈을 비동기적으로 로드한다. 이는 모듈이 필요할 때만 로드되며, 페이지 로딩 속도에 영향을 주지 않는다.
- `모듈 정의`: AMD에서 모듈은 'define'함수를 사용하여 정의된다. 이 함수는 의존성 목록과, 모듈의 구현을 담은 콜백 함수를 인자로 받는다.
- `의존성 관리`: 'define'함수 내에서 선언된 의존성은 모듈이 실행되기 전에 먼저 로드된다. 이렇게 함으로써 모듈 간의 의존성을 명확하게 관리할 수 있게된다.
- `모듈 식별자`: AMD모듈은 고유한 식별자(ID)를 가질 수 있다. 하지만 식별자를 명시적으로 제공하지 않는 경우, 모듈의 파일 경로가 식별자로 사용된다.

## 장점

- 성능향상: 필요한 모듈만 비동기적으로 로드함으로써 초기 페이지 로드 시간을 단축시킬 수 있다.
- 코드 조직: 모듈 별로 파일을 분리하여 코드를 조직할 수 있으므로, 대규모 어플리케이션에서 코드의 가독성과 유지보수성이 향상된다.
- 의존성 제어: 모듈 간의 의존성이 명시적으로 관리되므로, 코드의 실행 순서와 의존성 문제를 보다 쉽게 해결할 수 있다.

## AMD Module 예시

```javascript
define(["dependency1", "dependency2"], function (dep1, dep2) {
  // 모듈의 기능을 정의
  var myModule = {
    doSomething: function () {
      // 기능 구현
    }
  };

  return myModule;
});
```

> 'define'함수는 두 개의 의존성 'dependency1'과 'dependency2'를 사용한다. 콜백 함수는 이 의존성들이 로드된 후 실행되며, 모듈의 기능을 구현하고 반환한다.
