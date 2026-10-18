# Research Notes — 팀 연구 블로그

네 명이 함께 쓰는 공부·연구 기록 블로그입니다.
[Hugo](https://gohugo.io/)로 만들고 **GitHub Pages**로 배포합니다. 글은 전부 마크다운(`.md`)으로 작성합니다.

- **팀 공식 자료** → `content/posts/` 에 올립니다.
- **개인 공부 기록** → `content/members/본인폴더/` 에 올립니다.

---

## 1. 처음 한 번만 하는 세팅 (대표 1명)

### 1) GitHub Organization과 저장소 만들기
1. GitHub에서 팀 Organization을 만듭니다. (예: `my-research-team`)
2. Organization 안에 **`<조직이름>.github.io`** 라는 이름의 저장소를 만듭니다.
   - 예: 조직이 `my-research-team` 이면 저장소 이름은 `my-research-team.github.io`
   - 이 이름이어야 주소가 `https://my-research-team.github.io/` 로 깔끔하게 나옵니다.
3. 팀원 3명을 이 Organization의 멤버로 초대합니다. (Settings → People)

### 2) 이 폴더를 저장소에 올리기
이 폴더(`TEAM_Blog`)에서 아래를 실행하세요. `<조직이름>` 부분만 바꾸면 됩니다.

```bash
git init
git add .
git commit -m "블로그 초기 세팅"
git branch -M main
git remote add origin https://github.com/<조직이름>/<조직이름>.github.io.git
git push -u origin main
```

### 3) GitHub Pages 켜기
저장소 페이지에서 **Settings → Pages → Build and deployment → Source** 를 **GitHub Actions** 로 바꿉니다.
그러면 push할 때마다 자동으로 빌드·배포됩니다. (왼쪽 "Actions" 탭에서 진행 상황 확인)

### 4) 블로그 정보 바꾸기
`hugo.toml` 파일을 열어 위쪽 ❶ ❷ 두 줄만 바꾸면 됩니다.
- `baseURL` → `https://<조직이름>.github.io/`
- `title` → 블로그 제목

> 멤버 폴더 이름/이름도 바꾸고 싶다면 `content/members/` 아래 `member1`~`member4` 폴더명과
> 각 `_index.md`의 `title`, `role` 을 본인에 맞게 수정하세요.

---

## 2. 평소 글 쓰는 방법

글쓰기는 **마크다운 파일 하나 추가**가 전부입니다. 두 가지 방법 중 편한 걸 쓰세요.

### 방법 A. GitHub 웹에서 바로 쓰기 (설치 필요 없음)
1. 저장소에서 글을 둘 폴더로 이동
   - 팀 글: `content/posts/`
   - 내 글: `content/members/본인폴더/`
2. **Add file → Create new file** 클릭, 파일명을 `제목.md` 로 입력
3. 아래 형식으로 작성 후 **Commit** → 몇 분 뒤 자동 반영

```markdown
---
title: "글 제목"
date: 2026-10-18
author: "내 이름"
description: "한 줄 요약 (목록/검색에 표시됩니다)"
tags: ["태그1", "태그2"]
---

여기에 마크다운으로 본문을 씁니다.
```

> 맨 위 `---` 사이 부분을 **front matter**라고 합니다. 제목·날짜·태그 같은 글 정보입니다.
> `draft: true` 를 넣으면 공개되지 않는 초안으로 저장됩니다. 완성되면 그 줄을 지우세요.

### 방법 B. 내 컴퓨터에서 쓰기 (미리보기 가능)
Hugo가 설치돼 있으면(아래 3번) 명령어로 새 글 틀을 만들 수 있습니다.

```bash
# 팀 글
hugo new posts/내-글-제목.md

# 내 개인 글
hugo new members/member1/내-글-제목.md
```

작성 후 저장소에 push하면 배포됩니다.

```bash
git add .
git commit -m "새 글: 제목"
git push
```

---

## 3. 로컬 미리보기 (선택)

글을 올리기 전에 내 화면에서 미리 보고 싶을 때.

### Hugo 설치
- **Windows**: `scoop install hugo-extended` 또는 `winget install Hugo.Hugo.Extended`
- **macOS**: `brew install hugo`

### 미리보기 실행
이 폴더에서:

```bash
hugo server -D
```

브라우저에서 `http://localhost:1313` 접속. `-D` 는 초안(draft)까지 보여주는 옵션입니다.
파일을 저장하면 화면이 자동으로 새로고침됩니다.

---

## 4. 폴더 구조

```
TEAM_Blog/
├── hugo.toml              # 블로그 설정 (제목·주소·메뉴)
├── content/
│   ├── posts/             # ★ 팀 공식 자료
│   ├── members/           # ★ 개인 공부 기록
│   │   ├── member1/       #   팀원별 폴더
│   │   ├── member2/
│   │   ├── member3/
│   │   └── member4/
│   └── about/             # 소개 페이지
├── layouts/               # 디자인(템플릿) — 보통 건드릴 일 없음
├── assets/css/main.css    # 색·폰트·간격 (디자인 수정 시 여기)
├── archetypes/            # 새 글 기본 양식
└── .github/workflows/     # 자동 배포 설정
```

## 5. 자주 쓰는 마크다운

| 쓰고 싶은 것 | 문법 |
| --- | --- |
| 제목 | `## 소제목`, `### 더 작은 제목` |
| 굵게 / 기울임 | `**굵게**`, `*기울임*` |
| 링크 | `[보이는 글자](https://주소)` |
| 이미지 | `![설명](이미지주소)` |
| 목록 | 줄 앞에 `- ` 또는 `1. ` |
| 인용 | 줄 앞에 `> ` |
| 코드 | `` `한 줄 코드` `` 또는 ```` ``` ````로 여러 줄 감싸기 |
| 표 | `\| 칸 \| 칸 \|` 형태 |

이미지는 글과 같은 폴더에 넣고 `![설명](이미지파일명.png)` 으로 넣으면 가장 간단합니다.
(글 파일을 `폴더/index.md` 형태로 만들면 같은 폴더의 이미지를 바로 참조할 수 있습니다.)
