# Activity Result API

다른 액티비티를 실행시키는 것은 단방향 작업이 아니어도 된다. 즉, **액티비티를 시작하고 그 결과를 받을 수도 있다.** 예를 들어, 앱에서 카메라 앱을 시작하고 그 결과로 촬영된 사진을 얻을 수도 있고, 연락처 앱을 시작하고 그 결과로 연락처에 대한 세부정보를 얻을 수도 있다. 

`startActivityForResult()`, `onActivityResult()` API는 모든 API 수준의 액티비티 클래스에서 이용 가능하지만, 구글은 AndroidX의 액티비티, 프래그먼트 클래스에 도입된 `Activity Result API` 사용을 적극 권장하고 있다. 

---

## 기존 startActivityForResult의 문제점

기존에 사용하던 startActivityForResult 함수는 2020년 5월 기준으로 deprecated 되었다. 그 이유에 대해 알아보기 위해 기존 방식의 코드부터 살펴보자. 

```kotlin
btn.setOnClickListener {
    val intent = Intent(this, WriteActivity::class.java)
    startActivityForResult(intent, 0)
}
```

위 코드는 버튼 클릭 시, 0이라는 requestCode를 갖고 WriteActivity로 전환하는 코드이다. 

WriteActivity에서 작업을 마치고 돌아올 때 requestCode를 그대로 갖고 원래 액티비티로 돌아오게 된다. 그러면 **원래 액티비티는 requestCode에 따라 어떤 액티비티로부터 돌아온 건지 구분**하게 된다. 이때 사용되는 메서드가 onActivityResult이다.

### onActivityResult 함수의 코드량 증가 

```kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)

    when (requestCode) {
        0 -> {
            // 작업
        }

        1 -> {
            // 작업
        }
    }
}
```

위와 같이 requestCode를 통해 어떤 액티비티로부터 돌아온 건지 구분하고, 그에 따라 작업을 처리하면 된다. 그런데 이 방식은 프로젝트 규모가 커질수록 유지보수 하기 힘들어진다. 

프로젝트 규모가 커질수록 액티비티의 수도 늘어나고 그에 따라 onActivityResult 함수에서 처리하는 코드량도 엄청나게 많아질 것이다. 결과를 전달하는 모든 액티비티에 대해 requestCode에 따라 분기 처리를 해줘야 하기 때문이다. 이는 유지보수 하기 힘든 코드가 된다. 

### 복잡한 퍼미션 요청 코드

```kotlin
// startActivityForResult와 같은 로직
requestPermissions(requiredPermissions, 0)
 
// onActivityResult와 같은 로직
override fun onRequestPermissionsResult(requestCode: Int, permissions: Array, grantResults: IntArray) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults)
    
    when(requestCode) {
        0 -> {
        
        }
    }
}
```

퍼미션 요청 시에도 마찬가지다. requestPermissions, onRequestPermissionsResult 함수는 이름과 역할은 다르지만, startActivityForResult, onActivityResult 함수와 작동하는 로직이 동일하다. 

> requestCode에 따라 분기 처리 → 해당 권한의 유무에 따라 분기 처리 → 퍼미션 개수만큼 반복

요청해야 하는 퍼미션 개수가 많을수록 코드가 훨씬 복잡해지게 된다. 

### requestCode가 갖는 의미의 모호함 

또한 requestCode에 임의의 숫자를 부여하게 되는데, 이 숫자가 갖는 의미가 애매하다는 문제점도 있다. 개발자가 정하기 나름인 값이기 때문에 다른 사람이 코드를 읽었을 때 해당 숫자가 의미하는 액티비티가 무엇인지 알아차리기 어렵다. 

이러한 이유로 startActivityForResult는 deprecated 되었고, registerForActivityResult가 그 역할을 대신하고 있다.

# 액티비티 결과에 대한 콜백 등록

