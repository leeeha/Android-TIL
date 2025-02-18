# WebView 개념

WebView는 **네이티브 앱에 내재되어 있는 웹 브라우저**를 의미한다. 

- View 클래스의 확장으로, **웹 페이지를 액티비티 레이아웃의 일부로 표시**할 수 있다.
- 탐색 컨트롤이나 주소 표시줄 등 완전히 개발된 웹 브라우저의 기능은 포함되어 있지 않으며, WebView의 모든 작업은 기본적으로 **웹 페이지를 표시**하는 것이다.

# WebView 장단점

### 장점

- **여러 플랫폼에서 사용 가능**
    - 하나의 웹 페이지만 만들어도 Android, iOS에서 모두 사용할 수 있으므로, 초기 개발 비용을 최소화하고 유지보수도 편리하다.
- **배포 없이 업데이트 가능**
    - 앱 스토어 심사 없이도 웹사이트 내용을 수정할 수 있다. 따라서, 자주 바뀌거나 빠르게 업데이트가 필요할 때 유용하다.
- **인터넷 연결이 필요한 데이터를 간단히 불러옴.**
    - 이메일 같은 데이터는 항상 인터넷 연결이 필요한데, 네이티브 앱에서 네트워크 요청 & 데이터 파싱 & 레이아웃에 렌더링 하는 과정보다
    - 항상 인터넷에 연결되어 있는 WebView를 사용하여 앱을 빌드하는 것이 더 쉽다.

### 단점

- **비교적 느린 로딩 속도**
    - 네이티브 앱에 비해 로딩 시간이 느린 편이다. 네이티브 앱은 이미 스토어에서 빌드가 완료되지만, 웹뷰는 해당 사이트에서 사용하는 리소스를 다운로드하고 보여주는 데 시간이 더 필요하다. 로딩 시간이 너무 길어지면 사용자 경험에 안 좋은 영향을 미친다.
- **제한적인 UI 구현**
    - 웹뷰는 HTML, CSS, JavaScript를 사용하기 때문에 네이티브 앱의 UI를 구성하는 것보다 제약이 많다.
- **스토어 심사가 어려울 수 있음.**
    - 허가 없는 웹사이트를 무단으로 사용하거나, 웹사이트만 보여주는 단순한 앱이 스토어에 등록되는 것을 방지하고 있기 때문에, 웹뷰만으로 구성된 앱은 스토어 심사가 어려울 수 있다.

# 앱에 WebView 추가

## 기본 설정 (인터넷 권한, 의존성 추가)

```xml 
<manifest ... >
    <uses-permission android:name="android.permission.INTERNET" />
    ...
</manifest>
```

```kotlin
dependencies {
    implementation("androidx.webkit:webkit:1.8.0")
}
```

## 액티비티 레이아웃에 웹뷰 추가

```xml
<WebView
    android:id="@+id/webview"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
/>
```

```kotlin
val myWebView: WebView = findViewById(R.id.webview)
myWebView.loadUrl("http://www.example.com")
```

## onCreate()에서 웹뷰 추가

```kotlin
val myWebView = WebView(activityContext)
setContentView(myWebView)

myWebView.loadUrl("http://www.example.com")
```

## HTML 문자열 인코딩하여 URL 로드

```kotlin
// Create an unencoded HTML string, then convert the unencoded HTML string into bytes. 
// Encode it with base64 and load the data.
val unencodedHtml = "<html><body>'%23' is the percent code for ‘#‘ </body></html>";
val encodedHtml = Base64.encodeToString(unencodedHtml.toByteArray(), Base64.NO_PADDING)
myWebView.loadData(encodedHtml, "text/html", "base64")
```

# WebView 커스텀

## WebChromeClient

