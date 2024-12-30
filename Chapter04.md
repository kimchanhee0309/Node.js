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

* 프로미스로 작성하기 어려운 루프나 조건 분기 처리를 쉽게 처리할 수 있게 해줌
* 프로미스를 이용한 비동기 처리를 동기적인 형태로 기술할 수 있음
* async를 붙여 함수를 선언하면 그 안에 await를 작성할 수 있음
* await는 이어진 식에서 반환된 프로미스의 결과가 나올 때까지 그 부분의 실행을 중지함

```javascript
async function someFunc() = {
  const foo = await 프로미스를_반환하는_식;
  const bar = await 프로미스를_반환하는_식; // 앞의 await가 완료될 때까지 실행되지 않음
  await 프로미스를_반환하는_식;
};

const someFuncArrow = async () => {
  await 프로미스를_반환하는_식;
};
```
```javascript
const { readFile, writefile, chmod } = require('fs/promises');

const main = async () => {
  const backupFile = `${__filename}-${Date.now()}`;

  const data = await readFile(__fimename);
  await writeFile(backupFile, data);
  await chmod(backupFile, 0o400);

  return 'done';
};

main()
  .then((data) => {
    console.log(data);
  })
  .catch((err) => {
    console.error(err);
  });
```
* 이 코드에서는 main 함수에 async를 선언
* async 함수의 return에서 반환한 결과를 then으로 받고 catch에서 포괄적인 에러 핸들링을 할 수 있음
* 프로미스와 async/await는 서로 호출할 수 있다는 점 중요
* async를 선언한 함수 안에서 await를 사용해 프로미스를 호출함으로써, 프로미스의 결과가 반환될 때까지 다음 처리의 실행을 기다릴 수 있음

</div>
</details>

___

<details>
<summary>4.5 스트림 처리</summary>
<div markdown="1">    

* async/await와 같은 비동기 흐름 제어 외에 이벤트 주도 방식의 비동기 흐름 제어(스트림 처리)가 존재함
  * **비동기 흐름 제어**
    * 콜백
    * 프로미스
    * async/await
  * **이벤트 주도 방식의 비동기 흐름**
    * 스트림 처리(EventEmitter/Stream)

* 이벤트 주도 방식의 흐름 제어에서는 다양한 시점에서 처리를 수행함
* 콜백과 같은 일회성 처리에 비해 데이터를 순차 처리함으로써 메모리를 효율적으로 이용할 수 있음
* 이벤트 루프를 오랜 시간 정지시키는 처리를 나누고 싶을 때도 효과적임

* 스트림 처리를 수행하는 대표적인 예시
  * HTTP 요청/응답
  * TCP
  * 표준 입출력

* 예시 코드
```javascript
const EventEmitter = require('events');

// EventEmitter의 베이스 클래스를 상속해 사용자 이벤트를 다루는 EventEmitter를 정의
class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

// myevent라는 이름의 event를 받는 리스너를 설정
myEmitter.on('myevent', (data) => {
  console.log('on myevent:', data);
});

// myevent 발행
myEmitter.emit('myevent', 'one');

setTimeout() => {
  // myevent 발행
  myEmitter.emit('myevent', 'two');
}, 1000);

=======================================
<결과>
$ node index.js
on myevent: one
on myevent: two
```

* EventEmitter는 '몇 번이고', '작게 쪼개져' 발생하는 비동기 이벤트를 제어하기 위한 구현임
* Stream은 EventEmitter에 데이터를 저장할 내부 버퍼를 조합한 것이라 이해하면 됨
  * 내부 버퍼에 데이터가 일정량 모이면 이벤트가 발생함
* Stream 객체 사이는 연결할 수 있음
  * 모은 데이터를 다른 형식으로 변환하는 Stream을 중간에 연결하면, 데이터가 전부 모이기 전에 변환 가능한 것부터 처리를 시작할 수 있음
  * 또한 모든 데이터를 메모리에 저장하지 않고 처리하므로 메모리 사용량을 쉽게 억제할 수 있음
* Stream을 이용하면 이벤트 연결, 데이터 흐름량 조정, 변환 처리 등 연속하는 데이터 흐름을 효율적으로 다룰 수 있음
* Node.js에서 사용하닌 4가지 Stream 처리 기반
Stream 종류 | 설명 
----------------- | ------------------ 
Writable | 데이터 쓰기에 이용한다(예: fs.createWriteStream).
Readable | 데이터 읽기에 이용한다(예: fs.createReadStream).
Duplex | 쓰기/읽기 양쪽에 대응한다(예: net.Socket).
Transform | Duplex를 상속해 읽고 쓴 데이터를 변환한다(ex: zlib.createDeflate).

