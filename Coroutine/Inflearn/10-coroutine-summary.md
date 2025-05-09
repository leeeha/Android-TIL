코루틴은 

- 비동기 프로그래밍에서 **callback hell을 해결**했고
- kotlin 언어 키워드가 아닌 **라이브러리로써 동작**한다는 특징을 갖고 있다. 

이런 특징을 활용해 코루틴은 **비동기 non-blocking이나 동시성이 필요한 곳에서 적극적으로 사용**될 수 있다! 

예를 들어, **클라이언트에서는 비동기 UI를 구성하는데 사용**할 수 있다. 비슷하게 **서버에서도 여러 API(I/O)를 동시에 호출**해야 한다면 활용할 수 있다. 

추가적으로 webflux와 같은 프레임워크에서도 코루틴을 사용할 수 있고, **동시성 테스트**를 할 때도 코루틴을 적용해볼 수 있다.

### 참고 자료 

- [2시간으로 끝내는 코루틴, 최태현 ](https://www.inflearn.com/course/2%EC%8B%9C%EA%B0%84%EC%9C%BC%EB%A1%9C-%EB%81%9D%EB%82%B4%EB%8A%94-%EC%BD%94%EB%A3%A8%ED%8B%B4)
