---
title: CleanCode) 오류 처리
author: cotchan 
date: 2020-12-16 08:52:21 +0800
categories: [CleanCode] 
tags: [cleancode]
---

CleanCode 책을 바탕으로 정리한 내용입니다.            
개인 공부 목적으로 작성한 글이며 지속적으로 업데이트할 예정입니다.        

---

## Intro. 오류 코드보다 예외를 사용하라

오류가 발생하면 예외를 던지는 편이 낫습니다. 그래야 호출자의 코드가 더 깔끔해집니다. 그래야 `논리가 오류 처리 코드와 뒤섞이지 않습니다.`

## 오류를 사용하는 경우

```java
public class DeviceController {
    ...
    public void sendShutDown() {
        DeviceHandle handle = getHandle(DEV1);

        //디바이스 상태를 점검한다.
        if (handle != DeviceHandle.INVALID) { 
            //레코드 필드에 디바이스 상태를 저장한다.
            retrieveDeviceRecord(handle);

            //디바이스가 일시정지 상태가 아니라면 종료한다.
            if (record.getStatus() != DEVICE_SUSPENDED) {
                pauseDevice(handle);
                clearDeviceWorkQueue(handle);
                closeDevice(handle);
            }
            else {
                logger.log("Device suspended. Unable to shut down");
            }
        }
        else {
            logger.log("Invalid handle for: " + DEV1.toString());
        }
    }
    ...
}
```

+ 위와 같은 방법을 사용하면 호출자 코드가 복잡해집니다. 
+ 함수를 `호출한 즉시 오류를 확인해야 하기 때문`입니다.
	+ 불행히도 이 단계는 잊어버리기가 쉽습니다. 
+ **그래서 오류가 발생하면 예외를 던지는 편이 낫습니다.** 
+ 그러면 호출자 코드가 더 깔끔해집니다. 논리가 오류 처리 코드와 뒤섞이지 않기 때문입니다.


---

## 예외를 사용하는 경우

```java
public class DeviceController {
    ...
    public void sendShutDown() {
        try { 
            tryToShutDown();
        } 
        catch (DeviceShutDownError e) {
            logger.log(e); 
        }
    }

    private void tryToShutDown() throws DeviceShutDownError {
        DeviceHandle handle = getHandle(DEV1);
        DeviceRecord record = retrieveDeviceRecord(handle);
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);	 
    }

    private DeviceHandle gethandle(DeviceID id) {
        ...
        throw new DeviceShutDownError("Invalid handle for: " + id.toString()); 
        ...
    }
	
    ...
}
```

---

## 1. Try-Catch-Finally 문부터 작성하라

+ **예외가 발생할 코드를 짤 때는 try-catch-finally문으로 시작하는 편이 낫습니다.** 
+ 그러면 try 블록에서 무슨 일이 생기든지 호출자가 기대하는 상태를 정의하기가 쉬워집니다.
+ try 블록에서 무슨 일이 생기든지 catch 블록은 프로그램 상태를 일관성 있게 유지해야 합니다.
 

+ 다음은 파일이 없으면 예외를 던지는지 알아보는 단위 테스트입니다.

```java
//part.1
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() { 
    sectionStore.retrieveSection("invalid - file");
}
```

+ 단위 테스트에 맞춰서 다음 코드를 구현합니다. 
+ 하지만 다음 코드는 예외를 던지지 않으므로 단위 테스트는 실패합니다. 

```java
//part.2
public List<RecordedGrip> retrieveSection(String sectionName) { 
    //실제로 구현할 때까지 비어 있는 더미를 반환합니다.
    return new ArrayList<RecordedGrip>();
}
```

+ 잘못된 파일에 접근을 시도하도록 아래와 같이 구현을 변경해봅니다.
+ 아래 코드는 예외를 던지므로 이제는 테스트가 성공합니다. 

```java
//part.3
public List<RecordedGrip> retrieveSection(String sectionName) { 
    try { 
        FileInputStream stream = new FileInputStream(sectionName);
    }
    catch (Exception e) {
        throw new StorageException("retrieval error", e); 
    }
    
    return new ArrayList<RecordedGrip>();
}
```

+ 코드가 예외를 던지므로 이제는 테스트가 성공합니다.
+ **이 시점에서 리팩터링이 가능합니다.**
+ `catch 블록에서 예외 유형을 좁혀` 실제로 FileInputStream 생성자가 던지는 FileInputStream 생성자가 던지는 FileNotFoundException을 잡아냅니다.