결과를 받기 위한 액티비티를 시작할 때 **메모리 부족으로 프로세스와 액티비티가 소멸**될 수 있다. 특히 카메라 사용과 같이 메모리를 많이 사용하는 작업의 경우에는 소멸될 확률이 매우 높다. 따라서, **Activity Result API는 다른 액티비티를 실행하는 코드와 그 결과를 처리하는 콜백을 분리**한다.

결과 콜백은 프로세스와 액티비티가 재생성될 때도 사용할 수 있어야 하므로, (다른 액티비티를 실행하는 로직이 사용자 입력 또는 기타 비즈니스 로직을 기반으로만 발생하더라도) **액티비티가 재생성될 때마다 콜백을 무조건 등록해야 한다.**

Activity Result API에서 제공하는 `registerForActivityResult()` API로 **결과 콜백을 등록**할 수 있다. 이는 `ActivityResultContract`, `ActivityResultCallback` 객체를 인자로 받아, 다른 액티비티를 실행시키는데 필요한 `ActivityResultLauncher` 를 반환한다. 

`ActivityResultContract`는 결과를 생성하는 데 필요한 **입력 타입**, 결과의 **출력 타입**을 정의한다. 이 API는 사진 촬영, 권한 요청 등과 같은 기본 인텐트 작업에 대한 [기본 Contract](https://developer.android.com/reference/androidx/activity/result/contract/ActivityResultContracts)를 제공한다. [커스텀 Contract](https://developer.android.com/training/basics/intents/result?hl=ko#custom)를 생성할 수도 있다.

`ActivityResultCallback`은 `ActivityResultContract`에 정의된 출력 타입의 객체를 가져오는 `onActivityResult()` 메서드가 포함된 싱글 메서드 인터페이스이다.

```java
// GetContent creates an ActivityResultLauncher<String> 
// to let you pass in the mime type you want to let the user select
ActivityResultLauncher<String> mGetContent = registerForActivityResult(new GetContent(),
    new ActivityResultCallback<Uri>() {
        @Override
        public void onActivityResult(Uri uri) {
            // Handle the returned Uri
        }
});
```

```kotlin
val getContent = registerForActivityResult(GetContent()) { uri: Uri? ->
    // Handle the returned Uri
}
```

결과를 반환하는 액티비티를 여러 개 호출하면서 다른 Contract를 사용하거나 별개의 콜백을 원하는 경우, `registerForActivityResult()`를 여러 번 호출하여 여러 개의 `ActivityResultLauncher` 인스턴스를 등록할 수 있다. 

진행 중인 결과가 올바른 콜백에 전달되도록 프래그먼트나 액티비티를 만들 때마다 `registerForActivityResult()` 를 동일한 순서로 호출해야 한다. 

또한 `registerForActivityResult()` 함수는 **프래그먼트나 액티비티를 만들기 ‘전에’ 호출하는 것이 안전**하므로, 반환되는 `ActivityResultLauncher` 인스턴스의 **멤버 변수를 선언할 때** `registerForActivityResult()` 함수를 직접 사용할 수 있다. 

프래그먼트나 액티비티가 생성되기 ‘전에’ `registerForActivityResult()` 함수를 호출하여 결과 콜백을 등록해야 한다. 그래야 갑작스런 예외 상황으로 인해 액티비티가 파괴되고 재성성되더라도, 결과 콜백은 그래도 유지할 수 있기 때문이다. 그리고 다시 프래그먼트나 액티비티의 라이프사이클이 CREATED 상태에 도달하면, `ActivityResultLauncher`로 새 액티비티를 실행시키면 된다.

## 결과를 받기 위한 액티비티 실행

`registerForActivityResult()` 함수는 결과 콜백을 등록하지만, 다른 액티비티를 실행시키거나 결과에 대한 요청을 시작하지는 않는다. 그 대신에 이 작업은 해당 함수에서 반환된  `ActivityResultLauncher` 인스턴스가 담당한다. 

입력이 들어오면, `ActivityResultLauncher`는 `ActivityResultContract` 타입과 일치하는 입력을 가져온다. 그리고 `launch()` 함수를 호출하면 결과를 생성하는 프로세스가 시작된다. 사용자가 새로 실행된 액티비티에서의 작업을 완료하고 결과를 반환하면, `ActivityResultCallback`의 `onActivityResult()` 메서드가 다음과 같이 실행된다. 

```java
// Java

ActivityResultLauncher<String> mGetContent = registerForActivityResult(new GetContent(),
    new ActivityResultCallback<Uri>() {
        @Override
        public void onActivityResult(Uri uri) {
            // Handle the returned Uri
        }
});

@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    // ...

    Button selectButton = findViewById(R.id.select_button);

    selectButton.setOnClickListener(new OnClickListener() {
        @Override
        public void onClick(View view) {
            // Pass in the mime type you want to let the user select
            // as the input
            mGetContent.launch("image/*");
        }
    });
}
```

```kotlin
// Kotlin 

val getContent = registerForActivityResult(GetContent()) { uri: Uri? ->
    // Handle the returned Uri
}

override fun onCreate(savedInstanceState: Bundle?) {
    // ...

    val selectButton = findViewById<Button>(R.id.select_button)

    selectButton.setOnClickListener {
        // Pass in the mime type you want to let the user select
        // as the input
        getContent.launch("image/*")
    }
}
```

launch() 함수를 오버로딩하여 입력 외에도 ActivityOptionsCompat을 전달할 수도 있다. 

```java
// ActivityOptionsCompat: Additional options for how the Activity should be started.
public abstract void launch(I input, @Nullable ActivityOptionsCompat options)
```

>Notice: launch() 함수를 호출하는 시점과 onActivityResult() 콜백이 트리거 되는 시점 사이에 프로세스와 액티비티가 소멸될 수도 있으므로, 결과를 처리하는 데 필요한 추가적인 상태는 이러한 API와 별도로 저장하고 복원해야 한다.

## 주의할 점 요약 

액티비티 A에서 새로운 액티비티 B를 띄우고 그 결과를 반환 받으려 한다고 가정하자. 

그런데, 공식문서에 따르면 메모리 부족으로 프로세스나 액티비티가 소멸될 수 있다고 했다. 

예를 들어, 액티비티 A에서 B를 띄웠을 때 메모리 부족으로 **액티비티 A가 소멸 후 재생성** 되고, 액티비티 B는 원래대로 결과를 반환하려고 한다면? 

`registerForActivityResult()` 함수로 등록해뒀던 액티비티 A의 결과 콜백이 제대로 작동하지 않을 수도 있다!! 

따라서, 액티비티가 소멸되었다가 재생성 될 때도 항상 액티비티에 결과 콜백이 등록되어 있어야 한다. 이러한 이유로 공식문서에는 **프래그먼트나 액티비티를 생성하기‘전에’ 클래스의 멤버 변수로 결과 콜백을 등록하는 것이 안전하다**고 적혀있는 것이다.

그렇지 않으면 다음과 같은 오류가 발생할 수 있다. 

>java.lang.IllegalStateException: Fragment DiscoverFragment{b9d2bfb} (7b91fb92-382c-409e-9ecd-5397974bbf92 id=0x7f0a0130) is attempting to registerForActivityResult after being created. Fragments must call registerForActivityResult() before they are created (i.e. initialization, onAttach(), or onCreate())

>프래그먼트 생성 이후에 registerForActivityResult 함수에 접근하려고 하고 있다. 프래그먼트는 반드시 자신이 생성되기 '전에' registerForActivityResult 함수를 호출해야 한다.  

# 참고자료

[활동에서 결과 가져오기  |  Android 개발자  |  Android Developers](https://developer.android.com/training/basics/intents/result?hl=ko)

[startActivityForResult는 왜 deprecated 되었는가?](https://todaycode.tistory.com/121)

[[Android] Activity Result API : Fragment에서 registerForActivityResult()](https://junyoung-developer.tistory.com/159?category=960204)

[[Android] registerForActivityResult()란?](https://velog.io/@ho-taek/Android-registerForActivityResult란)