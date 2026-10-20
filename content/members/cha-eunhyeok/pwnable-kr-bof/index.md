---
title: "[pwnable.kr] bof — 버퍼 오버플로"
date: 2026-10-21
author: "차은혁"
cover: "images/image.png"
tags: ["pwnable", "ctf", "stack", "buffer-overflow"]
---

![alt text](images/image.png)

## 문제의 목표

![](images/Pasted%20image%2020261021041444.png)

`bof`는 입력을 길게 넣어 스택에 있는 `key` 값을 바꾸는 문제다. 목표는 `key`에 `0xcafebabe`를 넣어 셸을 실행하는 조건을 만족시키는 것이다.

핵심은 입력 버퍼의 경계를 넘어 다른 스택 영역까지 데이터를 쓸 수 있다는 점이다.

## 취약점 분석

문제에서 확인한 bof.c의 핵심 코드는 다음과 같다.

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
	char overflowme[32];
	printf("overflow me : ");
	gets(overflowme);	// smash me!
	if(key == 0xcafebabe){
		setregid(getegid(), getegid());
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}

```

`overflowme`는 32바이트 크기의 지역 배열이다. `gets()`는 입력을 저장할 때 입력의 길이를 확인하지 않는다. 따라서 긴 입력이 들어오면 배열의 경계를 넘어서도 계속 메모리에 쓴다. 또한 입력 끝에는 null 바이트를 추가한다.

이 문제에서는 그 성질을 이용해 `key`를 검사하는 조건을 버퍼 이상으로 밀어넣어 만족시키면 된다.

```c
if (key == 0xcafebabe)
```
0xdeadbeef를 0xcafebabe로

## 스택에서 입력이 덮어써지는 과정

버퍼와 스택은 별개의 저장 공간이 아니다. 이 함수의 입력 버퍼는 스택 안에 배치되어 있고, 그 뒤의 높은 주소 쪽에는 다른 값들이 있다.

아래 그림은 버퍼에서 `key`까지의 관계를 설명하기 위한 개념도다. 중간의 세부 구조는 생략했다.

```text
낮은 주소
          ┌────────────────────────────┐
입력 시작  │ overflowme[32]             │
    ↓     │ 입력을 저장하는 버퍼          │
          ├────────────────────────────┤
    ↓     │ 중간 스택 영역               │
          │ 다른 값과 정렬 공간 등        │
          ├────────────────────────────┤
    ↓     │ key (4바이트)               │
          └────────────────────────────┘
높은 주소
```

입력이 길어지면 위 그림의 아래 방향, 즉 높은 주소 방향으로 연속해서 기록된다. 스택이 확장되는 방향과 입력 바이트가 기록되는 방향은 구분해야 한다.

`A`의 바이트 값은 `0x41`이다. `key` 위치까지 `A`를 채우면 기존 값이 `0x41414141`로 바뀔 수 있다. 목표는 그 위치에 `A` 대신 `0xcafebabe`의 바이트를 넣는 것이다.

![](images/Pasted%20image%2020261021041221.png)
checksec을 통한 확인 카나리도 있는게 확인 되었다.
## GDB로 확인하기

우선 바이너리를 GDB로 열고 함수의 어셈블리 코드를 확인한다.

```bash
gdb ./bof
```

```gdb
disas func
```

![](images/Pasted%20image%2020261021041706.png)

입력이 저장된 상태의 스택을 보기 위해 `gets` 호출 바로 다음 명령어에 브레이크포인트를 설정한다. 아래 `주소`는 `disas func` 출력에서 확인한 실제 명령어 주소로 바꿔야 한다.

```gdb
b* func+55
run
```

입력을 기다리면 `AAAA`를 넣는다. `gets`가 반환한 뒤 브레이크포인트에서 멈추면 다음 명령어로 스택 메모리를 확인한다.

```gdb
x/50wx $esp
```

![](images/Pasted%20image%2020261021042528.png)
AAAA를 20개 넣은 결과

- `x`: 메모리 확인
- `50`: 50개 단위 출력
- `w`: 한 단위를 4바이트로 지정
- `x`: 16진수로 출력
- `$esp`: 현재 스택 포인터 주소

즉, ESP부터 총 200바이트를 확인하는 명령어다. 출력에서 `0x41414141`을 찾으면 입력한 `AAAA`가 저장된 위치를 확인할 수 있다.

다만 해당 값만으로 버퍼 시작 위치를 단정하지 않고, 입력 버퍼를 넘기는 명령어와 함께 대조한다. 짧은 입력을 사용했을 때 버퍼의 나머지 공간은 모두 초기화되는 것이 아니다.

## key까지의 오프셋 계산

필요한 길이는 버퍼 크기 자체가 아니라 **버퍼 시작 주소에서 key 시작 주소까지의 거리**다. 중간에 다른 스택 값과 정렬 공간이 있으므로 버퍼가 32바이트라고 해서 32바이트 바로 다음에 `key`가 있다고 볼 수는 없다.

예를 들어 실제 디스어셈블리에서 다음 배치를 확인했다면:

```text
입력 버퍼 시작: EBP - 0x2c
key 위치:      EBP + 0x08
```

두 주소의 차이는 다음과 같다.

```text
(EBP + 0x08) - (EBP - 0x2c)
= 0x34
= 52바이트
```

이 배치에서는 52바이트를 먼저 채우면 다음 바이트부터 `key` 영역에 도달한다.

```text
버퍼 시작                                 key 시작
│                                         │
▼                                         ▼
[ 버퍼 32바이트 ][ 중간 영역 20바이트 ][ key 4바이트 ]
[              A × 52                ][ 목표 값     ]
```

## 페이로드 구성

오프셋이 52바이트로 확인되었다면 페이로드는 다음과 같이 구성할 수 있다.

![](images/Pasted%20image%2020261021042911.png)

# 실행 결과

Daddy_I_just_pwned_a_buff3r!
![](images/Pasted%20image%2020261021044702.png)

## 배운 점

이번 풀이에서 중요한 것은 스택에 있는 값의 위치 관계를 이해하는 것이다. `x/50wx $esp`로 입력이 저장된 모습을 확인하고, 어셈블리에서 버퍼와 `key`의 주소를 찾아 두 위치 사이의 거리를 계산할 수 있다.

버퍼의 크기와 목표 변수까지의 거리는 서로 다를 수 있다. 그 차이를 확인한 뒤 채우기 데이터와 목표 값을 결합하면, 입력으로 조건 검사에 쓰이는 값을 바꿀 수 있다.

## 참고

- [pwnable.kr](https://pwnable.kr/)
- [GDB 공식 문서 — 메모리 확인](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Memory.html)
- [GDB 공식 문서 — 실행과 단계 실행](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Continuing-and-Stepping.html)

