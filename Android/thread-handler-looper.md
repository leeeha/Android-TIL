# 안드로이드 UI의 동작 

안드로이드 UI는 기본적으로 **Main Thread** 하나만으로 구성된 **싱글 스레드 모델**로 동작한다. 

**메인 스레드가 아닌 다른 스레드에서 UI 작업을 수행하면 안 된다**는 점에서, 메인 스레드를 **UI 스레드**라고 부르기도 한다. 

싱글 스레드 모델에서 지켜야 하는 2가지 원칙은 다음과 같다. 

>1) Main Thread를 block 하지 말 것 
>2) 안드로이드 UI ToolKit (TextView, ImageView 등)은 Main Thread만 접근 가능하도록 할 것 

**안드로이드 UI는 왜 싱글 스레드 모델로 동작할까?** 그 이유는 생각해보면 간단하다. 

여러 스레드에서 TextView 같은 하나의 UI를 동시에 수정하려고 하면, 어떤 결과가 나올지 보장할 수 없기 때문이다. 즉, **UI의 무결성을 보장**하기 위해 오직 메인 스레드에서만 UI 작업을 수행할 수 있도록 설계되어 있다. 

# 메인 스레드가 블로킹 되면? 

아래 예시처럼 코드를 작성하고 버튼을 클릭해보자. 

한번 클릭했을 때는 Clicked! 로그가 정상적으로 출력되지만, 그 후로 여러 번 클릭하면 터치 이벤트가 지연되어 결국 ANR(Application Not Responding)이 발생한다. 

(ANR 발생 기준: 액티비티 5초, 서비스와 브로드캐스트 리시버 10초)

```kotlin 
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        binding.btnMain.setOnClickListener {
            Log.d("MainActivity", "Clicked!")
            Thread.sleep(6000)
            Log.d("MainActivity", "${Thread.currentThread().name} : this will be printed.")
        }
    }
}
```

<img width="300" src="https://github.com/user-attachments/assets/2c438bf8-0ac0-4b7c-8a6f-12d540c2ccb9"/> 


이렇게 메인 스레드가 블로킹 되어 UI 동작이 멈추면 **앱 퍼포먼스 저하로 사용자 경험에 악영향**을 미치게 되며, 심한 경우에는 위처럼 **ANR까지 발생**할 수 있다. 

따라서, 시간이 오래 걸리는 작업은 **메인 스레드가 아닌 다른 스레드에서 처리**해줘야 한다. 이러한 별도의 스레드를 보통 **Worker Thread (Background Thread)**라고 부른다. 

Worker Thread에서만 작업을 처리하고 끝나는 경우도 있지만, 많은 경우에는 Worker Thread의 작업 결과를 Main Thread의 UI에 표시해야 한다. 즉, **스레드 간의 통신 방법**이 필요한 것이다! 

# 스레드 간 통신 

안드로이드는 아래 그림처럼 **스레드 내부에 Handler, Looper**를 두고, 이를 기반으로 **여러 스레드가 Message 또는 Runnable 객체를 주고 받으며 통신**할 수 있도록 설계되어 있다. 

<img width="400" src="https://github.com/user-attachments/assets/59b3ffee-6c1c-4ebf-89e8-35c6cab7da58"/> 

1. 2번 스레드가 메인 스레드에 Message 또는 Runnable 객체를 보내려고 한다. 
2. 2번 스레드는 메인 스레드가 가진 Handler의 sendMessage(Message), post(Runnable) 메서드로 Message 또는 Runnable 객체를 보낸다. 
3. 메인 스레드의 Handler는 전달 받은 메시지를 Looper의 MessageQueue에 넣는다. 
4. 메인 스레드의 Looper는 MessageQueue에 담긴 작업을 FIFO 방식으로 하나씩 꺼내서, Handler에게 작업을 수행하도록 요구한다. 
5. 메인 스레드의 Handler는 Looper에게 받은 작업을 handleMessage() 메서드 등으로 처리한다. 

# Handler 

### Looper로 메시지를 전달하는 경우 

