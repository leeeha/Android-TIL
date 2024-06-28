# **1. Introduction**

클린 코드 철학은 소프트웨어 장인 정신의 기초에서 나온 것으로 보입니다. 소프트웨어 장인 정신 선언문에 서명한 많은 소프트웨어 전문가들에 따르면, **잘 만들어지고 설명하기 쉬운 소프트웨어를 작성하는 것은 작동하는 소프트웨어를 작성하는 것만큼이나 중요하다**고 합니다.

다음은 이 책에서 발췌한 내용으로, 네이밍이 어렵고 논쟁이 있는 이유와 이 문제를 해결하기 위한 마음가짐을 보여줍니다:

> *"좋은 이름을 고를 때 가장 어려운 점은 좋은 설명 능력과 문화적 배경을 공유해야 한다는 점입니다. 이는 기술, 비즈니스 또는 관리 문제라기보다는 교육 문제입니다.”*
> 

> *"코드가 단락과 문장처럼 읽히는지, 아니면 최소한 표와 데이터 구조처럼 읽히는지에 집중하세요.”*
> 

# 2. Naming Conventions

## 2.1. Use Intention-Revealing Names

: **의도가 드러나는 이름을 사용하라.** 

기본적으로 변수, 클래스, 메서드의 이름은 주석을 적지 않아도 이해할 수 있도록 지으세요. 

```java
int d; // elapsed time in days  
int s; // elapsed time in seconds
```

```java
int elapsedTimeInDays;  
int daysSinceCreation; 
int daysSinceModification;  
int fileAgeInDays;
```

## 2.2. Avoid Disinformation and Encodings

**: 잘못된 정보나 불필요한 인코딩을 피하라.** 

실제로는 목록이 아닌 계정 그룹을 'accountList'라고 부르지 마세요. 대신 "accountGroup", "accounts" 또는 "bunchOfAccounts"로 이름을 지정하세요.

변수 이름과 함께 데이터 타입을 불필요하게 인코딩하지 마세요.

```java
String nameString;
Float salaryFloat;
```

직원 이름에 일련의 문자가 포함될 것이라는 사실이 전 세계에 잘 알려진 경우에는 필요하지 않을 수 있습니다. 십진수/부동 소수점인 Salary도 마찬가지입니다.

## 2.3. Make Meaningful Distinctions

**: 의미 있게 구분 지어라.** 

같은 범위에 서로 다른 두 가지가 있는 경우, 한 가지 이름을 임의로 변경하고 싶은 유혹을 느낄 수 있습니다. 

customer와 다른 cust에 저장되는 것은 무엇인가요?

```java
String cust;  
String customer;
```

아래 클래스들은 무엇이 다른가요?

```java
class ProductInfo 
class ProductData
```

같은 클래스에서 아래와 같은 메서드는 어떻게 구분할 수 있나요?

```java
void getActiveAccount()
void getActiveAccounts()
void getActiveAccountsInfo()
```

따라서 여기서 제안하는 것은 의미 있게 구분 지으라는 것입니다. 

## 2.4. Use Pronounceable Names

**: 발음할 수 있는 이름을 사용하라.** 

이름이 발음 가능한지 확인하세요. 혀가 꼬이는 이름을 원하는 사람은 아무도 없습니다.

```java
Date genymdhms;
Date modymdhms;
Int pszqint;
```

vs. 

```java
Date generationTimeStamp;
Date modificationTimestamp;
Int publicAccessCount;
```

## 2.5. Use Searchable Names

**: 검색 가능한 이름을 사용하라.** 

텍스트 본문에서 쉽게 찾을 수 없는 한 글자 이름이나 리터럴 상수는 피하세요.

```java
grep 15 *.java
```

vs.

```java
grep "MAX_STUDENTS_PER_TEACHER"  *.java
```

짧은 메서드에서 로컬 변수로 사용되는 경우에만 한 글자 이름을 사용하세요. 한 가지 중요한 제안은 "**이름의 길이는 해당 이름이 사용되는 범위의 크기와 일치해야 한다**"는 것입니다.

## 2.6. Don't Be Cute, Don't Use Offensive Words

: 귀엽거나 공격적인, 즉 **감정적인 단어를 사용하지 말라.** 

```java
deleteAllItems() // best 
vs.
whack()
vs.
kill()
vs.
eatMyShorts()
vs.
abort()
```

## 2.7. Pick One Word per Concept 

**: 하나의 개념당 한 단어만 사용하라.** 

코드베이스 전체에서 동일한 개념을 사용하세요. 

**fetchValue() vs. getValue() vs. retrieveValue()** 

모든 코드에서 fetchValue()를 사용하는 대신 getValue(), retrieveValue()를 사용하면 독자에게 혼란을 줄 수 있으므로 동일한 개념을 사용하세요.

**dataFetcher() vs dataGetter() vs dataFetcher()** 

