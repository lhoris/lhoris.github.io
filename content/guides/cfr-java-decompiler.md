---
title: CFR Java Decompiler 사용법 — JAR 파일 디컴파일 실전 예제
date: 2026-06-18
tags:
  - guide
  - java
  - decompiler
  - cfr
  - reverse-engineering
---

**Java 클래스 파일(.class) / JAR 파일을 사람이 읽을 수 있는 소스코드로 되돌리는 가장 깔끔한 도구, CFR을 다뤄봅니다.**

`CFR (Class File Reader)`는 Java 1.0부터 최신 LTS(Java 21+)까지 폭넓게 지원하는 디컴파일러입니다.
단일 jar 파일만 있으면 동작하고, 외부 라이브러리 의존성이 없으며, lambda·record·sealed class 같은 최신 문법도 깔끔히 복원합니다.

> 공식 사이트: [https://www.benf.org/other/cfr/](https://www.benf.org/other/cfr/)
> GitHub: [leibnitz27/cfr](https://github.com/leibnitz27/cfr)

---

## 왜 CFR인가?

| 디컴파일러                | 특징                                                              |
| -------------------- | --------------------------------------------------------------- |
| **CFR**              | 순수 Java, 단일 jar, 최신 문법(record/sealed/lambda) 잘 복원, CLI 친화적     |
| Fernflower (IntelliJ 내장) | IDE 통합 우수, 단독 사용은 다소 불편                                         |
| Procyon              | 오래된 클래스 파일에 강함, 업데이트가 느림                                        |
| JD-GUI               | GUI 친화적이나 최신 문법(예: lambda, switch expression)에서 결과가 어색한 경우 있음 |

> [!tip]
> 빠르게 한 두 클래스만 보려면 JD-GUI, IDE에서 분석하려면 Fernflower,
> **CLI로 jar 전체를 일괄 디컴파일**하려면 CFR이 가장 합리적입니다.

---

## 사전 준비

- [x] **JDK 8 이상** 설치 (`java -version`으로 확인)
- [x] CFR jar 파일 1개 (다음 단계에서 다운로드)
- [x] 디컴파일할 대상 jar 파일

### Java 설치 확인

```bash
java -version
```

설치되지 않았다면 [Adoptium Temurin](https://adoptium.net/) 등에서 LTS 버전을 설치하세요.

---

## 1단계: CFR 다운로드

CFR은 단일 jar 파일로 배포됩니다. 별도 설치가 필요 없습니다.

1. [CFR 공식 다운로드 페이지](https://www.benf.org/other/cfr/) 접속
2. 최신 버전 jar 다운로드 (예: `cfr-0.152.jar`)
3. 작업 폴더에 저장 (예: `C:\tools\cfr-0.152.jar`)

> [!info]
> Maven Central에서도 받을 수 있습니다:
> [org.benf:cfr](https://central.sonatype.com/artifact/org.benf/cfr)

### 동작 확인

```bash
java -jar cfr-0.152.jar --help
```

옵션 목록이 길게 출력되면 정상입니다.

---

## 2단계: 단일 클래스 파일 디컴파일

가장 간단한 사용법부터 시작합니다.

```bash
java -jar cfr-0.152.jar HelloWorld.class
```

표준 출력으로 디컴파일된 Java 코드가 출력됩니다.
파일로 저장하려면 리다이렉트하거나 `--outputpath` 옵션을 씁니다.

```bash
java -jar cfr-0.152.jar HelloWorld.class > HelloWorld.java
```

---

## 3단계: JAR 파일 디컴파일 (메인)

실제 업무에서 가장 자주 쓰는 시나리오입니다.
`example-app.jar` 안에 있는 모든 클래스를 한 번에 디컴파일하고, 패키지 구조를 보존하면서 디렉터리에 풀어내봅니다.

### 기본 명령어

```bash
java -jar cfr-0.152.jar example-app.jar --outputpath ./decompiled
```

- `example-app.jar` — 디컴파일 대상 jar
- `--outputpath ./decompiled` — 결과물을 저장할 폴더 (없으면 자동 생성)

### 실행 결과 예시

```
./decompiled
├── com
│   └── example
│       ├── App.java
│       ├── service
│       │   ├── UserService.java
│       │   └── OrderService.java
│       └── model
│           ├── User.java
│           └── Order.java
└── summary.txt
```

> [!note]
> 패키지 구조(`com/example/...`)가 그대로 보존되며, `summary.txt`에 처리 결과 요약이 남습니다.

### Windows PowerShell 예시 (실전)

```powershell
# 작업 폴더로 이동
cd C:\workspace\decompile-test

# CFR로 jar 디컴파일
java -jar C:\tools\cfr-0.152.jar `
    .\spring-petclinic-3.2.0.jar `
    --outputpath .\out
```

---

## 4단계: 자주 쓰는 옵션

CFR은 옵션이 매우 많지만, 실전에서 자주 쓰는 것은 일부입니다.

| 옵션                                | 설명                                                              |
| --------------------------------- | --------------------------------------------------------------- |
| `--outputpath <dir>`              | 출력 디렉터리 지정 (jar 일괄 처리 시 거의 필수)                                 |
| `--outputdir <dir>`               | (구버전 호환) `--outputpath`와 동일                                    |
| `--silent true`                   | 상세 로그 숨김 (스크립트에서 사용 시 권장)                                      |
| `--comments false`                | 디컴파일러가 삽입하는 주석 제거                                              |
| `--decodeenumswitch true`         | enum switch 패턴 자동 복원 (기본값 true)                                |
| `--decodestringswitch true`       | String switch 패턴 자동 복원 (기본값 true)                              |
| `--sugarasserts true`             | `assert` 문법 복원                                                  |
| `--hideutf true`                  | non-ASCII 문자열을 escape로 표시                                       |
| `--showversion false`             | 헤더의 CFR 버전 정보 숨김                                                |
| `--analyseas JAR`                 | 입력이 jar임을 명시 (보통 자동 감지되므로 생략 가능)                              |
| `--jarfilter <regex>`             | jar 내부에서 디컴파일할 클래스만 정규식으로 필터링                                 |

### 옵션 조합 예시

깔끔한 결과물을 원할 때:

```bash
java -jar cfr-0.152.jar example-app.jar `
    --outputpath ./decompiled `
    --comments false `
    --showversion false `
    --silent true
```

특정 패키지만 디컴파일하고 싶을 때 (`com.example.service` 이하만):

```bash
java -jar cfr-0.152.jar example-app.jar `
    --outputpath ./decompiled `
    --jarfilter "com/example/service/.*"
```

---

## 5단계: 결과물 검증

디컴파일이 끝나면 `summary.txt`를 먼저 확인합니다.

```
OK: com/example/App.class
OK: com/example/service/UserService.class
WARN: com/example/util/Mangled.class — could not fully decompile (obfuscated?)
```

- `OK` — 정상 복원
- `WARN` / `ERROR` — 난독화(obfuscation) 또는 unusual bytecode 가능성

> [!warning] 난독화된 코드
> ProGuard, R8, Allatori 등으로 난독화된 jar는 변수명·메서드명이 `a`, `b`, `c`처럼 의미 없게 복원됩니다.
> CFR이 잘못 디컴파일한 것이 아니라, 원본이 그렇게 빌드된 것입니다.

### IDE에서 열어 검토하기

복원된 폴더를 IntelliJ IDEA / VS Code 등의 IDE에서 열면 syntax highlighting과 코드 네비게이션을 활용할 수 있습니다.
컴파일 가능한 형태로 복원되는 경우가 많아, 그대로 Maven/Gradle 프로젝트에 붙여 빌드 검증도 가능합니다.

---

## 응용: PowerShell 함수로 등록

자주 쓴다면 PowerShell 프로필(`$PROFILE`)에 함수를 등록해두면 편합니다.

```powershell
function Invoke-Cfr {
    param(
        [Parameter(Mandatory)] [string]$Jar,
        [string]$Out = ".\decompiled"
    )
    java -jar C:\tools\cfr-0.152.jar $Jar `
        --outputpath $Out `
        --comments false `
        --showversion false `
        --silent true
    Write-Host "Done → $Out" -ForegroundColor Green
}

Set-Alias cfr Invoke-Cfr
```

이후로는 짧게 호출 가능합니다:

```powershell
cfr .\example-app.jar
cfr .\example-app.jar -Out .\custom-out
```

---

## 자주 마주치는 문제

> [!question] `Error: Unable to access jarfile cfr-0.152.jar`
> CFR jar 파일 경로가 올바른지 확인하세요.
> 절대 경로(`C:\tools\cfr-0.152.jar`)로 지정하면 가장 안전합니다.

> [!question] 출력 폴더는 비어있는데 에러도 없음
> 입력 파일이 jar가 아니라 단일 `.class`라면 `--outputpath`가 무시되고 표준 출력으로 결과가 나갑니다.
> jar로 묶어서 실행하거나, 리다이렉트(`> out.java`)로 저장하세요.

> [!question] `// $FF: Couldn't be decompiled` 가 나옴
> 일부 메서드가 너무 복잡하거나 bytecode가 비정상적일 때 표시됩니다.
> 다른 디컴파일러(Procyon, Fernflower)와 비교해 결과를 교차 검증해보세요.

> [!question] 한글이 깨져 보임
> 콘솔 인코딩 문제일 수 있습니다. PowerShell에서:
> ```powershell
> chcp 65001       # UTF-8로 변경
> $OutputEncoding = [System.Text.Encoding]::UTF8
> ```

---

## 사용 시 주의 사항 (라이선스·법적)

> [!important]
> 디컴파일은 강력한 도구지만, 무엇이든 디컴파일해도 되는 것은 아닙니다.

- 본인이 작성한 코드의 디버깅 / 잃어버린 소스 복구
- 오픈소스 라이브러리의 동작 확인 / 학습
- 보안 연구·취약점 분석 (책임 있는 공개 전제)
- 호환성 검증을 위한 분석

위와 같은 합법적 사용에 한해 활용하세요.
상용 소프트웨어를 무단으로 디컴파일하는 것은 라이선스 위반 또는 저작권 침해가 될 수 있습니다.

---

## 정리

- CFR은 **단일 jar 하나로** 동작하는 가장 가벼운 Java 디컴파일러
- `java -jar cfr.jar <input.jar> --outputpath <dir>` 한 줄로 전체 jar 풀어내기 가능
- 자주 쓰는 옵션은 `--outputpath`, `--silent`, `--comments`, `--jarfilter` 정도
- `summary.txt`로 결과 검증 → IDE에서 코드 탐색
- 난독화된 jar는 한계가 있음 — 다른 디컴파일러와 교차 검증

---

## 참고 링크

- [CFR 공식 사이트](https://www.benf.org/other/cfr/)
- [CFR GitHub Repository](https://github.com/leibnitz27/cfr)
- [Maven Central — org.benf:cfr](https://central.sonatype.com/artifact/org.benf/cfr)
- [Procyon Decompiler](https://github.com/mstrobel/procyon)
- [Fernflower (IntelliJ)](https://github.com/JetBrains/intellij-community/tree/master/plugins/java-decompiler/engine)
