---
title: "bof"
date: 2026-10-31
author: "임수희"
cover: ""
tags: ["bof"]
---
# bof

## **01. Netcat으로 원격 프로그램 동작 확인**

프로그램은 `overflow me :` 메시지로 문자열을 받고, 목표 값이 아닌 상태라 `Nah..`를 출력.

![image.png](image\image.png)

## **02. 분석 파일과 소스 코드 확인**

로컬 디렉터리에서 `bof` 실행 파일, `bof.c` 소스 코드, Docker 관련 파일과 `flag`를 확인. `bof.c`를 읽어 **취약한 입력 함수 `gets()`**, **버퍼 크기 32바이트**, **초깃값 `0xdeadbeef`**, **목표 값 `0xcafebabe`**를 확인. 로컬 `flag`는 27바이트였고 이후 원격에서 본 `flag`는 29바이트여서 두 파일을 같은 정답으로 간주하지 않음.

![image.png](image\image%201.png)

![image.png](image\image%202.png)

![image.png](image\image%203.png)

## **03. 실행 파일 형식과 로컬 실행 확인**

`file` 출력에서 **ELF 32-bit, Intel i386, Little Endian(LSB), PIE, not stripped**임을 확인했다. 32비트 x86의 스택 프레임과 4바이트 정수 저장 방식을 분석하는 근거가 됨. `./bof`에서 짧은 문자열을 입력해도 `Nah..`가 출력되어 원격과 동일한 기본 흐름을 재현함.

![image.png](image\image%204.png)

![image.png](image\image%205.png)

## **04. `func` 역어셈블과 변수 위치 추정**

![image.png](image\image%206.png)

![image.png](image\image%207.png)

![image.png](image\image%208.png)

## **05. `gets()` 이후에 중단점 설정**

`func+63`은 `cmp [ebp+8],0xcafebabe` 명령어가 있는 위치. 따라서 `gets()`로 입력한 직후이면서 `key`를 비교하기 **직전**에 중단. 입력값 `hi`를 준 다음 GDB/Pwndbg의 레지스터·어셈블리·스택 표시를 확인.

![*비교 명령 직전에 중단점 설정 및 실행*](image\image%209.png)

*비교 명령 직전에 중단점 설정 및 실행*

![*첫 중단점 도착: 레지스터와 명령어*](image\image%2010.png)

*첫 중단점 도착: 레지스터와 명령어*

![*첫 중단점 도착: 스택과 호출 정보*](image\image%2011.png)

*첫 중단점 도착: 스택과 호출 정보*

## **06. 실제 메모리 주소와 입력값 확인**

| 항목 | 실제 값 | 설명 |
| --- | --- | --- |
| `EBP` | `0xffffcef8` | 해당 실행 시 함수 스택 기준점 |
| `ESP` | `0xffffcec0` | 현재 스택 맨 위 주소 |
| 버퍼 시작 | `0xffffcecc` | `EBP - 0x2c` |
| `key` 주소 | `0xffffcf00` | `EBP + 0x8` |
| `key` 값 | `0xdeadbeef` | 아직 변경 전 |

버퍼 처음 두 바이트 `0x68 0x69`는 ASCII 문자 `h`, `i`. 이후 `0x00`은 C 문자열의 종료 바이트. 배열의 나머지 데이터가 모두 0으로 초기화되어 있다는 뜻은 아님.

![*EBP와 ESP 확인*](image\image%2012.png)

*EBP와 ESP 확인*

![*key 원래 값 확인*](image\image%2013.png)

*key 원래 값 확인*

![*입력 버퍼의 바이트 확인*](image\image%2014.png)

*입력 버퍼의 바이트 확인*

## **07. 스택 56바이트를 한 번에 확인**

4바이트(`word`) 단위로 14개, 즉 56바이트를 출력했다. 중요한 구간은 다음과 같음.

```
EBP+0x08 : key          (0xdeadbeef)
EBP+0x04 : 복귀 주소
EBP+0x00 : 저장된 이전 EBP
EBP-0x04 : 저장된 ESI
EBP-0x08 : 저장된 EBX
EBP-0x0c : Stack Canary (0x32de9300)
EBP-0x2c : overflowme[32] 시작
```

