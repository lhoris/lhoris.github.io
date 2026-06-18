---
title: Fernflower Java Decompiler 사용법 — 직접 빌드해서 JAR 디컴파일하기
date: 2026-06-19
tags:
  - guide
  - java
  - decompiler
  - fernflower
  - reverse-engineering
---

**IntelliJ IDEA에 내장된 그 디컴파일러, Fernflower를 단독 CLI로 써봅니다.**

`Fernflower`는 JetBrains가 유지보수하는 Java용 분석형(analytical) 디컴파일러입니다.
IntelliJ IDEA에서 `.class` 파일을 열면 보이는 그 결과물을 만들어내는 엔진이 바로 이 도구입니다.
공식 릴리스 jar가 별도로 배포되지 않기 때문에, **소스에서 직접 빌드해서 jar를 얻어내는 과정**부터 다룹니다.

> 공식 저장소: [JetBrains/fernflower](https://github.com/JetBrains/fernflower)
> 공식 README: [README.md](https://github.com/JetBrains/fernflower/blob/master/README.md)

관련 글: [[cfr-java-decompiler|CFR Java Decompiler 사용법]]

---

## 왜 Fernflower인가?

| 디컴파일러                | 특징                                                              |
| -------------------- | --------------------------------------------------------------- |
| **Fernflower**       | JetBrains 공식 유지보수, IntelliJ 내장 엔진과 동일, 식별자 자동 리네임(`ren`) 강력 |
| [[cfr-java-decompiler\|CFR]] | 단일 jar 배포로 가장 가볍게 시작 가능, 최신 문법 복원 우수                          |
| Procyon              | 오래된 클래스 파일에 강함, 업데이트가 느림                                        |
| JD-GUI               | GUI 친화적이나 최신 문법에서 결과가 어색한 경우 있음                                |

> [!tip]
> **IntelliJ에서 보던 디컴파일 결과를 CLI에서 그대로** 얻고 싶다면 Fernflower가 정답입니다.
> 난독화된 jar에서 의미 없는 식별자(`a`, `b`, `c`)를 자동으로 `class_1`, `method_2` 같은 고유 이름으로 바꿔주는 `ren` 옵션이 특히 유용합니다.

---

## 사전 준비

- [x] **JDK 11 이상** 설치 (`java -version`으로 확인) — Gradle 빌드용
- [x] **Git** 설치
- [x] 디컴파일할 대상 jar 파일

> [!info]
> Fernflower는 CFR과 달리 **사전 빌드된 jar를 공식 사이트에서 받을 수 없습니다.**
> IntelliJ Community 저장소 안에 묶여 배포되거나, 소스에서 직접 빌드해야 합니다.
> 이 글에서는 직접 빌드해 jar를 얻는 방법을 다룹니다.

---

## 1단계: Fernflower 빌드 (jar 얻기)

### 1-1. 소스 클론

원하는 작업 폴더로 이동해서 저장소를 클론합니다.

```bash
git clone https://github.com/JetBrains/fernflower.git
cd fernflower
```

### 1-2. Gradle로 빌드

저장소에 포함된 Gradle Wrapper로 빌드합니다. (Gradle을 별도 설치할 필요 없습니다.)

```bash
./gradlew :installDist
```

처음 실행 시 Gradle 배포본을 자동 다운로드하므로 1~2분 정도 걸립니다.

```
Downloading https://services.gradle.org/distributions/gradle-9.1.0-bin.zip
............10%.............20% ... 100%

BUILD SUCCESSFUL in 1m 26s
4 actionable tasks: 4 executed
```

### 1-3. 빌드 산출물 위치 확인

빌드가 성공하면 다음 경로에 jar가 생성됩니다.

```
fernflower/build/install/fernflower/
├── bin/
│   ├── fernflower         # Linux/macOS 실행 스크립트
│   └── fernflower.bat     # Windows 실행 스크립트
└── lib/
    ├── fernflower.jar     # ← 우리가 쓸 jar
    └── annotations-26.1.0.jar
```

`fernflower.jar`를 원하는 도구 폴더로 복사해 두면 편합니다.

```bash
mkdir -p /c/tools
cp build/install/fernflower/lib/fernflower.jar /c/tools/fernflower.jar
```

PowerShell:

```powershell
Copy-Item .\build\install\fernflower\lib\fernflower.jar C:\tools\fernflower.jar
```

> [!note]
> `bin/fernflower` 스크립트를 그대로 써도 되지만, **`java -jar` 직접 실행**이 옵션 전달이 가장 명확합니다. 이 글에서는 jar 직접 실행 기준으로 설명합니다.

### 동작 확인

```bash
java -jar /c/tools/fernflower.jar
```

옵션 없이 실행하면 사용법(usage)이 출력됩니다. 정상 동작 시그널입니다.

---

## 2단계: 기본 명령어 구조

Fernflower의 CLI 문법은 CFR보다 자유로운 편입니다.

```
java -jar fernflower.jar [-<option>=<value>]* [<source>]+ <destination>
```

- `-<option>=<value>` — 옵션 (0개 이상, 자세한 표는 4단계 참고)
- `<source>` — 디컴파일할 입력 (1개 이상). `.class`, `.zip`, `.jar`, 디렉터리 모두 가능
- `<destination>` — **출력 디렉터리** (반드시 마지막 인자)

> [!warning] 인자 순서가 중요합니다
> 마지막 인자는 **항상 출력 디렉터리**여야 합니다.
> 출력 디렉터리를 빠뜨리면 마지막 입력 파일이 출력 폴더로 오해되어 엉뚱한 결과가 나옵니다.

---

## 3단계: JAR 파일 디컴파일 (메인)

`example-app.jar` 전체를 한 번에 디컴파일해 보겠습니다.

### 기본 명령어

```bash
mkdir decompiled
java -jar fernflower.jar example-app.jar ./decompiled
```

출력 폴더가 미리 존재해야 합니다. 없으면 `mkdir`로 먼저 만들어주세요.

### 실행 결과 예시

```
./decompiled
└── example-app.jar
```

> [!important] 출력 형태
> Fernflower는 jar 입력에 대해 **디컴파일된 .java가 들어있는 새 jar(zip)** 를 만들어냅니다.
> 디렉터리로 풀어내려면 `unzip` / `Expand-Archive`를 한 번 더 거쳐야 합니다.

PowerShell에서 풀기:

```powershell
Expand-Archive -Path .\decompiled\example-app.jar -DestinationPath .\decompiled\src
```

Linux/macOS:

```bash
unzip ./decompiled/example-app.jar -d ./decompiled/src
```

`./decompiled/src/` 아래에 패키지 구조 그대로 `.java` 파일이 풀립니다.

### Windows PowerShell 실전 예시

```powershell
# 작업 폴더로 이동
cd C:\workspace\decompile-test

# 출력 폴더 생성
New-Item -ItemType Directory -Force .\out | Out-Null

# Fernflower로 jar 디컴파일 (난독화 대비 ren 옵션 포함)
java -jar C:\tools\fernflower.jar `
    -ren=1 `
    -dgs=1 `
    .\spring-petclinic-3.2.0.jar `
    .\out

# 결과 jar 풀기
Expand-Archive -Path .\out\spring-petclinic-3.2.0.jar -DestinationPath .\out\src

# 결과 확인
Get-ChildItem -Recurse .\out\src -Filter *.java | Select-Object -First 10
```

---

## 4단계: 자주 쓰는 옵션

Fernflower의 옵션은 짧은 약자(`hes`, `dgs`, `ren` 등) 형태입니다.
모든 옵션은 `-<key>=<value>` 형식으로 전달하며, 대부분 `1`(켜기) / `0`(끄기)입니다.

README가 강조하는 **사용자가 실제로 자주 바꾸는 옵션**은 다음과 같습니다.

| 옵션          | 기본값 | 설명                                                                |
| ----------- | --- | ----------------------------------------------------------------- |
| `-hes=`     | 1   | 비어있는 super() 호출 숨김                                                |
| `-hdc=`     | 1   | 비어있는 기본 생성자 숨김                                                    |
| `-dgs=`     | 0   | **제네릭 시그니처 복원** (켜는 걸 권장)                                         |
| `-mpm=`     | 0   | 메서드당 최대 처리 시간(초). 0은 무제한                                          |
| `-ren=`     | 0   | **난독화된 식별자 자동 리네임** (난독화 jar에서 매우 유용)                             |
| `-urc=`     | -   | 사용자 정의 리네이머 클래스 (`IIdentifierRenamer` 구현)                          |
| `-din=`     | 1   | 내부 클래스 디컴파일                                                       |
| `-das=`     | 1   | `assert` 문법 복원                                                    |
| `-den=`     | 1   | enum 복원                                                           |
| `-udv=`     | 1   | 디버그 정보가 있으면 변수명 복원                                                |
| `-ump=`     | 1   | 파라미터 이름 속성이 있으면 복원                                                |
| `-lac=`     | 0   | lambda를 익명 클래스 형태로 디컴파일                                           |
| `-asc=`     | 0   | 문자열 내 non-ASCII를 Unicode escape로                                  |
| `-log=`     | INFO | 로그 레벨 (`TRACE`/`INFO`/`WARN`/`ERROR`)                              |
| `-ind=`     | 3 spaces | 들여쓰기 문자열                                                          |
| `-crp=`     | 0   | record pattern 사용                                                 |
| `-cps=`     | 0   | switch with patterns 사용                                            |
| `-e=`       | -   | **라이브러리 파일 지정** (디컴파일 대상은 아니지만 분석 시 참고)                            |

### 옵션 조합 예시

깔끔한 결과 + 제네릭 복원:

```bash
java -jar fernflower.jar `
    -dgs=1 `
    -hes=1 -hdc=1 `
    example-app.jar `
    ./out
```

난독화된 jar (식별자 자동 리네임):

```bash
java -jar fernflower.jar `
    -ren=1 `
    obfuscated.jar `
    ./out
```

라이브러리 정보 함께 제공 (`-e=`로 컨텍스트 확장):

```bash
java -jar fernflower.jar `
    -dgs=1 `
    -e=./libs/spring-core-6.1.0.jar `
    -e=./libs/jackson-databind-2.17.0.jar `
    example-app.jar `
    ./out
```

`-e=` 옵션은 디컴파일 대상은 아니지만, 클래스/메서드 간 관계 분석에 사용됩니다.
**`ren` 옵션과 함께 쓸 때 특히 결과 품질이 좋아집니다.**

> [!info] README가 말하는 "전문가용" 옵션
> `rbr`, `rsy`, `dc4`, `ner`, `rgn`, `bto`, `nns`, `uto`, `rer`, `fdi`, `inn`, `cci` 등은
> 기본값 그대로 두는 게 좋습니다. (워크어라운드 / 미세 튜닝 옵션)

---

## 5단계: 결과물 검증

### 풀어낸 .java 파일 확인

`Expand-Archive`로 풀어낸 `out/src/` 폴더를 IntelliJ IDEA / VS Code에서 그대로 엽니다.
패키지 구조가 보존되어 있어 IDE의 코드 네비게이션이 곧바로 동작합니다.

### IntelliJ 결과와 비교

IntelliJ IDEA에서 같은 `.class` 파일을 열어 결과와 비교해 보면 사실상 동일합니다.
**CLI에서도 IDE 수준의 디컴파일을 얻을 수 있다는 것**이 Fernflower의 강점입니다.

> [!warning] 난독화된 코드
> `-ren=1`을 켜도 의미 있는 이름이 아니라 `class_1`, `method_2` 같은 고유 이름으로 바뀝니다.
> 원본의 의미를 복원하는 게 아니라, 충돌 없이 재컴파일 가능하게 만들어주는 용도입니다.

---

## 응용: PowerShell 함수로 등록

자주 쓴다면 PowerShell 프로필(`$PROFILE`)에 함수로 등록해두면 편합니다.

```powershell
function Invoke-Fernflower {
    param(
        [Parameter(Mandatory)] [string]$Jar,
        [string]$Out = ".\decompiled",
        [switch]$Extract
    )
    if (-not (Test-Path $Out)) {
        New-Item -ItemType Directory -Force $Out | Out-Null
    }
    java -jar C:\tools\fernflower.jar `
        -ren=1 -dgs=1 `
        $Jar `
        $Out

    if ($Extract) {
        $name = [System.IO.Path]::GetFileNameWithoutExtension($Jar)
        Expand-Archive -Path (Join-Path $Out (Split-Path $Jar -Leaf)) `
                       -DestinationPath (Join-Path $Out $name) -Force
        Write-Host "Extracted → $(Join-Path $Out $name)" -ForegroundColor Green
    }
    Write-Host "Done → $Out" -ForegroundColor Green
}

Set-Alias ff Invoke-Fernflower
```

이후로는 짧게 호출 가능합니다:

```powershell
ff .\example-app.jar
ff .\example-app.jar -Out .\custom-out -Extract
```

---

## 자주 마주치는 문제

> [!question] `./gradlew: Permission denied` (Linux/macOS)
> 실행 권한이 없는 경우입니다.
> ```bash
> chmod +x ./gradlew
> ```

> [!question] Windows에서 `./gradlew`가 안 먹힘
> Git Bash가 아니라 PowerShell이라면 `.\gradlew.bat`을 사용하세요.

> [!question] 출력 폴더에 jar만 덩그러니 생김
> 의도된 동작입니다. Fernflower는 jar 입력 → jar 출력입니다.
> `Expand-Archive` / `unzip`으로 한 번 더 풀어야 `.java` 파일을 볼 수 있습니다.

> [!question] `Error: destination directory does not exist`
> 마지막 인자(출력 디렉터리)가 미리 존재해야 합니다.
> `mkdir out` / `New-Item -ItemType Directory -Force .\out`으로 먼저 만들어 주세요.

> [!question] 디컴파일 결과 식별자가 죄다 `a`, `b`, `c`
> 원본이 난독화된 상태입니다. `-ren=1`을 켜면 충돌 없는 고유 이름으로 자동 변경됩니다.

> [!question] 빌드한 jar 경로가 README와 다름
> 현재(`master`) 빌드 산출물은 `build/install/fernflower/lib/fernflower.jar`에 생성됩니다.
> README에 적힌 `build/install/engine/bin` 경로는 과거 모듈 이름 시점의 설명일 수 있습니다.
> 실제 빌드 결과 폴더(`build/install/`)를 한 번 살펴보고 확인하는 게 가장 정확합니다.

---

## 사용 시 주의 사항 (라이선스·법적)

> [!important]
> Fernflower 자체는 [Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0)으로 배포되지만,
> **디컴파일 대상의 라이선스는 별개**입니다.

- 본인이 작성한 코드의 디버깅 / 잃어버린 소스 복구
- 오픈소스 라이브러리의 동작 확인 / 학습
- 보안 연구·취약점 분석 (책임 있는 공개 전제)
- 호환성 검증을 위한 분석

위와 같은 합법적 사용에 한해 활용하세요.
상용 소프트웨어를 무단으로 디컴파일하는 것은 라이선스 위반 또는 저작권 침해가 될 수 있습니다.

---

## 정리

- Fernflower는 **IntelliJ IDEA 내장 디컴파일러의 CLI 버전** — JetBrains 공식 유지보수
- 공식 릴리스 jar가 없어서 **소스 클론 → `./gradlew :installDist`** 로 직접 빌드
- 결과 jar 위치: `build/install/fernflower/lib/fernflower.jar`
- 사용법: `java -jar fernflower.jar [-옵션=값]* <입력...> <출력디렉터리>`
- **출력은 디렉터리가 아니라 jar** — `Expand-Archive` / `unzip`으로 풀어서 사용
- 자주 쓰는 옵션: `-dgs=1`(제네릭), `-ren=1`(난독화 리네임), `-e=`(라이브러리 컨텍스트)
- 난독화 jar는 [[cfr-java-decompiler|CFR]]과 교차 검증 권장

---

## 참고 링크

- [JetBrains/fernflower (공식 저장소)](https://github.com/JetBrains/fernflower)
- [Fernflower README — CLI 옵션 전체](https://github.com/JetBrains/fernflower/blob/master/README.md)
- [IntelliJ Community Repository (릴리스)](https://www.jetbrains.com/intellij-repository/releases)
- [Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0)
- 관련 글: [[cfr-java-decompiler|CFR Java Decompiler 사용법]]