```java
//part.4
public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
        FileInputStream stream = new FileInputStream(sectionName);
	stream.close();
    }
    catch (FileNotFoundException e) {
        throw new StorageException("retrieval error", e);
    }
    
    return new ArrayList<RecordedGrip>();
}
```

try-catch 구조로 범위를 정의했으므로 TDD를 사용해 필요한 나머지 논리를 추가합니다.     
나머지 논리는 FileInputStream을 생성하는 코드와 close 호출문 사이에 넣으며 오류나 예외가 전혀 발생하지 않는다고 가정합니다.    
    
+ **`먼저 강제로 예외를 일으키는 테스트 케이스를 작성`한 후 테스트를 통과하게 코드를 작성하는 방법을 권장합니다.**
+ 그러면 자연스럽게 try 블록의 트랜잭션 범위부터 구현하게 되므로 범위 내에서 트랜잭션의 본질을 유지하기 쉬워집니다.


---

## 2. 미확인(Unchecked) 예외를 사용하라

+ 미확인 예외란 
    + `jang.lang.RuntimeException` 클래스를 상속한 클래스(`RuntimeException의 하위 클래스`)
    + **명시적인 예외 처리를 강제하지 않습니다.**
    + 실행 단계에서 확인이 가능합니다.
    + 예외 발생 시 트랜잭션을 rollback 합니다.
    + `NullPointerException`이나 `illegalArgumentException` 등이 있습니다.

![Desktop View](/assets/img/post/cleancode/2020-12-17-cleancode-unchecked-exception.png)


**`확인된 예외는` 개방-폐쇄 원칙을 위반합니다.** 메서드에서 확인된 예외를 던졌는데 catch 블록이 호출순서에서 세 단계 위에 있다면, 그 사이 메서드 모두가 선언부에 해당 예외를 정의해야 합니다. 즉, `하위 단계에서 코드를 변경하면 상위 단계 메서드 선언부를 전부 고쳐야 한다는 뜻입니다.` 모듈과 관련된 코드가 전혀 바뀌지 않았더라도 (선언부가 바뀌었으므로) 모듈을 다시 빌드한 다음 배포해야 한다는 뜻입니다.


---


## 3. 예외에 의미를 제공하라

예외를 던질 때는 전후 상황을 충분히 덧붙여줍니다. 그러면 오류가 발생한 원인과 위치를 찾기가 더 쉬워집니다. 자바는 모든 예외에 호출 스택을 제공합니다. 하지만 실패한 코드의 의도를 파악하려면 호출 스택만으로는 부족합니다. 

+ 개선방법
    + **오류 메시지에 정보를 담아 예외와 함께 던집니다.** 
    + **실패한 연산 이름과 실패 유형도 언급해줍니다.**
    + **애플리케이션이 로깅 기능을 사용한다면 catch 블록에서 오류를 기록하도록 충분한 정보를 넘겨줍니다.**
  

---

## 4. 호출자를 고려해 예외 클래스를 작성하라

+ 오류를 분류하는 방법은 수없이 많습니다. 
    + 오류가 발생한 위치로 분류 (ex. 오류가 발생한 컴포넌트로 분류)
    + 오류가 발생한 유형으로 분류 (ex. 디바이스 실패, 네트워크 실패, 프로그래밍 오류 등)
+ **하지만 애플리케이션에서 오류를 정의할 때 프로그래머에게 가장 중요한 관심사는 `오류를 잡아내는 방법`이 되어야 합니다.**

+ **결론부터 말하자면 `Wrapper 클래스를 사용`하여 하나의 예외 클래스만 두는 것입니다.**    

## 4-1. Before: Wrapper 클래스 사용 X

+ 아래 코드는 외부 라이브러리를 호출하는 try-catch-finally 문을 포함한 코드로, 외부 라이브러리가 던질 예외를 모두 잡아냅니다.

```java
//Before
ACMEPort port = new ACMEPort(12);

try {
	port.open();

} catch (DeviceResponseException e) {
	reportPortError(e);
	logger.log("Device response exception", e);

} catch (ATM1212UnlockedException e) {
	reportPortError(e);
	logger.log("Unlock exception", e);

} catch (GMXError e) {
	reportPortError(e);
	logger.log("Device response exception");

} finally {
	...
}
```