버퍼 시작부터 `key` 시작까지의 차이는 `(EBP+8) - (EBP-44) = 52바이트`. 이 **52바이트**가 공격 데이터의 패딩 길이(오프셋)임.

![*버퍼부터 key까지 연속 메모리 확인*](image\image%2015.png)

*버퍼부터 key까지 연속 메모리 확인*

## **08. A 36개로 카나리 덮어쓰기 실험**

버퍼가 32바이트이므로, `A` 36개를 입력해 **배열 32바이트 + 바로 뒤 카나리 4바이트**를 덮어쓰는지 시험.

```
run
# overflow me : 에 A 36개 입력
x/wx $ebp-0xc
x/wx $ebp+8
```

카나리는 기존 `0x32de9300`에서 `0x41414141`로 바뀜. ASCII `A`의 바이트 값이 `0x41`이기 때문. 반면 52바이트 떨어진 `key`는 여전히 `0xdeadbeef`임. `gets()`는 끝에 `\0`도 기록하므로, 엄밀히는 A 36개 외에 종료 바이트의 초과 기록도 발생.

![*A 36개 입력 후 중단점 도착*](image\image%2016.png)

*A 36개 입력 후 중단점 도착*

![*카나리 변조 확인*](image\image%2017.png)

*카나리 변조 확인*

![*36바이트 입력 후 key 유지*](image\image%2018.png)

*36바이트 입력 후 key 유지*

## **09. Stack Canary 보호 동작 확인**

프로그램은 목표 값에 도달하지 못해 `Nah..`를 출력했고, 함수 종료 시 카나리 검사에서 변조가 감지되어 `*** stack smashing detected ***`와 함께 `SIGABRT`로 종료. **카나리는 `gets()` 직후가 아닌 함수가 끝날 때 검사된다**는 사실을 확인. 이 문제에서는 비교문과 `system()` 호출이 종료 검사보다 앞에 있다는 점이 중요.

![*Stack smashing detected*](image\image%2019.png)

*Stack smashing detected*

## **10. 52바이트 패딩과 목표 값으로 `key` 변경**

목표 값은 32비트 정수 `0xcafebabe`. 리틀 엔디언에서는 메모리에 `be ba fe ca` 순서로 기록.

```python
payload = b"A" * 52 + p32(0xcafebabe)
```

처음 52바이트는 `key` 위치까지 도달하기 위한 패딩, 다음 4바이트는 `key`를 덮어쓰는 값. 로컬 검증에서는 `payload.bin`을 만들고 `run < payload.bin`으로 실행한 다음 아래 명령어로 확인.

```
x/wx $ebp+8
# 0xcafebabe
```

이로써 **소스 코드 변경 없이 사용자 입력만으로 함수 인자의 값을 변조**.

![image.png](image\image%2020.png)

![image.png](image\image%2021.png)

![image.png](image%2022.png)

![image.png](image\image%2023.png)

![*목표 key 값으로 변경 확인*](image\image%2024.png)

*목표 key 값으로 변경 확인*

## **11. `cmp`, `ZF`, `jne`로 조건문 통과 검증**

중단점이 `cmp`에 있는 상태에서 `ni`로 한 명령씩 진행.

```
ni
info registers eflags
ni
x/i $eip
```

`0xcafebabe`와 `0xcafebabe`의 비교 결과가 같으므로 **ZF(Zero Flag)=1**이 되었고, `jne`(서로 다르면 점프)는 분기하지 않음. 다음 명령어가 `call getegid@plt`로 나타난 것은 `if (key == 0xcafebabe)` 블록에 진입했다는 뜻. Pwndbg의 백트레이스에 보인 `0x41414141`은 패딩이 저장된 EBP/복귀 주소까지 덮어썼음을 보여줌.

![*cmp 실행 후 jne 위치 도착*](image\image%2025.png)

*cmp 실행 후 jne 위치 도착*

![*cmp 실행 시 손상된 스택 및 호출 추적*](image\image%2026.png)

*cmp 실행 시 손상된 스택 및 호출 추적*