* 콜백과 스트림의 차이
  * 콜백은 완료 시 한 번 호출되지만, 스트림 처리는 하나의 처리에 대해 여러 번의 처리가 발생함

* 예시 코드
```javascript
const http = require('http');

//서버에 대해 요청하는 객체를 생성
const req = http.request('http://localhost:3000', (res) => {
  //흘러오는 데이터를 utf8로 해석한다.
  res.setEncoding('utf8');

  // data 이벤트를 받는다.
  res.on('data', (chunk) => {
    console.lg(`body: ${chunk}`);
  });

  // end 이벤트를 받는다.
  res.on('end', () => {
    console.log('end');
  });
});

// 여기에서 처음으로 요청이 송신됨
req.end();
```
* .on
  * http.request의 콜백에 전달된 res는 HTTP 요청의 응답을 나타내는 스트림 객체임
  * 즉, 다음에 표시하는 위치에서 리스너가 각각 data와 end라는 이벤트를 받도록 설정함
* data 이벤트는 상황에 따라 여러 차례 호출되어 리스너 내용이 여러 번 처리될 수 있음
  * 큰 데이터를 가져오는 동안 다른 처리를 하지 못한다면 리소스가 아까움
  * But, 스트림으로 처리하면 해당 리스터가 호출되는 시점까지 Node.js는 다른 처리를 할 수 있음
  * 또한 데이터를 작게 쪼개 처리하기 때문에 한 번에 돌리는 루프가 작아지며, 더 작은 메모리에서도 동작이 가능하다는 장점이 있음

```javascript
const server = http.createServer();

//독립적인 리스너로 정의할 수 있음
server.on('request', (req, res) => {
  res.write('hello world\n');
  res.end();
});

server.listen(3000);
```
* 서버 입장에서는 클라 요청이 올 때까지 계속 기다리기만 하기에는 리소스가 아까움
* So, 각각 클라 연결은 request 이벤트로 다루고 리스너를 등록함
* 이렇게 하면 요청이 오지 않는 동안에 다른 요청을 받거나 다른 처리를 수행할 수 있음

### 4.5.1 스트림 처리의 에러 핸들링
* 스트림 처리에서 에러 핸들링이 누락되면 에러를 포괄적으로 잡아낼 수 없고, 에러 발생 시 프로세스가 깨지므로 주의해야 함

</div>
</details>

___

<details>
<summary>4.6 Asynclterator</summary>
<div markdown="1">    

* 스트림 처리시 몇 가지 문제점
  * async/await 등의 흐름 제어에 내장하기 어렵다.
  * 에러 핸들링을 잊기 쉽고, 잊었을 때 영향이 크다.

* **AsyncIterator(for await ... of)**
  * 이것이 등장하면서 async/await와 스트림 처리의 호환성이 극적으로 향상됨
  * AsyncIterator는 이벤트를 for문처럼 표현하여 async/await의 콘텍스트에서도 스트림 처리를 다룰 수 있게 해줌

### 4.6.1 Asynclterator가 도움이 되는 이유
* AsyncIterator 유용성
  * 자신의 파읽을 읽는다.
  * 잠시 기다린 뒤 읽은 내용을 파일에 추가한다.
  * 다음을 읽는다.
 
#### AsyncIterator 없이 구현하기
* 파일 쓰기 이외의 처리를 스트림 처리로 구현한 예시
```javascript
const fs = require('fs');

// 파일을 읽는 Stream을 생성(64바이트 씩)
const readStream = fs.createReadStream(__filename, { encoding: 'utf8', highWaterMark: 64 });

let counter = 0;
// 파일 데이터를 읽는 동안 실행되는 리스너
readStream.on('data', (chunk) => {
  console.log(counter, chunk);
  counter++;
});

// 파일 읽기를 종료했을 때 실행되는 리스너
readStream.on('close', () => {
  console.log('close stream:');
});

readStream.on('error', (e) => {
  console.log('error:', e);
});
```
* fs.createReadStream은 파읽을 읽는 ReadableStream을 생성하는 함수임. 대상 파일을 읽는 Stream을 생성함
* highWaterMark 옵션에는 데이터양을 설정함

