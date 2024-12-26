# 4장. Node.js에서의 비동기 처리

* Node.js에서 비동기 처리를 다루는 방법을 다루는 장
  * 비동기 코드의 4가지 패턴 이해
  * 각 패턴의 에러 핸들링에 대한 설명
 
* Node.js 에서 이용되는 이벤트 핸들링 패턴 4가지
  * 콜백(callback)
  * 프로미스(promise)
  * async/await
  * EventEmitter/Stream

* 권장 설계 패턴
  * 되도록 async/await를 사용한다.
  * 스트림 처리가 필요한 경우에만 EventEmiiter/Stream을 사용한다.

<details>
<summary>4.1 동기 처리와 비동기 처리</summary>
<div markdown="1">    
 
* Node.js에는 이벤트 루프가 있어 싱글 프로세스/싱글 스레드여도 여러 요청을 효율적으로 처리할 수 있음
* 반대로 말하면 이벤트 루프를 장시간 정지시키는 코드는 여러 요청을 받을 때 성능을 발휘하기 어려움!!
* So, Node.js 코드를 작성할 때는 **이벤트 루프가 장시간 정지되지 않도록** 유의해야 함
* Node.js의 이벤트 루프를 정지시키는 건? 동기 처리(비동기 이외의 처리)

</div>
</details>

___

<details>
<summary>4.2 콜백</summary>
<div markdown="1">    

### 4.2.1 Node.js와 Callback

</div>
</details>

___

<details>
<summary>4.3 프로미스</summary>
<div markdown="1">    

### 4.3.1 콜백의 프로미스화

</div>
</details>

___

<details>
<summary>4.4 async/await</summary>
<div markdown="1">    

</div>
</details>

___

<details>
<summary>4.5 스트림 처리</summary>
<div markdown="1">    

### 4.5.1 스트림 처리의 에러 핸들링

</div>
</details>

___

<details>
<summary>4.6 Asynclterator</summary>
<div markdown="1">    

### 4.6.1 Asynclterator가 도움이 되는 이유

</div>
</details>

___

<details>
<summary>4.7 에러 핸들링 정리</summary>
<div markdown="1">    

### 4.7.1 비동기 에러 핸들링

</div>
</details>

___

<details>
<summary>4.8 Top-Level Await</summary>
<div markdown="1">    

### 4.8.1 Top-Level Await와 Asynclterator의 주의점

</div>
</details>
