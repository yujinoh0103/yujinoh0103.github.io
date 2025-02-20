---
layout: post
title: "[스나이퍼팩토리] 한컴AI아카데미 3주차 - JS 실습_todo app react로 변환하기기"
date: 2025-02-20
categories: [한컴ai]
author: "yujinoh0103"
---
 
> **목표**: 기존 JavaScript 기반의 React 코드를 TypeScript + React(= .tsx)로 전환하기  
---

## 1. 배경
처음에는 React를 JavaScript로만 개발했습니다.  
**개발자 경험(autocomplete, 타입 추론 등)**을 개선하기 위해 TypeScript로 전환하기로 결정했습니다.

---

## 2. 전환 과정

### 2.1 프로젝트 세팅
1. **`create-react-app --template typescript`**를 이용해 새 프로젝트 생성, 혹은 `tsconfig.json` 구성
2. `npm install --save typescript @types/react @types/react-dom`
3. `tsconfig.json`에서 `"strict": true`로 설정해 **엄격 모드** 활성화

### 2.2 파일 확장자 변경
- 기존 `.js` 파일들을 `.tsx` 또는 `.ts`로 변경
- **JSX를 사용**하는 파일은 `.tsx` 확장자를,  
  로직만 담긴 **유틸** 파일은 `.ts` 확장자를 사용

### 2.3 타입 에러 수정
- `localStorage.getItem(...)` 결과가 `string | null`인 문제
- `window.customProperty` 처럼 전역변수를 썼을 때 오류 발생
  - **글로벌 타입 확장**(declare global)을 통해 해결
- 함수 파라미터 타입(예: `(date: Date) => string`) 미지정으로 인한 `any` 에러
- React 컴포넌트에서는 **`React.FC<Props>`** 활용

### 2.4 ESLint & Prettier 설정
- **`@typescript-eslint/eslint-plugin`**, **`@typescript-eslint/parser`** 설치
- `.eslintrc`에서 `"no-unused-vars"`, `"no-explicit-any"` 규칙 확인
- 수정:
  - **미사용 변수**는 지우거나 실제 사용
  - 가능하면 **`any`** 대신 **구체적 타입** 지정

---

## 3. 주요 이슈와 해결 방법

1. **`window.currentPenColor` 오류**  
   ```bash
   Property 'currentPenColor' does not exist on type 'Window & typeof globalThis'.
   ```

- 원인: TS가 전역 Window 객체에 선언되지 않은 속성에 접근하면 에러
- 해결: `global.d.ts` 파일에 `declare global { interface Window { currentPenColor: string } }`

2. `JSON.parse(...)`에 null 가능성

```bash
Argument of type 'string | null' is not assignable to parameter of type 'string'.
```
- 원인: localStorage.getItem → string | null
- 해결: null 체크 후 JSON.parse 호출
ts
const tasksStr = localStorage.getItem("tasksByDate");
let tasksByDate = tasksStr ? JSON.parse(tasksStr) : {};
매개변수 any 에러

원인: TypeScript 엄격 모드에서는 파라미터에 명시적으로 타입 지정 필요
해결:
ts
복사
편집
function formatLocalDate(date: Date): string {
  // ...
}
미사용 변수/함수 ESLint 경고

원인: 변환 후 사용하지 않는 임시 함수가 남음
해결: 필요 없는 건 삭제, 필요한 건 실제로 호출
1. 성과 및 느낀 점
장점:
자주 틀리던 부분(예: undefined 체크, 오탈자)에서 IDE가 즉시 경고해 줌
협업 시 코드를 이해하는 데 도움이 됨 (자동 완성 & 타입 문서화)
프로덕션 빌드 시 안전성 증가
단점:
초반에 타입 정의가 귀찮고 에러가 많이 남
전역변수 사용이 많은 구조에서는 수정 범위가 커짐
1. 향후 계획
Redux, Recoil 등 타입 친화적인 상태관리 라이브러리 적용
Context API에서 커스텀 훅 적용해 전역 상태를 더욱 엄격히 관리
CI 환경에서 **tsc --noEmit**로 타입 검사 자동화
1. 참고 자료
Official React + TypeScript Cheatsheet
TypeScript Handbook
declare global 사용법 (MDN)
최종적으로 모든 파일이 .tsx/.ts로 정리되었고, 빌드 시 에러도 사라졌습니다.
곧바로 프러덕션 환경에 배포하고, 이후 유지보수가 훨씬 편해졌어요!