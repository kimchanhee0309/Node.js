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

기능 | 구현 | 동기/비동기
----------------- | ------------------ | ------------------
JSON operation | V8 | 동기
Array operation | V8 | 동기
File I/O | libuv | 비동기(Sync 함수 제외)
Child process | libuv | 비동기(Sync 함수 제외)
Timer | libuv | 비동기
TCP | libuv | 비동기

</div>
</details>

___

<details>
<summary>4.2 콜백(callback)</summary>
<div markdown="1"> 

* 예시 코드(Node.js에서 파일을 다루는 표준 모듈인 fs를 활용)
```javascript
const { readFile } = require('fs');

console.log('A');

readFile(__filename, (err, data) => {
  console.log('B', data);
});

console.log('C');
```
* 콜백은 **'처리가 끝났을 때 호출되는 함수를 등록하는'** 인터페이스임
* 위 코드에서 readFile은 2번째 인수에 콜백(콜백 함수)을 제공함

* 콜백 예시(파일 자신을 불러오고 // 파일 이름을 포맷 후 다른 이름으로 쓰고 // 백업한 파일을 ReadOnly하는 사양)
```javascript
const { readFile, writeFile, chmod } = require('fs');

const backupFile = `${__filename}-${Date.now()}`;

readFile(__filename, (err, data) => {
  if (err) {
    return console.error(err);
  }
  writeFile(backupFile, data, (err) => {
    if (err) {
      return console.error(err);
    }
    chmod(backupFile, 0o400, (err) => {
      if (err) {
        return console.error(err);
      }
      console.log('done')
    });
  });
});
```
* 처리가 종료된 시점에 콜백이 실행되므로 처리를 직렬로 연결한 경우에는 앞의 코드와 같이 중첩이 깊어짐
* 중첩이 계속 깊어지는 코드를 **콜백 지옥(callback hell)** 이라 부름
* 콜백 지옥은 가급적 피하는 게 좋음

### 4.2.1 Node.js와 Callback
* Node.js의 표준 모듈에 구현된 콜백 API에 존재하는 독특한 관례
  
관례 | - 
----------------- | ------------------ 
API의 가장 마지막 인수가 콜백 | -
콜백의 1번째 인수가 에러 객체 | -

* 콜백에 익숙해지기 전까지는 **에러 핸들링** 에 특히 주의해야 함
* 1번째 인수가 에러 객체가 되므로, 에러 핸들링은 반드시 **1번재 인수에 대해 null 체크** 를 해야 함
```javascript
const { readFile } = require('fs');

readFile(__filename, (err, data) => {
  if (err) {
    console.error(err);
    return;
  }
  console.log(data);
});
```

* null 체크 안의 return도 중요!!!
* return을 입력하지 않으면 에러가 발생해도 다음 console.log가 실행됨
```javascript
const { readFile } = require('fs');

readFile(__filename, (err, data) => {
  if (err) {
    console.error(err);
    // return; // 여기를 생략하거나 빼먹은 경우 다음 처리를 진행한다.
  }
  console.log(data);
});
```

* 또한, try-catch로 콜백 안의 에러를 잡을 수 없음
  * So, 콜백 처리를 중첩하는 경우에는 모든 콜백마다 에러에 대한 null 체크를 반드시 해야 함.
```javascript
const { readFile } = require('fs');

try {
  readFile(__filename, (err, data) => {
    console.log(data);
  });
} catch (err2) {
  // 콜백의 인수에 들어 있는 에러는 잡을 수 없다.
  console.error(err2);
}
```

</div>
</details>

___

<details>
<summary>4.3 프로미스</summary>
<div markdown="1">    

* 콜백(callback)의 단점
  * 중첩이 깊어지기 쉽고, 포괄적인 에러 핸들링을 수행할 수 없음
  * 이러한 약점들을 해소한 비동기 처리를 구현한 것이 **프로미스(promise)**

* 프로미스(promise)란?
  * 성공이나 실패를 반환하는 객체임
  * 프로미스 객체 생성 시, 상태가 정해지지 않고 비동기 처리가 완료된 시점에 둘 중 하나의 상태로 변함
    * 성공하면 **then 메서드**에 설정된 성공 시의 핸들러가 호출됨
    * 실패하면 **catch 메서드**에 설정된 실패 시의 핸들러가 호출됨

* 프로미스는 **resolve(성공)** 와 **reject(실패)** 시에 호출하는 함수를 생성자로 생성함
```javascript
const promiseFunc = new Promis((resolve, reject) => {
  // ---
  // 비동기로 수행하는 처리를 작성한다.
  // ---
  if (error_발생_시) {
    // 에러가 발생했을 때는 reject를 호출한다.
    return reject(에러_내용);
  }
  // 성공했을 때는 resolve를 호출한다.
  resolve(성공_시의_내용);
});

promiseFunc.then(성공=resolve_시_실행할_함수);

promiseFunc.catch(실패=reject_시_실행할_함수);
```
```javascript
const promiseA = new Promise((resolve, reject) => {
  resolve('return data');
});

promiseA.then((data) => console.log(data));

const promiseB = new Promise((resolve, reject) => {
  reject(new Error('return error'));
});

primiseB.catch((err) => console.error(err));

console.log('done');

=============================================================
<결과>
$ node index.js
done
return data
Error: return error
  at /home/xxx/tmp/index.js:8:10
  at new Prom,ise (<anonymous>)
```
* then이나 catch는 연결(체인)할 수 있음
* 이를 통해 콜백 시에 있던 중첩이 깊어지는 것을 방지 + 포괄적인 에러 핸들링을 할 수 있음

```javascript
const promiseX = (x) => {
  return new Promise((resolve, reject) => {
    if (typeof x === 'number') {
      resolve(x);
    } else {
      reject(new Error('return error'));
    }
  })
};

const logAndDouble = (num) => {
  console.log(num);
  return num * 2;
};

// then으로 성공했을 때를 연결할 수 있고, 실패했을 때는 catch로 건너뛴다.
promiseX(1)
  .them((data) => logAndDouble(data))
  .then((data) => logAndDouble(data))
  .catch(console.log(data))
```

* 프로미스의 장단점
  * 장점 : 콜백 지옥을 피할 수 있고 작성하기에도 쉬움
  * 단점 : 기존의 루프나 조건 분기와 조합하기 어렵다는 문제

### 4.3.1 콜백의 프로미스화
* 콜백을 이용한 비동기 처리는 콜백을 프로미스 객체로 감싸서 프로미스화 할 수 있음
```javascript
const { readFile, writeFile, chmod } = require('fs');

const readFileAsync = (path) => {
  return new Promise((resolve, reject) => {
    readFile(path, (err, data) => {
      if (err) {
        reject(err);
        return;
      }
      resolve(data);
    });
  });
};

const writeFileAsync = (path, data) => {
  return new Promise((resolve, reject) => {
    writeFile(path, data, (err) => {
      if (err) {
        reject(err);
        return;
      }
      resolve();
    });
  });
};

const chmodAsync = (path, mode) => {
  return new Promise((resolve, reject) => {
    chmod(backupFile, mode, (err) => {
      if (err) {
        reject(err);
        return;
      }
      resolve();
    });
  });
};

const backupFile = `${__filename)-$(Data.now())`;

readFileAsync(__filename)
  .then((data) => {
    return writeFileAsync(backupFile, data);
  })
  .then(() => {
    return chjmodAsync(backupFile, 0o400);
  })
  .catch((err) => {
    console.error(err);
  });
```

* 프로미스 체인을 이용하면 콜백에 비해 중첩을 깊게 만들지 않고 처리를 연결할 수 있음

#### promisify와 promise 인터페이스
* util.promisify는 다음 관례에 따르는 콜백 함수를 프로미스로 바꿀 수 있음
관례 | - 
----------------- | ------------------ 
API의 가장 마지막 인수가 콜백 | -
콜백의 1번째 인수가 에러 객체 | -
처리 완료 시에 1번만 호출되는 콜백 함수 | -

* promisity 사용 예시
```javascript
const { promisify } = require('util');
const { readFile, writeFile, chmod } = require('fs');
const readFileAsync = promisify(readFile);
const writeFileAsync = promisify(writeFile);
const chmodAsync = promisify(chmod);
```
```javascript
const { readFile, writeFile, chmod } = require('fs/promises');

const backupFile = `${__filename}-${Date.nopw()}`;

readFile(__filename)
  .then((data) => {
    return writeFile(backupFile, data);
  })
  .then(() => {
    return chmod(backupFile, 0o400);
  })
  .catch((err) => {
    console.error(err);
  });
```

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
