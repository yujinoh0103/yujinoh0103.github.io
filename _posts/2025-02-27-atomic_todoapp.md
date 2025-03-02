---
layout: post
title: "[스나이퍼팩토리] 한컴AI아카데미 5주차 - react todo app에  atomic design 적용하기"
date: 2025-02-27
categories: [한컴ai]
author: "yujinoh0103"
---
<img src="https://yujinoh0103.github.io/assets/img/todo_atomic.png">

# 리액트 TODO 프로젝트 리팩토링 및 Atomic Design 적용기

## 소개
이 문서는 **React + TypeScript** 기반의 투두(TODO) 프로젝트를 리팩토링한 과정을 정리한 글입니다.  
**Atomic Design** 패턴을 적용해 컴포넌트를 구조적으로 나누고, **hook**과 **utils** 폴더를 분리하여 관리성을 높였습니다.  
또한 **전역 상태**(Context / 로컬스토리지 / useState) 관리, **타입** 정의, **리액트 이벤트 처리**에 관한 내용을 포함합니다.

---

## 목차
1. [Atomic Design으로 컴포넌트 나누기](#atomic-design으로-컴포넌트-나누기)
2. [Hooks와 Utils의 차이](#hooks와-utils의-차이)
3. [전역 상태 관리 & 폴더 구조](#전역-상태-관리--폴더-구조)
4. [예시 코드](#예시-코드)
   - App.tsx
   - CategoryList & CategoryCard
   - TaskListForDate
   - Calendar
5. [인터페이스와 타입 정의](#인터페이스와-타입-정의)
6. [기타 에러 해결 사례 (TS, ESLint)](#기타-에러-해결-사례-ts-eslint)
7. [결론](#결론)

---

## Atomic Design으로 컴포넌트 나누기

### Atomic Design 개념
- **Atoms**: 더 이상 쪼갤 수 없는 UI 기본 요소 (Button, Input 등)
- **Molecules**: 여러 Atoms가 모여 작은 기능 단위를 형성 (예: TaskList, CategoryForm 등)
- **Organisms**: Molecules가 합쳐져 더 큰 단위 (예: CategoryCard, WhenToMeet 등)
- **Templates**: 페이지 레이아웃과 구조를 정의
- **Pages**: 최종적으로 라우팅되는 페이지

### 예시 폴더 구조
```
src/
├── components/
│   ├── atoms/
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── ProgressBar.tsx
│   ├── molecules/
│   │   ├── TaskInput.tsx
│   │   ├── TaskList.tsx
│   ├── organisms/
│   │   ├── CategoryCard.tsx
│   │   ├── CategoryList.tsx
│   │   ├── TaskListForDate.tsx
│   ├── templates/
│   │   ├── CalendarLayout.tsx
│   ├── pages/
│   │   ├── AppPage.tsx
├── hooks/
├── utils/
├── types/
├── App.tsx
└── index.tsx
```
- 각 폴더별 역할이 명확해져서 유지보수와 확장성에 유리합니다.

---

## Hooks와 Utils의 차이

### Hooks(`hooks/`)
- **React 상태(state)를 관리**하기 위한 함수들을 배치.
- `useState`, `useEffect`, `useReducer`, `useContext` 등을 활용해 **React 내부 상태**를 변경.
- 여러 컴포넌트에서 재사용할 가능성이 크거나 **비즈니스 로직**을 캡슐화할 때 사용.

예시) `useCategory.ts`
```tsx
import { useState, useEffect } from "react";
import { Category } from "../types";

export const useCategory = () => {
  const [categories, setCategories] = useState<{ [name: string]: Category }>({});

  useEffect(() => {
    const saved = localStorage.getItem("categories");
    if (saved) setCategories(JSON.parse(saved));
  }, []);

  useEffect(() => {
    localStorage.setItem("categories", JSON.stringify(categories));
  }, [categories]);

  return { categories };
};
```

### Utils(`utils/`)
- **데이터 변환 및 순수 함수(Pure Function)**를 배치.
- React와 무관하며, 상태 변경 없이 입출력 변환만 담당.
- 테스트 및 재사용이 쉬움.

예시) `taskUtils.ts`
```ts
import { Task } from "../types";

export const addTaskToList = (tasks: Task[], text: string, deadline: string | null): Task[] => {
  if (!text.trim()) return tasks;
  return [...tasks, { text, deadline, completed: false }];
};
```

---

## 전역 상태 관리 & 폴더 구조
- Context API (혹은 Redux, Zustand, Recoil 등)를 통해 전역 상태를 관리할 수 있음.
- 소규모 프로젝트에서는 Context API만으로도 충분.

예시) `usePenColor.ts`
```ts
import { useState } from "react";

export const usePenColor = () => {
  const [currentPenColor, setCurrentPenColor] = useState("#000000");
  return { currentPenColor, setCurrentPenColor };
};
```

---

## 인터페이스와 타입 정의

### 타입 정의(`types/`)
```ts
export interface Task {
  text: string;
  deadline: string | null;
  completed: boolean;
}

export interface Category {
  name: string;
  color: string;
  tasks: Task[];
}

export interface TasksByDate {
  [date: string]: { text: string; category: string }[];
}
```

---

## 기타 에러 해결 사례 (TS, ESLint)
### `Cannot find name 'category'. (TS2304)`
변수 선언 없이 `category`를 사용했을 때 발생.
`categories[task.category]`처럼 안전하게 가져와야 함.

### `Type '{ key: string; category: Category; ... }' is not assignable to type 'CategoryCardProps'. (TS2322)`
`CategoryCard`가 기대하는 props와 전달하는 props가 다를 경우 발생.

### `no-unused-vars (ESLint)`
변수를 선언했지만 사용하지 않으면 경고 발생.

---

## 결론
이번 글에서는 React + TypeScript 기반 TODO 프로젝트를 Atomic Design, Hooks/Utils 분리, Context/로컬스토리지 전역 관리, 타입 정의 등을 통해 리팩토링하는 과정을 살펴봤습니다.

- **Atomic Design**으로 컴포넌트를 분할해 유지보수성과 재사용성이 상승.
- **Hook**에서는 React 상태를 관리, **Utils**에서는 순수 데이터 조작을 담당.
- **TypeScript 인터페이스**로 데이터 구조를 명확히 정의해, 런타임 오류를 줄임.
- **ESLint & TypeScript 오류 해결** 과정을 통해 더 안정적인 코드 작성 가능.

감사합니다. ✨

