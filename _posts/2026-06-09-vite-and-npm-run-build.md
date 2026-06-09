---
title: "Vite와 npm run build (React 코드가 dist 폴더로 빌드되는 과정)"
categories:
  - React
tags:
  - React
  - Vite
  - Frontend
last_modified_at:
---

# Vite와 npm run build

React 코드를 짜면 그대로 배포되는 줄 알았는데, `npm run build`라는 걸 한 번 거쳐야 한다고 함. 이게 뭘 하는 건지 정리함


## 1. 내가 짠 React 코드는 브라우저가 못 읽는다

React로 코드를 짜면 `.jsx` 파일에 이런 걸 씀.

```jsx
function App() {
  return <h1>안녕</h1>;
}
```

근데 브라우저는 `.jsx`도 모르고, `import` 같은 최신 문법도 그대로는 잘 못 읽음. 브라우저가 아는 건 결국 순수 HTML/CSS/JS뿐임.

그래서 내가 짠 코드를 **브라우저가 읽을 수 있는 형태로 번역**해줘야 함. 이 번역 작업이 빌드(build)임.


## 2. Vite가 그 번역을 해준다

Vite(비트)는 React 프로젝트를 빌드해주는 도구임. 두 가지 일을 함.

- **개발할 때**: `npm run dev` — 코드 고치면 화면에 바로 반영해주는 개발 서버를 띄워줌.
- **배포할 때**: `npm run build` — 내 코드를 브라우저용으로 번역해서 결과물을 만들어줌.

지금 정리할 건 배포용인 `npm run build`임.


## 3. npm run build를 하면 dist 폴더가 생긴다

`npm run build`를 실행하면 Vite가 이런 일을 함.

```
내 React 코드(.jsx, import 여러 개, CSS 등)
   → Vite가 번역 + 합치기 + 압축
   → dist 폴더에 결과물 생성
```

`dist` 폴더 안에는 이런 게 들어 있음.

```
dist/
 ├─ index.html      ← 시작 파일
 ├─ assets/
 │   ├─ index-abc123.js   ← 내 코드 전부 합쳐 압축한 JS
 │   └─ index-abc123.css  ← 스타일 합쳐 압축한 CSS
```

수십 개로 나뉘어 있던 내 코드가, 브라우저가 바로 읽을 수 있는 몇 개 파일로 합쳐지고 줄어듦. 파일 이름에 붙은 `abc123` 같은 건 캐시 때문에 붙는 거임(내용 바뀌면 이름도 바뀜).


## 4. 그 다음엔?

이 `dist` 폴더가 곧 정적 파일 덩어리임. 이걸 nginx한테 주면 사용자한테 화면을 띄워줄 수 있음.

```
npm run build  →  dist 폴더  →  nginx가 서빙  →  사용자 화면
```


## 5. 정리

- 내가 짠 React 코드는 브라우저가 바로 못 읽음.
- Vite가 `npm run build`로 그걸 브라우저용으로 번역·압축해서 `dist` 폴더에 모아줌.
- `dist`는 HTML/CSS/JS 정적 파일이라, 이걸 nginx가 서빙하면 배포 끝.

> 한마디로 빌드는 "내 코드를 브라우저가 알아듣는 말로 번역해서 한 폴더에 담는 일"임.
