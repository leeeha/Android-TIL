# Gradle Version Catalog

Gradle은 Groovy 또는 Kotlin 언어를 이용한 **빌드 자동화 도구**이다. Android Studio를 사용한 이래 안드로이드 개발에서도 적극 활용되고 있으며, **모듈에 의존성 및 플러그인 등을 선언하기 위해 필수적**인 기술이다.

Gradle Version Catalog는 **Gradle의 의존성과 플러그인 추가 및 유지보수를 효율적으로 관리하는 기술**이다.

- 멀티 모듈 사용 시, 의존성과 플러그인 관리를 쉽게 할 수 있다.
- 하나의 버전 목록을 만들어서 여러 모듈이 Type-Safe 하게 참조할 수 있다.

# buildSrc

buildSrc는 Gradle을 사용하는 프로젝트에서 멀티 모듈의 의존성, 플러그인 버전 관리를 필요로 할 때, **공통 Gradle 의존성, 플러그인 경로 및 버전 정보를 명시해두는 모듈**이다.

![image](https://github.com/leeeha/Android-TIL/assets/68090939/03b10c69-1356-4ce7-9441-32dec826bb63)

|  | Gradle Version Catalog | buildSrc |
| --- | --- | --- |
| 버전 검색  | 자동 | 수동 |
| 버전 수정 후 Rebuild | 버전의 영향을 받는 모듈만 | 프로젝트의 전체 모듈 |

# Gradle Version Catalog 사용 이유

Gradle Version Catalog를 사용하는 주된 이유는 **의존성, 플러그인을 하나의 공간에서 관리할 수 있고 유지보수가 용이**하기 때문이다. 

- 멀티 모듈 프로젝트 개발 시, 모듈별로 의존성 또는 플러그인 path를 다르게 선언하는 것을 방지
- 의존성, 플러그인을 불러오기 위해 모듈마다 path 문자열을 하드코딩 하는 것을 방지
- IDE에서 의존성, 플러그인 목록에 대한 코드 어시스턴스 제공
- libs 변수에 접근하여 Type-Safe 하게 의존성 또는 플러그인 참조 가능

# 버전 카탈로그 파일 구성

## 섹션 구분

`libs.versions.toml` 파일은 [versions], [libraries], [plugins] 라는 주요 섹션으로 구성되어 있다.

- [libraries] : 의존성 정보를 담는 변수 정의
- [plugins] : 플러그인 정보를 담는 변수 정의
- [versions] : [libraries], [plugins] 섹션에 정의된 의존성, 플러그인의 버전을 저장하는 변수 정의

## 각 섹션에서 path 지정

Gradle Version Catalog에서 의존성, 플러그인 등의 정보를 담은 변수를 선언할 때, path 키워드를 통해 Gradle 검색 경로를 구분할 수 있다. 

`group`

```kotlin
androidx-core = { group = "androidx.core", name = "core-ktx", version.ref = "core" }
```

- group 키워드로 **여러 모듈을 갖고 있는 그룹의 path 지정**
- name 키워드를 함께 사용해 **특정 모듈 참조** 가능

`module`

```kotlin
androidx-core = { module = "androidx.core:core-ktx", version.ref = "core" }
```

- module 키워드로 **특정 모듈의 path 지정**

`id`

```kotlin
android-application = { id = "com.android.application", version.ref = "agp" }
```

- id 키워드로 **플러그인의 path 지정**

# 기존 코드 마이그레이션

Gradle Version Catalog를 적용할 루트 프로젝트의 `gradle` 폴더 아래에 `libs.versions.toml` 파일을 추가한다. 

Gradle은 기본적으로 `libs.versions.toml` 이라는 파일 이름을 검색하여 의존성, 플러그인의 접근자를 생성하므로 기본 이름을 사용하는 게 좋다. 

```kotlin
[versions]
core = "<version>"

[libraries]
androidx-core = { group = "androidx.core", name = "core-ktx", version.ref = "core" }
```

버전 카탈로그 내 변수는 kebab case (app-haeun-example) 를 사용하여 정의한다. 

![image](https://github.com/leeeha/Android-TIL/assets/68090939/830f3c0a-b35e-49ce-ac06-fa275601eb03)

그러면 더 나은 코드 어시스턴스를 제공 받을 수 있다. 

## 루트 프로젝트의 build.gradle.kts

**AS-IS**

```kotlin
plugins {
  id("com.android.application") version "<version>" apply false
  // ...
}
```

**플러그인 path 선언** 

```kotlin
[versions]
agp = "<version>"

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
```

**TO-BE**

```kotlin
plugins {
  alias(libs.plugins.android.application) apply false
  // ...
}
```

## 모듈의 build.gradle.kts

### plugins 블록

**AS-IS**

```kotlin
plugins {
   id("com.android.application")
}
```

**플러그인 path 선언** 

```kotlin
[versions]
agp = "<version>"

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
```

**TO-BE**

```kotlin
plugins {
   alias(libs.plugins.android.application)
}
```

### dependencies 블록

**AS-IS**

```kotlin
dependencies {
  implementation("androidx.core:core-ktx:<version>")
  implementation("androidx.appcompat:appcompat:<version>")
  implementation("com.google.android.material:material:<version>")
}
```

**의존성 path 선언** 

```kotlin
[versions]
core = "<version>"
appcompat = "<version>"
material = "<version>"

[libraries]
androidx-core = { group = "androidx.core", name = "core-ktx", version.ref = "core" }
androidx-appcompat = { module = "androidx.appcompat:appcompat", version.ref = "appcompat" }
material = { module = "com.google.android.material:material", version.ref = "material" }
```

**TO-BE**

```kotlin
dependencies {
  implementation(libs.androidx.core)
  implementation(libs.androidx.appcompat)
  implementation(libs.material)
}
```

## 대표적인 이슈

![image](https://github.com/leeeha/Android-TIL/assets/68090939/d4b1b00b-7251-418d-982c-a40bb94b2c74)

> *libs can't be called in this context by implicit receiver. Use the explicit one if necessary*

Gradle 8.1 미만 버전에서 발생하는 IntelliJ 상의 오류이다. (참고: [KTIJ-19369](https://youtrack.jetbrains.com/issue/KTIJ-19369/False-positive-cant-be-called-in-this-context-by-implicit-receiver-with-plugins-in-Gradle-version-catalogs-as-a-TOML-file))

오류가 발생하는 파일에 `@Suppress` 어노테이션을 명시하여 해결할 수 있다. 

```kotlin
// TODO: Remove once KTIJ-19369 is fixed
@file:Suppress("DSL_SCOPE_VIOLATION")

plugins { /* ... */ }
```

>Note: If you are using a version of Gradle below 8.1, you need to annotate the `plugins{}` block with `@Suppress("DSL_SCOPE_VIOLATION")` when using version catalogs. Refer to [issue #22797](https://github.com/gradle/gradle/issues/22797) for more info.

Gradle 8.1 이상 버전에서는 위와 같은 문제가 발생하지 않는다. 

## 참고자료

[Migrate your build to version catalogs  |  Android Studio  |  Android Developers](https://developer.android.com/build/migrate-to-catalogs)

[Gradle Version Catalog 소개](https://medium.com/team-aliens/gradle-version-catalog-적용기-上-gradle-version-catalog-소개-264beee22342)

[Migrating to Version Catalog](https://proandroiddev.com/migrating-to-version-catalog-ce737cb94233)

[Version Catalog 도입을 위한 온보딩](https://brunch.co.kr/@purpledev/46)