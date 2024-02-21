---
title: "[Javs] Lambda와 Capturing"
date: 2024-02-19 20:07:00 +0900
categories: [Program-Language, Java]
tags: [Java, Javstudy]
---


## 1. Introduction

도서 [모던 자바 인 액션](https://m.yes24.com/Goods/Detail/77125987) 챕터 3을 읽으면 자바의 람다식이 무엇인지 알 수 있다. **람다 표현식**은 메서드로 전달할 수 있는 익명 함수를 단순화한 것으로, 간략히 말하면 `(parameters) -> {statements}` 와 같은 형식을 취한다고 할 수 있다. 람다식은 **함수형 인터페이스(Functional Interface)**를 기대하는 문맥에 사용할 수 있고, 함수형 인터페이스란 정확히 하나의 추상 메서드(Single Abstract Method, SAM)를 가진 인터페이스로 정의되어 있다.

람다식은 좌변에 정의된 람다 파라미터(`(parameters)`)로 넘겨진 변수를 활용할 수도 있지만, 다음과 같이 외부에서 정의된 변수를 사용할 수 있다.
```java
void lambdaCapturing() { 
    int localVariable = 1000; 
    Runnable r = () -> {
        System.out.println(localVariable);
    }; 
}
```
`localVariable`은 람다식 외부에 정의된 변수인데 이러한 변수를 자유 변수라고 한다. 람다에서 자유 변수를 사용하는 예시를 더 알아보자.



## 2. Lambda Capturing

```java
import java.util.function.DoubleUnaryOperator;
import java.util.function.Function;

class StaticLambdaSupplier {
    public static Function<Integer, String> get() {
        return i -> "number: " + i;
    }
}

class LambdaSupplier {
    public static DoubleUnaryOperator get(double a, double b, double c) {
        return x -> a*x*x + b*x + c;
    }
}

public class LambdaCaptureEx {
    static void ex1() {
        var f = LambdaSupplier.get(4, 0, -1); //y1 = 4x^2 - 1
        var g = LambdaSupplier.get(2, -3, -2); //y2 = (x - 2)(2x + 1) = 2x^2 -3x -2
        System.out.println("y1(2) = " + f.applyAsDouble(2));
        System.out.println("y1(-0.5) = " + f.applyAsDouble(-0.5));
        System.out.println("y2(2) = " + g.applyAsDouble(2));
        System.out.println("y2(2) = " + g.applyAsDouble(0));
        System.out.println("y2(-0.5) = " + g.applyAsDouble(-0.5));

        System.out.println("f == g : " + (f == g));
    }

    static void ex2() {
        var f = StaticLambdaSupplier.get();
        var g = StaticLambdaSupplier.get();
        System.out.println(f.apply(5));
        System.out.println(g.apply(19));

        System.out.println("f == g : " + (f == g));
    }

    public static void main(String[] args) {
        ex1();
        ex2();
    }
}
```

### 2.1. 2차 다항함수

위 코드에서 `LambdaSupplier`의 스태틱 메서드를 보면 람다식 `x -> a*x*x + b*x + c`을 그대로 리턴하고 있음을 알 수 있다. 그리고 `a`, `b`, `c`는 람다식 외부에 정의된 자유 변수이다. 이것은 수학의 2차식을 반환하는 것으로, 메서드 인자로 계수 $a, b, c$를 넘겨주면 그에 해당하는 2차 타항함수 $f(x) = ax^2 + bx + c$ 를 반환하는 것으로 이해할 수 있다. 이것의 활용은 `ex1()`에 나와있다.

```java
static void ex1() {
    var f = LambdaSupplier.get(4, 0, -1); //y1 = 4x^2 - 1
    var g = LambdaSupplier.get(2, -3, -2); //y2 = (x - 2)(2x + 1) = 2x^2 -3x -2
    System.out.println("y1(2) = " + f.applyAsDouble(2));
    System.out.println("y1(-0.5) = " + f.applyAsDouble(-0.5));
    System.out.println("y2(2) = " + g.applyAsDouble(2));
    System.out.println("y2(2) = " + g.applyAsDouble(0));
    System.out.println("y2(-0.5) = " + g.applyAsDouble(-0.5));

    System.out.println("f == g : " + (f == g));
}
```
`var f`는 $f(x) = 4x^2 - 1$을 담고 있고, `var g`는 $g(x) = (x - 2)(2x + 1) = 2x^2 - 3x - 2$를 담고 있다고 이해할 수 있다. 두 람다함수는 서로 다른 매개변수로 생성되었으므로, `f == g`는 `false`가 된다.

### 2.2. 자유 변수가 없는 평범한 람다
```java
class StaticLambdaSupplier {
    public static Function<Integer, String> get() {
        return i -> "number: " + i;
    }
}

static void ex2() {
    var f = StaticLambdaSupplier.get();
    var g = StaticLambdaSupplier.get();
    System.out.println(f.apply(5));
    System.out.println(g.apply(19));

    System.out.println("f == g : " + (f == g));
}
```
`StaticLambdaSupplier`를 보면 자유 변수를 참조하지 않은 평범한 람다식을 반환하고 있다. `StaticLambdaSupplier.get()`으로 얻는 람다식은 항상 동일하므로, `f == g`의 결과는 `true`가 된다.



## 3. 자유 변수의 제약 조건

자유 변수가 항상 규제로부터 자유로운 것은 아니다. 몇 가지 경우에서 특수한 제약사항이 존재한다. 간단히 알아보자. 우선 자유 변수는 람다식 외부에 선언된 변수이므로, 그 변수가 선언된 범위에 따라 유형이 다르다. 크게 3가지 유형이 있다.

1. static 자유 변수
2. instance 자유 변수
3. local 자유 변수

이 중 static 자유 변수와 instance 자유 변수는 큰 제약이 없으나, `local 자유 변수`에 대해서는 제약사항이 있다. 람다식 외부의 local 변수를 캡처하는 것을 **Local Lambda Capturing**이라 하며 어떤 제약사항이 있는지 알아보자.

### 3.1. Local 람다 캡처링의 제약사항

local 자유 변수에 대해서는 아래의 제약 사항이 있다.
1. 자유 변수는 final이거나 effectively final(사실상 final)이어야 한다.
2. 람다식 내에서 local 자유 변수의 값을 변경할 수 없다.

따라서 아래와 같은 코드는 불가능하다.
```java
void func() {
    int local = 10;
    Runnable r = () -> {
        local = 15; // local은 final로 취급됨
    };
}

void func2() {
    int local = 10;
    Runnable r = () -> {
        System.out.println(local);
    };
    local = 15; // local은 effectively final이므로 불가능
}

```
이러한 제약 사항이 있는 이유는 무엇일까? 사실 책이나 인터넷을 살펴보면 이유를 쉽게 알 수 있으므로 여기서는 간략히 설명하겠다. 

위 코드에서 람다식 `() -> { local = 15; }`이 `Runnable r`에 대입되므로, 람다식은 인스턴스 변수라고 할 수 있다. 반면 `local`은 지역 변수다. VM 관점에서 인스턴스 변수와 지역 변수는 아주 큰 차이가 있는데, 인스턴스 변수는 `힙` 영역에 저장되어 있어 서로 다른 스레드끼리 공유할 수 있는 반면 지역 변수는 `스택` 영역에 저장되어 있다. 스택 영역은 스레드마다 별도로 생성되므로, 스택 영역의 메모리는 다른 스레드와 공유할 수 없다. 즉 지역 변수는 스레드끼리 공유되지 못한다. 

지역 변수의 수명은 그 변수가 선언된 지역(보통 이는 중괄호`{}`로 표현된다)의 실행이 끝나는 순간 끝난다. 하지만 람다식, 즉 **인스턴스 변수**는 그렇지 않기 때문에 지역 변수가 사라져도 메모리에 존재할 수 있다. 자바에서 사라진 메모리 공간에 접근하는 것은 허용하지 않는다. 따라서 자바 언어 설계자는 애초에 람다식에서 참조하는 지역 변수를 final로 취급하게 함으로써 이런 복잡한 문제를 피한 것이다.



## 4. 람다 캡처링의 과정

지역 변수 캡처링의 제약 사항을 살펴보았다. 그런데 람다 캡처링은 어떻게 수행되는 것일까? 원리를 알아보자.

로컬 지역 변수을 캡처링하는 것이 녹록치 않음에도 불구하고 그래도 가능한 이유는, 람다 내부에서 사용되는 지역 변수는 **원본 지역 변수를 복사한 것**이기 때문이다. 따라서 원본이 되는 지역 변수의 할당이 해제되어도 람다 내부에서 사용되는 데이터는 그대로 유지될 수 있는 것이다.

이에 대해 조금 더 자세히 알아보자. [stackoverflow의 한 질문 답변](https://stackoverflow.com/questions/4732544/why-are-only-final-variables-accessible-in-anonymous-class)에서는 다음과 같이 말하고 있다.
> As noted in comments, some of this becomes irrelevant in Java 8, where final can be implicit. Only an effectively final variable can be used in an anonymous inner class or lambda expression though.
> 
> It's basically due to the way Java manages closures.
> 
> When you create an instance of an anonymous inner class, any variables which are used within that class have their values copied in via the autogenerated constructor. This avoids the compiler having to autogenerate various extra types to hold the logical state of the "local variables", as for example the C# compiler does... (When C# captures a variable in an anonymous function, it really captures the variable - the closure can update the variable in a way which is seen by the main body of the method, and vice versa.)
> 
> As the value has been copied into the instance of the anonymous inner class, it would look odd if the variable could be modified by the rest of the method - you could have code which appeared to be working with an out-of-date variable (because that's effectively what would be happening... you'd be working with a copy taken at a different time). Likewise if you could make changes within the anonymous inner class, developers might expect those changes to be visible within the body of the enclosing method.
> 
> Making the variable final removes all these possibilities - as the value can't be changed at all, you don't need to worry about whether such changes will be visible. The only ways to allow the method and the anonymous inner class see each other's changes is to use a mutable type of some description. This could be the enclosing class itself, an array, a mutable wrapper type... anything like that. Basically it's a bit like communicating between one method and another: changes made to the parameters of one method aren't seen by its caller, but changes made to the objects referred to by the parameters are seen.
> 
> If you're interested in a more detailed comparison between Java and C# closures, I have an article which goes into it further. I wanted to focus on the Java side in this answer :)

요약하자면 다음과 같다. 
익명 내부 클래스의 인스턴스를 생성하게 되면, 해당 클래스 내에 사용된 모든 변수의 값은 자동으로 생성된 생성자를 통해 복사된다. 즉 람다식에서 로컬 외부 변수를 캡처링하게 되면, 람다식이 익명 클래스의 인스턴스로 변환되는 과정에서 **외부 변수의 값이 자동 생성된 생성자를 통해 람다식 내부로 복사된다**고 이해할 수 있다. 이러한 동작이 바로 람다 캡처링의 원리인 것이다.

다시 말하면, 람다식 외부에 선언된 변수 `localVariable`이 있다 가정하면, 람다식 내부의 `localVariable` 변수는 이름이 같을 뿐, 서로 완전히 다른 존재다. 외부의 변수는 람다식 입장에서 **자유 변수**이고, 람다식 내부의 변수는 외부 변수를 의미하는 것이 아니라 생성자에서 복사된 완전히 별개의 변수라는 것이다. 그러므로 만약 람다식 내부에서 로컬 지역 변수를 수정할 수 있었다면 이상하게 보인다는 것이다.

요약하면, 람다에서 사용하는 변수는 capture라는 과정을 통해 생성되며, 읽기 전용으로 복사해서 사용하기 때문에 동시성 문제가 생기지 않도록 final을 요구한다고 할 수 있다.



## 5. 이 모든 것은 사실 로컬 익명 클래스 인스턴스의 규칙이었다

람다와 캡처링에 관련된 모든 규칙은 사실 로컬 내부 클래스 인스턴스의 규칙과 정확히 동일하다. 로컬 익명 클래스 인스턴스를 생성할 때도 바깥 지역 변수를 참조할 때 지금까지 본 게시물에서 람다에 대해 논의된 모든 규칙이 적용된다. 그러면 람다 = 로컬 익명 클래스 인스턴스인 것일까?

### 5.1. 하지만 람다는 로컬 익명 클래스 인스턴스가 아니다

람다와 로컬 익명 클래스 인스턴스는 놀랍도록 동일하지만, 이 둘은 조금이지만 분명 다르다. JM 관점에서 보면, 로컬 익명 클래스 인스턴스가 생성되는 곳의 바이트코드를 살펴 보면 새 클래스가 생성된다. 하지만 람다가 생성되는 곳의 바이트코드를 살펴보면 새 클래스가 생성되는 대신, byte코드의 invokedynamic 이라는 instruction으로 함수가 참조되는 것을 확인할 수 있다. 즉 람다식을 사용하는 것이 로컬 익명 클래스 인스턴스를 사용하는 것보다 조금 더 효율적이라고 할 수 있다. invokedynamic에 대한 자세한 내용은 본 포스팅의 주제를 넘어서는 것 같으니, [관련된 링크](https://blog.hexabrain.net/400)로 이를 갈음한다.



## 6. 마치며

이렇게 람다와 캡처링에 대한 심도 있는 주제를 다루어 보았다. 그러면 마지막으로 다음과 같은 코드는 어떻게 실행될 지 추측해보자.

```java
import java.util.function.IntConsumer;
import java.util.function.IntSupplier;

class ModifyInstanceValue {
    int k;

    ModifyInstanceValue(int k) { this.k = k; }

    public IntSupplier getSupplyLambda() {
        return () -> k++;
    }
    public IntConsumer getConsumeLambda() {
        return var -> k = var;
    }
}

public class LambdaCaptureEx2 {

    static IntSupplier getSupplier() {
        ModifyInstanceValue inner = new ModifyInstanceValue(10);
        return inner.getSupplyLambda(); //inner 인스턴스가 사라진다!
    }
    static IntConsumer getConsumer() {
        ModifyInstanceValue inner = new ModifyInstanceValue(100);
        return inner.getConsumeLambda(); //inner 인스턴스가 사라진다!
    }

    public static void main(String[] args) {
        var getK = getSupplier();
        var setK = getConsumer();

        for (int i = 0; i < 10; i++)
            System.out.println(getK.getAsInt());

        System.out.println("consume a value");
        setK.accept(300);
        System.out.println(getK.getAsInt());
    }
}
```
인스턴스 변수 `k`를 참조하는 람다식을 반환한다. 그런데 그 `k`를 가지는 인스턴스(`inner`)는 사라진다. 람다식을 호출하면 어떻게 될까?



## 참고 자료
- [[Java] 람다의 변수 범위](https://newwisdom.tistory.com/29)
- [[Java] 람다: 람다 캡처링](https://bugoverdose.github.io/development/lambda-capturing-and-free-variable/)
- [Lambda Capturing](https://velog.io/@dev_osj/Lamda-Capturing)
- [[자바] Lambda에서 사용 가능한 변수(Stack과 Heap)](https://darkstart.tistory.com/199)
- [Why are only final variables accessible in anonymous class?](https://stackoverflow.com/questions/4732544/why-are-only-final-variables-accessible-in-anonymous-class)
- [Java8 Lambdas vs Anonymous classes](https://stackoverflow.com/questions/22637900/java8-lambdas-vs-anonymous-classes)
- [invokedynamic의 내부 동작](https://blog.hexabrain.net/400)