# [도사티비] 함수형 프로그래밍 맛보기

## 함수형 프로그래밍이란?

- 오래된 글이지만 참고: "Why Functional Programming Matters", John Hughes

- 사전적 의미(위키백과)

> 함수형 프로그래밍은 자료 처리를 **수학적 함수의 계산**으로 취급하고 **상태와 가변 데이터를 멀리하는** 프로그래밍 패러다임의 하나이다. 명령형 프로그래밍에서는 상태를 바꾸는 것을 강조하는 것과는 달리, 함수형 프로그래밍은 함수의 응용을 강조한다. 함수형 프로그래밍은 1930년대에 계산가능성, 결정문제, 함수정의, 함수응용과 재귀를 연구하기 위해 개발된 형식체계인 람다 대수에 근간을 두고 있다. 다수의 함수형 프로그래밍 언어들은 람다 연산을 발전시킨 것으로 볼 수 있다.

### 질문: 상태와 가변 데이터가 없는 프로그래밍이 정말 가능한가?

### 답: 그렇다!

- 충분히 복잡한 계산을 변수 없이 할 수 있다! 엑셀을 보라.
-- 엑셀에서 셀간의 의존성을 DAG로 표현할 수 있으며, 연산순서는 각 셀간의 의존관계에 의해 자동 계산된다.
-- 엑셀에서 한 셀의 값이 정해지고 나면, 사용자가 그 셀의 값을 변경하거나, 입력 역할을 하는 셀(계산 의존성 DAG에서 다른 셀에는 의존하지 않으면서, 다른 셀에 값을 제공하기만 하는 셀들)을 변경하지 않는 이상 결코 그 셀의 값이 바뀌지 않는다.


### 엑셀의 단점: 스프레드 시트 수준에서 추상화가 불가능하다.

- 저수준 추상화는 copy & paste시 엑셀이 자동으로 해준다고 볼 수도 있다.
-- 예: x(t+1) = x(t) - f(x(t))/f'(x(t))를 입력시 셀 위치를 가지고 잘 공식을 정의한 다음, 셀의 복사시 상대 주소 자동 조정 기능을 활용함)

- 이런 셀 복사시 상대 주소 자동 변경만으로 적절한 추상화가 어렵다.
- 만들어진 스프레드 시트에서 공통적인 추상 패턴을 알아보기가 힘들다. 즉, 코딩은 비주얼하게 편하게 가능하지만, 스프레드 시트를 보고 공통 패턴을 파악하는 것은 어렵다.

- 그럼에도 불구하고, 우리는 엑셀을 통해 불변 값 위주의 계산이 유용함을 알 수 있고, 불변 값 중심의 코딩이 결코 어렵지 않음도 알 수 있다(자바 프로그래머 숫자 <<< 엑셀 프로그래머 숫자)

## 그럼, 엑셀 코드를 프로그래밍 언어로 표현해 보자.

### 자바스크립트 콘솔 - 우리의 놀이터

- REPL(read-eval-print-loop, 레플이라고 읽음): 입력을 받아(read), 평가/계산하고(eval), 결과를 출력(print)하는 과정을 반복한다.
-- 자바스크립트 콘솔이 우리의 REPL이라 할 수 있다.

- 엑셀에서 12개월 만기 이율 5% 정기적금의 1년후 만기 금액을 계산하는 식을 변수 값을 한번 설정한 다음에는 바꾸지 않는 자바스크립트로 옮겨보자.

