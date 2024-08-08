기본 JWT 방식의 인증(보안)을 강화시킨 액세스 & 리프레시 토큰 인증 방식에 대해 알아보자. 

## 리프레시 토큰이 필요한 이유

액세스 토큰만을 이용한 인증 방식의 문제점은 **제 3자가 토큰을 탈취하면 대처가 어렵다**는 것이다. 

액세스 토큰은 발급된 이후 서버에 저장되지 않고 토큰 자체로 검증을 하며 사용자 권한을 인증한다. 따라서, **액세스 토큰이 탈취되면 토큰이 만료되기 전까지 토큰을 획득한 누구나 사용자 권한을 얻게 된다.** 

JWT은 발급 이후 임의 삭제가 불가능하기 때문에, **액세스 토큰에 '유효 시간'을 부여하는 방식으로 탈취 문제에 대응**할 수 있다. 

액세스 토큰의 유효기간을 짧게 하면 토큰을 남용하는 것을 방지할 수 있지만, 유효기간이 짧은 만큼 사용자는 더 자주 로그인을 하여 새롭게 토큰을 발급 받아야 하므로 불편하다는 단점이 있다. 그렇다고 유효기간을 무턱대로 늘리면 토큰을 탈취 당했을 때 보안에 취약해진다. 

이때 필요한 것이 바로 **리프레시 토큰**이다! 

리프레시 토큰도 액세스 토큰과 마찬가지로 JWT이다. 단지 액세스 토큰은 유저 정보 **접근**에 관여하는 토큰이고, 리프레시 토큰은 액세스 토큰의 **재발급**에 관여하는 토큰이라는 점에서 역할의 차이가 있을 뿐이다. 

## 액세스 / 리프레시 토큰 인증 방식

1. 사용자가 서버에 로그인 요청을 보낸다. 
2. 서버는 로그인을 성공시키면서 클라이언트에게 **액세스 토큰, 리프레시 토큰을 동시에 발급**한다. 
3. **서버는 리프레스 토큰을 DB에 저장**하고, **클라이언트는 액세스 토큰, 리프레시 토큰**을 쿠키, 세션, 웹스토리지, **로컬 저장소 등에 저장**하고 요청이 있을 때마다 두 가지 토큰을 헤더에 담아서 보낸다. 
4. 클라이언트가 만료된 액세스 토큰을 서버에 보내면, 서버는 같이 보내진 **리프레시 토큰을 DB에 저장된 것과 비교하여 일치하면 액세스 토큰을 재발급**한다. 
5. 사용자가 **로그아웃 하면 액세스 토큰, 리프레시 토큰을 모두 만료**시킨다. 
6. 새로 로그인 하면 서버로부터 다시 액세스 & 리프레시 토큰을 발급 받고, 서버는 리프레시 토큰을 DB에 저장하여 나중에 토큰 인증 시 사용한다. 

## 토큰 만료에 따른 처리 

| 액세스 토큰 | 리프레시 토큰  |  |
| --- | --- | --- |
| 만료 X | 만료 X | 정상 처리  |
| 만료 O | 만료 X | DB에 저장된 리프레시 토큰으로 인증 과정을 거쳐 액세스 토큰 재발급  |
| 만료 X | 만료 O | 액세스 토큰이 유효하다는 건 이미 인증된 것이므로 바로 리프레시 토큰 재발급  |
| 만료 O | 만료 O | 강제 로그아웃 시키고 다시 로그인 하여, 액세스, 리프레시 토큰 모두 재발급  |

## OkHttp Token Interceptor

