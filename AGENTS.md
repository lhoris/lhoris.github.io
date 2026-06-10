# AGENTS.md — lhoris.github.io (Personal Wiki)

이 저장소는 Obsidian + Quartz 5 기반의 개인 wiki이자 디지털 가든입니다.
배포는 `git push` 후 GitHub Actions가 자동으로 GitHub Pages에 수행합니다.

---

## 저장소 구조 요약

```
content/         ← 실제 wiki 문서 (Obsidian 마크다운) — 주요 작업 대상
quartz/          ← Quartz 프레임워크 소스 (직접 수정 금지)
quartz.config.yaml  ← 사이트 설정 (신중하게 변경)
public/          ← 빌드 산출물 (직접 수정 금지)
.github/workflows/  ← CI/CD 워크플로우 (직접 수정 금지)
```

---

## 금지 사항 (하지 말 것)

### 콘텐츠

- `content/private/` 폴더의 파일을 공개 경로로 이동하거나 내용을 출력·요약하지 않는다.
- 기존 문서의 frontmatter를 사용자 지시 없이 변경하지 않는다.
- 문서를 삭제하거나 내용을 대규모로 재작성하지 않는다. 대규모 변경 전 반드시 확인한다.
- `ignorePatterns`(`private`, `templates`, `.obsidian`)에 해당하는 폴더를 빌드 대상에 포함시키지 않는다.

### 빌드·배포

- `public/` 폴더를 직접 편집하지 않는다. 빌드 산출물이며 CI가 자동 생성한다.
- `git push`는 사용자 확인 없이 실행하지 않는다.
- `main` 브랜치에 `--force` 푸시하지 않는다.

### 프레임워크·설정

- `quartz/` 내부 소스를 수정하지 않는다.
- `quartz.config.yaml`의 `baseUrl`, `theme`, `plugins` 섹션은 명시적 요청 없이 건드리지 않는다.
- `package.json`, `package-lock.json`을 직접 편집하지 않는다.
- `.github/workflows/` CI 파일을 임의로 수정하지 않는다.

---

## 콘텐츠 작성 규칙

- 모든 신규 문서에 frontmatter를 포함한다:
  ```yaml
  ---
  title: 제목
  date: YYYY-MM-DD
  tags: [tag1, tag2]
  ---
  ```
- 내부 링크는 Obsidian 위키링크 형식(`[[파일명]]`)을 사용한다.
- 이미지·첨부파일은 `content/assets/` 아래에 둔다.
- 민감한 정보(개인 정보, 비밀키, 토큰 등)는 절대 `content/` 아래 어디에도 기록하지 않는다.
