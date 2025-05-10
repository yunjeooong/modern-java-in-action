# Chapter 11. null 대신 Optional 클래스

## 11.1 값이 없는 상황을 어떻게 처리할까?

자바에서 `null`을 통해 값이 없다는 것을 표현하는 방식은 다양한 문제를 일으킨다. 특히 `NullPointerException`(NPE)은 자바에서 가장 흔한 에러 중 하나다. 아래 코드처럼 단순한 상황에서도 NPE가 발생할 수 있다:

```java
public String getCarInsuranceName(Person person) {
  return person.getCar().getInsurance().getName();
}
```

### 11.1.1 보수적인 자세로 NullPointerException 줄이기

NPE를 방지하기 위해 필요한 곳에 다양한 null 확인 코드를 추가하는 방법이 있다:

```java
public String getCarInsuranceName(Person person) {
  if(person != null) {
    Car car = person.getCar();
    if(car != null) {
      Insurance insurance = car.getInsurance();
      if(insurance != null) {
        return insurance.getName();
      }
    }
  }
  return "Unknown";
}
```

이러한 반복 패턴 코드를 **깊은 의심(deep doubt)** 이라 부른다. 변수에 접근할 때마다 중첩된 if가 추가되면서 코드 들여쓰기 수준이 증가하고 가독성이 떨어진다.

다른 방법으로는 중첩 if 블록을 없애고 여러 출구를 두는 방식이 있지만, 이 경우에도 출구가 많아 유지보수가 어려워질 수 있다.

### 11.1.2 null 때문에 발생하는 문제

`null`로 값이 없다는 사실을 표현하는 것은 다음과 같은 문제를 일으킨다:

1. **에러의 근원**: NPE는 자바에서 가장 흔하게 발생하는 에러다.
2. **코드를 어지럽힘**: 중첩된 null 확인 코드를 추가해야 해서 코드 가독성이 떨어진다.
3. **아무 의미가 없음**: null은 아무 의미도 표현하지 않는다. 값이 없음을 표현하는 방식으로 적절하지 않다.
4. **자바 철학에 위배**: 자바는 개발자로부터 모든 포인터를 숨겼지만, null 포인터는 예외다.
5. **형식 시스템에 구멍**: 모든 참조 형식에 null을 할당할 수 있어, null이 할당되기 시작하면 시스템 전체로 퍼져 원래 어떤 의미로 사용되었는지 알 수 없게 된다.

### 11.1.3 다른 언어의 null 대체재

그루비(Groovy)와 같은 언어에서는 안전 내비게이션 연산자(safe navigation operator) `?.`를 도입해 null 문제를 해결했다:

```groovy
def carInsuranceName = person?.car?.insurance?.name
```

## 11.2 Optional 클래스 소개

자바 8은 하스켈과 스칼라의 영향을 받아 `java.util.Optional<T>`라는 새로운 클래스를 제공한다. Optional은 선택형 값을 캡슐화하는 클래스다:

- 값이 있으면 Optional 클래스는 값을 감싼다.
- 값이 없으면 `Optional.empty` 메서드로 Optional을 반환한다. (`Optional.empty`는 Optional의 특별한 싱글턴 인스턴스를 반환하는 정적 팩토리 메서드다)

null 참조와 Optional.empty에는 중요한 차이가 있다. null을 참조하려 하면 NPE가 발생하지만, Optional.empty()는 Optional 객체이므로 다양한 방식으로 활용할 수 있다.

Optional 클래스를 사용하면:
- 모델의 의미(semantic)가 더 명확해진다. 변수가 Optional이면 그 값이 있을 수도, 없을 수도 있다는 것을 명시적으로 표현한다.
- 더 이해하기 쉬운 API를 설계할 수 있다. 메서드 시그니처만 보고도, 값이 없을 수 있는 상황인지 여부를 판단할 수 있다.
- Optional이 등장하면 값이 없을 수 있는 상황에 대응하도록 강제하는 효과가 있다.

예를 들어, Person/Car/Insurance 데이터 모델을 Optional을 사용해 재정의하면 다음과 같다:

```java
public class Person {
    private Optional<Car> car; // 사람은 차를 소유했을 수도, 소유하지 않았을 수도 있음

    public Optional<Car> getCar() {
        return car;
    }
}

public class Car {
    private Optional<Insurance> insurance; // 자동차가 보험에 가입되어 있을 수도, 가입되어 있지 않았을 수도 있음

    public Optional<Insurance> getInsurance() {
        return insurance;
    }
}

public class Insurance {
    private String name; // 보험회사에는 이름이 반드시 있음

    public String getName() {
        return name;
    }
}
```

## 11.3 Optional 적용 패턴

### 11.3.1 Optional 객체 만들기

**빈 Optional**
```java
Optional<Car> optCar = Optional.empty();
```

