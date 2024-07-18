코루틴이 **어떻게 특정 지점에서 중단도 하고 재개도 하는지 그 내부적인 동작 원리**를 예제와 함께 알아보자! 

아래의 UserService 클래스에서 findUser라는 suspend 함수로 유저 정보를 가져오는데, 그 안에서 findProfile, findImage라는 또 다른 2개의 suspend 함수도 호출하고 있다. 

```kotlin
package org.example

import kotlinx.coroutines.delay

class UserService {
    private val userProfileRepository = UserProfileRepository()
    private val userImageRepository = UserImageRepository()

    suspend fun findUser(userId: Long): UserDto {
        println("findProfile")
        val profile = userProfileRepository.findProfile(userId)

        println("findImage")
        val image = userImageRepository.findImage(profile)

        return UserDto(profile, image)
    }
}

data class UserDto(
    val profile: Profile,
    val image: Image
)

class Profile
class Image

class UserProfileRepository {
    suspend fun findProfile(userId: Long): Profile {
        delay(1_000L)
        return Profile()
    }
}

class UserImageRepository {
    suspend fun findImage(profile: Profile): Image {
        delay(1_000L)
        return Image()
    }
}
```

<img width="600" src="https://github.com/user-attachments/assets/5dd8fec1-922f-42bc-aed1-956e8b992dd0"/>

suspend 함수인 findUser가 내부적으로 어떻게 동작하는지 알아보자! 

우선 findUser 함수에서 중단될 수 있는 지점이 어디인지 확인해보자. 바로 **suspend 함수를 호출하는 두 곳이 중단될 수 있는 지점**이다. 

이 지점을 경계로 메소드를 나누면 findUser 함수는 총 3단계로 이루어진다. 

```kotlin
suspend fun findUser(userId: Long): UserDto {
    // 0단계 - 초기 시작 
    println("findProfile")
    val profile = userProfileRepository.findProfile(userId)

    // 1단계 - 1차 중단 후 재시작 
    println("findImage")
    val image = userImageRepository.findImage(profile)

    // 2단계 - 2차 중단 후 재시작 
    return UserDto(profile, image)
}
```

이제 **각 단계를 라벨로 표시**해보자. 이를 위해 **라벨 정보를 갖고 있는 객체** 하나를 만들어야 한다. 

`Continuation`이라는 인터페이스를 만들고, 이를 구현한 익명 클래스의 객체가 라벨을 갖고 있도록 구현해보자. 

```kotlin
interface Continuation {
  // 라벨을 갖고 있을 인터페이스 
}

suspend fun findUser(userId: Long): UserDto {
    // state machine의 약자 (라벨을 기준으로 상태를 관리하므로)
    val sm = object: Continuation {
        var label = 0 // 익명 클래스 객체가 라벨을 갖게 만든다. 
    }
    
    when(sm.label) {
        0 -> {
            println("findProfile")
            val profile = userProfileRepository.findProfile(userId)
        }
        
        1 -> {
            println("findImage")
            val image = userImageRepository.findImage(profile)
        }
        
        2 -> return UserDto(profile, image)
    }
}
```

1번, 2번 라벨에서 각각 profile, image 객체를 가져올 수 없어서 에러가 발생하는데, sm 객체가 이 데이터들을 갖고 있게 만들자. 그리고 이 데이터를 필요할 때 꺼내서 쓰면 된다. 

```kotlin
suspend fun findUser(userId: Long): UserDto {
    val sm = object: Continuation {
        var label = 0 
        var profile: Profile? = null
        var image: Image? = null
    }

    when(sm.label) {
        0 -> {
            println("findProfile")
            val profile = userProfileRepository.findProfile(userId)
        }

        1 -> {
            println("findImage")
            val image = userImageRepository.findImage(sm.profile!!)
        }

        2 -> {
            return UserDto(sm.profile!!, sm.image!!)
        }
    }
}
```

다음으로 sm 객체가 profile, image 데이터를 갖고 있도록 만들자. 현재는 라벨이 항상 0으로 고정되어 있으므로, 중단 지점 직전에 라벨도 하나씩 올려보자. 