+ 위 코드는 중복이 심하지만 대다수 상황에서 우리가 오류를 처리하는 방식은 오류를 일으킨 원인과 무관하게 일정합니다.
    1. 오류를 기록한다.
    2. 프로그램을 계속 수행해도 좋은지 확인한다.

---

## 4-2. After: Wrapper 클래스 사용 O

+ 4-1의 경우는 `예외에 대응하는 방식이` 예외 유형과 무관하게 `거의 동일`합니다. 
+ 그래서 호출하는 라이브러리 `API를 감싸면서` `예외 유형 하나를 반환`하면 됩니다.

```java
LocalPort port = new LocalPort(12);

try {
	port.open();

} catch (PortDeviceFailure e) {
	reportError(e);
	logger.log(e.getMessage(), e);

} finally {
	...
}
```

+ **`LocalPort 클래스`는 단순히 ACMEPort 클래스가 던지는 예외를 잡아 변환하는 `Wrapper 클래스` 입니다.**

```java
public class LocalPort {
	private ACMEPort innerPort;

	public LocalPort(int portNumber) {
		innerPort = new ACMEPort(portNumber);
	}

	public void open() {
		try {
			innerPort.open();

		} catch (DeviceResponseException e) {
			throw new PortDeviceFailure(e);

		} catch (ATM1212UnlockedException e) {
			throw new PortDeviceFailure(e);

		} catch (GMXError e) {
			throw new PortDeviceFailure(e);

		} finally {
			...
		}
	}
	...
}
```

+ LocalPort 클래스처럼 ACMEPort를 감싸는 클래스는 매우 유용합니다.
+ **`실제로 외부 API를 사용할 때는 감싸기 기법이 최선입니다.`**
    + 외부 API를 감싸면 외부 라이브러리와 프로그램 사이에서 의존성이 크게 줄어듭니다. 
    + 그러므로 나중에 다른 라이브러리로 갈아타도 비용이 적습니다.
    + 또한 Wrapper 클래스에서 외부 API를 호출하는 대신 테스트 코드를 넣어주는 방법으로 프로그램을 테스트하기도 쉬워집니다.

+ Wrapper 기법을 사용하면 특정 업체가 API를 설계한 방식에 발목 잡히지 않습니다. 프로그램이 사용하기 편리한 API를 정의하면 그만입니다. 
    + 위 예제에서는 port 디바이스 실패를 표현하는 예외 유형 하나를 정의했는데, 그랬더니 프로그램이 훨씬 깨끗해졌습니다.

+ 흔히 예외 클래스가 하나만 있어도 충분히 코드는 많습니다.
    + 예외 클래스에 포함된 정보로 오류를 구분해도 괜찮은 경우가 그렇습니다.
    + **한 예외는 잡아내고 다른 예외는 무시해도 괜찮은 경우라면 여러 예외 클래스를 사용하면 됩니다.** 


---


## 5. 정상 흐름을 정의하라 

향후에 업데이트할 예정입니다. (p137)


---


## 6. null을 반환하지 마라

```java
//example
public void registerItem(Item item) {
	if (item != null) {
		ItemRegistry registry = peristentStore.getItemRegistry();
		if (registry != null) {
			Item existing = registry.getItem(item.getID());
			if (existing.getBillingPeriod().hasRetailOwner()) {
				existing.register(item); 
			}
		}
	}
}
```

+ 위 처럼 `null`을 반환하는 코드는 일거리를 늘릴 뿐만 아니라 호출자에게 문제를 떠넘깁니다. 누구 하나라도 `null` 확인을 빼먹는다면 애플리케이션이 통제 불능에 빠질지도 모릅니다.    
+ `null`로 인한 문제는 실상은 `null 확인이 너무 많아서` 문제가 됩니다.    
+ **메서드에서 null을 반환하고픈 유혹이 든다면 그 대신 `예외를 던지거나` `특수 사례 객체를 반환`하는 게 낫습니다.**
+ 사용하려는 외부 API가 null을 반환한다면 `Wrapper 메서드를 구현해` 예외를 던지거나 특수 사례 객체를 반환하는 방식을 고려해봅니다.

많은 경우에 특수 사례 객체가 손쉬운 해결책입니다. 

