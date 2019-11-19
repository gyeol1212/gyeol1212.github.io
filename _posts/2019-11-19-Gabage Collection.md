---
published: true
layout: single
title: "Error Boundaries"
comments: true
---

JavaScript의 Gabage Collection에 대해 알아보자

## JS Gabage Collection

### 메모리 관리

메모리의 생존 주기

1. 할당 : 값의 초기화, 함수 호출을 통한 할당 등
2. 사용
3. 해제 : 문제 발생

할당된 메모리가 더 이상 필요 없을 때 해제
<br>
_=> 더 이상 필요 없을때를 알아내기가 어려움_

C와 같은 언어에서는 개발자가 직접 결정하고 해제

JS의 경우 _가비지 콜렉션(Gabage Collection)_ 이라는 자동 메모리 관리법을 사용

---

### Gabage Collection

더 이상 필요하지 않은 메모리를 판단하여 해제하는 자동화 프로그램

GC 알고리즘의 핵심 개념은 _참조_

A라는 메모리를 통해 B에 접근 가능하다면 "B는 A에 참조된다"

#### Mark and Sweep

닿을 수 없는 오브젝트 = 더 이상 필요없는 오브젝트

roots 오브젝트 집합( 전역 변수 )으로 부터 참조하는 오브젝트를 단계적으로 참조하여 '닿을 수 있는 오브젝트'로 판단하고, 닿을 수 없는 오브젝트에 대해 GC를 실행.

순환 참조 문제 해결

---

### JavaScript의 Gabage Collection

흔한 자바스크립트의 메모리 누수

1. 전역 변수

2. DOM 요소 이중 참조

3. Closure