![*EFLAGS에서 ZF 확인*](image\image%2027.png)

*EFLAGS에서 ZF 확인*

![*jne 미분기 후 if 블록 진입*](image\image%2028.png)

*jne 미분기 후 if 블록 진입*

![*if 블록 진입 시 스택 표시*](image\image%2029.png)

*if 블록 진입 시 스택 표시*

![*EIP가 getegid 호출을 가리킴*](image\image%2030.png)

*EIP가 getegid 호출을 가리킴*

## **12. `/bin/sh` 호출 직전까지 도달**

`func+107`은 `call system@plt` 명령어의 위치. 중단점에 도달한 뒤 EAX가 가리키는 문자열을 읽으면 `"/bin/sh"`가 확인. 즉 **`system("/bin/sh")`가 호출되기 직전까지 실행 흐름을 바꾼 것**. 이 시점에는 호출 직전이므로 '이미 셸이 실행됐다'고 표현하면 안됨.

![*system 호출 직전 중단점 설정*](image\image%2031.png)

*system 호출 직전 중단점 설정*

![*system 호출 직전에 도착*](image\image%2032.png)

*system 호출 직전에 도착*

![*system 인자가 준비된 스택*](image\image%2033.png)

*system 인자가 준비된 스택*

![*EAX 문자열 /bin/sh 확인*](image\image%2034.png)

*EAX 문자열 /bin/sh 확인*

## **13. GDB에서 셸 실행 및 종료 메시지 관찰**

GDB 출력에서 `/usr/bin/dash` 실행이 보임. Ubuntu의 `/bin/sh`가 `dash`를 가리키는 환경이므로 **셸 프로세스 실행이 실제로 이루어졌다는 증거**. 이어진 `No symbol "func" in current context` 메시지는 새 실행 파일에 기존 `bof`의 `func` 심볼이 없어서 중단점을 재설정하지 못했다는 뜻. 마지막의 `stack smashing detected`는 함수 종료 시 카나리 검사가 실패한 결과임. `payload.bin`을 입력으로 주면 셸이 추가 입력 없이 종료될 수 있음.

![*GDB continue: dash 실행과 심볼 오류*](image\image%2035.png)

*GDB continue: dash 실행과 심볼 오류*

![*GDB continue: exec 이후 오류 및 종료*](image\image%2036.png)

*GDB continue: exec 이후 오류 및 종료*

## **14. 원격 서버에 페이로드 전송하여 셸 획득**

앞의 `cat payload.bin`은 공격 데이터를 보내고, 뒤의 `cat`은 키보드로 입력한 후속 명령어를 연결에 전달. 원격에서 `id`를 실행하자 아래와 같은 응답.

![*Netcat 파이프로 원격 id 실행*](image\image%2037.png)

*Netcat 파이프로 원격 id 실행*

## **15. 원격 디렉터리와 Flag 파일 확인**

작업 디렉터리는 `/home/bof_pwn`이고, 원격 `flag` 파일은 `root:bof_pwn` 소유 및 그룹으로 표시. 현재 셸이 `bof_pwn` 그룹에 속하므로 **그룹 읽기 권한으로 접근 가능한 상황**. 원격 파일 크기는 29바이트로 확인.

![*원격 경로와 파일 권한 확인*](image\image%2038.png)

*원격 경로와 파일 권한 확인*

## **16. Flag 출력이 지연되어 여러 입력 방법 시험(문제의 시작…)**

한 차례 `cat flag` 이후 출력과 다음 프롬프트가 바로 나타나지 않았고, `echo START; cat /home/bof_pwn/flag; echo END`를 입력해도 즉시 응답하지 않음. 이후 `nc` 파이프에서 명령어를 자동 전달하는 방법을 시도했으나 Flag 내용은 보이지 않음. 

![image.png](image\image%2039.png)

![image.png](image\image%2040.png)

![image.png](image\image%2041.png)

![image.png](image\image%2042.png)

![image.png](image\image%2043.png)

![image.png](image\image%2044.png)

## **17. 최종 결과**

![image.png](image\image%2045.png)

![image.png](image\image%2046.png)

![image.png](image\image%2047.png)