---
title: "V8 basic"
date: 2026-10-19
author: "맹트농후작부인프랑소와즈도비네 Team"
cover: ""
tags: ["v8", "d8"]
---

> 해당 블로그는 한국정보보호산업협회(KISIA)에서 진행된 교육인 "기업수요기반화이트해커양성과정"에서 진행한 프로젝트를 문서화하기 위해 제작되었습니다.

프로그래밍 언어로 작성된 코드는 CPU가 실행 가능한 형태로 변환 되어야한다.

이 과정에는 코드를 한줄씩 해석하여 실행하는 Interpreter 방식과 전체를 다른 형태의 코드로 변환하는 Compiler 방식이 사용된다.

**V8**은 인터프리터와 컴파일러를 함께 사용하는 **JavaScript Engine**이다.

# V8?

V8은 Google이 개발하는 C++ 기반의 오픈소스 JavaScript/WebAssembly Engine이다.

Chrome과 Node.js 등에서 사용되며, JavaScript 표준인 ECMAScript를 구현한다.

V8은 JavaScript 소스 코드를 컴파일하고 실행한다. JavaScript 객체에 필요한 메모리를 할당하기도 하며, 사용하지 않는 객체는 Garbage Collection을 통해 정리한다.


JavaScript 코드는 가장 먼저 **Ignition Bytecode**로 컴파일된다.

Ignition은 이 Bytecode를 해석해서 실행하고, 실행 과정에서 값의 타입과 객체의 형태 같은 정보를 수집한다.

수집된 정보와 Bytecode는 이후 컴파일러가 기계어 코드를 생성할 때 사용한다.

## Bytecode

Bytecode는 JavaScript 소스 코드와 CPU 기계어 사이에서 사용되는 V8 내부 명령 형식을 말한다.

## Ignition

Ignition은 V8의 Bytecode Interpreter이다.

JavaScript 프로그램은 Ignition Bytecode로 컴파일되며, Ignition이 해당 Bytecode를 실행하게 된다.

## JIT Compilation

JIT(Just-In-Time) Compilation은 프로그램이 실행되는 동안 코드를 기계어로 컴파일 하는 방식을 말한다.

예를 들어 다음과 같은 함수가 반복적으로 실행된다고 해보자.

```JavaScript
function add(a, b) {
    return a + b;
}

for (let i = 0; i < 100000; i++) {
    add(i, 1);
}
```

Ignition은 함수를 실행하면서 입력된 값의 타입과 객체 형태 등의 실행 정보를 수집한다.

```plaintext
add(0, 1)
add(1, 1)
add(2, 1)
add(3, 1)
...
```

이렇게 수집된 정보는 Sparkplug, Maglev, TurboFan과 같은 컴파일러가 기계어 코드를 생성할 때 사용된다.

```plaintext
Ignition
    │
    ├── Sparkplug
    │     └── Baseline Compiler
    │
    ├── Maglev
    │     └── Optimizing Compiler
    │
    └── TurboFan
          └── Optimizing Compiler
```

컴파일러는 다음과 같은 위치에 있으나, 각 컴파일 계층이 항상 순서대로 도는 것은 아니고... V8은 실행 상태와 함수의 동작에 따라 사용할 실행 계층을 결정하게 된다.

# V8과 Chrome

V8은 Chrome이 아니다...

V8은 Chrome을 구성하는 여러 구성요소 중에서 JavaScript와 WebAssembly 실행을 담당하는 엔진이다.

# d8

`d8`은 Engine을 브라우저 없이 터미널에서 돌려보는 개발자용 JavaScript Shell이다.

V8 소스 코드를 수정한 뒤 동작을 확인하거나, JavaScript 코드의 실행 결과를 관찰하고, 디버깅할 때 사용이된다.

브라우저가 아니기에, 일반적인 환경에서는 document, window와 같은 브라우저 DOM 객체가 있지는 않다.

# 다음에는...

V8 취약점 분석에서는 특정 버전의 V8을 빌드하거나, Debug 빌드에서 내부 객체와 실행 상태를 확인하게 된다.

V8을 빌드하면 d8도 함께 생성되는데, 이 과정을 진행해볼 예정이다.