```kotlin
suspend fun findUser(userId: Long): UserDto {
    val sm = object: Continuation {
        var label = 0
        var profile: Profile? = null
        var image: Image? = null
    }

    when(sm.label) {
        0 -> {
            println("findProfile")
            sm.label = 1
            sm.profile = userProfileRepository.findProfile(userId)
        }

        1 -> {
            println("findImage")
            sm.label = 2
            sm.image = userImageRepository.findImage(sm.profile!!)
        }

        2 -> {
            return UserDto(sm.profile!!, sm.image!!)
        }
    }
}
```

이제 라벨 1번, 2번이 호출되게 만들어야 한다. 현재는 findUser가 호출되면, sm 객체가 만들어지며 0번 라벨을 갖게 되고, findProfile 직전에 1번 라벨로 변경되긴 하지만, 그대로 함수가 종료되어 버린다. 

이를 해결하기 위해 suspend 함수가 가장 마지막 매개변수로 Continuation을 받도록 변경할 것이다. findUser, findProfile, findImage 함수 모두에 적용해보자. 

```kotlin
package org.example

import kotlinx.coroutines.delay

class UserService {
    private val userProfileRepository = UserProfileRepository()
    private val userImageRepository = UserImageRepository()

    interface Continuation { }

    suspend fun findUser(userId: Long, continuation: Continuation): UserDto {
        val sm = object: Continuation {
            var label = 0
            var profile: Profile? = null
            var image: Image? = null
        }

        when(sm.label) {
            0 -> {
                println("findProfile")
                sm.label = 1
                sm.profile = userProfileRepository.findProfile(userId, sm)
            }

            1 -> {
                println("findImage")
                sm.label = 2
                sm.image = userImageRepository.findImage(sm.profile!!, sm)
            }

            2 -> {
                return UserDto(sm.profile!!, sm.image!!)
            }
        }
    }
}

data class UserDto(
    val profile: Profile,
    val image: Image
)

class Profile
class Image

class UserProfileRepository {
    suspend fun findProfile(userId: Long, continuation: UserService.Continuation): Profile {
        delay(1_000L)
        return Profile()
    }
}

class UserImageRepository {
    suspend fun findImage(profile: Profile, continuation: UserService.Continuation): Image {
        delay(1_000L)
        return Image()
    }
}
```

이제 Continuation에 `resumeWith(data: Any?)`라는 함수를 하나 만들고, findUser에서 익명 클래스로 만든 sm 객체에서 `resumeWith` 함수를 오버라이드 한다. 

오버라이드 된 resumeWith에서는 다시 한번 findUser를 호출할 것이다. (재귀 호출)

```kotlin
package org.example

import kotlinx.coroutines.delay

class UserService {
    private val userProfileRepository = UserProfileRepository()
    private val userImageRepository = UserImageRepository()

    interface Continuation {
        suspend fun resumeWith(data: Any?)
    }

    suspend fun findUser(userId: Long, continuation: Continuation): UserDto {
        val sm = object: Continuation {
            val userId = userId
            var label = 0
            var profile: Profile? = null
            var image: Image? = null

            override suspend fun resumeWith(data: Any?) {
                findUser(this.userId, this) // 재귀 호출 
            }
        }

        when(sm.label) {
            0 -> {
                println("findProfile")
                sm.label = 1
                sm.profile = userProfileRepository.findProfile(userId, sm)
            }

            1 -> {
                println("findImage")
                sm.label = 2
                sm.image = userImageRepository.findImage(sm.profile!!, sm)
            }

            2 -> {
                return UserDto(sm.profile!!, sm.image!!)
            }
        }
    }
}

data class UserDto(
    val profile: Profile,
    val image: Image
)

class Profile
class Image

class UserProfileRepository {
    suspend fun findProfile(userId: Long, continuation: UserService.Continuation): Profile {
        delay(1_000L)
        return Profile()
    }
}

class UserImageRepository {
    suspend fun findImage(profile: Profile, continuation: UserService.Continuation): Image {
        delay(1_000L)
        return Image()
    }
}
```

