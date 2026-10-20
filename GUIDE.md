# 팀원 가이드 (글 쓰는 법)

이 블로그는 **글 하나 = 폴더 하나**입니다.
폴더 안에 `index.md`(글 내용)와 `images/`(그 글의 이미지)가 함께 들어갑니다.
글을 올리면 1~2분 뒤 자동으로 사이트( https://kisiav8research.github.io/ )에 반영됩니다.

- **팀 글** → `content/posts/글이름/index.md` → **홈**에 표시
- **개인 글** → `content/members/본인폴더/글이름/index.md` → **개인 글** 메뉴에 표시

본인 폴더:
`lee-seungwon`(이승원) · `cha-eunhyeok`(차은혁) · `kim-minjeong`(김민정) · `lim-suhee`(임수희)

> 예) 이승원이 "v8" 이라는 개인 글을 쓰면 → `content/members/lee-seungwon/v8/index.md`

---

## 0. 처음 한 번만

1. 팀장이 보낸 **저장소 초대(Invite)** 를 이메일/GitHub 알림에서 **수락**합니다.
2. 이미지를 붙여넣기로 쓰려면 **VS Code**로 작업하는 걸 추천합니다 (아래 B + 이미지 섹션).

---

## A. 글 양식 (공통)

`index.md` 맨 위에는 글 정보(front matter)를 넣습니다.

```markdown
---
title: "글 제목"
date: 2026-10-19
author: "내 이름"
cover: ""
tags: ["태그1", "태그2"]
---

여기에 마크다운으로 본문을 씁니다.
```

> - `draft: true` 를 한 줄 넣으면 **비공개 초안**입니다. 완성되면 그 줄을 지워 공개하세요.
> - `cover` 는 비워 두면 **본문 첫 이미지**가 자동으로 카드 썸네일이 됩니다.
>   특정 이미지를 쓰려면 파일명을 적으세요. 예) `cover: "images/image.png"`

각 멤버 폴더의 `example/` 가 복사용 샘플입니다. 그대로 복사해서 폴더 이름만 바꾸면 새 글이 됩니다.

---

## B. VS Code로 쓰기 (이미지 붙여넣기 가능, 추천)

### 최초 1회
```bash
git clone https://github.com/kisiaV8research/kisiaV8research.github.io.git
```
그리고 **VS Code에서 이 폴더를 엽니다** (File → Open Folder). 저장소에 설정이 들어 있어
별도 확장 설치 없이 이미지 붙여넣기가 동작합니다. (VS Code 1.79 이상)

### 글 쓸 때마다
```bash
git pull                      # ① 항상 먼저 (최신화)
```
② 글 폴더와 `index.md`를 만듭니다. (예: `content/posts/v8/index.md`)
   - VS Code 탐색기에서 폴더 추가 → 그 안에 `index.md` 생성
   - 또는 터미널에서 `hugo new posts/v8/index.md` (팀 글) / `hugo new members/lee-seungwon/v8/index.md` (개인 글)

③ **이미지는 복사 → 붙여넣기(Ctrl+V)** 하면 끝:
   - 그 글의 `images/` 폴더에 저장되고
   - 커서 위치에 **상대경로 링크** `![](images/파일명)` 가 자동 삽입됩니다
   - 처음 넣은 이미지는 자동으로 카드 썸네일이 됩니다
   - 안 되면: ① VS Code로 **저장소 폴더**를 열었는지(파일만 열면 설정이 안 먹음)
     ② 글이 `글이름/index.md` 인지 ③ `git pull` 로 최신 설정을 받았는지 확인

④ 저장하고 올리기:
```bash
git add .
git commit -m "새 글: 제목"
git push                      # main에 바로 push OK
```

> 💡 올리기 전에 미리보기: Hugo 설치(`winget install Hugo.Hugo.Extended`) 후
> 저장소 폴더에서 `hugo server -D` → 브라우저 `http://localhost:1313`

---

## C. GitHub 웹에서 쓰기 (설치 없이, 글자 위주)

이미지가 없는 글이면 웹이 가장 간단합니다.

1. 저장소 → **Add file → Create new file**
2. 파일 이름에 경로까지 입력: `content/posts/v8/index.md`
   (슬래시 `/` 를 넣으면 폴더가 자동으로 만들어집니다)
3. 위 **A. 글 양식** 을 붙여넣고 작성 → **Commit**

> 웹에서는 이미지 자동 저장이 안 됩니다. 이미지가 필요하면 B(VS Code) 방식을 쓰세요.

---

## 자주 겪는 상황

| 상황 | 해결 |
| --- | --- |
| 글이 사이트에 안 보임 | 1~2분 대기 → 그래도 없으면 `draft: true` 가 남았는지 확인 |
| 이미지가 안 보임 | `images/파일명` 링크가 맞는지, 이미지가 같은 글 폴더 안에 있는지 확인 |
| push가 rejected | 다른 사람이 먼저 올림 → `git pull` 후 다시 `git push` |
| 썸네일을 바꾸고 싶음 | front matter `cover` 에 원하는 이미지 파일명 지정 |
| 글 수정/삭제 | 해당 글 폴더의 `index.md`(또는 폴더 전체)를 고치거나 지우고 commit/push |

## 자주 쓰는 마크다운

| 쓰고 싶은 것 | 문법 |
| --- | --- |
| 소제목 | `## 제목`, `### 더 작은 제목` |
| 굵게 / 기울임 | `**굵게**`, `*기울임*` |
| 링크 | `[보이는 글자](https://주소)` |
| 이미지 | 붙여넣기(Ctrl+V) 또는 `![설명](images/파일명.png)` |
| 목록 | 줄 앞에 `- ` 또는 `1. ` |
| 인용 | 줄 앞에 `> ` |
| 코드 | `` `한 줄` `` 또는 ```` ``` ````로 여러 줄 감싸기 |
| 표 | `\| 칸 \| 칸 \|` |