```
var i_year = 0.05;         // 5% 이율
var i_month = i_year / 12; // 월이율
var installment = 30000.0;   // 불입금
var pi_0 = installment + installment * (12-0) * i_month; // 첫 불입금의 최종 원리금
var pi_1 = installment + installment * (12-1) * i_month; // 2번째 불입금의 최종 원리금
var pi_2 = installment + installment * (12-2) * i_month; // 2번째 불입금의 최종 원리금
var pi_3 = installment + installment * (12-3) * i_month; // 2번째 불입금의 최종 원리금
var pi_4 = installment + installment * (12-4) * i_month; // 2번째 불입금의 최종 원리금
var pi_5 = installment + installment * (12-5) * i_month; // 2번째 불입금의 최종 원리금
var pi_6 = installment + installment * (12-6) * i_month; // 2번째 불입금의 최종 원리금
var pi_7 = installment + installment * (12-7) * i_month; // 2번째 불입금의 최종 원리금
var pi_8 = installment + installment * (12-8) * i_month; // 2번째 불입금의 최종 원리금
var pi_9 = installment + installment * (12-9) * i_month; // 2번째 불입금의 최종 원리금
var pi_10 = installment + installment * (12-10) * i_month; // 2번째 불입금의 최종 원리금
var pi_11 = installment + installment * (12-11) * i_month; // 12번째 불입금의 최종 원리금
var sum = pi_0 + pi_1 + pi_2 + pi_3 + pi_4 + pi_5 + pi_6 + pi_7 + pi_8 + pi_9 + pi_10 + pi_11; // 12번의 불입에 대한 원금+이자의 총 합
```

- 추상화(abstraction): 별것 아니고, 코드에서 공통 패턴을 하나로 묶고 부르기 편하게 이름을 붙이는 것임
- 반복되는 패턴이 무엇일까? `installment + installment * (12-<불입회차>) * i_month;`이 반복된다. 
- 이 패턴은  `installment`, `ith`, `i_month`에 따른 불입금과 최종이자의 합을 표현한다.
- 반복되는 부분은 불입금과 그 불입금에 대한 이자의 합계를 계산하는 식 부분이다.
- 매번 달라지는 부분은 `installment`, `ith`, `i_month`이다(사실은 12도 만기 개월수에 따라 바뀐다).
- 전체 식에 이름을 붙이되, 매번 달라지는 부분을 입력(함수의 입력을 다른 말로 매개변수라 한다)으로 만들고, 계산 결과를 돌려주게 하자.

```
// 코드 읽는 방법: 
//    getPIValue라는 이름은 함수(function)인데, 
//    인자로 installment, i_month, t_month, ith를 받아서
//    installment + installment * (t_month-ith) * i_month를 반환한다.
var getPIValue = function(installment, i_month, t_month, ith) {
  return installment + installment * (t_month-ith) * i_month;
}
```

- 여기서 `function(매개변수) { ... }` 부분을 익명함수(anonymous)라고 함.

> 내가 그 함수에 var로 이름을 붙이기 전에는 그는 다만 익명함수에 지나지 않았다 - 익명함수 by 익명작가

- 생각해보면 이 함수는 적금 계산 엑셀 워크시트의 테이블의 각 행을 getPTValue라는 함수로 바꾼 것이다. 사실 함수값을 식으로 계산하는 것과, 표로 만들어 놓은 입력값과 출력값 사이의 관계를 해석하는 데는 일맥상통하는 부분이 있다. 다만, 표는 유한하지만, 함수는 입력이 무한한 경우 출력도 무한할 수 있다.

- 여기서, 함수 내부는 외부에 대해 닫힌 세상이다.

1. 외부에서 주입되는 매개변수는 함수의 결과를 변화시키지만, 함수가 외부를 변화시키는 일은 없다.
2. 외부에서 함수의 결과를 바꾸는 유일한 방법은 매개변수에 다른 값을 전달하는 것 뿐이며, 매개변수가 같은 경우엔 함수의 결과도 항상 같다.
3. 1,2를 다른말로 하자면, 함수의 결과가 어떤 상태에 의존하지 않고 매개변수에만 의존하며, 함수의 실행으로 인해 반환값 외에 다른 어떤 상태가 바뀌는 일도 없다. 이를 *부수효과(side effect)가 없다*고 말한다.

-- 이런 경우, 매개변수가 같으면 함수의 결과도 같고, 함수가 함수 바깥에 영향을 끼치지 못하기 때문에, 같은 매개변수로 함수를 호출하는 경우가 여러번 있다면, 그 모든 호출을 결과값으로 바꿔치기 해도 안전하고, 함수를 호출해 계산한 것이나, 호출하는 대신 결과값으로 바꿔치기 해서 계산한 것이나 그 결과는 동일하다. 이를 *참조 투명성(referential transparency)*이라 한다.
-- 이렇게 부수효과가 없으며, 입력이 같으면 출력이 같은 함수를 순수함수(pure function)라고 부른다. 


