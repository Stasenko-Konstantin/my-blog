+++
date = '2025-03-13'
title = 'Go, ADT, интерфейсы и щепотка парсер-комбинаторов'
+++

---

## Содержание:

- [Мотивация](#мотивация)
- [Что такое ADT?](#что-такое-adt)
- ["Перечисления" в Go](#перечисления-в-go)
- [Интерфейсы как перечисления](#интерфейсы-как-перечисления)
- [Функциональные паттерны Tuple и Either в Go](#функциональные-паттерны-tuple-и-either-в-go)
- [Тип Parser для создания парсер-комбинаторов](#тип-parser-для-создания-парсер-комбинаторов)

---

## Мотивация:

Пытаясь понять парсер-комбинаторы я захотел реализовать свою [небольшую библиотеку](https://github.com/Stasenko-Konstantin/p) чтобы разобраться с этим вопросом. Для этого я обратился к разным статьям объясняющим принцип работы парсер-комбинаторов, в частности к одной [статье от serokell](https://serokell.io/blog/parser-combinators-in-haskell) в которой довольно понятная реализация и её объяснение. Я решил частично переписать их пример на `Go` и столкнулся с тем что в `Go` система типов значительно слабее чем в `Haskell` (которым пользовались в статье), например в `Go` нет нативной поддержки `ADT` (алгебраических типов данных), по крайней мере сумм-типов. Что это такое и как можно приготовить я объясню ниже.

---

## Что такое ADT?

Вот как выглядят `ADT` в `Haskell`:
```haskell
data Sum = One | Two           -- перечисление
data Product a b = Product a b -- структура

s :: Sum -- возможные значения 's' определяются по формуле:
s = One  -- C1 + C2 + ... + CN | где 'C' - один из возможных 
         -- конструкторов типа 'Sum'. В данном случае это
		 -- One + Two = 2. Это и есть тип-сумма 

p :: Product Int Int -- возможные значения 'p' определяются по другой формуле:
p = Product 2 2      -- a1 * a2 * ... * aN | где 'a' - типовой параметр (дженерик)
	                 -- структуры 'Product'. В данном случае это
					 -- |Int| * |Int|, т.е. мощность множества возможных значений `Int`
					 -- в квадрате.
```

`ADT` позволяют думать о типах данных в более определенной форме как программисту, так и
машине. Но вернемся к комбинаторам. В статье даны следующие типы данных:
```haskell
data Error i e     -- Error[I, E any]
  = EndOfInput 
  | Unexpected i   -- Unexpected[T any]
  | CustomError e  -- CustomError[T any]
  | Empty  
  deriving (Eq, Show)

newtype Parser i e a = Parser -- Parser[I, E, A any]
  { runParser :: [i] -> Either [Error i e] (a, [i])
--  (p Parser[I, E, A]) RunParser(i []I) Either[[]Error, Tuple2[A, []I]]
  }
```
Я не буду вдаваться в особенности синтаксиса `Haskell`, а сразу перейду к `Go`.

---

## "Перечисления" в Go:

В `Go` нет "нормальных" перечеслений, вместо них используется `iota` следующим образом:

```go
import "fmt"

type Sum int

const (
	One Sum = 1 + iota
	Two    // iota автоматически генерирует значения для всех
	       // констант перечисленных ниже, в данном случае только
		   // для Two. Их значениями будут 1 и 2 соответственно 
)

func (s Sum) String() string {
	switch s {
	case One:
		return "One"
	case Two:
		return "Two"
	default:
		return ""
	}
}

func main() {
	s := One
	fmt.Println(One)      // "One"
	fmt.Println(int(One)) // 1
}
```

Для простых случаев это вполне подходит, но когда необходимо в перечислении хранить некотрые данные, то этого уже недостаточно и либо приходится менять архитектуру подстраиваясь под такие недо-перечисления, либо использовать другие подходы. В качестве примера для первого случая могу привести свой простенький [интерпретатор лиспа](https://github.com/Stasenko-Konstantin/lisp), а именно [токены](https://github.com/Stasenko-Konstantin/lisp/blob/main/src/tokens.go) и [объекты](https://github.com/Stasenko-Konstantin/lisp/blob/main/src/objects.go). Вот как выглядят токены:
```go
type TokenType int

// Type in Token struct
const (
	NUM_T TokenType = iota
	NAME_T
	STRING_T
	LPAREN_T
	RPAREN_T
)

type Token struct {
	Type    TokenType
	Content string
	x       int
	y       int
}
```

Если бы `Go` наследовал также и от того же `Haskell`, то можно было бы ждать перечислений a la `Rust`, что-то типа такого:
```go
type Span struct {
	X, Y int
}

type Token enum {
	Num(int, Span)
	Name(string, Span)
	Lparen(Span)
	Rparen(Span)
}
```
Но имеем что имеем. 

---

## Интерфейсы как перечисления:

Если же нам нужно что-то не просто содержащее данные, но также и расширяемое, например чтобы пользователь мог определять свои типы на основе нашего, то можно воспользоваться следующим приемом:

```go
type Error interface {
	IsImplement()
}

type EndOfInput struct{}

// Unexpected I - input stream
type Unexpected[T any] struct {
	I T
}

// CustomError E - error
type CustomError[T any] struct {
	E T
}

type Empty struct{}

func (e EndOfInput) IsImplement()     {}
func (e Unexpected[T]) IsImplement()  {}
func (e CustomError[T]) IsImplement() {}
func (e Empty) IsImplement()          {}
```
Код получился не такой лаконичный как в `Haskell`, но он обладает одним важным преимуществом: данная версия не запечатана (`sealed`), т.е. пользователь может определить свой тип ошибки (`Error`) не только с помощью `CustomError`, но и определив свою собственную структуру и реализовав пустой метод `IsImplement()` чтобы удовлетворить интерфейс `Error`, что позволит пользователю делать более тонкую настройку и обработку своих ошибок.

Хочу отметить что без добавления дженериков в `Go` данный код был бы обмазан пустыми
интерфейсами (`interface{}`), был бы менее проверяем на этапе компиляции и пришлось бы везде где используются ошибки приводить пустой интерфейс к нужному типу, что также может быть чревато ошибками. 

---

## Функциональные паттерны Tuple и Either в Go:

Это была тип-сумма на `Go`. Теперь осталось реализовать структуру `Parser` - тип-произведение. Но прежде чем сделать это необходимо подготовиться:
```go
type Tuple2[T1, T2 any] struct {
	E1 T1
	E2 T2
}

type Either[L, R any] struct {
	left  *L
	right *R
}

func Left[L, R any](left *L) Either[L, R] {
	return Either[L, R]{left: left}
}

func Right[L, R any](right *R) Either[L, R] {
	return Either[L, R]{right: right}
}

func (e Either[L, _]) Left() (left *L, ok bool) {
	if e.left != nil {
		return e.left, true
	}
	return nil, false
}

func (e Either[_, R]) Right() (right *R, ok bool) {
	if e.right != nil {
		return e.right, true
	}
	return nil, false
}

func (e Either[L, R]) String() string {
	var (
		res  any = 0
		side string
	)
	if l, ok := e.Left(); ok {
		res, side = *l, "Left"
	} else if r, ok := e.Right(); ok {
		res, side = *r, "Right"
	}
	return fmt.Sprintf("%s( %v )", side, res)
}
```
Данный код реализует два типичных паттерна из функциональных языков программирования (ФЯП): тип-произведение `Tuple2` - двухэлементный кортеж (на основе которого можно построить односвязный список - излюбленную структуру данных в ФЯП) и тип-произведение `Either` предназначенный для обработки ошибок. Но `Go` не функциональный язык, поэтому такой код в нем считается неидиоматичным и в подавляющем большинстве случаев стоит использовать стандартный тип `error`, а не костылять подобные вещи - здесь мы это сделали только в качестве эксперимента. 

`Either` работает следующим образом:
```go
import (
	"fmt"
	"errors"
	"github.com/Stasenko-Konstantin/p"
)

func mkEither(err error) p.Either[error, int] {
	if err != nil {
		return p.Left[error, int](&err)
	}
	res := 42
	return p.Right[error, int](&res)
}

func handle(e p.Either[error, int]) int {
	if _, ok := e.Left(); ok {
		return 0
	} else if res, ok := e.Right(); ok {
		return *res
	}
	return -1
}

func main() {
	e1 := mkEither(errors.New("error"))
	fmt.Println(handle(e1)) // 0
	fmt.Println(e1)         // Left( error )
	
	e2 := mkEither(nil)
	fmt.Println(handle(e2)) // 42
	fmt.Println(e2)         // Right( 42 )
}
```

---

## Тип Parser для создания парсер-комбинаторов:

Теперь можно перейти уже к самому парсеру:
```go
type Parser[I, E, A any] func([]I) Either[[]Error, Tuple2[A, []I]]

func (p Parser[I, E, A]) RunParser(i []I) Either[[]Error, Tuple2[A, []I]] {
	return p(i)
}
```
Из-за дженериков код может выглядеть несколько перегруженным, но если посмотреть вблизи на каждый отдельный логический (лексический?) элемент, то и глаза не будут разбегаться: сперва определяется тип `Parser` представляющий собой алиас для типа функции принимающей входной поток `[]I` и возвращающей либо поток ошибок `[]Error`, либо результат парсинга в виде двухэлементного кортежа вторым элементом содержащего в себе оставшийся нераспаршенным остаток входного потока. 

Метод `RunParser` объявленный на этом алиасе служит удобству применения (функции) парсера. Вот как это выглядит:
```go
import (
	"testing"

	"github.com/stretchr/testify/assert"
)

func satisfy[I, E any](f func(I) bool) p.Parser[I, E, I] {
	type FR = p.Either[[]p.Error, p.Tuple2[I, []I]]
	type L = []p.Error
	type R = p.Tuple2[I, []I]
	return func(i []I) FR {
		if len(i) == 0 {
			return p.Left[L, R](&L{p.EndOfInput{}})
		} else if f(i[0]) {
			return p.Right[L, R](&R{i[0], i[1:]})
		}
		return p.Left[L, R](&L{p.Unexpected[I]{i[0]}})
	}
}

func char[I comparable, E any](i I) p.Parser[I, E, I] {
	return satisfy[I, E](func(i2 I) bool {
		return i == i2
	})
}

func Test(t *testing.T) {
	e, ok := char[rune, string]('e').RunParser([]rune("eas")).Right()
	assert.True(t, ok)
	assert.Equal(t, "e", string(e.E1))  // результат парсинга
	assert.Equal(t, "as", string(e.E2)) // нераспаршенный остаток входного потока 'I'
}
```
И выглядит это несколько монструозно, но такова плата за использование ненативной и неидеоматичной функциональной парадигмы в `Go`. Для использования внутри каких-нибудь хитрых библиотек предоставляющих пользователям удобный и привычный интерфейс, а подобный код использующих под капотом думаю может подойти, но всё равно прежде чем использовать подобные паттерны нужно сперва хорошенько взвесить все за и против: стоят ли эти функциональные плюшки потери идиоматичности?

---

[наверх](#содержание)
