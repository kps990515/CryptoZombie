## 1장
 
### 1. 컨트랙트
```javascript
//1. 여기에 솔리디티 버전 적기
pragma solidity ^0.4.19;
//2. 여기에 컨트랙트 생성
contract HelloWorld {
    
}
```

### 2. 상태변수
 uint : 부호 없는 정수로, 값이 음수가 아니어야 한다는 의미
 ```javascript
 pragma solidity ^0.4.19;

contract ZombieFactory {

    uint dnaDigits = 16;

}
```

### 3. 구조체
 ```javascript
struct Person {
  uint age;
  string name;
}
```

### 4. 배열
1. 정적 배열 : uint[2] fixedArray;
2. 동적 배열 : uint[] dynamicArray;
3. 구조체 배열 : Person[] people;
4. Public 배열 : getter 메소드 자동 생성(읽기 가능, 쓰기 불가)
```javascript
Person[] public people;
```


### 5. 함수
- 함수 인자가 _ 인것은 지역변수임을 표시
```javascript
function eatHamburgers(string _name, uint _amount) {

}
```