- **전체 화면 지원** 설정
- 창을 만들거나 닫고 JavaScript 다이얼로그를 유저에게 전송하는 등 **호스트 앱의 UI를 변경**하는 권한이 필요할 때 사용
- 오버라이드 가능한 메서드 예시
    - `onCreateWindow`: 웹뷰에서 새 창을 띄울 때 호출된다.
    - `onCloseWindow`: 웹뷰에서 창을 닫을 때 호출된다.
    - `onProgressChanged`: 웹뷰가 로딩 중일 때 호출된다. (0~100 사이의 진행도를 파악하여 프로그레스바 표시 가능)
    - `onShowCustomView`, `onHideCustomView`: 웹 페이지가 전체 화면 모드에 진입 및 해제될 때 호출된다.
    - `onConsoleMessage`: 웹뷰의 콘솔 메시지를 Logcat에 표시하여 디버깅 할 때 사용한다.

## WebViewClient

- 네비게이션 오류, 양식 제출 오류 등 **콘텐츠 렌더링에 영향을 미치는 이벤트** 처리 담당
- 서브 클래스를 사용하여 **URL 가로채기** 가능
- 오버라이드 가능한 메서드 예시
    - `onPageStarted`, `onPageFinished`: 페이지가 로드되는 처음과 끝 시점에 호출된다. 로딩 애니메이션을 띄우는 경우, 로드의 시작과 종료에 따라 가시성을 조절한다.
    - `onReceivedError`: 페이지 로드 중에 오류가 발생하면 호출된다. 로그를 남기거나 에러 상황에 따른 추가 동작을 정의할 수 있다. 단, 이 콜백은 메인 페이지 뿐만 아니라 모든 리소스에 대해 호출되므로, 가능한 최소한의 작업만 수행하여 성능에 영향을 덜 미치도록 해야 한다.
    - `shouldOverrideUrlLoading`: 페이지의 URL을 가로채서 어떻게 제어할지 정의한다. false를 반환하면 현재 웹뷰에서 URL을 열고, true를 반환하면 별도로 정의한 스킴에 따라 URL이 열린다. 이때, startActivity 함수를 호출할 때는 try-catch문 등으로 예외 처리를 해주는 것이 좋다.
    - `doUpdateVisitedHistory`: 페이지 로드가 끝나면, WebView 히스토리 스택에 해당 페이지가 쌓이면서 호출된다.

## [WebSettings](https://developer.android.com/reference/android/webkit/WebSettings)

- `javaScriptEnabled` : JavaScript 사용 설정
- `domStorageEnabled` : WebView에서 로컬 스토리지를 사용하는 경우 활성화
- `mediaPlaybackRequiresUserGesture` : 자동 재생이 아닌 사용자 제스처가 필요한 경우 활성화
- `textZoom` : 안드로이드 시스템 설정에 의해 텍스트 크기가 변경되는 것을 방지
- `userAgentString` : 서버 측에서 웹 페이지를 요청하는 클라이언트가 안드로이드 앱인지 확인 가능
- `cacheMode` : WebView 캐시 전략 설정
    - `LOAD_DEFAULT` : 유효한 캐시가 있으면 사용, 캐시가 없거나 만료되었으면 네트워크에서 요청 (디폴트 값)
    - `LOAD_CACHE_ELSE_NETWORK` : 만료된 캐시여도 있으면 사용, 없으면 네트워크에서 요청 (네트워크 불안정한 환경에서 빠른 로딩이 필요할 때)
    - `LOAD_CACHE_ONLY` : 네트워크 요청 없이, 매번 캐시 사용 (오프라인 모드 지원)
    - `LOAD_NO_CACHE` : 캐시 사용 없이, 매번 네트워크 요청 (항상 최신 데이터가 필요할 때)
    

# WebView에서 Web ↔ Android 통신

안드로이드 네이티브 앱과 웹은 서로 분리된 환경이기 때문에 **서로 간의 통신을 위한 연결 다리**인 브릿지가 필요하다. 즉, 브릿지란 **안드로이드와 웹뷰의 통신을 위해 만들어진 JavaScript용 인터페이스**라고 할 수 있다. 안드로이드와 웹뷰는 각자의 환경에 존재하는 메서드를 직접 호출할 수 없으므로 **브릿지를 통해 호출**하게 된다.

