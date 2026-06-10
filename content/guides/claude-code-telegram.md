---
title: Claude Code × Telegram 연동 가이드
date: 2026-06-11
tags:
  - guide
  - claude-code
  - telegram
  - bot
---

**Claude Code CLI와 Telegram을 연동하여 나만의 AI 비서를 만드는 가이드**

Claude Code를 Telegram Bot과 연결하면 텔레그램 채팅으로 Claude에게 질문하거나 명령을 내릴 수 있습니다.
DM은 물론 그룹 채팅에서도 @멘션으로 사용할 수 있어요.

> 원본 저장소: [lhoris/claudeclaw-telegram](https://github.com/lhoris/claudeclaw-telegram)

---

## 운영체제별 가이드

| OS         | 상태             |
| ---------- | -------------- |
| ✅ Windows  | 이 문서에서 다룹니다    |
| 🔜 macOS  | 작성 예정          |
| 🔜 Linux  | 작성 예정          |

---

## 사전 준비 (Windows)

- [x] [Claude Code CLI](https://claude.ai/code) 설치 완료 (아래 설치 방법 참고)
- [x] [Telegram Desktop](https://desktop.telegram.org/) 설치 완료
- [ ] Telegram Bot 생성 (아래 1단계 참고)

### Claude Code 설치 방법

두 가지 방법 중 하나를 선택하세요.

#### 방법 A. Native Installer (권장)

Windows PowerShell에서 실행:

```powershell
irm https://claude.ai/install.ps1 | iex
```

| 장점        | 설명                              |
| --------- | ------------------------------- |
| 자동 업데이트   | 새 버전 출시 시 자동으로 업데이트             |
| 추가 설치 불필요 | Node.js, Bun 별도 설치 없이 바로 사용 가능  |
| 빠른 실행     | 네이티브 바이너리로 실행 속도 빠름             |

#### 방법 B. npm 설치

Node.js가 이미 설치된 경우:

```bash
npm install -g @anthropic-ai/claude-code@latest
```

> [!warning] 주의사항
> - 자동 업데이트 미지원 — 업그레이드 시 수동으로 재설치 필요
> - Telegram 플러그인 사용을 위해 [Bun](https://bun.sh/)을 별도로 설치해야 함
> - Bun 설치: `powershell -c "irm bun.sh/install.ps1 | iex"`

> [!tip] npm 업그레이드 시 ENOTEMPTY 오류 발생하면
> ```bash
> npm uninstall -g @anthropic-ai/claude-code
> npm cache clean --force
> npm install -g @anthropic-ai/claude-code@latest
> ```

### 설치 확인

```bash
claude --version
```

---

## 단계별 역할 구분

| 단계             | 담당          | 설명                                             |
| -------------- | ----------- | ---------------------------------------------- |
| 1단계: Bot 생성    | 👤 사용자     | Telegram 앱에서 BotFather와 직접 대화                  |
| 2단계: 플러그인 설치   | 🤖 Claude 가능 | `/plugin install` 명령 실행                       |
| 3단계: 토큰 설정     | 🤖 Claude 가능 | `/telegram:configure <토큰>` 실행                 |
| 4단계: 재시작       | 👤 사용자     | `--channels` 플래그로 Claude Code 재실행 필요           |
| 5단계: 페어링 코드 받기 | 👤 사용자     | Telegram에서 봇에게 `/start` 전송                    |
| 5단계: 페어링 승인    | 🤖 Claude 가능 | `/telegram:access pair <코드>` 실행               |
| 6단계: 보안 설정     | 🤖 Claude 가능 | `/telegram:access policy allowlist` 실행        |

> [!important] 핵심
> 4단계 `--channels` 플래그 재시작은 Claude가 자신의 세션을 재시작할 수 없으므로 반드시 사용자가 직접 해야 합니다.

---

## 1단계: Telegram Bot 생성

1. Telegram에서 `@BotFather` 검색 후 대화 시작
2. `/newbot` 명령어 입력

   > **BotFather 응답:**
   > ```
   > Alright, a new bot. How are we going to call it?
   > Please choose a name for your bot.
   > ```

3. Bot 표시 이름 입력 (예: `My Claude Assistant`)

   > **BotFather 응답:**
   > ```
   > Good. Now let's choose a username for your bot.
   > It must end in `bot`. Like this, for example: TetrisBot or tetris_bot.
   > ```

4. Bot username 입력 — 반드시 `bot`으로 끝나야 함 (예: `myclaudeassistant_bot`)

   > **username이 이미 사용 중인 경우:**
   > ```
   > Sorry, this username is already taken. Please try something different.
   > ```
   > 다른 username으로 다시 시도하세요.

   > **성공 시 BotFather 응답:**
   > ```
   > Done! Congratulations on your new bot. You will find it at t.me/your_bot.
   > You can now add a description, about section and profile picture for your bot,
   > see /help for a list of commands.
   >
   > Use this token to access the HTTP API:
   > 1234567890:AAxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
   >
   > Keep your token secure and store it safely,
   > it can be used by anyone to control your bot.
   > ```

5. 발급된 **Bot Token**을 복사해두기 (외부 유출 금지 ⚠️)

---

## 2단계: Telegram 플러그인 설치

Claude Code CLI 터미널에서 아래 명령어 실행:

```
/plugin install telegram@claude-plugins-official
```

설치가 완료되면 `/telegram` 관련 명령어가 활성화됩니다.

---

## 3단계: Bot Token 설정

Claude Code CLI 터미널에서 1단계에서 발급받은 Token을 인수로 넘겨 실행:

```
/telegram:configure <발급받은_토큰>
```

예시:

```
/telegram:configure 1234567890:AAxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

설정 확인은 인수 없이 실행:

```
/telegram:configure
```

Access Policy(접근 정책):

- `pairing` — 요청할 때마다 코드로 승인 (기본값)
- `allowlist` — 미리 등록된 사용자만 접근 가능 (보안상 권장)
- `disabled` — 채널 비활성화

---

## 4단계: Channels 플래그로 Claude Code 실행

기존 Claude Code 세션을 종료하고, 아래 명령어로 재시작:

```bash
claude --channels plugin:telegram@claude-plugins-official
```

> [!warning]
> 이 명령어로 실행된 Claude Code 터미널 창을 **닫으면 Telegram 연동이 끊깁니다.**
> Telegram으로 Claude를 사용하는 동안에는 이 창을 유지해야 합니다.

---

## 5단계: 봇 페어링 (DM 연동)

1. Telegram에서 내 봇에게 `/start` 전송
2. 봇이 **6자리 페어링 코드** 반환
3. Claude Code 터미널에서 아래 명령어 실행:

```
/telegram:access pair <6자리코드>
```

4. 승인 완료 → 이제 Telegram DM으로 Claude와 대화 가능 🎉

---

## 6단계: 보안 설정 (권장)

나만 사용할 경우, allowlist 정책으로 변경해 타인의 접근을 차단하세요:

```
/telegram:access policy allowlist
```

---

## 7단계: 그룹 채팅 연동 (선택)

### 7-1. 봇을 그룹에 초대

Telegram 그룹 채팅방에 내 봇을 멤버로 추가합니다.

### 7-2. BotFather에서 Group Privacy 비활성화

```
@BotFather → /mybots → 봇 선택 → Bot Settings → Group Privacy → Disable
```

> [!warning]
> 기본 설정(Enabled)이면 봇이 그룹 메시지를 읽지 못합니다.
> 변경 후 봇을 그룹에서 내보냈다가 다시 초대해야 적용될 수 있습니다.

### 7-3. 그룹 ID 확인 및 등록

그룹 ID를 확인한 뒤 Claude Code 터미널에서 실행:

```
/telegram:access group add <그룹ID>
```

> [!info] 그룹 ID 확인 방법
> 1. 그룹 채팅에서 아무 메시지나 선택
> 2. `@userinfobot`에게 해당 메시지를 포워딩
> 3. userinfobot이 Chat ID를 알려줌 — 음수(`-`)로 시작하는 숫자가 그룹 ID
>
> 그룹에 봇을 초대할 필요 없이 포워딩만으로 확인 가능합니다.

### 7-4. 그룹에서 사용 — @멘션에만 응답하게 하기

Group Privacy를 비활성화하면 봇이 그룹 내 모든 메시지를 읽을 수 있지만,
**플러그인은 @멘션된 메시지에만 응답**합니다.

```
@봇이름 안녕하세요!
```

@멘션 없이 보낸 메시지에는 응답하지 않으므로, 일반 대화를 방해하지 않습니다.

> [!question] 멘션 응답이 작동하지 않을 때 확인 사항
> - BotFather에서 Group Privacy가 Disable로 설정되어 있는지 확인
> - 비활성화 후 봇을 그룹에서 내보냈다가 다시 초대했는지 확인
> - `claude --channels plugin:telegram@claude-plugins-official` 로 실행 중인지 확인

---

## 활용 예시

- 📅 일정 관리 (파일 기반 메모)
- 📈 주식/투자 정보 조회
- 💬 질문 & 답변
- 💻 코드 작성 / 리뷰 요청
- 🔍 검색 및 요약

---

## 관련 명령어 모음

| 명령어                            | 설명                       |
| ------------------------------ | ------------------------ |
| `/telegram:configure`          | Bot Token 설정 및 채널 초기화    |
| `/telegram:access`             | 접근 현황 확인                 |
| `/telegram:access pair <코드>`   | 페어링 코드로 사용자 승인           |
| `/telegram:access allow <ID>`  | 특정 사용자 직접 허용             |
| `/telegram:access remove <ID>` | 사용자 제거                   |
| `/telegram:access group add <ID>` | 그룹 채팅 등록                |
| `/telegram:access policy <mode>` | 접근 정책 변경               |

---

## 참고 링크

- [Claude Code 공식 문서](https://claude.ai/code)
- [Telegram BotFather](https://t.me/botfather)
- [claudeclaw-telegram (원본 저장소)](https://github.com/lhoris/claudeclaw-telegram)