자 이제 findProfile, findImage에 넘겨준 Continuation 객체를 통해 `resumeWith` 함수를 호출하면, 다음 라벨 영역의 코드도 호출할 수 있게 된다! 

그림으로 이 과정을 설명하면 다음과 같다. 핵심은 **suspend 함수들 간에 Continuation을 지속적으로 전달하면서 콜백으로 활용한다**는 것이다. 

이를 **Continuation Passing Style (CPS)** 이라고 한다.

<img width="600" src="https://github.com/user-attachments/assets/aeb74883-5465-45fc-bf6a-4c585ffb4788"/>

<img width="600" src="https://github.com/user-attachments/assets/258e6246-534d-489d-8d4c-1c0e1aefbff2"/>

<img width="600" src="https://github.com/user-attachments/assets/0c607bc2-477e-4f59-8adc-d4a8c1a74136"/>

<img width="600" src="https://github.com/user-attachments/assets/4fae0036-68df-4418-a483-250b6954480b"/>

<img width="600" src="https://github.com/user-attachments/assets/a0ead809-76fb-4d40-b1c9-838f4bd88616"/>

<img width="600" src="https://github.com/user-attachments/assets/298875ac-6f62-493b-aa23-ddcc0011265e"/>

이렇게 동작하도록 구현하기 위해 findUser가 호출될 때마다 sm을 새로 만들지 않고, **들어온 Continuation 객체 타입에 따라 새로운 Continuation 객체를 만들도록 수정**한다. 

또한, Continuation의 resumeWith 함수를 오버라이드 한 함수에서 라벨과 데이터를 넣어주도록 수정한다. 

```kotlin
package org.example

import kotlinx.coroutines.delay

suspend fun main() {
    val userService = UserService()
    println(userService.findUser(userId = 1L, continuation = null))
}

class UserService {
    private val userProfileRepository = UserProfileRepository()
    private val userImageRepository = UserImageRepository()

    interface Continuation {
        suspend fun resumeWith(data: Any?)
    }

    private abstract class FindUserContinuation(val userId: Long) : Continuation {
        var label = 0
        var profile: Profile? = null
        var image: Image? = null
    }

    suspend fun findUser(userId: Long, continuation: Continuation?): UserDto {
        // FindUserContinuation 추상 클래스를 구현하는 sm 객체 초기화
        // resumeWith 추상 메서드 오버라이딩
        // continuation 객체가 널이 아니면 새로 초기화 하지 않고 그대로 사용
        val sm = continuation as? FindUserContinuation ?: object: FindUserContinuation(userId) {
            // 중단되었다가 재개될 때 continuation이 갖고 있는 라벨과 데이터 변경 
            override suspend fun resumeWith(data: Any?) {
                when(super.label) {
                    0 -> {
                        profile = data as Profile
                        label = 1
                    }

                    1 -> {
                        image = data as Image
                        label = 2
                    }
                }

                // 재귀 호출
                findUser(this.userId, this)
            }
        }

        // 라벨에 따라 suspend 함수 실행 
        // 일정 시간 동안 중단되었다가 재개 -> resumeWith 실행 
        when(sm.label) {
            0 -> {
                println("findProfile")
                userProfileRepository.findProfile(userId, sm)
            }

            1 -> {
                println("findImage")
                userImageRepository.findImage(sm.profile!!, sm)
            }
        }

        // 최종 유저 데이터 반환 
        return UserDto(sm.profile!!, sm.image!!)
    }
}

data class UserDto(
    val profile: Profile,
    val image: Image
)

class Profile
class Image

class UserProfileRepository {
    suspend fun findProfile(userId: Long, continuation: UserService.Continuation) {
        delay(1_000L)
        continuation.resumeWith(Profile())
    }
}

class UserImageRepository {
    suspend fun findImage(profile: Profile, continuation: UserService.Continuation) {
        delay(1_000L)
        continuation.resumeWith(Image())
    }
}
```

