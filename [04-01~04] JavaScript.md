## [04/01-04] JavaScript



### JavaScript

- var vs. let

  var에서 중복 선언 가능

  var에서 호이스팅이 됨 (선언된 변수보다 먼저 써도 문제가 없음 / 단, 초기화는 되지 않아 undefined 상태)

  var: 함수 단위 / let: 블록 단위



- 배열

  push(값): 마지막에 값 추가

  indexOf(값): 값이 어떤 인덱스에 있는지 확인, 찾지 못하면 -1 return

  sort((a, b) => a - b): 버블 정렬 알고리즘을 이용하여 배열 오름차순 정렬



- document 관련

  getElementById(): DOM 메소드, HTML 문서에서 특정 id를 가진 요소를 찾음

  value: HTML 요소(element)의 속성(attribute) 중 하나로 사용자 입력 요소(input), textarea 등과 같이 값을 가지는 요소들에서 해당 요소의 값을 가져오거나 설정하는 데 사용

  innerHTML: HTML 요소의 내부 HTML 코드를 나타내는 속성으로 내부 HTML 코드를 읽거나 수정할 수 있음



- 이벤트

  onkeydown: 키보드에 입력이 될 때마다 발생하는 이벤트



- 연산자

  - 논리 연산자를 사용한 단축 평가

    - 논리 AND (&&)

      첫 번째 피연산자가 true로 평가되면 두 번째 피연산자의 값 반환, false인 경우 첫 번째 피연산자의 값을 반환

    - 논리 OR (||)

      첫 번째 피연산자가 false로 평가되면 두 번째 피연산자의 값 반환, true인 경우 첫 번째 피연산자의 값을 반환

    - 논리 Nullish (??)

      첫 번째 피연산자가 null이나 undefined가 아니면 첫 번째 피연산자의 값 반환, 그렇지 않으면 두 번째 피연산자의 값을 반환

  - 조건부(삼항) 연산자

    조건이 true로 평가되면 표현식1, false인 경우 표현식2를 실행

  - 할당 연산자를 이용한 단축

    할당과 동시에 연산하는 연산자(+=, -=, *=, /=, %= 등)를 사용하여 값을 업데이트 하면서 할당 가능



#### 함수

JS의 함수는 일급 객체로 취급

함수도 type이 될 수 있음

함수는 중복 정의 가능

메소드 오버로딩 X, 마지막에 정의된 함수가 저장됨

```jsx
function add(a) {
	return a + a;
}

function add(a, b) {
	return a + b;
}

// 메소드가 오버로딩되지 않기 때문에 나중에 선언된 add만 실행됨
add(3);
add(3, 5);
```

상수에 함수 저장 가능

호이스팅되지 않음

```jsx
function test() {
    console.log("test");
}

// let c = test;   // test면 c에 함수 전체 대입, test()면 함수 실행 후 값을 대입

// 함수의 매개 변수로 함수를 받을 수 있음
function c(abc) {
    abc();
}

c(test);
```

파라미터 = arguments 배열로 저장됨

return 이후로는 호출되지 않음

- 화살표 함수

  화살표 이후 중괄호가 없으면 자동으로 return, 중괄호를 사용하면 return을 명시해야 함



- 객체

  함수가 객체 안에 들어가면 this는 자신이 속한 객체를 가리킴

  객체 안에서 화살표 함수를 사용하면 this는 Windows가 됨



- 배열

  배열 안의 타입은 모두 같지 않아도 됨

  `const array = new Array(...n)` 선언 시 인자가 1개라면 그 크기의 배열을, 2개 이상이라면 인덱스의 값이 됨

  - 내장 함수

    forEach: 함수 형태의 파라미터를 전달하는 콜백함수

    map: 배열 안의 각 원소를 변환할 때 사용, 새로운 배열이 만들어짐

    indexOf: 객체가 아닌 타입의 인덱스 위치 반환

    findIndex: 객체의 인덱스 위치 반환

    find: 객체에서 찾아낸 값 반환

    filter: 조건을 만족하는 값만 추출

    splice(index, n): index부터 n개 제거

    slice: 원본 배열에서 추출

    shift: 첫번째 원소를 배열에서 꺼내기

    unshift: 맨앞에 원소 추가

    join(구분자): 배열 안의 값을 문자열 형태로 합침 / 기본 구분자는 ,(comma)

    reduce(콜백함수, 초기값)



- for in vs. of

  of: 배열을 순회하면서 값 출력

  in: 객체를 순회하면서 값 출력



#### 실습문제

