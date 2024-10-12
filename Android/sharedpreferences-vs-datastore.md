## SharedPreferences vs Datastore 

작고 간단한 데이터 세트를 저장할 때 과거에는 SharedPreferences를 주로 사용했지만, 이 API는 몇가지 단점을 갖고 있다. 

Jetpack에서 제공하는 Datastore 라이브러리는 이러한 문제점을 해결하며, 더 간단하고 안전하고 비동기적으로 데이터를 저장할 수 있는 방법을 제시한다. 

다음과 같은 2가지 구현을 제공한다. 

- Preferences Datastore
- Proto Datastore

|  | Shared Preferences | Preferences DataStore | Proto Datastore | 
| --- | --- | --- | --- |
| 비동기 API | O (리스너를 통해 변경된 값을 읽을 때만) | O (via Flow) | O (via Flow) |
| 동기 API | O (but, UI 스레드에서 호출하기에 안전하지 않음) | X | X |
| UI 스레드에서 호출하기에 안전 | X -- (1) | O (Dispatchers.IO 에서 작업 처리) | O (Dispatchers.IO 에서 작업 처리) |
| 오류 신호 전송 | X | O  | O |
| 런타임 예외로부터 안전 | X -- (2)| O | O |
| 강한 일관성을 보장하는 트랜잭션 API | X | O | O |
| 데이터 마이그레이션 처리 | X | O | O |
| 타입 안전성 | X | X | O (with 프로토콜 버퍼) |

(1) SharedPreferences에서 디스크 I/O 작업을 수행하는 동기식 API가 UI 스레드를 차단할 수 있다. 또한 apply() 함수는 fsync() 함수에서 UI 스레드를 차단한다. Service 또는 Activity가 시작되거나 중지될 때마다, apply()에 의해 예약되어 있던 fsync() 함수가 호출되어 UI 스레드를 차단할 수 있다. 이는 ANR의 원인이 되기도 한다. 

(2) SharedPreferences는 파싱 오류를 런타임 예외로 처리한다.

## Preferences Datastore vs Proto Datastore

- `Preference Datastore` : SharedPreferences와 마찬가지로 스키마를 정의하지 않은 상태에서 **key-value 쌍**을 기반으로 데이터를 읽고 쓴다.
- `Proto Datastore` : **프로토콜 버퍼를 사용하여 스키마를 정의**한다. Protobuf를 사용하기 때문에 **강하게 타입이 지정된 (strongly typed)** 데이터를 유지할 수 있다. 이러한 데이터는 XML 등 다른 유사한 데이터 형식보다 빠르고 작고 간결하며 덜 모호하다. Proto Datastore를 사용하려면 새로운 직렬화 메커니즘을 배워야 하지만, Proto Datastore의 strongly typed의 이점이 그만한 가치가 있습니다.

cf) [프로토콜 버퍼](https://protobuf.dev/) : 구조화된 데이터를 **직렬화**하기 위한, 언어 중립적이고 플랫폼 중립적이며 확장 가능한 Google의 메커니즘이다. XML과 비슷하지만 더 작고, 더 빠르고, 더 간단하다. 데이터를 어떻게 구조화할지 스키마를 정의하면, 특수하게 생성된 소스 코드를 사용하여 **다양한 데이터 스트림과 다양한 언어를 사용하여 구조화된 데이터를 쉽게 쓰고 읽을 수 있다.**

## Room vs Datastore 

**부분적인 업데이트, 참조 무결성 또는 대규모의 복잡한 데이터 세트가 필요한 경우에는 DataStore 대신 Room 사용을 고려**해야 한다. DataStore는 작거나 단순한 데이터 세트에 적합하며 부분적인 업데이트나 참조 무결성을 지원하지 않는다. 

## 참고자료 

https://developer.android.com/codelabs/android-preferences-datastore#3