**null이 아닌 값으로 Optional 만들기**
```java
Optional<Car> optCar = Optional.of(car);
```
car가 null이면 즉시 NPE가 발생한다.

**null값으로 Optional 만들기**
```java
Optional<Car> optCar = Optional.ofNullable(car);
```
car가 null이면 빈 Optional 객체가 반환된다.

### 11.3.2 맵으로 Optional의 값을 추출하고 변환하기

스트림의 map과 비슷하게 Optional의 map 메서드는 값이 존재할 때만 제공된 함수를 적용한다:

```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

이렇게 하면 기존에 null 체크를 하던 코드를 다음과 같이 간결하게 변경할 수 있다:

```java
// 기존 코드
String name = null;
if(insurance != null) {
    name = insurance.getName();
}

// Optional 사용
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

### 11.3.3 flatMap으로 Optional 객체 연결

여러 계층의 객체에 접근할 때는 flatMap을 사용해야 한다. 다음과 같이 map만 사용하면 컴파일되지 않는다:

```java
Optional<Person> optPerson = Optional.of(person);
Optional<String> name =
        optPerson.map(Person::getCar) // Optional<Optional<Car>>
                .map(Car::getInsurance) // Optional<Optional<Insurance>>
                .map(Insurance::getName);
```

flatMap을 사용하면 중첩된 Optional을 평준화할 수 있다:

```java
public String getCarInsuranceName(Optional<Person> person) {
    return person.flatMap(Person::getCar)
           .flatMap(Car::getInsurance)
           .map(Insurance::getName)
           .orElse("Unknown"); // Optional이 비어있으면 기본값 사용
}
```

### 11.3.4 도메인 모델에 Optional을 사용했을 때 데이터를 직렬화할 수 없는 이유

Optional 클래스는 필드 형식으로 사용할 것을 가정하지 않았으므로 `Serializable` 인터페이스를 구현하지 않았다. 따라서 도메인 모델에 Optional을 사용하면 직렬화 모델을 사용하는 도구나 프레임워크에서 문제가 생길 수 있다.

직렬화 모델이 필요하다면 변수는 일반 객체로 두되, Optional로 값을 반환받는 메서드를 추가하는 방식이 권장된다:

```java
public class Person {
    private Car car;
    
    public Optional<Car> getCarAsOptional() {
        return Optional.ofNullable(car);
    }
}
```

### 11.3.5 Optional 스트림 조작

자바 9에서는 Optional을 포함하는 스트림을 쉽게 처리할 수 있도록 Optional에 stream() 메서드를 추가했다:

```java
public Set<String> getCarInsuranceNames(List<Person> persons) {
    Stream<Optional<String>> stream = persons.stream()
        .map(Person::getCar) // 사람 목록을 각 사람이 보유한 자동차의 Optional<Car> 스트림으로 변환
        .map(optCar -> optCar.flatMap(Car::getInsurance)) // Optional<Car>를 해당 Optional<Insurance>로 변환
        .map(optIns -> optIns.map(Insurance::getName)); // Optional<Insurance>를 해당 이름의 Optional<String>으로 변환
        
    // 자바 9 이전
    return stream.filter(Optional::isPresent)
        .map(Optional::get)
        .collect(toSet());
        
    // 자바 9 이후
    return stream.flatMap(Optional::stream)
        .collect(toSet());
}
```

### 11.3.6 디폴트 액션과 Optional 언랩

Optional 클래스는 값을 읽는 다양한 방법을 제공한다:

1. **get()**: 값이 있으면 값을 반환하고, 값이 없으면 NoSuchElementException을 발생시킨다. 값이 반드시 있다고 확신할 수 있는 상황에서만 사용해야 한다.

2. **orElse(T other)**: Optional이 값을 포함하지 않을 때 기본값을 제공한다.
   ```java
   Insurance insurance = optInsurance.orElse(new Insurance("Unknown"));
   ```

3. **orElseGet(Supplier<? extends T> other)**: Optional이 비어있을 때만 Supplier가 실행된다(게으른 평가).
   ```java
   Insurance insurance = optInsurance.orElseGet(() -> new Insurance("Unknown"));
   ```

4. **orElseThrow(Supplier<? extends X> exceptionSupplier)**: Optional이 비어있을 때 지정한 예외를 발생시킨다.
   ```java
   Insurance insurance = optInsurance.orElseThrow(() -> 
       new RuntimeException("해당 제품 ID는 없는 ID입니다."));
   ```

5. **ifPresent(Consumer<? super T> consumer)**: 값이 존재할 때 해당 값으로 동작을 수행한다.
   ```java
   optionalValue.ifPresent(value -> System.out.println(value));
   ```