```jsx
// 문제 1: 배열 모든 요소 출력
const numbers = [1, 2, 3, 4, 5];
console.log(numbers);

// 문제 2: 객체의 모든 프로퍼티와 값 출력
const person = {
    name: '홍길동',
    age: 25,
    email: 'hong@example.com'
};
console.log(person);

// 문제 3: 특정 값의 개수 찾기
const number = [1, 3, 7, 8, 3, 3, 4, 3];
const targetNumber = 3;
let count = 0;
for(let i of number) {
    if (i === targetNumber) {
			count++;
    }
}
console.log(count);

// 문제 4: 최대값 찾기
const num = [10, 36, 54, 89, 12];
let max = num[0];
for(let i of num) {
    if (i > max) {
			max = i;
    }
}
console.log(max);

// 문제 5: 객체 배열에서 특정 조건의 객체 찾기
// 나이 30 이상인 이름 출력
const users = [
    { name: 'Alice', age: 25 },
    { name: 'Bob', age: 30 },
    { name: 'Charlie', age: 28 },
    { name: 'Dave', age: 35 }
];
let overThirty = [];
for (let user of users) {
    if(user.age >= 30) {
			overThirty.push(user);
    }
}
console.log(overThirty);
```

```javascript
// 1. 배열 내용 제곱
const numbers = [1, 2, 3, 4, 5];
const pow = [];
numbers.map(i => pow.push(Math.pow(i, 2)));
console.log(pow);

// 2. 짝수만 필터링
const number = [1, 2, 3, 4, 5, 6];
const even = [];
number.filter(i => {
    if (i%2 === 0) {
even.push(i);
    }
});
console.log(even);

// 3. 배열 인덱스의 총 합
console.log(numbers.reduce((sum, i) => sum += i, 0));

// 4. 나이가 30 이상만 추출
const people = [
    { name: 'Alice', age: 25 },
    { name: 'Bob', age: 30 },
    { name: 'Charlie', age: 28 }
];
const overThiry = people.find(i => i.age >= 30);
console.log(overThiry);

// 5. 10 이상인지 확인
const num = [10, 20, 30, 40, 50];
const overTen = num.every(i => i>=10);
console.log(overTen);

// 6. 홀수가 하나라도 있는지 확인
const n = [2, 4, 6, 8, 10, 11];
const result = n.some(n => n%2!==0);
console.log(result);
```



#### 객체

객체 생성자: 함수를 통해서 새로운 객체를 만들고 그 안에 넣고 싶은 값 or 함수를 구현할 수 있게 해줌

새로운 객체를 만들 때 new 키워드 필요

프로토타입: 객체 생성자 함수 아래에 `.prototype.[키] = 코드` 입력

```javascript
function Animal(type, name, sound) {
    this.type = type;
    this.name = name;
    this.sound = sound;
}

Animal.prototype.say = function() {
    console.log(this.sound);
}

Animal.prototype.sharedValue = 1;

const dog = new Animal('dog', 'mimi', 'bark bark!');
const cat = new Animal('cat', 'nana', 'meow meow!');

console.log(dog.sharedValue);
console.log(cat.sharedValue);

// 재정의 시 해당 항목에만 적용됨
cat.sharedValue = 2;
console.log(dog.sharedValue);
console.log(cat.sharedValue);

// 부모의 say를 쓰고 싶을 떄
Animal.prototype.say.call(cat);
```

- 웹 서버 (Web Server)

  데이터베이스에 접속하여 해당 사이트에 포함된 정적 컨텐츠를 보여줌

  ex. html, css, image

- 웹 어플리케이션 서버 (WAS, Web Application Server)

  동적인 컨텐츠를 처리하기 위해 사용하는 미들웨어



#### 비동기 처리

- 동기 처리: 이전 작업이 끝날 때까지 대기하는 동안 중지
- 비동기 처리: 동시에 여러 가지 작업 처리 가능

```jsx
function work() {
    const start = Date.now();
    for (let i=0; i<1000000000; i++) {
    }
    const end = Date.now();
    console.log(end - start + "ms");
}

work();
console.log('next work');

function hi() {
    console.log('hello');
}

// 1000ms 후에 실행
setTimeout(hi, 1000);

console.log('hi next work');
```

해당 코드를 실행하면 hi 함수보다 'hi next work'이라는 문자가 먼저 출력됨

setTimeout를 0으로 변경해도 동일



#### Promise

비동기 작업을 편하게 처리할 수 있도록 하는 기능

콜백 함수가 여러 개 있으면 복잡하다는 단점 해결

```jsx
const myPromise = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(1);
    }, 1000);
});
// resolve: 성공했을 때
// reject: 실패했을 때
const myPromise = new Promise((resolve, reject) => {
    setTimeout(() => {
resolve(1);
    }, 1000);
});

myPromise.then(n => {
    console.log(n);
});

const myPromise2 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject(new Error());
    }, 1000);
});

myPromise2
    .then(n => {
        console.log(n);
    })
    .catch(error => {
        console.log(error);
    });

function increaseAndPrint(n) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            const value = n + 1;
                if (value === 5) {
                    const error = new Error();
                    error.name = 'ValueIsFiveError';
                    reject(error);
                    return;
                }
            console.log(value);
            resolve(value);
        }, 1000);
    });
}

increaseAndPrint(0)
    .then(n => {
        return increaseAndPrint(n);
    })
    .then(n => {
        return increaseAndPrint(n);
    })
    .then(n => {
        return increaseAndPrint(n);
    })
    .then(n => {
        return increaseAndPrint(n);
    })
    .then(n => {
        return increaseAndPrint(n);
    })
    .catch(e => {
        console.error(e);
    });

increaseAndPrint(0)
    .then(increaseAndPrint)
    .then(increaseAndPrint)
    .then(increaseAndPrint)
    .then(increaseAndPrint)
    .then(increaseAndPrint)
    .catch(e => {
        console.error(e);
    });
```



