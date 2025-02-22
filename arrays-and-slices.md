# Низови и кришке

**[Сав код за ово поглавље можете пронаћи овде](https://github.com/marcetin/nauci-go-sa-testovima/tree/main/arrays)**

Када имате низ, врло је уобичајено да морате прећи преко њих. Па хајде
користите [наше ново пронађено знање о `for`](iteration.md) да направите функцију `Sum`. `Sum` ће узети низ бројева и вратити збир.

Користимо своје ТДД вештине

## Прво напишите тест

Направите нову фасциклу у којој ћете радити. Направите нову датотеку под називом `sum_test.go` и убаците следеће:

```go
package main

import "testing"

func TestSum(t *testing.T) {

	numbers := [5]int{1, 2, 3, 4, 5}

	got := Sum(numbers)
	want := 15

	if got != want {
		t.Errorf("got %d want %d given, %v", got, want, numbers)
	}
}
```

Низови имају _фиксни капацитет_ који дефинишете када декларишете променљиву.
Низ можемо иницијализовати на два начина:

* \[N\]type{value1, value2, ..., valueN} e.g. `numbers := [5]int{1, 2, 3, 4, 5}`
* \[...\]type{value1, value2, ..., valueN} e.g. `numbers := [...]int{1, 2, 3, 4, 5}`

Понекад је корисно и исписати улазе у функцију у поруци о грешци.
Овде користимо резервоар `%v` за испис" подразумеваног "формата, који добро функционише за низове.

[Прочитајте више о низовима формата](https://golang.org/pkg/fmt/)

## Покушајте да покренете тест

Покретањем `go test` компајлер неће успети са `./sum_test.go:10:15: undefined: Sum`

## Напиши минималну количину кода за покретање теста и провери неуспешне резултате теста

У `sum.go`

```go
package main

func Sum(numbers [5]int) int {
	return 0
}
```

Тест би сада требало да пропадне са _јасном поруком о грешци_

`sum_test.go:13: got 0 want 15 given, [1 2 3 4 5]`

## Напишите довољно кода да прође

```go
func Sum(numbers [5]int) int {
	sum := 0
	for i := 0; i < 5; i++ {
		sum += numbers[i]
	}
	return sum
}
```

Да бисте добили вредност из низа на одређеном индексу, само користите `array[index]` синтаксу. У овом случају користимо `for` да поновимо 5 пута да бисмо прошли кроз низ и додали сваку ставку у `sum`.

## Рефактор

Уведимо [`range`](https://gobyexample.com/range) да бисмо помогли у чишћењу нашег кода

```go
func Sum(numbers [5]int) int {
	sum := 0
	for _, number := range numbers {
		sum += number
	}
	return sum
}
```

`range` вам омогућава да прелазите низом. На свакој итерацији, `range` враћа две вредности - индекс и вредност.
Одлучили смо да игноришемо вредност индекса користећи `_` [празан идентификатор](https://golang.org/doc/effective_go.html#blank).

### Низови и њихов тип

Занимљиво својство низова је да је величина кодирана у свом типу. Ако покушате да проследи `[4]int` у функцију која очекује `[5]int`, неће се компајлирати.
Они су различитих типова, па је то исто као и покушај предавања `string` функција која жели `int`.

Можда мислите да је прилично незгодно што низови имају фиксну дужину, и то већину често их вероватно нећете користити!

Го има _slices_ који не кодирају величину колекције, већ уместо тога имају било коју величину.

Следећи услов биће збир збирки различитих величина.

## Прво напишите тест

Сада ћемо користити [slice type][slice] који нам омогућава да имамо колекције било које величине. Синтакса је врло слична низовима, само изостављате величину када проглашавајући их

`mySlice := []int{1,2,3}` rather than `myArray := [3]int{1,2,3}`

```go
func TestSum(t *testing.T) {

	t.Run("collection of 5 numbers", func(t *testing.T) {
		numbers := [5]int{1, 2, 3, 4, 5}

		got := Sum(numbers)
		want := 15

		if got != want {
			t.Errorf("got %d want %d given, %v", got, want, numbers)
		}
	})

	t.Run("collection of any size", func(t *testing.T) {
		numbers := []int{1, 2, 3}

		got := Sum(numbers)
		want := 6

		if got != want {
			t.Errorf("got %d want %d given, %v", got, want, numbers)
		}
	})

}
```

## Покушајте и покрените тест

Ово се не саставља

`./sum_test.go:22:13: cannot use numbers (type []int) as type [5]int in argument to Sum`

## Напиши минималну количину кода за покретање теста и провери неуспешне резултате теста

Проблем је што можемо и ми

* Разбијте постојећи АПИ тако што ћете аргумент променити у `Sum` да би био пререзан
  него низ. Када то учинимо, потенцијално ћемо упропастити
  нечији дан јер се наш _други_ тест више неће састављати!
* Направите нову функцију

У нашем случају, нико други не користи нашу функцију, па уместо да имамо две функције за одржавање, имајмо само једну.

```go
func Sum(numbers []int) int {
	sum := 0
	for _, number := range numbers {
		sum += number
	}
	return sum
}
```

Ако покушате да покренете тестове који се и даље неће компајлирати, мораћете да промените први тест да би прошао у пресеку, а не у низу.

## Напишите довољно кода да прође

Испоставило се да је овде било потребно само да поправимо проблеме са компајлером и да тестови прођу!

## Рефактор

Већ смо извршили факторизацију `Sum` - све што смо урадили је заменити низове кришкама, тако да нису потребне додатне промене.
Имајте на уму да не смемо занемарити наш тестни код у фази рефакторирања - можемо додатно побољшати своје `Sum` тестове.

```go
func TestSum(t *testing.T) {

	t.Run("collection of 5 numbers", func(t *testing.T) {
		numbers := []int{1, 2, 3, 4, 5}

		got := Sum(numbers)
		want := 15

		if got != want {
			t.Errorf("got %d want %d given, %v", got, want, numbers)
		}
	})

	t.Run("collection of any size", func(t *testing.T) {
		numbers := []int{1, 2, 3}

		got := Sum(numbers)
		want := 6

		if got != want {
			t.Errorf("got %d want %d given, %v", got, want, numbers)
		}
	})

}
```

Важно је испитати вредност ваших тестова. Није циљ да имате што више тестова, већ да имате што је могуће више _поверења_ у базу вашег кода. Ако имате превише тестова, то може да створи прави проблем и само додаје више трошкова у одржавању. ** Сваки тест има цену **.

У нашем случају можете видети да је имати два теста за ову функцију сувишно.
Ако ради за парче једне величине, врло је вероватно да ће успети и за парче било које величине \ (у оквиру разлога \).

Го-ов уграђени комплет алата за тестирање садржи [алат за покривање](https://blog.golang.org/cover).
Иако тежња ка 100% покривености не би требао бити ваш крајњи циљ, алат за покривање може вам помоћи идентификујте подручја вашег кода која нису обухваћена тестовима. Ако сте били строги према ТДД-у, сасвим је вероватно да ћете ионако имати близу 100% покривености.

Покушајте да покренете

`go test -cover`

Требало би да видите

```bash
PASS
coverage: 100.0% of statements
```

Сада избришите један од тестова и поново проверите покривеност.

Сад кад смо срећни што имамо добро тестирану функцију коју бисте требали да извршите
сјајан посао пре него што преузмете следећи изазов.

Потребна нам је нова функција звана `SumAll` која ће заузети различит број
кришке, враћајући нови пресек који садржи укупне вредности за сваки предати пресек.

На пример

`SumAll([]int{1,2}, []int{0,9})` би вратио `[]int{3, 9}`

или

`SumAll([]int{1,1,1})` би вратио `[]int{3}`

## Прво напишите тест

```go
func TestSumAll(t *testing.T) {

	got := SumAll([]int{1, 2}, []int{0, 9})
	want := []int{3, 9}

	if got != want {
		t.Errorf("got %v want %v", got, want)
	}
}
```

## Покушајте и покрените тест

`./sum_test.go:23:9: undefined: SumAll`

## Напиши минималну количину кода за покретање теста и провери неуспешне резултате теста

Морамо да дефинишемо `SumAll` према ономе што жели наш тест.

Го вас може пустити да напишете [_variadic functions_](https://gobyexample.com/variadic-functions) који могу да имају променљив број аргумената.

```go
func SumAll(numbersToSum ...[]int) (sums []int) {
	return
}
```

Ово је валидно, али наши тестови се и даље неће компајлирати!

`./sum_test.go:26:9: invalid operation: got != want (slice can only be compared to nil)`

Го вам не дозвољава да користите операторе једнакости са кришкама. Можете _писати_ функцију за итерацију по сваком `got` и `want` одсечку и провери њихове вредности али ради погодности можемо да користимо [`reflect.DeepEqual`][deepEqual]  што је корисно за утврђивање да ли су _било које_ две променљиве исте.

```go
func TestSumAll(t *testing.T) {

	got := SumAll([]int{1, 2}, []int{0, 9})
	want := []int{3, 9}

	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}
```

It's important to note that `reflect.DeepEqual` is not "type safe" - the code will compile even if you did something a bit silly. To see this in action, temporarily change the test to:
\ (обавезно `import reflect` у врху датотеке да би имао приступ` DeepEqual` \)

Важно је напоменути да `reflect.DeepEqual` није "сигуран за тип" - код саставиће се чак и ако сте учинили нешто помало глупо. Да бисте ово видели на делу, привремено промените тест у:

```go
func TestSumAll(t *testing.T) {

	got := SumAll([]int{1, 2}, []int{0, 9})
	want := "bob"

	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}
```

Оно што смо овде урадили је покушати да упоредимо `slice` са` string`. Ово прави
нема смисла, али тест се саставља! Дакле, док користите `reflect.DeepEqual` је
прикладан начин упоређивања кришки \ (и других ствари \) морате бити опрезни
приликом његовог коришћења.

Вратите тест поново и покрените га. Требали бисте имати тест резултате као што је приказано у наставку

`sum_test.go:30: got [] want [3 9]`

## Напишите довољно кода да прође

Оно што треба да урадимо је да прелиставамо варарге, израчунамо збир користећи наш
постојећу функцију `Sum`, а затим је додајте у пресек који ћемо вратити

```go
func SumAll(numbersToSum ...[]int) []int {
	lengthOfNumbers := len(numbersToSum)
	sums := make([]int, lengthOfNumbers)

	for i, numbers := range numbersToSum {
		sums[i] = Sum(numbers)
	}

	return sums
}
```

Много нових ствари које треба научити!

Постоји нови начин за стварање пресека. `make` вам омогућава да направите пресек помоћу
почетни капацитет `len`` numbersToSum` који морамо да решимо.

Можете да индексирате кришке као низове са `mySlice[N]` да бисте извадили вредност или
доделите му нову вредност са `=`

Тестови би сада требало да прођу.

## Рефактор

Као што је поменуто, кришке имају капацитет. Ако имате парче капацитета
2 и покушајте да урадите `mySlice [10] = 1`, добићете грешку _рунтиме_.

Међутим, можете користити функцију `append` која узима пресек и нову вредност,
затим враћа нови пресек са свим ставкама у њему.


```go
func SumAll(numbersToSum ...[]int) []int {
	var sums []int
	for _, numbers := range numbersToSum {
		sums = append(sums, Sum(numbers))
	}

	return sums
}
```

У овој имплементацији мање бринемо о капацитету. Почињемо са
празан пресек `sums` и додајте му резултат` Sum` док радимо кроз варарге.

Наш следећи услов је да променимо `SumAll` у` SumAllTails`, где ће и бити
израчунајте укупне вредности „репова“ сваке кришке. Реп колекције је
сви предмети у колекцији, осим првог \ ("глава" \).

## Прво напишите тест

```go
func TestSumAllTails(t *testing.T) {
	got := SumAllTails([]int{1, 2}, []int{0, 9})
	want := []int{2, 9}

	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}
```

218 / 5000
Резултати превода
## Покушајте и покрените тест

`./sum_test.go:26:9: undefined: SumAllTails`

## Напиши минималну количину кода за покретање теста и провери неуспешне резултате теста

Преименујте функцију у `SumAllTails` и поново покрените тест

`sum_test.go:30: got [3 9] want [2 9]`

## Напишите довољно кода да прође

```go
func SumAllTails(numbersToSum ...[]int) []int {
	var sums []int
	for _, numbers := range numbersToSum {
		tail := numbers[1:]
		sums = append(sums, Sum(tail))
	}

	return sums
}
```

Резине се могу исећи! Синтакса је `slice[low:high]`. Ако изоставите вредност на
једна од страна `:` снима све на тој страни. У нашем
случају, кажемо „узми од 1 до краја“ са `numbers[1:]`. Можда желите
проведите неко време пишући друге тестове око кришки и експериментишите са
резник оператора да бисте га боље упознали.

## Рефактор

Овог пута није много за реконструкцију.

Шта мислите да би се догодило када бисте празан део прешли у наш
функцију? Шта је „реп“ празне кришке? Шта се дешава када кажете Иди на
хватање свих елемената из `myEmptySlice[1:]`?

## Прво напишите тест

```go
func TestSumAllTails(t *testing.T) {

	t.Run("make the sums of some slices", func(t *testing.T) {
		got := SumAllTails([]int{1, 2}, []int{0, 9})
		want := []int{2, 9}

		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %v want %v", got, want)
		}
	})

	t.Run("safely sum empty slices", func(t *testing.T) {
		got := SumAllTails([]int{}, []int{3, 4, 5})
		want := []int{0, 9}

		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %v want %v", got, want)
		}
	})

}
```

## Покушајте и покрените тест
```text
panic: runtime error: slice bounds out of range [recovered]
    panic: runtime error: slice bounds out of range
```

О, не! Важно је напоменути да је тест _компајлиран_, то је грешка у извршавању.
Грешке у времену компајлирања су нам пријатељ јер нам помажу да напишемо софтвер који
ради, рунтиме грешке су нам непријатељи јер утичу на наше кориснике.

## Напишите довољно кода да прође

```go
func SumAllTails(numbersToSum ...[]int) []int {
	var sums []int
	for _, numbers := range numbersToSum {
		if len(numbers) == 0 {
			sums = append(sums, 0)
		} else {
			tail := numbers[1:]
			sums = append(sums, Sum(tail))
		}
	}

	return sums
}
```

## Рефактор

Наши тестови поново имају поновљени код око тврдњи, па извуцимо их у функцију

```go
func TestSumAllTails(t *testing.T) {

	checkSums := func(t testing.TB, got, want []int) {
		t.Helper()
		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %v want %v", got, want)
		}
	}

	t.Run("make the sums of tails of", func(t *testing.T) {
		got := SumAllTails([]int{1, 2}, []int{0, 9})
		want := []int{2, 9}
		checkSums(t, got, want)
	})

	t.Run("safely sum empty slices", func(t *testing.T) {
		got := SumAllTails([]int{}, []int{3, 4, 5})
		want := []int{0, 9}
		checkSums(t, got, want)
	})

}
```

Згодан споредни ефекат овога је што додаје мало сигурности у типичност нашем коду. Ако програмер грешком додаје нови тест са компајлером `checkSums(t, got, "dave")`зауставиће их на путу.

```bash
$ go test
./sum_test.go:52:21: cannot use "dave" (type string) as type []int in argument to checkSums
```

## Окончање

Покрили смо

* Низови
* Кришке
    * Разни начини да их направите
    * Како имају _фиксни_ капацитет, али можете створити нове резове од старих
      користећи `append`
    * Како резати, кришке!
* `len` да бисте добили дужину низа или пресека
* Алат за покривање теста
*  `reflect.DeepEqual` и зашто је то корисно, али може смањити сигурност типа вашег кода

Користили смо кришке и низове са целим бројевима, али они раде са било којим другим типом такође, укључујући саме низове / кришке. Дакле, можете прогласити променљиву од `[][]string` ако треба.

[Погледајте Го блог пост на кришкама] [блог-слице] за детаљнији увид кришке. Покушајте да напишете још тестова како бисте учврстили оно што сте научили читајући га.

Још један згодан начин експериментисања са Гоом, осим писања тестова, је Го игралиште. Можете испробати већину ствари и можете лако делити свој код ако треба да постављате питања. [Направио сам го игралиште са комадом у којем можете експериментисати.](https://play.golang.org/p/ICCWcRGIO68)

[Ево примера](https://play.golang.org/p/bTrRmYfNYCp) сечења низа и како промена пресека утиче на оригинални низ; већ „копија“ пресека неће утицати на оригинални низ.
[Још један пример](https://play.golang.org/p/Poth8JS28sc) зашто је то добра идеја да направите копију кришке након резања врло велике кришке.

[for]: ../iteration.md#
[blog-slice]: https://blog.golang.org/go-slices-usage-and-internals
[deepEqual]: https://golang.org/pkg/reflect/#DeepEqual
[slice]: https://golang.org/doc/effective_go.html#slices