```java
//before
List<Employee> employees = getEmployees();
if (employees != null) { 
	for (Employee e : employees) {
		totalPay += e.getPay();
	}
}
```

+ 위에서 getEmployees는 null도 반환합니다. 하지만 반드시 null을 반환할 필요가 있을까요? getEmployees를 변경해 빈 리스트를 반환한다면 코드가 훨씬 깔끔해집니다. 

```java
//after
List<Employee> employees = getEmployees();
for (Employee e : employees) {
	totalPay += e.getPay(); 
}
```

+ 다행스럽게 자바에는 `Collections.emptyList()`가 있어 미리 정의된 읽기 전용 리스트를 반환합니다. 우리의 목적에 적합한 리스트입니다.     

```java
public List<Employee> getEmployees() { 
	if (.. 직원이 없다면 ..)
		return Collections.emptyList();
}
```

+ 이렇게 코드를 변경하면 코드도 깔끔해질뿐더러 NullPointerException이 발생할 가능성이 줄어듭니다.  


---


## 7. null을 전달하지 마라

+ 메서드에서 null을 반환하는 방식도 나쁘지만 메서드로 `null을 전달하는 방식은 더 나쁩니다.` 정상적인 인수로 null을 기대하는 API가 아니라면 메서드로 null을 전달하는 코드는 최대한 피해야 합니다.
+ 예제를 살펴보면 이유가 드러납니다. 아래는 두 지점 사이의 거리를 계산하는 간단한 메서드입니다.

```java
public class MetricsCalculator {
	public double xProjection(Point p1, Point p2) {
		return (p2.x - p1.x) * 1.5;
	}
	...
}
```

+ 누군가 인수로 null을 전달하면 어떤 일이 일어날까요? 당연히 NullPointerException이 발생합니다.

```java
calculator.xProjection(null, new Point(12, 13));
```


---


## 7-1. 솔루션: 새 예외 유형 만들어 던지기

```java

public class MetricsCalculator {
    public double xProjection(Point p1, Point p2) {
        if (p1 == null || p2 == null) {
            throw InvalidArgumentException(
                "Invalid argument for MetricsCalculator.xProjection");
        }
        return (p2.x - p1.x) * 1.5;	
    }
}
```

+ 위 코드가 원래 코드보다 나을까요? NullPointerException보다는 조금 나을지도 모르겠습니다. 하지만 위 코드는 `InvalidArgumentException`을 잡아내는 처리기가 필요합니다. 처리기는 InvalidArgumentException 예외를 어떻게 처리해야 좋을지는 또 고민해볼 문제입니다.


---


## 7-2. 솔루션: assert문 사용하기

+ 다음은 또 다른 대안으로 `assert문을 사용`하는 방법도 있습니다.

```java
public class MetricsCalculator {
	public double xProjection(Point p1, Point p2) {
		assert p1 != null : "p1 should not be null";
		assert p2 != null : "p2 should not be null";
		return (p2.x - p1.x) * 1.5;  
	}	
}
```

+ 위 코드는 문서화가 잘 되어 코드 읽기는 편하지만 문제를 해결하지는 못합니다. 누군가 null을 전달하면 여전히 실행 오류가 발생합니다.
+ 대다수 프로그래밍 언어는 호출자가 실수로 넘기는 null을 적절히 처리하는 방법이 없습니다. 
    + **그렇다면 애초에 null을 넘기지 못하도록 금지하는 정책이 합리적일 것입니다.** 
    + **즉, 인수로 null이 넘어오면 코드에 문제가 있다는 뜻입니다.**
+ 이런 정책을 따르면 그만큼 부주의한 실수를 저지를 확률도 작아집니다.


---


## 결론

깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 합니다. 이 둘은 상충하는 목표가 아닙니다. **`오류 처리를 프로그램 논리와 분리해` 독자적인 사안으로 고려하면 튼튼하고 깨끗한 코드를 작성할 수 있습니다.** 오류 처리를 프로그램 논리와 분리하면 독립적인 추론이 가능해지며 코드 유지보수성도 크게 높아집니다.  
 

---

+ 출처	
    + 로버트 C. 마틴, 『Clean Code』, 인사이트(2013), p129-p142
    + [JAVA Error와 Checked/Unchecked Exception](https://live-everyday.tistory.com/203)
    + [[Java]자바의 예외 - Exception, RuntimeException 그리고 Error](https://steady-hello.tistory.com/55)
