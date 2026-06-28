# CLAUDE.md — lhoris.github.io (Personal Wiki)

이 저장소는 Obsidian + Quartz 5 기반의 개인 wiki이자 디지털 가든입니다.
주된 작업은 `content/` 아래 마크다운 파일을 추가·수정하는 것입니다.
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

- `content/private/` 폴더의 파일을 공개 경로로 이동하거나 내용을 외부에 노출하지 않는다.
- 기존 문서의 frontmatter(`title`, `tags`, `date`, `aliases` 등)를 사용자 지시 없이 임의로 변경하지 않는다.
- 문서를 삭제하거나 내용을 대규모로 교체하지 않는다. 수정 전 반드시 확인한다.
- `ignorePatterns`에 등록된 폴더(`private`, `templates`, `.obsidian`)의 내용을 빌드 대상으로 만들지 않는다.

### 빌드·배포

- `public/` 폴더를 직접 편집하지 않는다. 빌드 산출물이며 CI가 자동 생성한다.
- `git push`는 반드시 사용자 확인 후에만 실행한다.
- `main` 브랜치에 `--force` 푸시하지 않는다.

### 프레임워크·설정

- `quartz/` 내부 TypeScript 소스를 수정하지 않는다 (플러그인 개발 요청 시 예외).
- `quartz.config.yaml`의 `baseUrl`, `theme`, `plugins` 섹션은 명시적 요청 없이 변경하지 않는다.
- `package.json`, `package-lock.json`을 직접 편집하지 않는다.
- `.github/workflows/`의 CI 파일을 임의로 수정하지 않는다.

---

## 콘텐츠 작성 규칙

- 모든 문서에 frontmatter를 포함한다:
  ```yaml
  ---
  title: 제목
  date: YYYY-MM-DD
  tags: [tag1, tag2]
  ---
  ```
- 내부 링크는 Obsidian 위키링크 형식(`[[파일명]]`)을 사용한다.
- 이미지·첨부파일은 `content/assets/` 아래에 둔다.
- 공개하면 안 되는 내용은 반드시 `content/private/` 아래에 둔다.

---

## 필수 작업 지침 (새 문서 추가 시)

새로운 공개 콘텐츠 문서를 `content/` 아래에 추가할 때는 반드시 다음 작업을 함께 수행한다.

- `content/index.md`의 **"## 최근 문서"** 목록 **맨 위**에 새 문서의 위키링크를 추가한다.
  - 형식: `- [[경로/파일명|문서 제목]]` (예: `- [[devlog/study-bmad|BMAD를 사용하며 깨달은 …]]`)
  - 가장 최근 글이 항상 맨 위에 오도록 한다.
- 이 작업은 새 문서 커밋과 같은 PR/푸시 사이클 안에서 같이 처리한다 (별도 후속 커밋으로 미루지 않는다).
- 단, `content/private/` 아래의 비공개 문서는 최근 문서 목록에 추가하지 않는다.