6. **ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)** (자바 9): Optional이 값을 가질 때와 비어있을 때 각각 다른 동작을 수행한다.
   ```java
   optionalValue.ifPresentOrElse(
       value -> System.out.println("값 존재: " + value),
       () -> System.out.println("값 없음")
   );
   ```

### 11.3.7 두 Optional 합치기

두 Optional 객체를 조합하여 연산을 수행할 때는 flatMap을 활용할 수 있다:

```java
public Insurance findCheapestInsurance(Person person, Car car) {
    // 다양한 보험회사가 제공하는 서비스 조회
    // 모든 결과 데이터 비교
    return cheapestCompany;
}

// Optional을 사용한 버전
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
    if (person.isPresent() && car.isPresent()) {
        return Optional.of(findCheapestInsurance(person.get(), car.get()));
    } else {
        return Optional.empty();
    }
}

// flatMap과 map 활용
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
    return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
}
```

### 11.3.8 필터로 특정값 거르기

filter 메서드를 이용해 Optional 값이 특정 조건을 만족할 때만 값을 유지하고, 그렇지 않으면 빈 Optional을 반환할 수 있다:

```java
// 보험회사 이름이 'CambridgeInsurance'인지 확인
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurance -> "CambridgeInsurance".equals(insurance.getName()))
        .ifPresent(x -> System.out.println("ok"));

// 나이 제한 확인
public String getCarInsuranceName(Optional<Person> person, int minAge){
    return person.filter(p -> p.getAge() >= minAge)
        .flatMap(Person::getCar)
        .flatMap(Car::getInsurance)
        .map(Insurance::getName)
        .orElse("Unknown");
}
```

## 11.4 Optional을 사용한 실용 예제

### 11.4.1 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기

기존 API가 null을 반환하는 경우, Optional로 감싸서 안전하게 사용할 수 있다:

```java
// Map.get() 메서드의 결과를 Optional로 감싸기
Optional<Object> value = Optional.ofNullable(map.get("key"));
```

### 11.4.2 예외와 Optional 클래스

값을 제공할 수 없을 때 예외를 발생시키는 API도 Optional을 활용해 개선할 수 있다:

```java
// Integer.parseInt() 예외 처리를 Optional로 변환
public static Optional<Integer> stringToInt(String s) {
    try {
        return Optional.of(Integer.parseInt(s));
    } catch(NumberFormatException e) {
        return Optional.empty();
    } 
}
```

이렇게 하면 복잡한 try/catch 구문 없이 값이 없는 상황을 깔끔하게 처리할 수 있다.

### 11.4.3 기본형 Optional을 사용하지 말아야 하는 이유

Optional도 기본형에 특화된 `OptionalInt`, `OptionalLong`, `OptionalDouble` 등의 클래스를 제공한다. 하지만 다음 이유로 사용을 권장하지 않는다:

1. Optional의 최대 요소 수는 한 개이므로 기본형 특화 클래스로 성능 개선이 미미하다.
2. 기본형 특화 Optional은 map, flatMap, filter 등 중요한 메서드를 제공하지 않는다.
3. 기본형 Optional과 일반 Optional은 호환되지 않아 함께 사용하기 어렵다.

## 11.5 실전 활용 예시

### ifPresent 사용 예제

기존 코드:
```java
Member member = memberRepository.findById(id);
if (member != null) {
    if (member.isAdmin()) {
        member.addAdminPermissions();
    } else {
        member.addDefaultPermissions();
    }
}
```

Optional 적용 후:
```java
Optional<Member> memberOptional = memberRepository.findById(id);
memberOptional.ifPresent(member -> {
    if (member.isAdmin()) {
        member.addAdminPermissions();
    } else {
        member.addDefaultPermissions();
    }
});
```

### 예외 대신 Optional 반환

```java
private Product findProductById(Long productId) {
  return Optional.ofNullable(productRepository.findOne(productId))
    .orElseThrow(() -> new RuntimeException("해당 제품 ID는 없는 ID입니다."));
}
```

### orElse와 orElseGet 사용 예

```java
Optional<Object> objectOptional = Optional.empty();

Object object1 = objectOptional.orElse(new Object()); // 항상 새 객체 생성
Object object2 = objectOptional.orElseGet(() -> new Object()); // 필요할 때만 새 객체 생성
Object object3 = objectOptional.orElseGet(Object::new); // 메서드 참조 활용
```

### 객체 변환 시 Optional 활용

```java
Optional<UserVo> userVo = Optional.ofNullable(findUserById(employeeVo));
Employee employee = new Employee();
employee.setName(userVo.map(UserVo::getName).orElse(""));
// 또는 예외 발생
// employee.setName(userVo.map(UserVo::getName)
//    .orElseThrow(() -> new RuntimeException("ID에 해당하는 유저가 존재하지 않습니다.")));
```
