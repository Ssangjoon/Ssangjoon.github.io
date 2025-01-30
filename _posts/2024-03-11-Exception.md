---
date: '2024-03-10 22:30:30 +0900'
categories:
  - Book
tags:
  - study
published: false
---
## 프로그램 오류

- 컴파일 에러 : 컴파일 시에 발생하는 에러
- 런타임 에러 : 실행 시에 발생하는 에러
- 논리적 에러 : 실행은 되지만, 의도와 다르게 동작하는 것

자바에서는 실행 시 발생할 수 있는 에러를 에러와 예외 두가지로 구분하였다.

모든 예외의 최고 조상은 Exception 클래스다. 

- Object
    - Throwable
        - Error(프로그램 코드에 의해서 수습될 수 없는 심각한 오류)
            - OutOfMemoryError
            - StackOverflowError
        - Exception(프로그램 코드에 의해서 수습될 수 있는 다소 미약한 오류)
            - IOException
            - ClassNotFoundException
            - …
            - RuntimeException
                - NullPointerException
                - ClassCastException
                - IndexOutOfBoundsException
                - ArthmeticException

크게 보면 RuntimeException의 하위와 그렇지 않은 것들로 나뉘어 진다. 

- Exception 클래스들 : 사용자의 실수와 같은 외적인 요인에 의해 발생하는 예외
    
    존재하지 않는 파일의 이름 입력 → `FileNotFoundException`
    
    데이터 형식이 잘못 → `DataFormatException`
    
- RuntimeException 클래스들 : 프로그래머의 실수로 발생하는 예외
    
    배열의 범위를 벗어남 → `ArrayIndexOutOfBoundsException`
    
    값이 null인 참조변수의 멤버를 호출 → `NullPointerException`
    
    클래스간 형변환을 잘못 → `ClassCastException`
    
    정수를 0으로 나누는 경우 → `ArithmeticException`
    

### 예외 발생과 catch 블럭

**printStackTrace() 와 getMessage()** 

- printStackTrace()  ⇒ 예외발생 당시의 호출스택 (Call Stack)에 있었던 메서드의 정보와 예외 메시지를 화면에 출력한다.
- getMessage()  ⇒발생한 예외클래스의 인스턴스에 저장된 메시지를 얻을 수 있다.

**멀티 catch 블럭**

```java
try {
} catch (ExceptionA e) {
	e.printstackTrace(); 
} catch (ExceptionB e2) { 
	e2.printStackTrace();
}
---------------------------------------

try {
} catch (ExceptionA | ExceptionB e) { 
	e .printstackTrace();
}
```

```java
try{...}
catch (ParentException | ChildException e){ // 에러!
	e.printStackTrace();
}
	-----------------------------------------------------
// 두 예외 클래스가 조상, 자손 관계라면 그냥 조상만 써주는것과 동일하기 때문이다. 
try{...}
catch (ParentException e){ // 에러!
	e.printStackTrace();
}
```

### 예외 발생시키기

```java
Exception e = new Exception("고의로 발생시킴");
throw e;

throw new Exception("고의로 발생시킴"); // 위 두줄과 동일함
// 생성시 생성자에 String을 넣어 주면, getMessage()를 이용해 얻을 수 있다. 
```

위 예제의 경우 컴파일 에러가 나지만 

RuntimeException 클래스들이라면 컴파일은 성공하며 실행단계에서 에러가 난다. RuntimeException 클래스들에 해당하는 예외는 프로그래머의 실수로 발생하는 것들이기 때문에 예외처리를 강제하지 않는다. 

컴파일러가 예외처리를 확인하지 않는 RuntimeException 클래스들은 unchecked 예외라고 부르고 예외처리를 확인하는 Exception클래스들은 checked예외 라고 부른다.  

### 사용자 정의 예외 만들기

```java
class MyException extends Exception {
	// 에러코드값을저장하기위한필드를추가 했다.
	private final int ERR_C0DE; // 생성를 통해 초기화 한다.
	MyException (String msg, int errCode) { // 생성자
		super(msg);
		ERR_C0DE = errCode;
	} 

	MyException (String msg) { // 생성자
		this (msg, 100); // ERR_C0DE 를 100(기본값) 으로 초기화한다.
	}

	public int getErrCode() { // 에러 코드를 얻을 수 있는 메서드도 추가했다.
		return ERR_C0DE; // 이 메서드는 주로 getMessage ()와 함께사용될 것이다.
	}
}
```

기존의 예외 클래스는 주로 Exception을 상속 받아 checked예외로 작성하는 경우가 많았지만, 
요즘은 예외처리를 선택적으로 할 수 있도록 RuntimeException을 상속 받아서 작성하는 쪽으로 바뀌어가고 있다.