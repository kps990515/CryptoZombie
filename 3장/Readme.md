## 3장
 
### 1. 컨트랙트의 불변성
  컨트랙트는 배포하고 나면 변경 불가
  그러므로 인터페이스 사용시 주소선언이 아닌 주소set함수로 사용해야한다
  (외부 주소가 변경됐을 때 변경 가능하도록)
```javascript
  KittyInterface kittyContract;

  function setKittyContractAddress(address _address) external {
    kittyContract = KittyInterface(_address);
  }
```

### 2. Ownable 컨트랙트
 위처럼 external을 사용하면 외부 모두가 접근가능
 컨트랙트 소유자만 해당 함수 사용가능하도록 Ownable 컨트랙트 사용(상속받아서)

 #### 1. 생성자 
 컨트랙트가 생성될 때 한번만 실행
 ```javascript
  function Ownable() public {
    owner = msg.sender;
  }
```

 #### 2. 함수제어자 
 다른 함수들에 대한 접근을 제어할 때 사용
 ```javascript
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;
  }
```

### 3. 함수제어자
  modifier : 다른 함수들에 대한 접근을 제어할 때 사용

  1. likeABoss 호출
  2. modifer로 선언된 onlyOwner 함수 실행
  3. onlyOwner의 _; 를 만나면 likeABoss로 돌아감
  4. onlyOwner를 추가하면 해당 함수는 owner만 호출 가능하게 됨

 ```javascript
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;
  }

  contract MyContract is Ownable {
  event LaughManiacally(string laughter);

  function likeABoss() external onlyOwner {
    LaughManiacally("Muahahahaha");
  }
}
```

### 4. 구조체 압축을 통한 가스비 절감
 - uint의 경우 uint8, 16, 32 상관없이 256비트 저장공간 확보
 - 예외적으로 구조체의 경우는 선언한대로 저장공간 확보

```javascript
  struct MiniMe {
    uint32 a;
    uint32 b;
    uint c;
  }
```

### 5. 시간 단위(Time Units)
 - now : 1970년 1월 1일 ~ 현재까지의 초 단위 합
 - seconds, minutes, hours, days, weeks, years : 길이 만큼의 초 단위 uint로 변환

```javascript
  uint lastUpdated;

  // `lastUpdated`를 `now`로 설정
  function updateTimestamp() public {
    lastUpdated = now;
  }

  // 마지막으로 `updateTimestamp`가 호출된 뒤 5분이 지났으면 `true`를, 5분이 아직 지나지 않았으면 `false`를 반환
  function fiveMinutesHavePassed() public view returns (bool) {
    return (now >= (lastUpdated + 5 minutes));
  }
```

### 6. 시간 단위 활용 + 구조체 인수 전달
 - Zombie storage _zombie 구조체 인수 자체 전달
 - _isReady 함수를 통해 readyTime이 지났는지 확인

```javascript
  function _triggerCooldown(Zombie storage _zombie) internal {
    _zombie.readyTime = uint32(now + cooldownTime);
  }

  function _isReady(Zombie storage _zombie) internal view returns (bool) {
      return (_zombie.readyTime <= now);
  }
```

### 7. public 함수와 보안
 - 보안점검을 위해 public / external을 검사
 - 가장 쉬운 방법은 internal 선언

```javascript
  // 1. 이 함수를 internal로 만들게
  function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) internal {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    // 2. isReady로 쿨타임찼는지 확인
    require(_isReady(myZombie))
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    if (keccak256(_species) == keccak256("kitty")) {
      newDna = newDna - newDna % 100 + 99;
    }
    _createZombie("NoName", newDna);
    // 3. 새로운 쿨타임 부여
    _triggerCooldown(myZombie);
  }
```

### 8. 인수를 받는 함수제어자(modifier)
```javascript
// 사용자의 나이를 저장하기 위한 매핑
mapping (uint => uint) public age;

// 사용자가 특정 나이 이상인지 확인하는 제어자
modifier olderThan(uint _age, uint _userId) {
  require (age[_userId] >= _age);
  _;
}

// 차를 운전하기 위햐서는 16살 이상이어야 하네(적어도 미국에서는).
// `olderThan` 제어자를 인수와 함께 호출하려면 이렇게 하면 되네:
function driveCar(uint _userId) public olderThan(16, _userId) {
  // 필요한 함수 내용들
}
```

```javascript
contract ZombieHelper is ZombieFeeding {

  modifier aboveLevel(uint _level, uint _zombieId) {
    require(zombies[_zombieId].level >= _level);
    _;
  }

  function changeName(uint _zombieId, string _newName) external aboveLevel(2, _zombieId) {
    require(msg.sender == zombieToOwner[_zombieId]);
    zombies[_zombieId].name = _newName;
  }

  function changeDna(uint _zombieId, uint _newDna) external aboveLevel(20, _zombieId) {
    require(msg.sender == zombieToOwner[_zombieId]);
    zombies[_zombieId].dna = _newDna;
  }

}
```

### 9. view함수활용해 가스절약
 - view 함수는 사용자 호출 시 가스 소모 0
 - 가스는 수정(트랜잭션 생성)할 때 소모 
 - 읽기 전용 external view로 가스 최적화
```javascript
  function getZombiesByOwner(address _owner) external view returns(uint[]) {

  }
```

### 10. storage는 가스를 소모
 - storage 쓰기 연산 = 트랜잭션 생성 = 가스 소모
 - 배열 값 찾을때에는 memory 사용
```javascript
// memory 배열 선언
function getArray() external pure returns(uint[]) {
  // 메모리에 길이 3의 새로운 배열을 생성한다.
  // solidity에서는 배열크기 선 지정 필요
  uint[] memory values = new uint[](3);
  // 여기에 특정한 값들을 넣는다.
  values.push(1);
  values.push(2);
  values.push(3);
  // 해당 배열을 반환한다.
  return values;
}
```
 - memory로 배열 가져오는 방식
```javascript
  function getZombiesByOwner(address _owner) external view returns(uint[]) {
    uint[] memory result = new uint[](ownerZombieCount[_owner]);

    return result;
  }
```

### 11. for 반복문
 - 배열 내부 값이 하나 없어지면 가스 소모 극심
  1. 전달할 좀비를 새로운 소유자의 ownerToZombies 배열에 넣는다.
  2. 기존 소유자의 ownerToZombies 배열에서 해당 좀비를 지운다.
  3. 좀비가 지워진 구멍을 메우기 위해 기존 소유자의 배열에서 모든 좀비를 한 칸씩 움직인다.
  4. 배열의 길이를 1 줄인다.

 - owner가 소유한 좀비들 가져오기
```javascript
  function getZombiesByOwner(address _owner) external view returns(uint[]) {
    uint[] memory result = new uint[](ownerZombieCount[_owner]);
    uint counter = 0;
    for (uint i = 0; i < zombies.length; i++) {
      if (zombieToOwner[i] == _owner) {
        result[counter] = i;
        counter++;
      }
    }
    return result;
  }
```



