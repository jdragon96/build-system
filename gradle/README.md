# Gradle 테스트

## 1. 프로젝트 구성

```bash
gradle init --type java-application --dsl kotlin
```

```
📦프로젝트
 ┣ 📂.gradle --------- (1)
 ┣ 📂app
 ┃ ┣ 📂build
 ┃ ┣ 📂src
 ┃ ┗ 📜build.gradle.kts --------- (2)
 ┣ 📂build
 ┃ ┗ 📂reports
 ┣ 📂gradle --------- (3)
 ┃ ┣ 📂wrapper
 ┃ ┃ ┣ 📜gradle-wrapper.jar
 ┃ ┃ ┗ 📜gradle-wrapper.properties
 ┃ ┗ 📜libs.versions.toml --------- (4)
 ┣ 📜.gitattributes
 ┣ 📜.gitignore
 ┣ 📜gradle.properties
 ┣ 📜gradlew
 ┣ 📜gradlew.bat
 ┣ 📜README.md
 ┗ 📜settings.gradle.kts
```

> (1) : 프로젝트 캐시파일  
> (2) : app 서브프로젝트를 위한 빌드 스크립트  
> (3) : wrapper 파일을 위해 생성된 폴더  
> (4) : 의존성을 위한 버전 카탈로그

</br>
</br>

## 2. Gradle 파일 살펴보기

### 2.1. settings.gradle.kts
```kotlin
plugins {
  id("org.gradle.toolchains.foojay-resolver-convention") version "0.8.0"
}

rootProject.name = "gradle_test"
include("app")
```

* `rootProject.name`은 빌드에 할당되는 이름
  * 폴더 이름보다 우선순위가 높음
* `include(app)`은 `app`이라고 불리는 하나의 서브 프로젝트로 구성됨
  * app은 자신만의 코드 및 빌드 로직을 포함
  * 더 많은 서브 프로젝트는 `include()` 명령으로 추가 가능

### 2.2. app/build.gradle.kts
```kotlin
// Java에서 CLI 어플리케이션 빌드 지원을 위해 추가되는 플러그인
// JVM을 호출하는 역할을 함
plugins {
  id("application")                                               
}

// 의존성 해결을 위한 Maven Central 저장소를 이용
repositories {
  mavenCentral()                                                  
}

dependencies {
  // 테스트를 위해 JUnit 주피터를 사용
  testImplementation(libs.junit.jupiter)                          
  testRuntimeOnly("org.junit.platform:junit-platform-launcher"
  // 어플리케이션이 사용하는 의존성
  implementation(libs.guava)                                      
}

java {
  toolchain {
    // 툴체인 버전
    languageVersion = JavaLanguageVersion.of(11)                
  }
}

application {
  // 어플리케이션을 위한 메인 클래스 정의
  mainClass = "org.example.App"                                   
}

tasks.named<Test>("test") {
  // 단위 테스트를 위해 JUnut 플랫폼을 사용
  useJUnitPlatform()                                              
}
```

</br>
</br>

## 3. 실행 및 배포
### 3.1. 어플리케이션 실행
```bash
# Windows
$.\gradlew.bat run
# macOS
$ .\gradlew run
```

### 3.2. 번들링
* `application` 플러그인은 의존성과 함께 해당 어플리케이션을 감싸줌
* 아카이브에는 단일 명령으로 어플리케이션을 실행 할 스크립트를 포함
  * 번들링 성공 시, 아래 파일이 생성됨  
    * app/build/distribution/app.jar
    * app/build/distribution/app.zip
```bash
./gradlew.bat build
```

### 3.3. 빌드 스캔 배포하기
* `--scan` 옵션을 통해 배포
```bash
$ ./gradlew build --scan
```

</br>
</br>

## 4. 빌드 라이프사이클
### Phase 1. 초기화
* settings.gradle.kts 파일 탐색
* Gradle은 어떤 프로젝트가 빌드에 사용 될 것인지 걸정
* 각 프로젝트 마다 `Project` 인스턴스를 생성

### Phase 2. 설정
* 각 프로젝트의 빌드 스크립트 탐색
* Gradle은 실행 가능한 Task의 묶음을 결정

### Phase 3. 실행
* Gradle은 선택된 Task를 호출

</br>
</br>


## 5. 멀티 프로젝트 빌드
### 5.1. 하위 프로젝트 만들기

</br>

1. lib 폴더 생성
```bash
mkdir lib
mkdir -p lib/src/main/java/com/gradle
```
</br>

2. build.gradle.kts 파일 생성

```kotlin
plugins {
  id("java")
}

repositories {
  mavenCentral()
}

dependencies {
  teorm:junit-platform-launcher")
  implestImplementation("org.junit.jupiter:junit-jupiter:5.9.3")  
  testRuntimeOnly("org.junit.platfmentation("com.google.guava:guava:32.1.1-jre")
}
```

</br>

3. settings.gradle.kts 수정

```kotlin
plugins {
  id("org.gradle.toolchains.foojay-resolver-convention") version "0.7.0"
}
rootProject.name = "authoring-tutorial"
include("app")
// 새 라이브러리 추가
include("lib")
```

</br>

4. app/build.gradle.kts 파일 수정
```kotlin
dependencies {
  // 라이브러리 의존성 추가하기
  implementation(project(":lib")) 
}
```

</br>

### 5.2. 빌드 추가하기(플러그인)

1. 새 프로젝트 만들기

```
cd gradle
mkdir license-plugin
cd license-plugin
gradle init --dsl kotlin --type kotlin-gradle-plugin --project-name license
```

</br>

2. gradle/license-plugin/settings.gradle.kts 수정하기

```kotlin
rootProject.name = "license"
include("plugin")
```

</br>

3. settings.gradle.kts 수정하기

```kotlin
plugins {
  id("org.gradle.toolchains.foojay-resolver-convention") version "0.7.0"
}

rootProject.name = "authoring-tutorial"

include("app")
include("lib")

// 새로 추가한 프로젝트
includeBuild("gradle/license-plugin") 
```

</br>

4. 확인하기
```kotlin
$ ./gradlew.bat projects


Root project 'gradle_test'
+--- Project ':app'
\--- Project ':lib'

Included builds:
\--- Included build ':license-plugin'
```

</br>

5. 빌드 관련 명령어
```bash
# Builds app and lib
./gradlew build
# Builds app and lib
./gradlew :app:build
# Builds lib only
./gradlew :lib:build
# Builds license-plugin only
./gradlew :license-plugin:plugin:build
```

</br>
</br>

# 6. Settings 파일 작성하기
## 6.1. 선행지식
* Java 앱 만드는 방법
* Gradle 빌드 라이프사이클
* 서브프로젝트와 별도의 빌드 추가하기

## 6.2. Gradle 스크립트

* 빌드 스크립트와 세팅 파일은 Kotlin 또는 Groovy 코드
  * [Gradle API](https://docs.gradle.org/current/javadoc/org/gradle/api/initialization/Settings.html)
  * [Kotlin Setting API](https://docs.gradle.org/current/kotlin-dsl/gradle/org.gradle.kotlin.dsl/-kotlin-settings-script/index.html)


## 6.3. Settings 객체

* settings 파일은 Gradle 빌드의 시작점
  * 초기화 단계에서 Gradle은 root 프로젝트에서 settings 파일을 탐색
  * settings.gradle(.kts) 탐색 성공 시, Settings 객체를 생성


## 참고자료
https://docs.gradle.org/current/userguide/part3_multi_project_builds.html