---
title: "React nativeEvent (SyntheticEvent와의 차이, 언제 쓰는가)"
categories:
  - React
tags:
  - React
  - Event
last_modified_at:
---

# React nativeEvent

React에서 이벤트를 다루다 보면 `e.nativeEvent`라는 걸 보게 됨. 평소엔 `e`만 써도 됐는데 이게 왜 따로 있는지 정리함


## 1. React의 이벤트는 진짜 이벤트가 아니다(?)

React에서 버튼 클릭 같은 이벤트를 받으면 `e`를 줌.

```jsx
function handleClick(e) {
  console.log(e.target);
}
```

근데 이 `e`는 브라우저가 만든 진짜 이벤트가 아니라, **React가 한 겹 감싸서 만든 가짜 이벤트**임. 이걸 SyntheticEvent(합성 이벤트)라고 부름.


## 2. 왜 React가 감싸는가

브라우저마다 이벤트 동작이 조금씩 다름. React가 한 겹 감싸서 **어느 브라우저든 똑같이 동작하게** 맞춰주는 거임. 그래서 우리는 브라우저 차이를 신경 안 쓰고 `e`만 쓰면 됨.

대부분의 경우 이 SyntheticEvent(`e`)만으로 충분함.


## 3. nativeEvent = 그 안에 든 진짜 이벤트

`e.nativeEvent`는 React가 감싸기 전의 **브라우저 원본 이벤트**임.

```jsx
function handleClick(e) {
  console.log(e);              // SyntheticEvent (React가 감싼 것)
  console.log(e.nativeEvent);  // 브라우저 원본 이벤트
}
```

선물 상자로 치면, `e`는 React가 포장한 상자고 `e.nativeEvent`는 그 안에 든 진짜 알맹이임.


## 4. 언제 nativeEvent를 쓰나

대부분은 안 씀. SyntheticEvent로 충분함. 다만 **React가 안 감싸주는, 브라우저 원본에만 있는 정보가 필요할 때** 씀.

- 한글 입력(IME) 조합 중인지 같은 세밀한 입력 상태가 필요할 때 (`e.nativeEvent.isComposing`)
- 원본 이벤트에만 있는 특정 속성에 접근해야 할 때
- 같은 요소에 React 이벤트랑 직접 단(addEventListener) 이벤트가 섞여서 순서를 따져야 할 때

예를 들어 한글 입력 중에 엔터를 처리할 때 자주 만남.

```jsx
function handleKeyDown(e) {
  if (e.nativeEvent.isComposing) return; // 한글 조합 중이면 무시
  if (e.key === 'Enter') submit();
}
```

이걸 안 하면 한글 입력 도중 엔터가 두 번 먹는 버그가 생기는데, 이 `isComposing`은 원본 이벤트에 있어서 `nativeEvent`로 봐야 함.


## 5. 정리

- React가 주는 `e`는 SyntheticEvent(브라우저 차이를 맞춘 가짜 포장).
- `e.nativeEvent`는 그 안의 브라우저 원본 이벤트.
- 평소엔 `e`로 충분하고, 원본에만 있는 정보(예: 한글 조합 `isComposing`)가 필요할 때만 `nativeEvent`를 씀.
