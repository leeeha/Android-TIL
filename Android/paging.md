# Paging 라이브러리 소개 

[Paging 라이브러리](https://developer.android.com/topic/libraries/architecture/paging/v3-overview?hl=ko)를 사용하면 **로컬 저장소나 네트워크로부터 대규모 데이터를 페이지 단위로 로드하고 표시**할 수 있다. 

Paging 라이브러리의 구성요소는 [Android 앱 아키텍처](https://developer.android.com/topic/architecture?hl=ko) 가이드에 맞게 설계되었으며, 다른 Jetpack 구성요소와 원활하게 통합된다. 

# Paging 라이브러리 사용 시 이점 

어플리케이션에서 대규모 데이터를 한번에 로드하려고 하면, **메모리 사용량이 급증하고 어플리케이션의 응답성이 떨어지며, 이는 좋지 않은 사용자 경험으로 이어지게 된다.** 

그래서 대규모 데이터는 보통 페이징 처리를 하는 것이 일반적이다. 페이지 단위로 **필요한 데이터만 부분적으로 로드하여, 네트워크 대역폭을 줄이고 메모리 같은 시스템 리소스도 절약**할 수 있다. 

이때 Paging 라이브러리를 사용하면 얻을 수 있는 이점은 다음과 같다.

- **페이징 된 데이터의 메모리 내 캐싱**을 지원하여, 네트워크 통신 부담을 줄이고 사용자 응답성 증가 
- **중복 요청 삭제 기능**을 기본으로 제공하여, 네트워크 대역폭과 시스템 리소스 모두 효율적으로 사용 가능 
- 사용자가 로드된 데이터의 끝까지 스크롤 할 때, **RecyclerView Adapter가 자동으로 다음 페이지 요청** 
- **Kotlin Coroutine의 Flow** 뿐만 아니라 **LiveData, RxJava와의 호환성**도 좋다. 
- **새로고침 및 재시도 기능**을 포함하여, 오류 처리를 기본으로 지원 
- **Jetpack Compose**에서는 Adapter 없이도 무한 스크롤 구현 가능 

# Paging 라이브러리 아키텍처 

<img width="600" src="https://github.com/user-attachments/assets/ffc808ec-ec37-4fe2-96c2-bb9c475d4e16" />

Paging 라이브러리를 사용해서 데이터를 가져오는 방법은 크게 3가지로 나뉜다. 

1. 로컬 DB에서 내부 데이터를 가져오는 경우 
2. 서버 API 등 네트워크 통신으로 외부 데이터를 가져오는 경우 
3. 서버 API로 가져온 외부 데이터를 로컬 DB에 캐싱하는 경우 

어떤 상황에서든 PagingConfig와 PagingSource는 필수적으로 구현해야 되고, RemoteMediator는 3번 상황에서만 구현하면 된다. 

# 참고 자료

https://developer.android.com/topic/libraries/architecture/paging/v3-overview?hl=ko

https://dev-inventory.com/16