#### async/await

async를 함수 앞에 붙이고, Promise 함수 앞에 await를 넣으면 해당 Promise가 끝날 때까지 기다렸다가 다음 작업 수행

```jsx
function sleep(ms) {
    return new Promise((resolve) => setTimeout(resolve, ms));
}

// async를 넣지 않으면 기다리지 않고 바로 출력
async function process() {
    console.log('hi');
    await sleep(1000);    // 비동기
    console.log('nice to meet you');
}

console.log('before process');
process();
console.log('after process');
```

Promise.all: 작업을 동시에 수행

Promise.race: 다른 Promise가 먼저 성공하기 전에 먼저 끝난 Promise가 실패하면 실패로 간주



#### 이벤트

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>event</title>
</head>
<body>
    <button id="btn">click</button>
    <script>
        // source에 eventListener(이벤트명, 이벤트핸들러(함수), [ToCapture])

        function hi() {
            alert('hi');
        }
        document.addEventListener('click', hi); // body와 button 둘다 적용됨
    </script>
</body>
</html>
```

- document 관련

  querySelector: 처음 나오는 태그 리턴

  querySelectorAll: 태그에 해당하는 리스트 리턴

  getElementTagName: 태그에 해당하는 리스트 리턴

  createElement: 해당 요소 추가

  appendChild(): 해당 태그에 값 추가



- 이벤트 관련

  removeEventListener(): 이벤트가 한번만 실행되고 삭제됨

  target: 이벤트가 발생한 요소



- 이벤트 전파

  위쪽에 있는 box 영향이 밑쪽의 box에도 미침

  bubbling: 가장 아래 box부터 전파 (기본 설정)

  capturing: 가장 위 box부터 전파

  stopPropagation(): 이벤트 전파를 막는 함수

  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8" />
      <title>Event 전파</title>
      <style>
        .pink {
          position: relative;
          background-color: pink;
          height: 200px;
          width: 200px;
          display: flex;
          justify-content: center;
          align-items: center;
        }
        .red {
          position: absolute;
          top: 0;
          left: 0;
          background-color: red;
          width: 100px;
          height: 100px;
        }
      </style>
    </head>
    <body>
      <div class="pink">
        <div class="red"></div>
      </div>
  
      <script>
        let pinkbox = document.querySelector(".pink");
        let redbox = document.querySelector(".red");
        let body = document.querySelector("body");
        let html = document.querySelector("html");
        pinkbox.addEventListener(
          "click",
          function (e) {
          //   e.stopPropagation();   // 이벤트 전파를 막는 함수
            alert("pink box!!");
          },
          false   // false: bubbling으로 진행, true: capturing으로 진행
        );
        redbox.addEventListener(
          "click",
          function (e) {
            //   e.stopPropagation();
            alert("red box");
          },
          false
        );
  
        body.addEventListener(
          "click",
          function (e) {
            //   e.stopPropagation();
            alert("body box");
          },
          false
        );
  
        html.addEventListener(
          "click",
          function (e) {
            e.stopPropagation();
            alert("html box");
          },
          false
        );
      </script>
    </body>
  </html>
  ```



### DOM (Document Object Model)

XML이나 HTML 문서에 접근하기 위한 일종의 인터페이스

문서 내의 모든 요소를 정의하고 각각의 요소에 접근하는 방법 제공



### jQuery

장점: 간결한 문법, 편리한 API, 크로스 브라우징

code.jquery.com에서 -> code intergration을 해야 사용 가능

$(선택자).val(): 선택자의 요소 가져오기 / 값을 설정할 때는 val 안에 내용을 입력하면 됨

click([함수명]): 클릭 이벤트

fadeIn(): 나타나게 하는 이벤트

fadeOut(): 사라지게 하는 이벤트

animate(properties, []..): css 요소가 어떻게 변화할 건지 설정

css(propertyName): css 요소 변경

text(): 선택된 요소를 해당 문자열로 설정

------

https://www.youtube.com/watch?v=z1Jj7Xg-TkU

https://www.youtube.com/watch?v=8aGhZQkoFbQ&t=2s

https://velog.io/@sugyinbrs/Event-Loop-Call-Stack-이-작동하는-법

https://velog.io/@ckanywhere/Eventloop-와-asyncawait

https://developer-talk.tistory.com/848