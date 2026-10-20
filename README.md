# Research Notes

KISIA V8 팀 연구 블로그 · **https://kisiav8research.github.io/**
Hugo로 만들고 GitHub Pages로 배포합니다. `main` 에 push하면 1~2분 뒤 자동 반영됩니다.

- 글을 쓰는 상세 안내는 [`GUIDE.md`](GUIDE.md) 참고.

---

## 1. 초기 세팅 (Windows)

### 1) 필요한 프로그램 설치
PowerShell에서:
```powershell
winget install Git.Git
winget install Microsoft.VisualStudioCode
winget install Hugo.Hugo.Extended   # 로컬 미리보기용(선택)
```
설치 후 PowerShell을 새로 켜야 명령어가 인식됩니다.

### 2) 저장소 받기
```powershell
git clone https://github.com/kisiaV8research/kisiaV8research.github.io.git
```

### 3) VS Code로 "폴더" 열기 (중요)
VS Code → **File → Open Folder…** → 받은 `kisiaV8research.github.io` **폴더**를 엽니다.
> 파일 하나만 열면 이미지 붙여넣기 설정(`.vscode/settings.json`)이 적용되지 않습니다. 반드시 폴더로 여세요.

### 4) (선택) 로컬 미리보기
글을 올리기 전에 내 화면에서 확인하려면, 저장소 폴더에서:
```powershell
hugo server -D
```
브라우저에서 `http://localhost:1313` 접속 (`-D` = 초안도 표시). 저장하면 자동 새로고침.

---

## 2. 폴더 구조

```
kisiaV8research.github.io/
├── hugo.toml                     # 블로그 설정(제목·주소·메뉴)
├── content/
│   ├── posts/                    # ★ 팀 글 → 홈에 표시
│   │   └── 글이름/               #   글 1개 = 폴더 1개
│   │       ├── index.md          #     글 본문
│   │       └── images/           #     그 글의 이미지
│   ├── members/                  # ★ 개인 글 → "개인 글" 메뉴에 표시
│   │   ├── lee-seungwon/         #   이승원   (각자 폴더 안에 글이름/index.md)
│   │   ├── cha-eunhyeok/         #   차은혁
│   │   ├── kim-minjeong/         #   김민정
│   │   └── lim-suhee/            #   임수희
│   └── about/                    # 소개 페이지
├── layouts/                      # 디자인 템플릿(보통 안 건드림)
├── assets/css/main.css           # 색·폰트·간격
├── .vscode/settings.json         # 이미지 붙여넣기 자동화 설정
└── .github/workflows/deploy.yml  # 자동 배포
```

---

## 3. 글 작성 방법

### 규칙: 글 1개 = 폴더 1개 + 그 안의 `index.md`
평범한 `.md` 파일 하나로 만들지 **마세요**. 반드시 폴더를 만들고 안에 `index.md` 를 둡니다.

| 종류 | 경로 |
| --- | --- |
| 팀 글 | `content/posts/글이름/index.md` |
| 개인 글 | `content/members/본인폴더/글이름/index.md` |

> 만드는 법: VS Code 탐색기에서 `posts`(또는 본인 폴더) 우클릭 → **New Folder**(`글이름`) → 그 안에 **New File**(`index.md`).
> 또는 터미널: `hugo new posts/글이름/index.md`

### 글 양식 (`index.md` 맨 위)
```markdown
---
title: "글 제목"
date: 2026-10-19
author: "이승원"
cover: ""
tags: ["v8", "pwn"]
---

여기부터 본문(마크다운).
```

| 항목 | 필수 | 설명 |
| --- | --- | --- |
| `title` | ✅ | 글 제목 |
| `date` | ✅ | 작성일 `YYYY-MM-DD` |
| `author` | ✅ | 작성자 이름 |
| `tags` | | 태그 목록. 예 `["v8", "d8"]` |
| `cover` | | 썸네일 이미지. 비우면 **본문 첫 이미지**가 자동 썸네일 |
| `draft` | | `true` 면 비공개 초안. 공개할 때 이 줄 삭제 |

### 이미지 넣기
`index.md` 를 연 상태에서 이미지를 **복사 → 붙여넣기(Ctrl+V)**:
- 그 글의 `images/` 폴더에 저장
- 커서 위치에 **상대경로 링크** `![](images/파일명)` 자동 삽입
- 처음 넣은 이미지는 자동으로 카드 썸네일

수동으로 넣을 때는 이미지를 글 폴더의 `images/` 에 넣고 `![설명](images/파일명.png)` 라고 쓰면 됩니다.

### 자주 쓰는 마크다운
| 쓰고 싶은 것 | 문법 |
| --- | --- |
| 소제목 | `## 제목` · `### 더 작은 제목` |
| 굵게 / 기울임 | `**굵게**` · `*기울임*` |
| 링크 | `[보이는 글자](https://주소)` |
| 목록 | 줄 앞에 `- ` 또는 `1. ` |
| 인용 | 줄 앞에 `> ` |
| 코드 | `` `한 줄` `` 또는 ```` ``` ````로 여러 줄 감싸기 |
| 표 | `\| 칸 \| 칸 \|` |

---

## 4. 올리기 (push)

저장소 폴더에서 PowerShell로:
```powershell
git pull                              # ① 항상 먼저 (최신화)
git add .
git commit -m "새 글: V8 basic"
git push                              # ③ main에 바로 push
```
`main` 에 push하면 **GitHub Actions**가 자동으로 빌드·배포합니다.

> **push가 거부(rejected)되면** 다른 사람이 먼저 올린 것입니다 → `git pull` 후 다시 `git push`.

### 웹에서 올리기(설치 없이, 글자 위주)
이미지 없는 글은 GitHub 웹에서 **Add file → Create new file** 로 파일 이름에
`content/posts/글이름/index.md` 처럼 경로까지 입력하면 폴더째 만들어집니다. (이미지 자동저장은 VS Code에서만)

---

## 5. 배포 확인

- 저장소 **Actions** 탭에서 "Deploy Hugo site to Pages" 작업이 **초록 체크**면 완료 (1~2분)
- 사이트: **https://kisiav8research.github.io/**
- 빨간 X(실패)면 해당 로그를 확인하거나 팀에 공유

---

## 참고: 배포 구성 (유지보수용)
- GitHub Organization `kisiaV8research` 의 **user-page 저장소**(`<org>.github.io`)
- **Settings → Pages → Source = GitHub Actions** 로 설정되어 있음
- `main` push → `.github/workflows/deploy.yml` 이 Hugo 빌드 후 Pages로 배포
- 블로그 제목·주소·메뉴는 `hugo.toml` 에서 수정