#### 개선한 프로그램

```
var i_year = 0.05;         // 5% 이율
var i_month = i_year / 12; // 월이율
var installment = 30000.0;   // 불입금

// installment를 월이율 i_month로 불입하는 t_month 만기 적금에서, ith번째(ith는 0부터 t_month-1 까지) 불입금에서 얻을 수 있는 이자와 원금의 합을 구한다.
var getPIValue = function(installment, i_month, t_month, ith) {
  return installment + installment * (t_month-ith) * i_month;
}
var pi_0 = getPIValue(installment, i_month, 12, 0);
var pi_1 = getPIValue(installment, i_month, 12, 1);
var pi_2 = getPIValue(installment, i_month, 12, 2);
var pi_3 = getPIValue(installment, i_month, 12, 3);
var pi_4 = getPIValue(installment, i_month, 12, 4);
var pi_5 = getPIValue(installment, i_month, 12, 5);
var pi_6 = getPIValue(installment, i_month, 12, 6);
var pi_7 = getPIValue(installment, i_month, 12, 7);
var pi_9 = getPIValue(installment, i_month, 12, 8);
var pi_10 = getPIValue(installment, i_month, 12, 9);
var pi_11 = getPIValue(installment, i_month, 12, 10);

var sum = pi_0 + pi_1 + pi_2 + pi_3 + pi_4 + pi_5 + pi_6 + pi_7 + pi_8 + pi_9 + pi_10 + pi_11; // 12번의 불입에 대한 원금+이자의 총 합

```

- 행복한가? 조금.... 왜? 그래도 노가다. 머리가 나쁘면 몸이 고생
- 현재까지 우리가 아는 언어 요소(순수함수 + 불변 값 + 기본 산술연산) 만으로 해결이 불가능함

### 반복적인 작업을 프로그램이 하게 만들기

- 함수가 자기 자신을 호출할 수 있다면 반복이 가능하다!
- 어떻게 하면 함수안에서 자기 자신을 부르게 할 수 있을까?
- 결국, 함수를 정의하면서 내부에서 자신을 부를 이름을 지정하게 해야 한다.
- 예: n이 10보다 작은 동안 루프 돌기

```
// 여기서 loopUntil10은 만들어진 익명함수에 부여한 이름이고,
/  function 다음에 있는 loop는 함수 안에서 자기 자신을 부를 때 쓰기 위한 
// 이름이다.
var loopUntil10 = function loop(n) {
  if(n<10) {
    loop(n+1);
  }
}
```

- 이제, 이런 재귀를 사용해 적금 총액을 계산하자.

```
// 적금 합계를 구하는 재귀 함수
// sum_so_far에 합계를 누적해 나감
var getJukgumSum = function loop(i, sum_so_far) {
  if(i<12) {
    return loop(i+1, sum_so_far + getPIValue(installment, i_month, 12, i));
  } else {
    return sum_so_far;
  }
}

// 사용법
var sum = getJukgumSum(0, 0);
```

- 단점: 사용할 때 0, 0을 지정해야 함. 호출하면서 실수하기도 쉬울 뿐 아니라, 외부에 불필요한 인터페이스를 노출하는 것임.
- 함수 안에서 함수를 정의해 써먹고, 밖에는 최소의 인터페이스만 제공하자.

```
var getJukgumSum = function loop() {
  function loopInner(i, sum_so_far) {
    if(i<12) {
      return loop(i+1, sum_so_far + getPIValue(installment, i_month, 12, i));
    }
    else {
      return sum_so_far;
    }
  }
  
  loopInner(0,0);
}

// 사용법
var sum = getJukgumSum();
```

### 지금까지 중간 정리
- 값 중심 코딩 -> 추상화 -> 함수
- 함수 -> 추상화 -> 함수를 입력으로 받는 함수
#### 함수 프로그래밍이라도 특별히 다른 원칙이 적용되는 것이 아니다.
- 인터페이스를 깔끔하게 하기
- 공통 부분을 추상화하기
- 다만, 대입을 사용하지 않고 최대한 함수를 활용하기

