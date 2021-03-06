# Learning FP

[Ref: Functinal Programming in Scala](http://www.amazon.com/Functional-Programming-Scala-Paul-Chiusano/dp/1617290653/ref=sr_1_3?ie=UTF8&qid=1432526049&sr=8-3&keywords=scala)

## Chapter 1

> 함수형 프로그래밍은 프로그램을 작성하는 **방식**에 대한 제약이지,  
표현 가능한 프로그램의 **종류** 에 대한 제약이 아니다.

참조 투명성을 지키면, 치환모형이 사용 가능해지며 이로 인해 컴퓨터가 수행하는 계산뿐만 아니라 
개발자의 머릿속에서 진행되는 추론도 간단해진다. 

모듈적인 프로그램은 독립적으로 이해하고, 재사용할 수 있는 컴포넌트로 이루어진다. 
프로그램 전체의 의미는 오직 컴포넌트의 의미와 합성에 관한 규칙들로 정의 된다. 
이는 함수형 프로그래밍의 개념과 동일하다. **순수함수는 재사용 가능하고, 독립적인, 그리고 합성 가능한 컴포넌트다.** 

## Chapter 2

`compose` 같은 고차함수는 자신이 수백만줄의 코드로 이루어진 거대한 함수를 다루는지, 
아니면 간단한 한 줄 짜리 함수를 다루는지 신경쓰지 않는다. 이는 다형적 고차함수가 공통적인 패턴을 추상화하여 제공하기 때문인
데 이로인해 **큰 규모의 프로그래밍도 작은 규모의 프로그래밍과 아주 비슷한 느낌으로 진행할 수 있다.**

## Chapter 3

함수적 자료구조란 오직 순수 함수만으로 조작되는 자료구조이다. 따라서 함수적 자료구조는 정의에 의해 **immutable** 이다.

대규모 프로그램에서는 방어적인 복사가 문제가 될 수 있다. 변이 자료가 일련의 느슨하게 결합된 구성 요소들을 거쳐간다면, 
각 구성요소는 자신의 복사본을 만들어야 한다. 다른 구성요소가 그 자료를 변경할 수도 있기 때문이다. 

반면 **immutable** 자료구조는 항상 안전하게 공유할 수 있으므로 복사본을 만들 필요가 없으며, 부수효과에 의존하는 
프로그래밍 방식보다 FP 가 더 효율적인 결과를 산출하는 경우가 많다.

## Chapter 4

`map2` 함수는, 인자가 2개인 어떤 함수라도 수정 없이 `Option` 에 대응하도록 만들 수 있다.

```scala
def map2[A, B, C](a: Option[A], b: Option[B])(f: (A, B) => C): Option[C] = {
  for {
    aValue <- a
    bValue <- b
  } yield f(aValue, bValue)
}

// sequence using traverse
def sequence[A](a: List[Option[A]]): Option[List[A]] = {
  traverse(a)(x => x)
}

// simply,
def traverse[A, B](a: List[A])(f: A => Option[B]): Option[List[B]] = {
  a.foldRight[Option[List[B]]](Some(Nil))((x, y) => map2(f(x), y)(_ :: _))
}
```

`map`, `lift`, `sequence`, `map2`, `map3` 같은 함수들이 있으면, 생략적 값을 다루기 위해 기존 함수를 수정해야 할일이 **전혀** 없어야 하는 것이 정상이다.


4장의 핵심은 실패와 예오를 보통의 값으로 표현할 수 있다는 점과 오류 처리 및 복구에 대한 공통의 패턴을 추상화 하는 함수를 작성할 수 있다는 점이다. 
`Option` 이 그러한 목적으로 많이 쓰이긴 하지만, 에러에 대한 정보를 주지 않는다. 

실패에 대한 원인을 얻기 위해, `Either` 를 사용할 수 있다.  

## Chapter 5

Non-strictness (혹은 laziness) 를 이용하면 컬렉션을 변환하고 탐색하는 `map`, `filter` 등을 하나의 패스로 융합해서 더 효율적인 계산을 해 낼 수 있다.
  
> **Strictness**
> 어떤 표현식의 평가가 무한히 실행되면, 또는 한정돈 값을 돌려주지 않고 오류를 던진다면, 그러한 표현식을 일컬어 **bottom** 으로 평가되는 표현식이라고 부른다. 
> 만약 **bottom** 으로 평가되는 식 모든 `x` 에 대해 `f(x)` 가 *bottom** 이면 `f` 는 **strict** 하다. 

<br/>

함수형 프로그래밍에서는 **계산의 서술** 과 **실행** 을 분리할 수 있다. 그러면, **필요한 것 보다 더 큰 표현식을 서술**하되, 그 **표현식의 일부만 평가** 할 수 있다.

- 일급 함수는 일부 계산을 자신의 본문에 담고 있으나, 그 계산은 인수가 전달되어야 실행된다.
- `Option` 은 오류가 발생했다는 사실만 담고 있을 뿐, 오류에 대해 무엇을 수행할 것인가는 그와 분리되어 있다.
- `Stream` 은 요소들의 순차열을 생성하되, 그 평가는 실제로 요소가 필요할 때 까지 미룰 수 있다.

다음은 `foldRight` 의 느긋한 버전이다.

```scala
def exists(p: A => Boolean): Boolean =
  foldRight(false)((h, t) => p(h) || t)
  
def foldRight[B](z: => B)(f: (A, => B) => B): B = this match {
  case Cons(h, t) => f(h(), t().foldRight(z)(f))
  case _ => z
}
```

이를 이용하면 `map, append, flatMap` 등을 만들 수 있다.

```scala
def map[B](f: A => B): Stream[B] =
  foldRight(empty[B])((a, b) => cons(f(a), b))

def filter(p: A => Boolean): Stream[A] =
  foldRight(empty[A])((a, b) => if (p(a)) cons(a, b) else b)

def append[B >: A](s: => Stream[B]): Stream[B] =
  foldRight(s)((h, t) => cons(h, t))

def flatMap[B](f: A => Stream[B]): Stream[B] =
  foldRight(empty[B])((h, t) => f(h) append t)
```

## Chapter 7

항상 대수적 추론에 기반하면, API 가 특정 법칙(law)을 따르는 하나의 대수로 서술될 수 있다는 사실을 깨달을 것이다. 이런식으로 코딩할 때에는 
구체적인 문제 영역을 완전히 잊어버리고, 형식들이 잘 맞아떨어지게 하는 데에만 집중할 수 있다. 이는 속임수가 아니라, 대수 방정식을 단순화할 때 하는 추론과 비슷한 
자연스러운 추론 방식이다. API 를 하나의 **대수(algebra)**, 즉 일단의 **법칙(law)** 또는 참이라고 가정하는 **속성(property)** 들을 가진 연산 집합으로 간주하고, 
그 대수에 정의된 게임 규칙에 따라 그냥 형식적으로 기호를 조작하면서 문제를 풀어 나갈 수 있다.

> 하나의 대수는 하나 이상의 집합과 그 집합들의 원소에 작용하는 함수들 그리고 일단의 **공리(axiom)** 들로 이루어진다. 공리는 
항상 참이라고 간주되는 명제이며, 공리로부터 **정리(theorem)** 를 유도할 수 있다.

<br/>

`map(y)(id) == y` 라고 할 때, `map(map(y)(g))(f) == map(y)(f compose g)` 라는 공짜 정리가 성립한다, 이를 **사상융합(map fusion)** 
이라고도 부르며, 일종의 최적화로 사용할 수 있다. 즉, 두 번째 사상을 계산하기 위해 개별적인 병렬 계산을 띄우는 대신, 그것을 첫 번째 사상으로 접을 수 있다.

## Chapter 8

검례 최소화는 검사 실패 시 검사 프레임워크가 해당 검사를 실패하게 만드는 가장 작은, 또는 가장 간단한 검례를 찾아내는 방법이다. 
검례 최소화 방법에는 두 가지가 있다. 

- **수축**: 실패한 검례가 나왔다면 개별적인 절차를 띄워, 그 검례의 크기를 크기를 점점 줄여가면서 검사를 반복하되, 검사가 더이상 실패하지 않으면 멈춘다. 이러한 최소화 공정을 
수축이라 부르며, 이를 구현하려면 각 자료 형식마다 개별적인 코드르 작성해야 한다. (**ScalaCheck** 는 **수축**을 사용한다) 
- **크기별 생성**: 검사 실패 후에 검례들을 수축하는 대신 애초에 크기와 복잡도를 점차 늘려가면서 검례들을 생성한다.

<br/>

소프트웨어의 세계에는 겉으로 보기에 서로 다른 **문제점** 들이 대단히 많이지만 **그에 대한 함수적 해법들의 공간은 의외로 작다.** 

## Chapter 9

**파서 생성기** 는 주어진 문법 명세에 기초해서 파서의 구현코드를 생성해 준다. 이런 접근방식은 유용하고, 효율적이지만, 코드 생성에 따르는 
일반적인 문제점을 그대로 가지고 있다. 구체적으로 말하면, 디버깅하기 어려운 통일적인 코드 덩어리를 출력한다는 문제점이다. 또한 논리의 조각들을 재사용하기도 어렵다. 
파서에서 나타나는 공통적인 패턴을 추출하기 위해 새 조합기나 보조 함수를 도입할 수 없기 때문이다.

반면 파서 조합기에서는 파서가 보통 일급의 값이다. 파싱 논리를 재사용하기가 아주 간단하며, 프로그래밍 언어 이외의 외부 도구는 전혀 필요하지 않다.

<br/>

`slice` 는 구현에 일정한 제약을 가한다. 구체적으로 말하면, `p.many.map(_.size)` 라는 파서를 실행하면 임시 목록이 생성되지만, 
`p.many.slice.map(_.size)` 를 실행하면 임시 목록이 생성되지 않아야 한다. 이를 만족하려면 `slice` 가 파서 내부 구현에 접근해야 한다. 
이는 `slice` 가 하나의 기본 수단임을 강하게 암시한다.

<br/>

```scala
def product[A, B](p1: Parser[A], p2: => Parser[B]): Parser[(A, B)] 
```

`"0"`, `"1ㅁ"`, `"2aa"` 과 같은 **context-sensitive grammar** 는 `product` 로 표현할 수 없다. 둘째 파서의 선택이 첫 파서의 결과에 의존하기 때문이다. 
이런 문법을 파싱하려면 `flatMap` 이 필요하다.

```scala
def flatMap[A, B](p1 Parser[A])(f: A => Parser[B]): Parser[B]
```

<br/>

전형적인 라이브러리 설계 시나리오에서는 함수가 표현에 어떤 영향을 미칠지가 주된 관심사가 된다. 그러나 대수적 방식으로 접근할 때에는 
그와는 다른 사고방식이 필요하다. 이 경우에는 가능한 구현에 대해 **함수가 어떤 정보를 명시하는지** 를 기준으로 함수를 고민해야 한다. 
함수의 서명은 구현에 어떤 정보가 주어지는 지를 결정하고, 명시된 법칙들을 구현이 존중하는 한, 그러한 정보를 어떻게 사용하는지는 구현의 자유이다.

<br/>

```scala
val spaces = " ".many
val p1 = scope("magic spell") {
  "abra" ** spaces ** "cadabra""
}

val p2 = scope("gibberish") {
  "abba" ** spaces ** "babba""
}

val p = p1 or p2
```

`run(p)("abra cAdabra")` 에 대해서 `ParseError` 가 발생한다면, `"magic spell"` 파싱 오류를 돌려주는 것이 더 적합하다. 지금의 구현으로는 
`"abra"` 단어만 파싱에 성공한다면, `"magic spell"` 브랜치(**branch**) 의 파싱이 확정(**commited**) 된 상태가 된다. 

이러면 파싱 오류를 만났다 하더라도 다음 분기 `"gibberish"` 를 조사하지 않는다. 프로그래머가 분기 시점을 지정할수 있도록 하기 위해 `attempt` 를 만들어 보자. 
`p1 or p2` 가 있을 때 `p1` 을 실행하고, 만일 미확정(**uncommitted**) 상태에서 `p1` 이 실패했다면 같은 입력에 대해 `p2` 를 실행하고, 아닐 경우 실패하도록 만들자.

모든 파서가 **문자를 하나라도 파싱했다면, 해당 파싱을 확정하게** 변경하고 확정 지연을 위한 `attempt` 를 도입하자.

```scala
def attempt[A](p: Parser[A]): Parser[A]

// law
attempt(p flatMap(_ => fail)) or p2 == p2
```

이제 이 `attempt` 를 이용하면 파싱 확정을 지연할 수 있다.

```scala
(attempt("abra" ** spaces ** "abra") or "cadabra") or ("abra" ** spaces ** "cadabra!")
```

이제 둘째 `"abra"` 가 파싱되기 전까지는 `"cadabra"` 도 파싱의 대상으로 고려된다.

- 만약 첫 번째 `"abra"` 를 파싱하다가 실패한다면 `"cadabra"` 를 파싱하고 (uncommitted)
- 만약 두 번째 `"abra"` 까지 파싱 후 실패한다면 `"cadabra"` 를 파싱하지 않는다. (committed)

```scala
(("abra" ** spaces ** "abra") or "cadabra") or ("abra" ** spaces ** "cadabra!")
```

만약 위를 실행했다면 첫 번째 `"abra"` 가 실패했다면 바로 확정상태가 되어 `"cadabra"` 를 파싱하지 않았을 것이다. 
이제는 프로그래머가 committed 시점을 조절할 수 있다.

<br/>

## Chapter 10

**모노이드는 하나의 형식이되, 그 형식에 대해 결합법칙을 만족하며 항등원을 가진 이항연산이 존재하는 형식이다**.

- `op(op(x, y), z) == op(x, op(y, z))`
- `op(x, zero) = op(zero, x)`

<br/>

### Dual

`Option` 모노이드를 다음처럼 구현할 수 있다.

```scala
implicit def optionMonoid[A] = new Monoid[Option[A]] {
  override def op(x: Option[A], y: Option[A]): Option[A] = x orElse y
  override def zero: Option[A] = None
}
```

이 때, `op` 를 `y orElse x` 로 구현해도 모노이드 법칙은 성립한다. 이는 결합법칙 뿐만 아니라 **교환법칙** 또한 성립하기 때문이다. 
이는 `intAdditionMonoid` 나 `booleanOrMonoid` 의 경우에도 마찬가지다.

```scala
implicit val intAddition = new Monoid[Int] {
  override def op(a1: Int, a2: Int): Int = a1 + a2
  override def zero: Int = 0
}

implicit val booleanOr = new Monoid[Boolean] {
  override def op(a1: Boolean, a2: Boolean): Boolean = a1 || a2
  override def zero: Boolean = false
}
```

`op` 에서 교환법칙이 성립하는 한, 모든 모노이드는 **dual** 을 가진다. 
 
```scala
def dual[A](implicit m: Monoid[A]): Monoid[A] = new Monoid[A] {
  override def op(x: A, y: A): A = m.op(y, x)
  override def zero: A = m.zero
}
```

### foldMap using dual, endoMonoid

`foldMap` 을 `foldLeft` 를 이용해서 만들 수 있지만, 거꾸로 `foldLeft`, `foldRight` 를 `foldMap` 을 이용해서 만들 수 있다.

```scala
// 항등함수 모노이드
implicit def endoMonoid[A] = new Monoid[A => A] {
  override def op(f: A => A, g: A => A): A => A = f compose g
  override def zero: (A) => A = a => a
}

def foldMap[A, B](as: List[A])(m: Monoid[B])(f: A => B): B =
  as.foldLeft(m.zero)((b, a) => m.op(b, f(a)))
```

이 때 `foldMap` 을 이용하면 `foldRight` 를 다음처럼 구현할 수 있다.
 
```scala
def foldRight[A, B](as: List[A])(z: B)(f: (A, B) => B): B =
  foldMap(as)(endoMonoid[B])(f.curried)(z)
```

여기서 `f.curried` 의 타입은 `A => (B => B)` 이다.
 
```scala
scala> val f = (a: Int, b: Int) => a + b
f: (Int, Int) => Int = <function2>

scala> f.curried
res0: Int => (Int => Int) = <function1>
```

결국 `endoMonoid[B]` 를 이용해서 `foldLeft` 의 

- 기본값 `zero` 를 `B => B` 로 지정하고, 
- 연산함수 `f` 를 `(B => B) compose f(a) where f: A => (B => B)` 로 사용하는것이므로 

`B => B` 에 대해 `foldLeft` 를 실행한다고 볼 수 있다. 이때 생기는 각각의 `B => B` 는 `f compose g` 를 이용해서 조합되므로 
(`f(g(B))`) `foldRight` 의 올바른 구현이다. `foldMap(as)(endMonoid[B])(f.curried)` 에 `z` 를 먹이면 최종 결과물인 `B` 가 나온다.

```scala
"foldRight" in {
  // custom impl
  foldRight(List(1, 2, 3, 4))("")(_ + _) shouldBe "1234"

  // scala standard foldRight
  List(1, 2, 3, 4).foldRight("")(_ + _) shouldBe "1234"
}
```

<br/>

`foldLeft` 의 경우에는 모노이드의 `op` 순서가 뒤집혀야 하므로 `dual` 을 이용하면 된다.

```scala
def foldLeft[A, B](as: List[A])(m: Monoid[B])(f: (A, B) => B): B = 
  foldMap(as)(dual(endoMonoid[B]))(a => b => f(b, a))(z)
```

```scala
"foldLeft" in {
  // custom impl
  foldLeft(List(1, 2, 3, 4))(0)(_ - _) shouldBe -10
  foldRight(List(1, 2, 3, 4))(0)(_ - _) shouldBe -2

  // scala standard foldRight
  List(1, 2, 3, 4).foldLeft(0)((a, b) => a - b) shouldBe -10
  List(1, 2, 3, 4).foldRight(0)((a, b) => a - b) shouldBe -2
}
```

### Balanced Fold

모노이드 연산이 결합적이라는 사실은, `List` 같은 자료구조를 접을 때 어느 방향으로 접을 것인지 선택이 가능하다는 뜻이다. (e.g `foldLeft`, `foldRight`) 
만약 `fold` 를 수행할 때 하나씩이 아니라, 여러개씩 잘라 접을 수 있다면 병렬처리를 할 수 있다. 두 내부 `Monoid.op` 호출이 독립적이기 때문이다.

```scala
// fold
op(op(op(a, b), c), d)

// balanced fold
op(op(a, b), op(c, d))
```

### Parallelism

<br/>

모노이드는 결합법칙이 성립하기 때문에 데이터를 적절히 분할한다면 병렬성으로 성능상의 이득을 볼 수 있다. 예를 들어 
거대한 텍스트 파일에서 단어를 셀 경우, 병렬로 조사하려면 여러 조각들로 분할 한 뒤 결과를 결합하는 전략을 사용할 수 있다. 
이 때 지금 조사하는 조각의 위치(중간인지 끝인지)에 상관없이 중간 결과를 결합할 수 있어야 하므로, 결합법칙을 만족해야 한다. 
이를 **Monoid** 표현해 볼 수 있다.

```sclaa
```

> 연산에서 결합법칙이 성립한다면, 혹은 병렬 연산을 도입하는 것이 가능하다면 **Monoid** 가 아닌지 의심해 볼 것
 
 

