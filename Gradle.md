
## Features

### Gradle Version Catalogs

* [Gradle Version Catalogs로 의존성 관리하기](https://cheonjaeung.com/posts/manage-dependencies-with-gradle-version-catalogs/)

Gradle 버전 카탈로그는 **Gradle 7.4**에서 도입된 기능으로, 전체 프로젝트에서 의존성 좌표와 버전을 중앙에서 일관되게 관리할 수 있게 해줍니다. 각 모듈의 빌드 파일마다 버전과 의존성을 따로 선언하는 대신, 단일 카탈로그 파일에 정의하여 확장성과 가독성을 높이고, 버전 불일치를 방지할 수 있습니다[1][3].

### 핵심 개념 및 장점

- 버전 카탈로그는 `libs.versions.toml` 파일을 통해 라이브러리 버전과 플러그인 정보를 한 곳에서 관리합니다.  
- 각 프로젝트나 모듈에서는 이 카탈로그를 참조해 의존성과 플러그인을 선언합니다.  
- IDE에서 타입 안전한 자동완성 기능을 지원하여 개발 편의성이 향상됩니다.  
- 대규모 프로젝트에서 의존성 중복 제거와 일괄 버전 업그레이드가 쉬워집니다.  
- 버전 충돌 해결 기능은 아니며, 선언된 버전은 기본 참조용입니다[1][3][4].

### `libs.versions.toml` 기본 구조

```toml
[versions]
# 버전 변수 선언
kotlin = "1.9.0"
spring = "6.0.11"

[plugins]
# 플러그인 선언: id와 참조하는 버전 변수 포함
spring-boot = { id = "org.springframework.boot", version.ref = "spring" }

[libraries]
# 라이브러리 의존성 선언: 그룹, 이름, 버전 변수 참조
kotlin-stdlib = { group = "org.jetbrains.kotlin", name = "kotlin-stdlib", version.ref = "kotlin" }
spring-boot-starter = { group = "org.springframework.boot", name = "spring-boot-starter", version.ref = "spring" }
```

`version.ref` 키를 통해 버전 변수를 참조해 관리가 쉽도록 하며, 필요할 경우 직접 버전을 지정할 수도 있습니다[1][3].

### 사용법

- `libs.versions.toml` 파일을 프로젝트 `gradle/` 디렉토리에 생성합니다. Gradle이 자동으로 인식합니다.  
- 각 모듈의 `build.gradle(.kts)` 파일에서 의존성 선언 시 `libs.<별칭>`으로 참조합니다.  
- 플러그인은 최상위 `build.gradle`에서 `alias`로 정의 후, 실제 모듈에서 적용합니다.  

예)

```kotlin
dependencies {
    implementation(libs.kotlin.stdlib)
    implementation(libs.spring.boot.starter)
}
```

```kotlin
plugins {
    alias(libs.plugins.spring.boot) apply false
}

plugins {
    alias(libs.plugins.spring.boot)
}
```

### Apache Iceberg 프로젝트의 Gradle 버전 카탈로그

Apache Iceberg 저장소에서는 `gradle/libs.versions.toml` 파일을 통해 주요 라이브러리 버전과 플러그인 ID, 라이브러리 모듈을 선언하여 여러 서브모듈 간 의존성을 일관되게 관리합니다.  

예)

```toml
[versions]
guava = "32.1.2-jre"
jackson = "2.15.2"

[plugins]
java-library = { id = "java-library" }

[libraries]
guava = { group = "com.google.guava", name = "guava", version.ref = "guava" }
jackson-core = { group = "com.fasterxml.jackson.core", name = "jackson-core", version.ref = "jackson" }
```

각 모듈은 `libs.guava`와 같이 선언된 별칭을 사용해 관리하며, 중앙 집중식으로 버전을 통제해 유지보수와 확장에 용이합니다.
