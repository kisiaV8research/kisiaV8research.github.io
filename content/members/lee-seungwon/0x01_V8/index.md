---
title: "CVE-2020-6507 Part1"
date: 2026-10-20
author: "이승원"
cover: ""
tags: ["1-day", "CVE-2020-6507"]
---

> 스스로 공부하면서 다시 보기 위해 개인적으로 정리한 글이며, 틀린 부분이 존재할 수 있습니다.

환경 세팅 전, 미리 준비해둔 환경은 다음과 같다.

![alt text](images/image.png)

![alt text](images/image-1.png)

ubuntu의 경우 설치 후 아무런 설정도 하지 않은 상태이다.

# 1. 도구 설치

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install -y \
  open-vm-tools open-vm-tools-desktop \
  linux-headers-$(uname -r) \
  build-essential git curl wget ca-certificates pkg-config \
  cmake ninja-build clang lld llvm gdb gdbserver \
  python3 python3-venv python3-dev python3-pip \
  file lsb-release unzip zip xz-utils zstd jq ripgrep rsync \
  libssl-dev libffi-dev \
  binutils patchelf strace ltrace nasm ccache \
  tmux hto checksec libc6-dbg
```

설치한 도구들의 역할은 다음과 같다.

| 도구 | 역할 |
| --- | --- |
| open-vm-tools | VMware 게스트 서비스 |
| clang, lld, llvm | V8 컴파일러와 LLVM 도구 |
| gdb, gdbserver | 네이티브 디버깅 |
| gn | Ninja 빌드 설정 생성 |
| ninja, autoninja | 실제 컴파일 |
| d8 | V8 JavaScript 셸 |
| rg | V8 소스 검색 |
| strace, ltrace | 시스템·라이브러리 호출 추적 |
| readelf, objdump, nm | ELF 구조와 심볼 분석 |
| checksec | PIE, NX, RELRO, Canary 확인 |
| pwntools | PoC 자동화와 바이트·주소 처리 |

gdb의 사용성을 강화하기 위해 pwndbg 플러그인도 설치해주었다.

```bash
cd ~
git clone https://github.com/pwndbg/pwndbg
cd ~/pwndbg
./setup.sh
```

디버거 사용 방법이나, 분석과정에서 등장하는 개념들은 나올 때 마다 정리하고,
추가적으로 상세한 문서 기록이 필요하다는 생각이 들면 link 처리하여 문서화 시킬 예정이다.

# 2. CVE-2020-6507 분석 환경 구축

디렉터리 구조는 다음과 같이 둘 것이다.

```
/home/ubuntu/security-lab/cve/CVE-2020-6507/
├── workspace/
│   ├── .gclient
│   └── v8/
│       └── out/
│           ├── vuln-asan/
│           ├── vuln-debug/
│           ├── fixed-asan/
│           └── fixed-debug/
├── poc/
├── notes/
├── logs/
├── scripts/
└── artifacts/
```

생성해주자.

```bash
mkdir -p /home/ubuntu/security-lab/cve/CVE-2020-6507/workspace
mkdir -p /home/ubuntu/security-lab/cve/CVE-2020-6507/poc
mkdir -p /home/ubuntu/security-lab/cve/CVE-2020-6507/notes
mkdir -p /home/ubuntu/security-lab/cve/CVE-2020-6507/logs
mkdir -p /home/ubuntu/security-lab/cve/CVE-2020-6507/scripts
mkdir -p /home/ubuntu/security-lab/cve/CVE-2020-6507/artifacts
```

다운로드량을 줄이기 위해, fetch v8를 사용하지 않고 진행하기로 했다.

기존 V8 저장소를 참고, `--dissociate`로 기존 저장소와 독립된 복제본을 만든다.

### 이렇게 하는 이유...?

여기서 저장소(Repository)는 git 저장소를 이야기하는데, V8 History는 커밋이 수심만개여서 클론 한번 하면 수 GB가 오고간다...

CVE 하나당 한 번씩 새로 받으면 디스크랑 네트워크가 남아나지를 않기 때문에, 이미 받아둔 것을 재활용하는 것이다.

## fetch v8

`fetch`는 depot_tools에 들어 있는 명령으로, V8/Chromium 체크 아웃을 처음부터 통째로 만들어주는 역할을 한다. 내부적으로는 2개의 단계를 수행한다.

1. `.gclient` 설정 파일 생성 (solution 이름 `v8`, URL은 공식 V8 git)
2. `gclient sync` 실행 - V8 본체를 클론하고, `DEPS` 파일을 읽어 의존 저장소들(build, buildtools, ICU, zlib, googletest, jinja2 등 수십 개)을 전부 내려받음

편하긴한데, 1단계를 사용자가 손댈 수 없다는게 문제이다.

클론 방식을 지정할 수 없어서 기존 저장소를 참조하지 못하고, 매번 전체를 네트워크로 새로 받게 된다.

그리고 기본이 managed 모드라 `gclient sync`가 V8 본체의 리비전까지 관리하려고 하기 때문에, 여기서는 `fetch v8`을 하지 않고 두 단계를 나눠서 진행한다.

##  gclient setttings

1단계이다. 

```bash
cd /home/ubuntu/security-lab/cve/CVE-2020-6507/workspace

/home/ubuntu/depot_tools/gclient config --unmanaged \
  https://chromium.googlesource.com/v8/v8.git
