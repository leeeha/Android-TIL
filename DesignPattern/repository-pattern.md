# Repository Pattern 

<img width="700" src="https://github.com/user-attachments/assets/0fb5ad36-d44e-4c20-b632-e95b7b3a6c2f"/>

Repository 패턴은 **데이터 출처 (Data Source)와 상관없이, 동일한 인터페이스로 데이터를 사용할 수 있게 해주는 패턴**을 의미한다. 

예를 들어, Repository 패턴은 Room으로 관리되는 데이터 (Local data source)와, Retrofit2을 이용해 서버에서 받아오는 데이터 (Remote data source)의 구분을 없애, **Domain Layer에서는 Data Source의 구현 방법을 알 필요 없이 그냥 사용하기만 하면 된다.** 

이처럼 **Data Layer를 캡슐화** 함으로써 얻을 수 있는 이점은 다음과 같다. 

- Data Layer가 바뀌어도 Domain Layer는 영향을 받지 않거나 최소한만 받을 수 있다. 즉, **계층 간의 결합도가 감소**한다. 
- Presentation Layer는 Repository 인터페이스를 통해, **일관된 방식으로 데이터를 요청**할 수 있다. 
- **단위 테스트**를 통한 검증이 쉬워진다. 

단점으로는 Data Layer에 대한 직접적인 접근을 막기 위해 인터페이스를 사용함에 따라, **필요한 코드와 파일의 양이 증가한다**는 것이다.

# 참고 자료 

https://heegs.tistory.com/90