세 가지 메서드가 모두 동일한 작업을 수행하는 경우 코드 베이스 전체에서 혼합하여 사용하지 마세요. 대신 하나만 사용하세요.

**FeatureManager vs. FeatureController**

관리자 또는 컨트롤러 중 하나를 사용하세요. 다시 말하지만, **동일한 작업에 대해 코드베이스 전체에서 일관성을 유지하세요.**

## 2.8. Don’t Pun

**: 말장난 하지 마라. (다른 목적인데 같은 단어 사용 X)** 

이는 이전 규칙과 정반대입니다. **두 가지 다른 목적에 같은 단어를 사용하지 마세요.** 서로 다른 두 가지 아이디어에 같은 용어를 사용하는 것은 말장난에 가깝습니다.

예를 들어 동일한 add() 메서드를 가진 두 개의 클래스가 있다고 가정해 봅시다. 한 클래스에서는 두 값의 **덧셈을 수행**하고 다른 클래스에서는 **요소를 목록에 삽입**합니다. 따라서 두 번째 클래스에서는 add() 대신 insert() 또는 append()를 사용하는 것이 현명합니다. 

## 2.9. Solution Domain Names vs Problem Domain Names

클린 코드 철학은 필요할 때마다 알고리즘이나 디자인 패턴 이름과 같은 **솔루션 도메인 이름**을 사용하고, **문제 도메인 이름**을 두 번째 선택 사항으로 사용하도록 제안합니다. 

예를 들어, Visitor 디자인 패턴에 익숙한 프로그래머에게 accountVisitor는 많은 의미를 가지게 됩니다. 프로그래머에게는 문제에 대한 해결 방법을 내포하고 있는 "JobQueue"가 더 의미 있을 것입니다.

참고로 "문제 도메인"과 "솔루션 도메인"이라는 용어는 개발 과정에서 중요한 개념을 나타냅니다.

- 문제 도메인 (Problem Domain): 이는 **개발자가 해결하려는 문제의 영역**을 의미합니다. 그것은 특정 시스템이나 소프트웨어가 해결하려고 하는 **실제 세계의 문제나 요구사항을 포함**합니다. 예를 들어, 은행 시스템을 개발한다면, 문제 도메인은 계좌 관리, 자금 이체, 대출 처리 등이 될 수 있습니다.
- 솔루션 도메인 (Solution Domain): 이는 **문제 도메인의 요구 사항을 해결하는데 사용되는 기술 및 방법론**을 참조합니다. 이는 프로그래밍 언어, 데이터베이스 관리 시스템, 사용하는 프레임워크, 설계 패턴 등을 포함할 수 있습니다. 즉, **문제를 해결하는 방법**에 초점을 둡니다.

따라서, 개발 과정에서는 먼저 문제 도메인을 이해하고, 그 후에 적절한 솔루션 도메인을 적용하여 문제를 해결하게 됩니다. 문제와 해결이라는 이 두 가지 도메인 사이의 잘못된 매핑은 실패한 프로젝트의 주요 원인 중 하나입니다.

## 2.10. Add Meaningful Context as a Last Resort

**: 최후의 수단으로 의미있는 컨텍스트를 추가하라.** 

그 자체로 의미가 있는 이름도 있지만 대부분은 그렇지 않습니다. 대신 이름을 잘 명명된 클래스, 함수 또는 네임스페이스에 묶어 독자의 맥락에 맞게 이름을 배치해야 합니다. 이것이 불가능하다면 최후의 수단으로 이름 앞에 접두사를 붙이는 것이 필요할 수 있습니다.

예를 들어, Address 클래스 내부의 "state"라는 멤버 변수는 그 안에 포함된 내용을 보여줍니다. 그러나 동일한 "state"를 문맥 없이 "주 이름"을 저장하는 데 사용하면 오해의 소지가 있을 수 있습니다. 따라서 이 시나리오에서는 "addressState" 또는 "StateName"과 같은 이름을 지정합니다.

주소에 대한 코딩을 할 때 "state"는 "addrState"로 명명하는 것이 더 의미 있을 수 있습니다.

# 3. Summary

로버트 C. 마틴은 "똑똑한 프로그래머와 전문 프로그래머의 한 가지 차이점은 전문가는 **명확성이 가장 중요하다**는 것을 이해한다는 점이다. 프로는 **자신의 능력을 좋은 일에 사용하고 다른 사람이 이해할 수 있는 코드를 작성**합니다." 라고 말합니다. 

이 글이 여러분이 더 전문적이고 깔끔한 코드를 작성하는 데 도움이 되기를 바라며, 더 자세한 내용은 클린 코드 책을 참조하세요.

# 참고자료

[이 글](https://dzone.com/articles/naming-conventions-from-uncle-bobs-clean-code-phil)을 번역해서 작성한 글입니다.