```

실행하면 `.gclient`라는 텍스트 파일 하나가 생긴다.

![alt text](images/image-2.png)

내용은 다음과 같다.

```bash
ubuntu@ubuntu-VMware-Virtual-Platform:~/security-lab/cve/CVE-2020-6507/workspace$ cat .gclient
solutions = [
  { "name"        : 'v8',
    "url"         : 'https://chromium.googlesource.com/v8/v8.git',
    "deps_file"   : 'DEPS',
    "managed"     : False,
    "custom_deps" : {
    },
    "custom_vars": {},
  },
]
```

이 파일의 의미는 "이 디렉터리가 체크아웃의 루트이고, 최상위 저장소(solution)는 `v8`이라는 하위 디렉터리이다" 라는 의미를 가진다.

나중에 `gclient sync`를 돌리면 해당 파일을 찾아 읽고, `v8/DEPS`에 적힌 의존성들을 그 리버전에 맞춰 가져오게 된다.

명령을 수행할 때 `--unmanaged` 옵션을 이용했는데, 이게 중요하다.

### --unmanaged

managed 모드이면, `gclient sync`가 V8 본체를 자기가 아는 리비전(보통 최신 버전)으로 되돌려버릴 수 있다.

1-day를 분석하는 입장에서는 취약한 커밋 환경을 고정해두고 연구를 진행해야하기 때문에, 본체의 체크아웃 위치는 사용자가 직접 git checkout으로 정하고 의존성만 맞추는 형태인 `--unmanaged`를 붙여준 것이다.

## V8 저장소 복제

2단계를 진행하기 위해 V8 저장소를 클론 해오자.

```bash
cd /home/ubuntu/security-lab/cve/CVE-2020-6507/workspace

git clone \
  --reference /home/ubuntu/security-lab/v8/v8 \
  --dissociate \
  https://chromium.googlesource.com/v8/v8.git \
  v8
```

여기서 붙은 옵션들은 다음과 같다.

- `--reference`: 기존 V8 저장소의 Git 객체를 복제 과정에서 활용
- `--dissociate`: 복제가 끝나면 기존 저장소 의존성을 제거

이렇게 하면, /home/ubuntu/security-lab/v8/v8을 수정하지 않는 별도 저장소가 만들어진다. (둘이 거의 세트로 사용한다)

`.gclient`는 체크아웃 최상위 설정이고, solution 이름에 따라 v8 디렉터리를 관리하게 된다.

## CVE 분석에 사용할 Revision 받기

```bash
cd /home/ubuntu/security-lab/cve/CVE-2020-6507/workspace/v8

git fetch origin main

git fetch origin \
  refs/branch-heads/8.3:refs/remotes/origin/branch-heads/8.3
```

분석에서 구분할 주요 리비전은 다음과 같다.

| 용도 | V8 커밋 |
|---|---|
| 취약 Chrome 83 릴리스 | 3b627511511f00c552ced504c1f182bdcc3480af |
| 수정 Chrome 83 릴리스 | 90904eb48b16b32f7edbf1f8a92ece561d05e738 |
| 핵심 패치 직전 | c6c0090532f08a09146f958865f8027c44bce020 |
| 핵심 수정 패치 | b8883649d08c758d5ba729078b1f747d21faa582 |

Chrome 83.0.4103.106에서 CVE-2020-6507 수정이 배포되었니 참고하자.

(https://chromereleases.googleblog.com/2020/06/stable-channel-update-for-desktop_15.html)

## gclient sync

sync 하기 전, 취약 버전을 체크아웃 해주자.

```bash
cd /home/ubuntu/security-lab/cve/CVE-2020-6507/workspace/v8

git switch --detach \
  3b627511511f00c552ced504c1f182bdcc3480af
```

여기서 `--detach` 옵션은 특정 커밋을 정확하게 체크아웃 할 때 사용하는 옵션이며, 별도의 연구용 브랜치를 만들지 않고 리비전 자체를 고정하는 방식이다.

이제 `gclient sync`를 실행해주자.

```bash
/home/ubuntu/depot_tools/gclient sync
```

gclient sync는 해당 V8 리비전의 DEPS에 기록된 빌드 도구와 하위 의존성을 맞춰준다.

- DEPS(dependencies): 특정 버전의 V8을 빌드하기 위한 의존성 목록으로, Python 문법으로 쓰여있다.

## 취약 버전 ASan 빌드

ASan(AddressSanitizer)는 컴파일러(clang/gcc)에 내장된 **메모리 오류 탐지기**이다.

이 녀석이 이용하는 핵심 아이디어는 2가지이다.

1. Shadow memory
프로그램 메모리 1바이트마다 접근해도 괜찮은지를 기록하는 별도의 맵을 둔다.

실제로는 8바이트당 1바이트 비율로 압축해서 관리하며, 모든 load/store 직전에 해당 맵을 조회한다.

2. Redzone
할당된 모든 메모리 블록 앞뒤에 독이 칠해진(poisoned) 완충 구역을 끼워 넣는다.
배열 밖으로 한칸만 넘어가도 레드존을 밟아 트리거 되는 형태이다.

해제된 메모리도 바로 재사용하지 않고 격리(quarantine) 큐에 넣어 독을 칠해둔다. 때문에 use-after-free도 잡아낼 수 있다.


