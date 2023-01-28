## 01. Why Coroutines

- JVM은 메인 쓰레드를 비롯한 여러 쓰레드로 구성될 수 있다.
- 메인 쓰레드가 블로킹되거나 종료된다면 프로그램 실행에 영향을 끼침
- 이러한 이유로 메인 쓰레드에 대한 부담을 줄이기 위해 여러 방법 제시

  - 새로운 쓰레드 생성
  - 새로운 쓰레드풀 생성
  - Rx

- 결국 쓰레드 단위의 작업이므로 비효율적인 면이 있다.

#### 코루틴

- 코투린은 이전에 자신이 실행이 중단했던 다음 장소부터 실행가능

  ![image](https://user-images.githubusercontent.com/67682840/215238405-2963297f-edfd-4480-bdb8-5b239b348647.png)

- 하나의 쓰레드는 여러 코루틴으로 이루어질 수 있다.
  
  <img width="845" alt="image" src="https://user-images.githubusercontent.com/67682840/215239137-cba71d88-b1f2-4331-b7d7-2d2bbfe683a4.png">

  작업의 단위가 쓰레드 일 경우, 하나의 쓰레드가 다른 쓰레드의 결과에 의존적이라면 쓰레드가 블로킹되고 컨텍스트 스위칭이 발생한다.
  
  <br>

  <img width="841" alt="image" src="https://user-images.githubusercontent.com/67682840/215239216-ff08ebef-8e28-4c65-b30d-5f9ff4838c87.png">
  
  하지만 작업의 단위가 코루틴일 경우, 현재 쓰레드가 블로킹되면 컨텍스트 스위칭을 통해 다른 쓰레드가 실행되는 대신 현재 쓰레드에서 다른 코루틴을 사용하도록 하면 되므로 컨텍스트 스위칭 발생이 현저히 줄어든다. 

<br>

## 02. BASICS

```kotlin
fun main(){
    GlobalScope.launch{
        delay(1000L)
        println("world!")
    }
    println("hello, ")
    Thread.sleep(2000)
}
```

> 출력결과<br>
> hello,
> world!

- `launch()`는 코루틴 빌더, 내부적으로 코루틴을 만들어 반환한다.
- 코루틴 빌더를 사용하기 위해서는 코루틴 스코프가 필요하다.
- `launch()`는 메인쓰레드를 블로킹하지 않으므로 hello가 먼저 출력됨을 알 수 있다.
- 쓰레드와 비슷한지 비교해보기 위해 아래 코드처럼 변경해보자


```kotlin
fun main(){
   thread {
       Thread.sleep(1000)
       println("world!")
   }
   println("hello, ")
   Thread.sleep(2000)
}
```

- `launch()`를 thread로 바꾸어도 제대로 동작한다.
- 코루틴은 light-weight thread임을 알 수 있다.

#### Thread.sleep, delay

- `delay`는 suspend function이므로 coroutinScope 안에서 작동하거나 다른 suspend function 안에서 작동

- 위 예제에서 메인 쓰레드는 coroutine scope가 아니므로 `delay`를 사용할 수 없다.

- 블로킹을 할 수 있는 스코프를 만들기 위해 `runBlocking`을 사용해보자.

```kotlin
fun main(){
  GlobalScope.launch{
      delay(1000L)
      println("world!")
  }
  println("hello, ")
  runBlocking {
      delay(2000L)
  }
}
```

- `runBlocking` 또한 스코프 빌더
- 메인 쓰레드를 블로킹한 채 새로운 쓰레드를 생성하여 작업을 할당한다.
- 메인 쓰레드 종료 방지를 위해 아래 `.runBlocking`을 사용했지만 딜레이가 1000이 아닌 3000이 되면 `hello,`만 출력이 된다.

```kotlin
fun main() = runBlocking {
    val job = GlobalScope.launch {
        delay(3000)
        println("world!")
    }
    println("hello, ")
    job.join()
}
```

- `.launch()`는 실행결과로 `job`을 반환한다
- `job.join()`은 코루틴의 실행결과를 기다린다.
- 매번 `.launch()`를 할 때마다 `.join()`을 하는 문제가 있다.

#### Structured concurrency

- 위 문제의 원인은 메인 쓰레드의 `runBlockig`과 `GlobalScope`가 아무런 관련이 없기 때문이다.
- `runBlocking`의 스코프에서 `launch`를 해주자
  - 이는 Top level 코루틴을 만드는 것이 아닌 부모 코루틴의 자식 코루틴을 만드는 것
  - 부모 코루틴이 자식 코루틴이 완료되는 것을 기다려 준다.

```kotlin
fun main() = runBlocking {
    launch {
        delay(3000)
        println("world!")
    }
    println("hello, ")
}
```



## Reference

- https://aaronryu.github.io/2019/05/27/coroutine-and-thread/ 