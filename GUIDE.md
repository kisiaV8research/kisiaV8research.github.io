# 팀원 가이드 (글 쓰는 법)

이 블로그는 **마크다운(.md) 파일 하나 추가 = 글 하나**입니다.
글을 올리면 1~2분 뒤 자동으로 사이트( https://kisiav8research.github.io/ )에 반영됩니다.

- **팀 글** → `content/posts/` 에 올림 → **홈**에 표시
- **개인 글** → `content/members/본인폴더/` 에 올림 → **개인 글** 메뉴에 표시

본인 폴더:
`lee-seungwon`(이승원) · `cha-eunhyeok`(차은혁) · `kim-minjeong`(김민정) · `lim-suhee`(임수희)

---

## 0. 처음 한 번만

1. 팀장이 보낸 **저장소 초대(Invite)** 를 이메일/GitHub 알림에서 **수락**합니다.
   (수락하면 글을 직접 올릴 수 있는 권한이 생깁니다.)
2. 글 쓰는 방법은 두 가지 중 편한 걸 고르면 됩니다 — **A. 웹에서 바로** / **B. 내 컴퓨터에서**.

---

## A. 웹에서 바로 쓰기 — 설치 필요 없음 (추천)

가장 간단합니다. 브라우저만 있으면 됩니다.

1. 저장소로 이동: https://github.com/kisiaV8research/kisiaV8research.github.io
2. 글을 둘 폴더로 들어갑니다.
   - 팀 글: `content` → `posts`
   - 개인 글: `content` → `members` → `본인폴더`
3. 오른쪽 위 **Add file → Create new file** 클릭
4. 파일 이름을 `영문-제목.md` 형식으로 입력 (예: `transformer-review.md`)
   - 한글 파일명도 되지만, 주소가 깔끔하려면 영문/숫자/하이픈 권장
5. 아래 양식을 붙여넣고 내용을 채웁니다.

```markdown
---
title: "글 제목"
date: 2026-10-18
author: "내 이름"
cover: ""
tags: ["태그1", "태그2"]
---

여기에 마크다운으로 본문을 씁니다.
```

6. 오른쪽 위 **Commit changes...** → 초록 버튼 **Commit** 클릭 → 끝
7. 1~2분 뒤 사이트에 반영됩니다. (저장소 **Actions** 탭에서 초록 체크 확인 가능)

> 📌 맨 위 `---` 사이를 **front matter**(글 정보)라고 합니다.
> - `draft: true` 를 넣으면 **비공개 초안**이 됩니다. 완성되면 그 줄을 지워 공개하세요.
> - `cover: "이미지주소"` 를 넣으면 홈/목록 카드의 **썸네일**로 쓰입니다. 비우면 제목 첫 글자로 자동 생성됩니다.

---

## B. 내 컴퓨터에서 쓰기 (git 사용)

미리보기를 하거나 여러 글을 한 번에 다루고 싶을 때.

### 최초 1회 — 클론
```bash
git clone https://github.com/kisiaV8research/kisiaV8research.github.io.git
cd kisiaV8research.github.io
```

### 글 쓸 때마다 — 이 순서를 반복
```bash
# 1) 최신 상태로 맞추기 (중요! 항상 먼저)
git pull

# 2) 글 파일 추가/수정
#    팀 글  : content/posts/내-글.md
#    개인 글: content/members/본인폴더/내-글.md
#    (위 A의 양식과 동일하게 작성)

# 3) 저장하고 올리기
git add .
git commit -m "새 글: 글 제목"
git push
```

- `main` 브랜치에 바로 push 해도 됩니다. (작은 팀이라 PR 안 거쳐도 OK)
- push하면 자동으로 빌드·배포됩니다.

> 💡 새 글 틀을 명령어로 만들 수도 있습니다 (Hugo 설치 시):
> ```bash
> hugo new posts/내-글.md               # 팀 글
> hugo new members/lee-seungwon/내-글.md  # 개인 글(본인 폴더)
> ```

### (선택) 올리기 전에 미리보기
Hugo를 설치하면 내 화면에서 먼저 확인할 수 있습니다.
- 설치 — Windows: `winget install Hugo.Hugo.Extended` / macOS: `brew install hugo`
- 미리보기 — 저장소 폴더에서:
  ```bash
  hugo server -D
  ```
  브라우저에서 `http://localhost:1313` 접속. 저장하면 자동 새로고침됩니다. (`-D` = 초안도 보임)

---

## 자주 겪는 상황

| 상황 | 해결 |
| --- | --- |
| 글을 올렸는데 사이트에 안 보임 | 1~2분 기다리기 → 그래도 없으면 `draft: true` 가 남아있는지 확인 |
| push할 때 거부(rejected)됨 | 다른 사람이 먼저 올린 것 → `git pull` 후 다시 `git push` |
| 이미지 넣고 싶음 | 이미지를 `static/images/` 에 올리고 본문에 `![설명](/images/파일명.png)` |
| 글 수정/삭제 | 해당 `.md` 파일을 고치거나 지우고 commit/push 하면 반영 |

## 자주 쓰는 마크다운

| 쓰고 싶은 것 | 문법 |
| --- | --- |
| 소제목 | `## 제목`, `### 더 작은 제목` |
| 굵게 / 기울임 | `**굵게**`, `*기울임*` |
| 링크 | `[보이는 글자](https://주소)` |
| 목록 | 줄 앞에 `- ` 또는 `1. ` |
| 인용 | 줄 앞에 `> ` |
| 코드 | `` `한 줄` `` 또는 ```` ``` ````로 여러 줄 감싸기 |
| 표 | `\| 칸 \| 칸 \|` |