* '잠시 기다린 뒤, 읽은 내용을 파일에 쓴다'는 처리를 추가하는 코드
  * 무작위로 몇 초 기다린 뒤 파일에 내용을 추가하는 write 함수 작성 + data 이벤트 핸들러로 호출
  * setTimeout을 프로미스로 감싸서 비동기로 슬립하는 함수 선언하는 예시 코드
```javascript
const fs = require('fs');
const { writeFile } = require('fs/promises');

// 잠시 기다리는 비동기 함수
const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms));

const readStream = fs.createReadStream(__filename, { encoding: 'utf8', highWaterMark: 64 });
const writeFileName = `${__filename}-${Date.now()}`

const write = async (chunk) => {
  // (Math.random() * 1000)ms 동안 기다린다.
  await sleep(Math.random() * 1000);
  // 파일에 추가 모드로 쓴다.
  await writeFile(writeFileName, chunk, { flag: 'a' });
}

let counter = 0;
readStream.on('data', async (chunk) => {
  console.log(counter);
  counter++;

  await write(chunk);
});

readStream.on('close', () => {
  console.log('close');
});

readStream.on('error', (e) => {
  console.log('error:', e);
});
```
* data 이벤트의 핸들러를 async 함수로 만들어도 이전 data 데이터 이벤트를 대기하지 않고 다음 처리를 실행하기 때문에 결과값이 정신없이 쓰여있음
* 이벤트 핸들러는 '언제', '얼마나' 호출될지 제어할 수 없으므로 모두 병렬로 처리됨
* 병렬로 처리되면 실행 속도면에서는 유리하지만, 순차 쓰기와 같은 상황에서는 호환성이 좋지 못함

#### AsyncIterator를 이용한 개선
```javascript
const fs = require('fs');
const { writeFile } = require('fs/promises');
// 잠시 기다리는 비동기 처리
const sleep = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

const writeFileName = `${__filename}-${Date.now()}`

const write = async (chunk) => {
  // (Math.random() * 1000)ms 동안 대기한다.
  await sleep(Math.random() * 1000);
  // 파일에 추가 모드로 쏜다.
  await writeFile(writeFileName, chunk, { flag: 'a' });
}

const main = async () => {
  const stream = fs.CreateReadStream(__filename, { encoding: 'utf8', highWaterMark: 64 });

  let counter = 0;
  // 비동기로 발생하는 이벤트를 직렬로 처리한다.
  for await (const chunk of stream) {
    console.log(counter);
    counter++;

    await write(chunk);
  }
}

main()
  .catch((e) => console.error(e));
```
* AsyncIterator는 비동기 처리를 구현한 객체를 마치 배열과 같이 반복 처리 할 수 있음
* 각각의 요소는 for await ... of 안에서 선언한 변수에서 접근할 수 있음
* Stream 객체에서는 for await ... of로 data 이벤트를 반복 처리할 수 있음
* data 이벤트 처리 자체를 대기시킴으로써 쓰기 처리가 순차적으로 동작하게 됨

</div>
</details>

___

<details>
<summary>4.7 에러 핸들링 정리</summary>
<div markdown="1">    

* Node.js 에서 에러 핸들링이 누락돼 프로세스가 중단되면 모든 요청에서 에러가 발생해 큰 영향을 미침
  * 이것은 하나의 요청에 하나의 프로세스를 할당하는 모델과 달리 여러 요청을 하나의 프로세스에서 받는 Node.js의 약점인
  * 각 에러 핸들링 정리 표
  * 
동기/비동기 | 설계 | 에러 핸들링 
----------------- | ------------------ | ------------------ 
동기 처리 |  | try-catch, async/await
비동기 처리 | 콜백 | if (err)
비동기 처리 | EventEmitter(Stream) | emitter.on('error')
비동기 처리 | async/await | try-catch, .catch()
비동기 처리 | AsyncIterator | try-catch, .catch()

### 4.7.1 비동기 에러 핸들링
* 비동기는 콜백, EventEmitter(Stream), async/await(AsyncIterator 포함)의 경우 각각 다음 에러 핸들링이 필요함
  * 콜백: 에러의 null 체크
  * EventEmitter(Stream): 에러 이벤트 핸들링
  * async/await: try-catch와 상위 함수에서의 .catch()

* Node.js 애플리케이션은 비동기 처리가 중심이므로 현재 환경에서는 async/await 위주로 설계하는 것이 좋음

</div>
</details>

___

<details>
<summary>4.8 Top-Level Await</summary>
<div markdown="1">    

### 4.8.1 Top-Level Await와 Asynclterator의 주의점

</div>
</details>