#### 왜 절차지향을 지양하는가? 
- Alan Perlis: "A programming language is low level when its programs require attention to the irrelevant"
- 앨런 펠리스: 어떤 프로그래밍 언어가 프로그램을 하면서 중요하지 않은 요소에 관심을 기울이게 만든다면 저수준이라 할 수 있다.
- 절차는 사실은 문제 해결에 있어 핵심 관심 대상이 아님.
-- 그럼 재귀는 중요하지 않은 요소에 관심을 기울이게 만드는 것 아니란 말인가? -> 글쎄... ^^

## 다른 방식 - 스트림 활용

### 정기적금

- 이번엔 스칼라를 사용하자. 스칼라 REPL이나 워크시트를 실행한다.
- 정기적금 문제 매 단계를 살펴보면:
-- 각 행의 결과는 개월수와만 관계 있음(이율 등 다른 요소는 행이 바뀐다고 해도 그대로임)
-- 엑셀을 그대로 인코딩한다면:
```
[0부터 11까지 개월을 표현하는 어떤것] 
    ===(getPIValue를 모든 원소에 적용한 결과)===>
[(불입금 * 원금)의 배열] 
    ===(배열의 모든 원소의 합을 구함)===>
합계
```
라는 식으로 파이프라이닝이 가능함
- 일단, `getPIValue` 정의 및 다른 값의 정의를 REPL에서 하자.

```
val i_year = 0.05         // 5% 이율
val i_month = i_year / 12 // 월이율
val installment = 30000.0   // 불입금

def getPIValue(installment:Double, i_month:Double, t_month:Int, ith:Int):Double = installment + installment * (t_month-ith) * i_month
    
// getPIValue에서 항상 같은 installment, i_month, 12를 지정해서 만든 새로운 함수. 인자를 ith만 받기 때문에 편리함.
def getPIValue2(ith:Int):Double = getPIValue(installment, i_month, 12, ith) 
```

- 0부터 11까지 개월을 표현하는 어떤것
  - 원소나열법: `Seq(0,1,2,3,4,5,6,7,8,9,10,11)`
  - 범위 사용: `(0 to 11)`

- `===(getPIValue를 모든 원소에 적용한 결과)===>` : 어떤 컬렉션의 모든 원소에 같은 함수를 적용: `컬렉션 map (함수)`

```
   Seq(0,1,2,3,4,5,6,7,8,9,10,11) map (getPIValue2)
   (0 to 11) map (getPIValue2)
```

- 만들어진 결과의 합계를 구함

```
// 직접 만든 재귀 합계 함수
def calcSum(c:Seq[Double], sum_so_far:Double):Double = {
  if(c.isEmpty)
    sum_so_far
  else
    calcSum(c.tail, sum_so_far + c.head)
}

// 사용법
calcSum(Seq(0,1,2,3,4,5,6,7,8,9,10,11) map (getPIValue2), 0)
calcSum(((0 to 11) map (getPIValue2)).toList, 0)
```

- 한번 더 추상화

```
   ((0 to 11) map (getPIValue2)).reduce(_+_)
```


> "It is better to have 100 functions operate on one data structure than 10 functions on 10 data structures." - Alan Perils
> http://www.cs.yale.edu/homes/perlis-alan/quotes.html

### 뉴튼법

```
def f = (x:Double) => x*x - 2.0
def df = (x:Double) => 2*x

val x0 = 2
val x1 = x0 - f(x0)/df(x0)
val x2 = x1 - f(x1)/df(x1)
...
x_n = x_n-1 - f(x_n-1)/df(x_n-1)
```

- 재귀를 사용한 스트림 정의

```
lazy val x:Stream[Double] = 2.0 #:: x.map(
  (x) => x - f(x)/df(x)
)

x.zipWithIndex.dropWhile( 
  tuple => tuple._1 - x(tuple._2+1) >= 0.0001 || tuple._1 - x(tuple._2+1) <= -0.0001
)(0)._1
```
