---
title: "[Library] Toast UI Grid에 대해서"
writer: chanho
date: 2024-03-14 22:30:00 +0900
categories: [Frontend, JavaScript]
tags: [frontend, javascript, library, toastui]
---

`Toast UI Grid`는 웹 어플리케이션에서 사용되는 데이터 그리드 라이브러리입니다. 이 라이브러리는 대용량 데이터를 효과적으로 표시하고 관리할 수 있는 풍부한 기능을 제공한다.

`🚶 활용 사례`

- 대시보드 및 리포트: 비즈니스 데이터 대시보드나 리포트에서 대용향의 데이터를 효과적으로 표시하고 분석할 수 있다. 필터링, 정렬, 그룹화 등의 기능을 사용하여 중요한 데이터를 쉽게 찾아볼 수 있다.
- 관리자 패널: 웹 사이트나 애플리케이션의 백엔드 관리자 패널에서 사용자, 주문, 상품 등의 데이터를 관리할 때 유용하다. 인라인 편집 기능을 사용하여 데이터를 직접 수정할 수 있다.
- 재고 관리 시스템: 재고 목록을 관리하고, 상품의 수량, 상태 등을 쉽게 업데이트하고 추적할 수 있다.
- 금융 데이터 분석: 주식, 외환 등의 금융 데이터를 실시간으로 표시하고 분석하는 데 사용할 수 있다. 데이터의 변동을 즉각적으로 파악하고 결정을 내릴 수 있도록 도와준다.
- 교육 데이터 관리: 학교나 교육 기관에서 학생, 성적, 출석 등의 데이터를 관리하는 데 사용할 수 있다. 데이터를 명확하게 정리하여 학생들의 학업 성과를 추적할 수 있다.
- 이벤트 관리: 참가자 목록, 일정 관리, 자원 할당 등 이벤트 관리와 관련된 다양한 데이터를 관리하는데 사용할 수 있다.

# Toast UI Grid 시작하기

- `NPM 사용`

```
$ npm inatall tui-grid
```

- `CDN 사용`

```
<link rel="stylesheet" href="https://uicdn.toast.com/grid/latest/tui-grid.css" />
<script src="https://uicdn.toast.com/grid/latest/tui-grid.js"></script>
```

# Toast UI Grid 옵션 알아보기

> 'Toast UI Grid'에서 옵션들을 사용하면 그리드의 기능과 외관을 사용자의 요구에 맞게 조정할 수 있다. 주요 옵션들을 소개하고자 한다.

## 데이터 관련 옵션

- `data`: 표시할 데이터. 일반적으로 객체의 배열 형태로 제공된다.
- `columns`: 열을 정의하는 객체의 배열. 각 열에 대한 설정을 포함한다.(ex\_이름, 타입, 편집 가능 여부 등)

## 스타일 및 레이아웃

- `rowHeight`: 각 행의 높이 설정
- `header`: 헤더 관련 설정(헤더 높이 등)
- `showRowNumber`: 행 번호 표시 여부
- `rowHeaders`: 행 헤더 설정(체크박스, 행 번호 등)

## 편집 기능

- `editingEvent`: 편집 모드를 활성화하는 이벤트('click', 'dblclick' 등)
- `filter`: 필터 옵션 설정

## 정렬 및 필터링

- `sortingMode`: 정렬 모드 설정('single', 'multiple')
- `filter`: 필터 옵션 설정

## 페이징 및 네비케이션

- `pagination`: 페이징 설정
- `pageOptions`: 페이징에 관한 추가 옵션(ex\_페이지당 행 수, 네비게이션 버튼)

## 선택 및 포커스

- `selectionUnit`: 선택 단위 설정(셀, 행)
- `focus`: 포커스 관련 설정

## 국제화 및 지역화

- `language`: 그리드의 언어 설정(en, ko)

## 기타 기능

- `contextMenu`: 컨텍스트 메뉴 관련 설정
- `scrollX, scrollY`: 가로 및 세로 스크롤 설정
- `height, minHeight, maxHeight`: 그리드의 높이 설정
- `copyOptions`: 복사 기능 관련 설정

# Toast UI Grid 사용해보기

```html
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Practice1</title>
    <link rel="stylesheet" href="node_modules/tui-grid/dist/tui-grid.css" />
  </head>
  <body>
    <div id="grid"></div>

    <script src="node_modules/tui-grid/dist/tui-grid.js"></script>
    <script src="app.js"></script>
  </body>
</html>
```

> 'grid' 아이디를 가진 'div' 객체를 통해 'Tosat ui grid'를 사용하여 그리고자 하는 데이터를 나타낼 수 있다.

## 기본 예제

```javascript
// app.js
const Grid = tui.Grid;

const data = [
  { id: 1, name: "Kong", age: 30, country: "USA" },
  { id: 2, name: "Siru", age: 25, country: "UK" },
  { id: 3, name: "Cheeze", age: 28, country: "Canada" }
];

const columns = [
  { header: "ID", name: "id" },
  { header: "Name", name: "name" },
  { header: "Age", name: "age" },
  { header: "Country", name: "country" }
];

const grid = new Grid({
  el: document.getElementById("grid"),
  data: data,
  columns: columns
});
```

`'el' 과 'columns' 옵션은 필수로 입력이 되어야 한다.`

- `el`: 그리드가 생성될 컨테이너 HTML 엘리먼트이다. 컨테이너 엘리먼트는 Grid 내부에서 자동으로 생성해주지 않기 때문에 인스턴스를 생성하기 전에 미리 만들어줘야 한다.
- `columns` : 데이터의 이름, 헤더, 에디터 등의 컬럼 정보 배열이다.
  - header: 이 속성은 그리드의 열 헤더에 표시되는 텍스트를 지정한다. (사용자에게 보이는 열의 이름)
  - name: 해당 열과 연결된 데이터 필드의 이름을 지정한다. 이는 데이터 객체의 키와 일치해야 한다. 즉, 그 열이 표시할 실제 데이터의 필드를 나타낸다.

`📋 기본예제 출력`

![image](https://github.com/chanhocode/chanhocode.github.io/assets/105937460/02991c87-24b6-40c6-beab-3c4d8c4ac37e)

`🗂️ 참조하면 좋은 자료`

[Toast UI Grid DOC](https://github.com/nhn/tui.grid/tree/master/packages/toast-ui.grid/docs/ko)

[예제 알아보기](https://nhn.github.io/tui.grid/latest/tutorial-example01-basic)