### JavaScript 사용 설정

기본적으로 JavaScript는 웹뷰에서 사용 중지되어 있다. 따라서 웹뷰에 연결된 WebSettings를 통해 JavaScript를 사용 설정해줘야 한다. 

```kotlin
val myWebView: WebView = findViewById(R.id.webview)
myWebView.settings.javaScriptEnabled = true
```

### Bridge 인터페이스 정의

```kotlin
/** Instantiate the interface and set the context.  */
class WebAppBridge(private val context: Context) {

    /** Show a toast from the web page.  */
    @JavascriptInterface
    fun showToast(message: String) {
        Toast.makeText(context, message, Toast.LENGTH_SHORT).show()
    }
}
```

```kotlin
val webView: WebView = findViewById(R.id.webview)
webView.addJavascriptInterface(WebAppBridge(this), "WebAppBridge")
```

```kotlin
/**
* object: JavaScript에 연결할 브릿지 클래스의 인스턴스 
* name: JavaScript가 브릿지 클래스에 접근할 때 사용할 이름 
*/
public void addJavascriptInterface(@NonNull Object object, @NonNull String name) {
    checkThread();
    mProvider.addJavascriptInterface(object, name);
}
```

>📌 **주의**: targetSdkVersion를 17 이상으로 설정하는 경우 JavaScript에서 사용할 수 있는 모든 메서드에 `@JavascriptInterface` 주석을 추가해야 한다. 
메서드는 공개 상태여야 한다. 이 주석을 제공하지 않으면 웹 페이지에서 메서드에 액세스할 수 없다. 

## Web → Android

웹뷰의 버튼을 클릭했을 때 안드로이드에서 토스트 메시지가 뜨도록 구현하려면, 다음과 같이 html, JavaScript 코드를 작성할 수 있다. 

```html
<input type="button" value="Say hello" onClick="showAndroidToast('Hello Android!')" />

<script type="text/javascript">
    function showAndroidToast(toast) {
        WebAppBridge.showToast(toast);
    }
</script>
```

WebView가 브릿지 인터페이스를 웹 페이지에서 사용할 수 있도록 자동 설정하기 때문에, JavaScript에서 브릿지 인터페이스를 초기화 할 필요가 없다. (참고로, JavaScript에 결합된 객체는 객체가 생성된 스레드가 아니라 다른 스레드에서 실행된다.) 

## Web ← Android

역으로, 안드로이드에서 문자열을 전달해 웹뷰에서 Alert를 띄우려면 어떻게 해야 할까? 

웹에는 다음과 같은 함수가 정의되어 있다고 가정하자. 

```html
<script>
    function showWebViewAlert(text) {
        alert(text);
    }
</script>
```

안드로이드에서는 웹의 `showWebViewAlert` 함수를 호출하기 위한 메서드를 브릿지 클래스 내부에 정의해주면 된다. 

```kotlin
class WebAppBridge(
    private val context: Context,
    private val webView: WebView,
) {
    ...

    fun showWebViewAlert(text: String) {
        val script = "window.showWebViewAlert('$text');"
        
        webView.evaluateJavascript(script) { result ->
		        Log.d("tag_test", result)
        }
    }
}
```

```kotlin
/**
* script: 웹에서 실행할 JavaScript 코드 
* resultCallback: 스크립트 실행 후 반환 값에 대한 처리 
*/
public void evaluateJavascript (String script, ValueCallback<String> resultCallback)
```

콜백 형태 대신에 `suspendCoroutine`을 활용하여 다음과 같이 코드를 작성할 수도 있다. 

```kotlin
suspend fun showWebViewAlertWithCoroutine(text: String) {
    val script = "window.showWebViewAlert('$text');"
    val result = evaluateJavascriptWithCoroutine(script)
    Log.d("tag_test", result)
}

private suspend fun evaluateJavascriptWithCoroutine(script: String): String {
    return suspendCoroutine { continuation ->
        webView.evaluateJavascript(script) { result ->
            continuation.resume(result)
        }
    }
}
```

