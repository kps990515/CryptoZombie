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
#### 1. 정적 배열 : uint[2] fixedArray;
#### 2. 동적 배열 : uint[] dynamicArray;
#### 3. 구조체 배열 : Person[] people;
#### 4. Public 배열 : getter 메소드 자동 생성(읽기 가능, 쓰기 불가)
```javascript
Person[] public people;
```
- 생성 & 추가
```javascript
// 새로운 사람을 생성한다:
Person satoshi = Person(172, "Satoshi");

// 이 사람을 배열에 추가한다:
people.push(satoshi);
```
```javascript
// 한줄로 표현
people.push(Person(16, "Vitalik"));
```


### 5. 함수
####1. 함수 인자가 _ 인것은 지역변수임을 표시
```javascript
function eatHamburgers(string _name, uint _amount) {

}
```
#### 2. public/private
- 기본적으로 public으로 선언
- private 선언 시 함수명에 _ & 맨 마지막에
```javascript
function _addToArray(uint _number) private {
  numbers.push(_number);
}
```

#### 3. return
- view 함수 : 데이터를 읽기만 하고 변경 X
```javascript
function sayHello() public returns (string) {
  return greeting;
}
```

- pure함수 : 어떤 데이터도 접근하지 않음(인자값만 활용)
```javascript
function _multiply(uint a, uint b) private pure returns (uint) {
  return a * b;
}
```

### 6. 이벤트
- 컨트랙트가 블록체인 상에서 특정 액션을 발생시켰을때 실행하기 위한 코드
```javascript
// 이벤트를 선언한다
event IntegersAdded(uint x, uint y, uint result);

function add(uint _x, uint _y) public {
  uint result = _x + _y;
  // 이벤트를 실행하여 앱에게 add 함수가 실행되었음을 알린다:
  IntegersAdded(_x, _y, result);
  return result;
}
```

7. 전체 코드
```javascript
pragma solidity ^0.4.19;
//컨트랙트 선언
contract ZombieFactory {
    //이벤트 선언
    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;
    //구조체 선언
    struct Zombie {
        string name;
        uint dna;
    }
    // 배열 선언
    Zombie[] public zombies;

    function createRandomZombie(string _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

    function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }
}
```