[](https://www.notion.so/3647902f79bb40d9b307178ecbadc0a9?pvs=21) 

```kotlin
package org.go.sopt.winey.data.interceptor

import // ...

class AuthInterceptor @Inject constructor(
    @ApplicationContext context: Context,
    private val json: Json,
    private val dataStoreRepository: DataStoreRepository
) : Interceptor {

    override fun intercept(chain: Interceptor.Chain): Response {        
        val originalRequest = chain.request()
        val headerRequest = originalRequest.newAuthBuilder().build()
        val response = chain.proceed(headerRequest)

        when (response.code) {
            CODE_TOKEN_EXPIRED -> { 
                try {
                    Timber.e("액세스 토큰 만료, 토큰 재발급 합니다.")
                    response.close()
                    return handleTokenExpired(chain, originalRequest, headerRequest)
                } catch (t: Throwable) {
                    Timber.e("예외 발생 ${t.message}")
                    saveAccessToken("", "")
                }
            }

            CODE_INVALID_USER -> {
                saveAccessToken("", "")
            }
        }
        
        return response
    }

    private fun handleTokenExpired(
        chain: Interceptor.Chain,
        originalRequest: Request,
        headerRequest: Request
    ): Response {
		    // 서버가 리프레시 토큰을 인증하여 액세스 토큰을 재발급 해준다. 
        val refreshTokenRequest = originalRequest.newBuilder().post("".toRequestBody())
            .url("$AUTH_BASE_URL/auth/token")
            .addHeader(REFRESH_TOKEN, runBlocking(Dispatchers.IO) { getRefreshToken() })
            .build()
            
        val refreshTokenResponse = chain.proceed(refreshTokenRequest)
        if (refreshTokenResponse.isSuccessful) {
            val responseToken = json.decodeFromString(
                refreshTokenResponse.body?.string().toString()
            ) as BaseResponse<ResponseReIssueTokenDto>
            
            // 새로 발급된 액세스, 리프레시 토큰을 데이터스토어에 저장 
            if (responseToken.data != null) {
                saveAccessToken(
                    responseToken.data.accessToken,
                    responseToken.data.refreshToken
                )
            }
            refreshTokenResponse.close()
            
            // 새로 발급 받은 액세스 토큰을 요청 헤더에 추가 
            val newRequest = originalRequest.newAuthBuilder().build()
            return chain.proceed(newRequest)
        } else {
            refreshTokenResponse.close()
            saveAccessToken("", "")
            return chain.proceed(headerRequest)
        }
    }
    
    private fun Request.newAuthBuilder() =
        this.newBuilder().addHeader(HEADER_TOKEN, runBlocking(Dispatchers.IO) { getAccessToken() })

    private suspend fun getAccessToken(): String {
        return dataStoreRepository.getAccessToken().first() ?: ""
    }

    private suspend fun getRefreshToken(): String {
        return dataStoreRepository.getRefreshToken().first() ?: ""
    }

    private fun saveAccessToken(accessToken: String, refreshToken: String) =
        runBlocking {
            dataStoreRepository.saveAccessToken(accessToken, refreshToken)
        }

    companion object {
        private const val CODE_TOKEN_EXPIRED = 401
        private const val CODE_INVALID_USER = 404
        private const val HEADER_TOKEN = "accessToken"
        private const val REFRESH_TOKEN = "refreshToken"
    }
}
```

## 리프레시 토큰이 만료된 경우

### 방법 1. 리프레시 토큰 만료 일주일 전, 액세스 토큰과 함께 재발급 진행

https://devtalk.kakao.com/t/refresh-token/19265

서버로부터 액세스 토큰을 재발급 받을 때 리프레시 토큰이 동일하다면, 리프레시 토큰의 만료 기간은 7일보다 더 많이 남은 것이라고 생각하면 된다. 

반면에, 리프레시 토큰 만료 시간이 7일 이하로 남았을 때 요청이 들어오면 서버는 리프레시 토큰도 새로운 값으로 내려줘야 한다. 

### 방법 2. 리프레시 토큰 만료 → 401 에러 → 로그아웃 & 로그인 → 액세스, 리프레시 토큰 재발급

<img width="600" src="https://github.com/leeeha/Android-TIL/assets/68090939/3e369cb7-03c0-4a0f-a210-e624f92f2c13"/>

현재 진행하고 있는 프로젝트인 위니 서버에서는 **액세스 토큰을 재발급 받을 때 리프레시 토큰도 갱신시킨다**고 한다. 

그래서 액세스 토큰 재발급 받을 때 응답으로 오는 액세스, 리프레시 토큰 모두 데이터스토어에 새로 저장해줘야 한다. 

## 참고자료

[🌐 Access Token & Refresh Token 원리](https://inpa.tistory.com/entry/WEB-📚-Access-Token-Refresh-Token-원리-feat-JWT)

[Refresh JWT Tokens in Android with OkHttp Interceptor](https://proandroiddev.com/refresh-jwt-tokens-in-android-with-okhttp-interceptor-575aaa4b6)