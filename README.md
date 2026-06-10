# lhoris 님의 개인 Wiki

Obsidian + Quartz 기반의 개인 기술 위키 및 디지털 가든(Digital Garden) 저장소입니다.

---

## Overview

이 저장소는 다음과 같은 목적으로 운영됩니다.

* 개인 기술 블로그 및 디지털 가든 운영
* AI Agent 및 LLM 관련 연구 내용 정리

---

## Tech Stack

* Obsidian
* Quartz 5
* GitHub Pages
* GitHub Actions
* Markdown

---

## Local Development

### Install Dependencies

```bash
npm install
```

### Run Local Server

```bash
npx quartz build --serve
```

로컬 기본 접속 주소

> [http://localhost:8080](http://localhost:8080)

---

## Deployment

GitHub Actions를 이용하여 GitHub Pages에 자동 배포합니다.

배포 흐름

```text
Obsidian
    ↓
Markdown 작성
    ↓
Git Push
    ↓
GitHub Actions
    ↓
Quartz Build
    ↓
GitHub Pages
```

---

## References

* Quartz

  * https://quartz.jzhao.xyz/

* Obsidian

  * https://obsidian.md/

* GitHub Pages

  * https://pages.github.com/