그러면, 그림에서 살펴본 것처럼 **Continuation을 통해 최초 호출인지, 콜백 호출인지 구분**할 수 있게 된다. 

그리고 **Continuation의 구현 클래스에서 라벨과 전달할 데이터 등을 관리**할 수 있게 된다. 

실행 결과 

```
findProfile
findImage
UserDto(profile=org.example.Profile@3a785e32, image=org.example.Image@4692f856)
```

초반에 작성했던 아래와 같은 간단한 findUser 함수를 디컴파일 해보면, 우리가 만들어봤던 예제와 비슷하게 Continuation을 사용한 CPS로 변하게 된다.

```kotlin
class UserService {
    private val userProfileRepository = UserProfileRepository()
    private val userImageRepository = UserImageRepository()

    suspend fun findUser(userId: Long): UserDto {
        println("findProfile")
        val profile = userProfileRepository.findProfile(userId)

        println("findImage")
        val image = userImageRepository.findImage(profile)

        return UserDto(profile, image)
    }
}
```

아래의 디컴파일 코드를 완전히 이해하긴 어렵지만, 큰 흐름은 **Continuation 객체의 라벨을 확인하여 그 값에 따라 switch-case문을 돌리고 최종적으로 UserDto를 반환한다**는 것이다. 

findUser 함수 안에서 사용되는 또 다른 suspend 함수인 findProfile, findImage도 비슷한 방식으로 동작한다. 

```kotlin
 @Nullable
 public final Object findUser(long userId, @NotNull Continuation $completion) {
    Object $continuation;
    label27: {
       if ($completion instanceof <undefinedtype>) {
          $continuation = (<undefinedtype>)$completion;
          if ((((<undefinedtype>)$continuation).label & Integer.MIN_VALUE) != 0) {
             ((<undefinedtype>)$continuation).label -= Integer.MIN_VALUE;
             break label27;
          }
       }

       $continuation = new ContinuationImpl($completion) {
          Object L$0;
          // $FF: synthetic field
          Object result;
          int label;

          @Nullable
          public final Object invokeSuspend(@NotNull Object $result) {
             this.result = $result;
             this.label |= Integer.MIN_VALUE;
             return UserService.this.findUser(0L, (Continuation)this);
          }
       };
    }

    Profile profile;
    Object var10000;
    label22: {
       Object $result = ((<undefinedtype>)$continuation).result;
       Object var8 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
       switch (((<undefinedtype>)$continuation).label) {
          case 0:
             ResultKt.throwOnFailure($result);
             String var9 = "findProfile";
             System.out.println(var9);
             UserProfileRepository var11 = this.userProfileRepository;
             ((<undefinedtype>)$continuation).L$0 = this;
             ((<undefinedtype>)$continuation).label = 1;
             var10000 = var11.findProfile(userId, (Continuation)$continuation);
             if (var10000 == var8) {
                return var8;
             }
             break;
          case 1:
             this = (UserService)((<undefinedtype>)$continuation).L$0;
             ResultKt.throwOnFailure($result);
             var10000 = $result;
             break;
          case 2:
             profile = (Profile)((<undefinedtype>)$continuation).L$0;
             ResultKt.throwOnFailure($result);
             var10000 = $result;
             break label22;
          default:
             throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
       }

       profile = (Profile)var10000;
       String var5 = "findImage";
       System.out.println(var5);
       UserImageRepository var12 = this.userImageRepository;
       ((<undefinedtype>)$continuation).L$0 = profile;
       ((<undefinedtype>)$continuation).label = 2;
       var10000 = var12.findImage(profile, (Continuation)$continuation);
       if (var10000 == var8) {
          return var8;
       }
    }

    Image image = (Image)var10000;
    return new UserDto(profile, image);
 }
}
```

실제 코루틴에서 사용되는 Continuation 인터페이스의 주요 함수는 다음과 같다. 

```kotlin
public interface Continuation<in T> {
    public val context: CoroutineContext

    public fun resumeWith(result: Result<T>)
}
```