# 페이지 탐색 처리

사용자가 웹뷰에서 웹페이지의 링크를 탭하면, 기본적으로 안드로이드에서 URL을 처리하는 앱을 실행한다. 일반적으로 **기본 웹브라우저**가 열리고 목적지 URL이 로드된다. 

이때, URL이 기본 웹브라우저가 아닌 **웹뷰 내에서 열리도록 웹뷰의 동작을 재정의** 하면, 웹 페이지 방문 기록을 통해 **사용자가 앞뒤로 화면을 탐색**하도록 만들 수 있다. 

사용자가 탭한 링크를 웹뷰 내에서 여는 방법을 알아보자! 

1. 링크가 로드되는 위치를 조정하기 위해, `shouldOverrideUrlLoading()` 메서드를 오버라이딩 하는 `WebViewClient` 클래스를 정의한다. 

```kotlin
private class MyWebViewClient : WebViewClient() {

    override fun shouldOverrideUrlLoading(view: WebView?, url: String?): Boolean {
        // url 호스트가 특정 도메인과 일치하는지 확인 
        if (Uri.parse(url).host == "www.example.com") {
            // 앱 내부에서 WebView로 페이지 로드 
            return false 
        }
        
        // 도메인이 일치하지 않는 경우, 다른 액티비티 실행하여 url 처리 
        Intent(Intent.ACTION_VIEW, Uri.parse(url)).apply {
            startActivity(this)
        }
        
        return true
    }
}
```

2. `WebViewClient` 인스턴스를 생성하여, `setWebViewClient()`를 사용해 `WebView`에 제공한다. 

```kotlin
val myWebView: WebView = findViewById(R.id.webview)
myWebView.webViewClient = MyWebViewClient()
```

## 커스텀 URL 처리

`WebView`는 커스텀 URL 스키마를 사용하는 링크를 확인하고, 리소스를 요청할 때 제약 사항을 건다. 예를 들어, `shouldOverrideUrlLoading()` 또는 `shouldInterceptRequest()`와 같은 콜백을 구현하는 경우 `WebView`는 유효한 URL인 경우에만 콜백을 호출한다. 

```html 
// Wrong
<a href="showProfile">Show Profile</a>

// Okay 
<a href="example-app:showProfile">Show Profile</a>
```

```kotlin
// The URL scheme must be non-hierarchical, meaning no trailing slashes.
const val APP_SCHEME = "example-app:"

override fun shouldOverrideUrlLoading(view: WebView?, url: String?): Boolean {
    return if (url?.startsWith(APP_SCHEME) == true) {
        urlData = URLDecoder.decode(url.substring(APP_SCHEME.length), "UTF-8")
        respondToData(urlData)
        true
    } else {
        false
    }
}
```

`shouldOverrideUrlLoading()` API는 주로 특정 URL의 인텐트를 실행하기 위한 것이다. 이를 구현할 때는 `WebView`가 처리하는 URL의 경우 `false`를 반환해야 한다. 인텐트 실행 외의 맞춤 동작도 정의 가능하다. 

>📌 **주의:** `shouldOverrideUrlLoading()` 내에서 `loadUrl()`, `reload()` 등과 유사한 메서드를 호출하면, 앱이 비효율적으로 동작할 수 있다. `false`를 반환하여 `WebView`가 기본 구현으로 URL을 계속 로드하도록 하는 것이 더 효율적이다. 

## 웹 페이지 방문 기록 탐색

- WebView가 URL 로드를 재정의하면, 웹페이지의 방문 기록이 자동으로 누적된다.
- `goBack()`, `goForward()`를 사용하여 방문 기록 페이지를 앞뒤로 탐색할 수 있다.

ex) 기기의 뒤로가기 버튼을 통해 이전 페이지로 돌아가는 경우 

