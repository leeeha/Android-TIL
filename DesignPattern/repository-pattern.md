<img width="700" src="https://github.com/user-attachments/assets/0fb5ad36-d44e-4c20-b632-e95b7b3a6c2f"/>

Repository 패턴은 **데이터 출처 (Data Source)와 상관없이, 동일한 인터페이스로 데이터를 사용할 수 있게 해주는 패턴**을 의미한다. 

예를 들어, Repository 패턴은 Room으로 관리되는 데이터 (Local data source)와, Retrofit2을 이용해 서버에서 받아오는 데이터 (Remote data source)의 구분을 없애, Domain Layer에서는 Data Source의 구현 방법을 알 필요 없이 그냥 사용하기만 하면 된다. 

즉, Data Source의 구현 방법이 바뀌더라도 Domain Layer의 Repository 인터페이스는 이에 영향을 받지 않거나 최소한만 받을 수 있다. 

