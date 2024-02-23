## CI(Continuous Integration)
- 개발자들이 작성한 코드를 공유 저장소에 정기적으로 통합하는 접근 방식이다.
- 이 과정에서 코드는 자동화된 빌드와 테스트를 거쳐 통합되기 때문에 오류가 발생하면 즉시 알 수 있다.
- 예를 들어, 오랜 시간 동안 변경된 코드가 통합되지 않다가 올라가게 되면 conflict가 발생하므로 해결하는 데 오랜 시간이 소요된다.
- 장점 : 작은 변경 사항이 자주 통합되므로, merge conflict가 발생할 가능성이 줄어들고, 발생한 경우에도 작은 단위의 문제로 나누어 빠르게 해결할 수 있다. 

## CD(Continuous Deployment)
- 변경 사항이 저장소에 병합되면 자동으로 프로덕션 환경에 배포되는 것을 의미한다.

## Github Actions
- CI/CD tool 중 하나로, YAML(야물) 언어를 사용하는 .yml 파일을 읽어 CI/CD를 수행한다.

## 용어
### Workflow
- 자동화된 전체 프로세스를 나타내며, 여러 Job으로 구성된다.
- 특정 Event에 의해 예약되거나 트리거된다.
  - 트리거 : 어느 특정한 동작에 반응해 자동으로 필요한 동작을 실행하는 것
- GitHub Actions에서는 .github/workflows/*.yml 파일에 정의된다.

### Event
- Workflow를 트리거하는 활동 또는 규칙
- 예를 들어, 특정 브랜치로의 push나 pull request 등이 해당된다.

### Jobs
- Workflow 내에서 실행되는 개별 작업 단위
- VM(Virtual Machine) 환경에서 실행된다.
  - VM 환경에서 실행되는 작업은 주로 빌드, 테스트, 배포 등과 같은 CI/CD 작업을 수행한다. 이를 통해 GitHub Actions는 다양한 환경에서 코드를 실행하고 테스트하여 안정성을 확인하고, 배포 준비를 하게된다.
- 하나의 Job은 여러 개의 Step으로 구성되며, 각 Job은 병렬로 실행될 수 있다.

### Step
- Job 내에서 순차적으로 수행되는 프로세스 단위

### Action
- Workflow 내에서 수행되는 가장 작은 작업 단위
- Step 내에서 수행되는 독립적인 명령
- 사용자가 직접 생성하거나, Github Marketplace에서 가져올 수 있다.

### Runner
- Workflow가 실행될 인스턴스

## 기본 Android yml 파일
```yml
# 해당 yml 파일의 이름
# 설정된 이름으로 Actions 탭의 workflows 항목에서 확인 가능
name: Android CI

# Event에 대해 작성하는 부분
# main으로 push나 pr 이벤트가 발생했을 때를 workflow를 실행시키는 트리거로 사용
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

# Job에 대해 작성하는 부분
jobs:
  build:

    # Job이 실행 될 OS : ubuntu, macos, window
    # -lastest : 제공하는 OS의 가장 최신 버전
    runs-on: ubuntu-latest

    # 하이픈(-)을 통해 구분되어 작성되어 있는 항목들이
    # Step 구문에 작성되어 있는 Job에서 수행되는 프로세스들을 의미한다.
    steps:
      - uses: actions/checkout@v3 # 해당 Step에서 수행할 Action
      - name: set up JDK 11 # 해당 Step의 이름
        uses: actions/setup-java@v3 # 해당 Step에서 실행될 커맨드 라인
        with: # 해당 Step에서 수행되는 Action에 정의되는 Parameter. Key-Value 형태
          java-version: '11'
          distribution: 'temurin'
          cache: gradle

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew
      - name: Build with Gradle
        run: ./gradlew build

```
- actions/checkout@v3 : 소스코드를 가져온다.
- actions/setup-java@v3 : 작업을 수행하기 위한 JDK를 설정한다.

## local.properties를 사용하여 키 관리하기
https://sonseungha.tistory.com/623

## ktlint check
- CI를 할 때 ktlint도 확인해줄 수 있다.
- 단, project 단의 build.gradle 파일에 ktlint dependency를 추가해주어야 한다.
- Gradle Groovy 설정 (build.gradle)
```kotlin
buildscript {
    repositories {
        maven {
            url 'https://plugins.gradle.org/m2/'
        }
    }
    dependencies {
        classpath 'org.jlleitschuh.gradle:ktlint-gradle:12.1.0'
    }
}

plugins {
    id 'org.jlleitschuh.gradle.ktlint' version '12.1.0'
}
```
- Gradle Kotlin 설정 (build.gradle.kts)
```kotlin
buildscript {
    repositories {
        maven(url = "https://plugins.gradle.org/m2/")
    }
    dependencies {
        classpath("org.jlleitschuh.gradle:ktlint-gradle:9.1.0")
    }
}
plugins {
    id("org.jlleitschuh.gradle.ktlint") version "9.1.0"
}
```
- yml 파일 추가
```yml
# steps 아래 작성
# ktlint test
- name: Run ktlint
  run: ./gradlew ktlintCheck 
```

## Gradle 캐싱
- https://kotlinworld.com/399
```yml
- name: Cache Gradle packages
    uses: actions/cache@v3
    with:
      path: |
        ~/.gradle/caches
        ~/.gradle/wrapper
      key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties', '**/buildSrc/**/*.kt') }}
      restore-keys: |
        ${{ runner.os }}-gradle-
```

## 최종
```yml
name: Android CI

on:
  # main, dev, feat branch pr 올리면 아래 jobs 수행
  pull_request:
    branches:
      - 'main'
      - 'develop'
      - 'feat/*'

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
      # timestamp
      - name: Print start time
        run: |
          echo "Workflow started at $(date)"
          
      # code branch checkout
      - uses: actions/checkout@v3
      - name: set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: gradle

      # gradle 캐싱 작업
      - name: Cache Gradle packages
        uses: actions/cache@v3
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties', '**/buildSrc/**/*.kt') }}
          restore-keys: |
            ${{ runner.os }}-gradle-
  
      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Build with Gradle
        run: ./gradlew build

      # ktlint test
      - name: Run ktlint
        run: ./gradlew ktlintCheck
```

#### 참고한 사이트
- https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions
- https://heegs.tistory.com/92
- https://zzsza.github.io/development/2020/06/06/github-action/
- https://happy-kmc.tistory.com/43
- https://blog.benelog.net/ktlint.html#gradle_%EB%B9%8C%EB%93%9C_%EC%84%A4%EC%A0%95