```kotlin
override fun onKeyDown(keyCode: Int, event: KeyEvent?): Boolean {
    // Check whether the key event is the Back button and if there's history.
    if (keyCode == KeyEvent.KEYCODE_BACK && myWebView.canGoBack()) {
        myWebView.goBack()
        return true
    }
    
    // If it isn't the Back button or there isn't web page history, bubble up to
    // the default system behavior. Probably exit the activity.
    return super.onKeyDown(keyCode, event)
}
```

앱에서 AndroidX AppCompat 1.6.0 이상을 사용하는 경우, 코드를 더 간소화 할 수 있다. 

```kotlin
onBackPressedDispatcher.addCallback {
    // Check whether there's history.
    if (myWebView.canGoBack()) {
        myWebView.goBack()
    }
}
```

## 기기 [구성 변경](https://developer.android.com/guide/topics/resources/runtime-changes?hl=ko) 처리

런타임에 사용자가 기기를 회전하거나 IME(Input Method Editor)를 닫는 등 기기의 구성이 변경되면, 액티비티 상태가 변경된다. 이로 인해 **액티비티의 재생성 뿐만 아니라, URL을 로드하는 WebView 객체도 재생성**된다. 

이러한 액티비티의 기본 동작을 수정하려면, Manifest 파일에서 액티비티의 `configChanges` 속성을 변경해줘야 한다. 

```xml 
<activity
	... 
	android:configChanges="fontScale|orientation|screenSize"
	... 
/>
```

| 속성 | 설명 |
| --- | --- |
| `fontScale` | 사용자가 시스템 글꼴 크기를 변경했을 때 `Activity`가 재생성 되지 않도록 방지 |
| `orientation` | 기기 회전 발생 시 `Activity`가 재생성 되지 않도록 방지 |
| `screenSize` | 화면 크기 변화(예: 멀티윈도우, 화면 비율 변경 등) 시 `Activity`가 재생성 되지 않도록 방지 |
| `keyboardHidden` | IME 키보드에 의해 `Activity`가 재생성 되지 않도록 방지  |

`onConfigurationChanged()` 메서드를 오버라이딩하여 처리할 수도 있다. 아래 예시 코드는 전체 화면 모드를 감지하여 웹뷰의 크기를 리사이징 하는 코드이다. 

```kotlin
override fun onConfigurationChanged(newConfig: Configuration) {
    super.onConfigurationChanged(newConfig)

    if (newConfig.orientation == Configuration.ORIENTATION_PORTRAIT) {
        webChromeClient.resizeChildView(displayWidthPixels, displayHeightPixels)

    } else if (newConfig.orientation == Configuration.ORIENTATION_LANDSCAPE) {
        webChromeClient.resizeChildView(displayHeightPixels, displayWidthPixels)
    }
}
```

# Window 관리

기본적으로 **새로운 창을 열려는 요청은 무시**된다. 이는 JavaScript에 의해 열리는 경우나 링크의 `target` 속성에 의해 열리는 경우 모두 해당된다. 사용자는 `WebChromeClient`를 커스텀하여 **다중 창 열기에 대한 동작을 직접 정의**할 수 있다. 

**하지만, 보안 강화를 위해 팝업 및 새 창이 열리는 것을 방지하는 것이 가장 좋다.** 이를 구현하는 가장 안전한 방법은 `setSupportMultipleWindows(true)`를 호출하되, 이 메서드가 의존하는 `onCreateWindow()` 메서드를 오버라이드하지 않는 것이다. 이러한 방식은 `target="_blank"` 속성을 사용하는 모든 페이지가 새 창에서 로드되지 않도록 막아준다. 

# 참고자료

https://developer.android.com/develop/ui/views/layout/webapps/webview?hl=ko

https://docs.tosspayments.com/resources/glossary/webview

https://velog.io/@yuuuzzzin/Android-WebView-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0-1%ED%83%84

https://velog.io/@yuuuzzzin/Android-WebView-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0-2%ED%83%84