- sendMessage(Message) 메서드로 MessageQueue에 Message 객체를 적재한다. 
- post(Runnable) 메서드로 MessageQueue에 Runnable 객체를 적재한다. 

### Looper로부터 메시지를 전달받는 경우 

- Message 객체가 전달된 경우: 메시지 내부의 Handler가 가진 handleMessage() 메서드로 작업을 처리한다. 
- Runnable 객체가 전달된 경우: Runnable 객체의 run() 메서드를 실행하여 작업을 처리한다. 

# Looper 

Looper는 오직 하나의 스레드만 담당한다. 안드로이드에서는 기본적으로 **MainActivity가 실행됨과 동시에 메인 스레드의 Looper가 돌기 시작**한다.

Looper는 궁극적으로, **MessageQueue에 있는 메시지들을 하나씩 꺼내서 이를 적절한 Handler로 전달하는 역할**을 한다.

Worker Thread를 Thread() 같이 기본 생성자로 만들면, 스레드 내부에 Looper가 생성되지 않는다. 이러한 스레드는 **다른 스레드에 메시지를 전달만 할 수 있고, 다른 스레드로부터 메시지를 받지는 못한다.** Worker Thread에서 백그라운드 작업만 수행하고 메시지를 전달할 용도라면 Looper를 생성하지 않아도 된다. 그러나, **다른 스레드로부터 메시지를 전달 받으려면 Looper를 반드시 생성**해야 한다. 

# 예시 코드 

### MainActivity (Main Thread)

```kotlin 
package com.practice.playground

import android.os.Bundle
import android.os.Handler
import android.os.Looper
import androidx.appcompat.app.AppCompatActivity
import com.practice.playground.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private lateinit var mainHandler: Handler
    private lateinit var timerThread: TimerThread

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // Main Thread의 핸들러 초기화
        mainHandler = Handler(Looper.getMainLooper())

        // Worker Thread 초기화
        timerThread = TimerThread(
            mainHandler = mainHandler,
            updateTimerText = ::updateTimerText
        )

        // Worker Thread 시작
        timerThread.start()

        // Worker Thread 종료
        binding.btnStopTimer.setOnClickListener {
            timerThread.stopTimer()
        }
    }

    private fun updateTimerText(text: String) {
        binding.tvTimer.text = text
    }
}
```

### TimerThread (Worker Thread)

```kotlin 
package com.practice.playground

import android.os.Handler

class TimerThread(
    private val mainHandler: Handler,
    private val updateTimerText: (String) -> Unit,
): Thread() {
    private var time = 0
    private var isRunning = true

    override fun run() {
        super.run()

        while (isRunning) {
            // Runnable 객체로 메인 스레드에 메시지 전달하여
            // UI 관련 작업 수행
            mainHandler.post {
                updateTimerText("Worker Thread 시작 후 ${time}초")
            }

            sleep(1000)

            time++
        }

        // 스레드 종료 전에 UI 관련 작업 수행
        mainHandler.post {
            updateTimerText("Worker Thread 종료")
        }
    }

    fun stopTimer() {
        isRunning = false
    }
}
```

<img width="300" src="https://github.com/user-attachments/assets/13e8a2fa-1f77-44be-9372-2717362b7132"/>

# 2025 안드로이드 탐구 영역 14번 문제 

📌 시험지 링크: https://android-exam25.gdg.kr/

(나) 지문은 **Looper의 메시지 큐가 포화 상태에 도달**했을 때, 이를 해결하는 방법에 대해 서술하고 있다. 

- 메시지 추가 빈도를 줄이고, **메시지의 중요도를 평가하여 꼭 필요한 작업만 큐에 추가**한다. 
- **작업을 더 작은 단위로 분리**하거나, 오래 걸리는 작업은 **백그라운드 스레드로 분산 처리**한다. 
- **메시지 큐 상태를 모니터링**하고, 병목을 유발하는 작업을 찾아 해결한다. 

# 참고 자료 

https://velog.io/@haero_kim/Android-Looper-Handler-기초-개념

https://velog.io/@buna1592/Android-Thread간-통신-using-Handler-Looper

https://hungseong.tistory.com/26#---%--Looper

