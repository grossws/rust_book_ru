% Стек и куча

Как любой системный язык программирования, Rust работает на низком уровне. Если
вы пришли из языка высокого уровня, то вам могут быть незнакомы некоторые
аспекты системного программирования. Наиболее важными из них являются те,
которые касаются работы с памятью в стеке и в куче. Если вы уже знакомы с тем,
как в C-подобных языках используется выделение памяти в стеке, то эта глава
освежит ваши знания. Если же вы еще не знакомы с этим, то в общих чертах узнаете
об этом понятии, но с акцентом на Rust.

# Управление памятью

Эти два термина касаются управления памятью. Стек и куча — это абстракции,
которые помогают вам определить, когда требуется выделение и освобождение
памяти.

Вот высокоуровневое сравнение.

Стек работает очень быстро; в Rust память
выделяется в стеке по умолчанию. Выделение памяти в стеке является локальным по
отношению к вызову функции, и имеет ограниченный размер. Куча, с другой стороны,
работает медленнее, а выделение памяти в куче осуществляется в программе
явно. Но такая память имеет теоретически неограниченный размер, и доступна
глобально.

# Стек

Давайте поговорим о следующей программе на Rust:

```rust
fn main() {
    let x = 42;
}
```

Эта программа имеет одно связанное имя, `x`. Память для него необходимо
где-то выделить. Rust по умолчанию «выделяет память в стеке», что означает, что
переменные «помещаются в стеке». Что это значит?

Когда функция вызывается, то выделяется некоторый объем памяти для всех её
локальных переменных и некоторой дополнительной информации. Это называется
«стековый кадр». В этом руководстве мы будем игнорировать эту дополнительную
информацию, и будем рассматривать лишь локальные переменные, которые мы
определяем. Таким образом, в этом случае, когда выполняется `main()`, мы
выделяем одно 32-битное целое число в нашем кадре стека. Как вы можете видеть,
это происходит автоматически — мы не должны писать какой-либо специальный код на
Rust для этого.

Когда функция завершается, её стековый кадр освобождается. Это происходит
автоматически — для этого нам не надо предпринимать никаких действий.

Вот и все, что касается этой простой программы. Главное, что здесь нужно
понять — это что выделение в стеке очень, очень быстро. Поскольку все локальные
переменные известны нам заранее, мы можем выделить память для них всех сразу. И
так как они, как правило, одновременно выходят из области видимости, мы можем
очень быстро освободить выделенную память.

Недостатком является то, что мы не можем хранить необходимые значения дольше,
чем в рамках одной функции.

А ещё мы не говорили о том, что же означает название «стек». Для этого мы должны
привести немного более сложный пример:

```rust
fn foo() {
    let y = 5;
    let z = 100;
}

fn main() {
    let x = 42;

    foo();
}
```

Эта программа имеет в общей сложности три переменные: две в `foo()` и одну в
`main()`. Так же как и раньше, когда вызывается `main()`, в её стековом кадре
выделяется одно целое число. Но, прежде чем мы сможем показать, что происходит,
когда вызывается `foo()`, мы должны визуализировать то, что происходит с
памятью. Ваша операционная система представляет отображение памяти для вашей
программы. Это довольно просто: огромный список адресов, от 0 до большого числа,
представляющего количество оперативной памяти у вашего компьютера. Например,
если у вас есть гигабайт оперативной памяти, то ваши адреса будут от `0` до
`1 073 741 824`. Это число равно 2<sup>30</sup>, количеству байтов в
гигабайте.

Эта память вроде гигантского массива: адреса начинаются с нуля и продолжаются до
конечного числа. Так вот схема нашего первого кадра стека:

| Адрес | Имя | Значение |
|-------|-----|----------|
| 0     | x   | 42       |

У нас есть переменная `x`, расположенная по адресу `0`, имеющая значение `42`.

Когда вызывается `foo()`, выделяется новый стековый кадр:

| Адрес | Имя | Значение |
|-------|-----|----------|
| 2     | z   | 100      |
| 1     | y   | 5        |
| 0     | x   | 42       |

Поскольку `0` было задействовано в первом кадре, для кадра `foo()` используются
`1` и `2`. При дальнейших вызовах функций стек будет расти вверх.

Здесь необходимо принять к сведению некоторые важные замечания. Адреса 0, 1 и 2
приведены исключительно в иллюстративных целях, и не имеют никакого отношения к
фактическим адресам, которые компьютер будет использовать. В частности, набор
адресов в действительности включает выравнивающие разделители, состоящие из
некоторого числа байтов, которые отделяют каждый из адресов. Размер этого
разделителя может даже превышать размер хранящегося значения.

После того, как `foo()` завершается, её кадр будет освобожден:

| Address | Name | Value |
|---------|------|-------|
| 0       | x    | 42    |

А потом, после `main()`, даже это последнее значение уходит. Легко!

Это называется «стек» (по-русски, стопка), потому что он работает как стопка
тарелок: первая тарелка, которую вы положили, будет последней тарелкой,
которую вы возьмете обратно. По этой причине стек иногда называют очередью
«последним пришел, первым вышел». Последнее значение, которое вы положили в
стек, будет первым, которое вы получите из него.

Давайте попробуем трёх-уровневый пример:

```rust
fn bar() {
    let i = 6;
}

fn foo() {
    let a = 5;
    let b = 100;
    let c = 1;

    bar();
}

fn main() {
    let x = 42;

    foo();
}
```

Сначала вызывается `main()`:

| Адрес | Имя | Значение |
|-------|-----|----------|
| 0     | x   | 42       |

Затем из `main()` вызывается `foo()`:

| Адрес | Имя | Значение |
|-------|-----|----------|
| 3     | c   | 1        |
| 2     | b   | 100      |
| 1     | a   | 5        |
| 0     | x   | 42       |

И затем из `foo()` вызывается `bar()`:

| Адрес | Имя | Значение |
|-------|-----|----------|
| 4     | i   | 6        |
| 3     | c   | 1        |
| 2     | b   | 100      |
| 1     | a   | 5        |
| 0     | x   | 42       |

Вот что мы имели ввиду раньше, говоря, что наш стек растет вверх.

После того, как `bar()` завершается, её кадр будет освобожден, оставляя только
`foo()` и `main()`:

| Адрес | Имя | Значение |
|-------|-----|----------|
| 3     | c   | 1        |
| 2     | b   | 100      |
| 1     | a   | 5        |
| 0     | x   | 42       |

А затем завершается `foo()`, оставляя только `main()`:

| Адрес | Имя | Значение |
|-------|-----|----------|
| 0     | x   | 42       |

И вот мы закончили. Уловили суть? Это как стопка тарелок: вы кладете наверх, и
берёте сверху.

# Куча

Такой способ выделения памяти работает очень хорошо, но он может быть
использован не всегда. Иногда вам необходимо передать некоторую память между
различными функциями или сохранить её валидность после окончания выполнения
функции. Для этого мы можем использовать кучу.

В Rust, вы можете выделить память в куче с помощью упаковки, т.е.
[типа `Box<T>`][box]. (Примечание переводчика: мы называем `Box<T>` упаковкой,
потому что `T` как бы «упакован» в `Box`: упаковка знает размер того, что лежит
внутри. Эта информация закодирована в типе `T`, поэтому во время исполнения, для
размерных типов, это просто указатель.) Вот пример:

```rust
fn main() {
    let x = Box::new(5);
    let y = 42;
}
```

[box]: http://doc.rust-lang.org/std/boxed/index.html

Вот что происходит с памятью, когда вызывается `main()`:

| Адрес | Имя | Значение  |
|-------|-----|-----------|
| 1     | y   | 42        |
| 0     | x   | ??????    |

Мы выделяем место для двух переменных в стеке. `y` представляет собой `42`,
тут всё как обычно. Но что насчёт `x`? Наш `x` представляет собой `Box<i32>`,
а упаковка выделяет память в куче. Фактическое значение упаковки — структура,
которая хранит указатель на «кучу». Когда начинает выполняться функция,
осуществляется вызов `Box::new()`, который выделяет некоторый объем памяти в
куче, и кладет туда `5`. Теперь память выглядит следующим образом:

| Адрес          | Имя | Значение       |
|----------------|-----|----------------|
| 2<sup>30</sup> |     | 5              |
| ...            | ... | ...            |
| 1              | y   | 42             |
| 0              | x   | 2<sup>30</sup> |

В нашем гипотетическом компьютере 1Гб оперативной памяти (2<sup>30</sup> байт).
А так как наш стек растет от нуля, то проще всего выделить память с другого
конца. Таким образом, наше первое значение находится на самом высоком месте в
памяти. А структура `x` хранит [сырой указатель][rawpointer] на адрес, который
мы выделили в куче, так что значение `x` равно 2<sup>30</sup> — это то самое
местоположение в памяти.

[rawpointer]: raw-pointers.html

Мы не слишком много говорили о том, что на самом деле означает «выделить» и
«освободить память» в этом контексте. Чрезмерное углубление в детали по этому
вопросу выходит за рамки данного руководства, но важно отметить, что куча — это
не просто стек, который растет с противоположного конца. Как мы увидим в
дальнейших примерах в этой книге, память из кучи может быть выделена и
освобождена в любом порядке, что в конечном итоге может привести к «дыркам». Вот
схема размещения памяти программы, проработавшей в течение некоторого времени:

| Адрес                | Имя  | Значение             |
|----------------------|------|----------------------|
| 2<sup>30</sup>       |      | 5                    |
| (2<sup>30</sup>) - 1 |      |                      |
| (2<sup>30</sup>) - 2 |      |                      |
| (2<sup>30</sup>) - 3 |      | 42                   |
| ...                  | ...  | ...                  |
| 3                    | y    | (2<sup>30</sup>) - 3 |
| 2                    | y    | 42                   |
| 1                    | y    | 42                   |
| 0                    | x    | 2<sup>30</sup>       |

В этом примере мы выделили четыре элемента в куче, но освободили лишь два из
них. Отсюда разрыв между 2<sup>30</sup> и (2<sup>30</sup>) - 3, который в
настоящее время не используется. Конкретные детали того, как и почему это
происходит, зависят от того, какую стратегию вы используете для управления
кучей. Различные программы могут использовать различные «распределители памяти»,
которые представляют собой библиотеки, которые управляют памятью за вас.
Программы на Rust используют для этого [jemalloc][jemalloc].

[jemalloc]: http://www.canonware.com/jemalloc/

Ладно, вернемся к нашему примеру. Так как эта память расположена в куче, то она
может оставаться валидной дольше, чем функция, которая выделяет упаковку. В
данном случае, однако, это не так.[^moving] Когда функция завершается, мы должны
освободить кадр стека для `main()`. Хотя у `Box<T>` для этого есть свой трюк:
[Drop][drop]. Реализация `Drop` для `Box` освобождает память, которая была
выделена при создании. Отлично! Поэтому, когда `x` уходит, сначала освобождается
память, выделенная в куче:

| Адрес | Имя | Значение  |
|-------|-----|-----------|
| 1     | y   | 42        |
| 0     | x   | ??????    |

[drop]: drop.html
[moving]: Мы можем продлить время жизни памяти путем передачи права
          собственности, что иногда называют «перемещение из упаковки» («moving
          out of the box»). Более сложные примеры будут рассмотрены позже.

А потом кадр стека уходит, освобождая всю нашу память.

# Аргументы и заимствование

У нас есть некоторые простые примеры со стеком и кучей, но что насчёт аргументов
функции и заимствования? Вот небольшая программа на Rust:

```rust
fn foo(i: &i32) {
    let z = 42;
}

fn main() {
    let x = 5;
    let y = &x;

    foo(y);
}
```

Когда мы входим в `main()`, память выглядит следующим образом:

| Адрес | Имя | Значение |
|-------|-----|----------|
| 1     | y   | 0        |
| 0     | x   | 5        |

Значением `x` является `5`, а `y` представляет собой ссылку на `x`. То есть, ее
значением является адрес памяти, по которому расположен `x`. В данном случае это
`0`.

А что насчёт случая, когда мы вызываем `foo()`, передавая `y` в качестве
аргумента?

| Адрес | Имя | Значение |
|-------|-----|----------|
| 3     | z   | 42       |
| 2     | i   | 0        |
| 1     | y   | 0        |
| 0     | x   | 5        |

Кадры стека используются не только для локальных имён, но также и для
аргументов. Таким образом, в этом случае, наш кадр должен содержать как `i`, наш
аргумент, так и `z`, наше локальное имя. `i` — это копия аргумента `y`.
Соответственно, значением `i`, как и значением `y`, является `0`.

Это одна из причин, почему заимствование переменной не освобождает какую-либо
память: значением ссылки является просто указатель на область памяти. Если мы
освободим находящуюся по этому указателю память, то это может привести к ошибкам
в дальнейшей работе.

# Сложный пример

Хорошо, давайте рассмотрим следующую, более сложную программу шаг за шагом:

```rust
fn foo(x: &i32) {
    let y = 10;
    let z = &y;

    baz(z);
    bar(x, z);
}

fn bar(a: &i32, b: &i32) {
    let c = 5;
    let d = Box::new(5);
    let e = &d;

    baz(e);
}

fn baz(f: &i32) {
    let g = 100;
}

fn main() {
    let h = 3;
    let i = Box::new(20);
    let j = &h;

    foo(j);
}
```

Сначала мы вызываем `main()`:

| Адрес           | Имя  | Значение       |
|-----------------|------|----------------|
| 2<sup>30</sup>  |      | 20             |
| ...             | ...  | ...            |
| 2               | j    | 0              |
| 1               | i    | 2<sup>30</sup> |
| 0               | h    | 3              |

Мы выделяем память для `j`, `i`, и `h`. `i` выделена в куче и поэтому содержит
указатель на значение в куче.

Далее, в конце вызова `main()`, вызывается `foo()`:

| Адрес           | Имя  | Значение       |
|-----------------|------|----------------|
| 2<sup>30</sup>  |      | 20             |
| ...             | ...  | ...            |
| 5               | z    | 4              |
| 4               | y    | 10             |
| 3               | x    | 0              |
| 2               | j    | 0              |
| 1               | i    | 2<sup>30</sup> |
| 0               | h    | 3              |

Пространство выделяется для `x`, `y` и `z`. Аргумент `x` имеет такое же
значение, как и `j`, так как мы передали `j` в качестве аргумента. Это указатель
на адрес `0`, так как `j` указывает на `h`.

Далее, `foo()` вызывает `baz()`, передавая `z`:

| Адрес           | Имя  | Значение       |
|-----------------|------|----------------|
| 2<sup>30</sup>  |      | 20             |
| ...             | ...  | ...            |
| 7               | g    | 100            |
| 6               | f    | 4              |
| 5               | z    | 4              |
| 4               | y    | 10             |
| 3               | x    | 0              |
| 2               | j    | 0              |
| 1               | i    | 2<sup>30</sup> |
| 0               | h    | 3              |

Мы выделили память для `f` и `g`. `baz()` очень короткая, и когда она
завершается, мы избавляемся от её кадра стека:

| Адрес           | Имя  | Значение       |
|-----------------|------|----------------|
| 2<sup>30</sup>  |      | 20             |
| ...             | ...  | ...            |
| 5               | z    | 4              |
| 4               | y    | 10             |
| 3               | x    | 0              |
| 2               | j    | 0              |
| 1               | i    | 2<sup>30</sup> |
| 0               | h    | 3              |

Далее `foo()` вызывает `bar()` с аргументами `x` и `z`:

| Адрес                | Имя  | Значение             |
|----------------------|------|----------------------|
|  2<sup>30</sup>      |      | 20                   |
| (2<sup>30</sup>) - 1 |      | 5                    |
| ...                  | ...  | ...                  |
| 10                   | e    | 9                    |
| 9                    | d    | (2<sup>30</sup>) - 1 |
| 8                    | c    | 5                    |
| 7                    | b    | 4                    |
| 6                    | a    | 0                    |
| 5                    | z    | 4                    |
| 4                    | y    | 10                   |
| 3                    | x    | 0                    |
| 2                    | j    | 0                    |
| 1                    | i    | 2<sup>30</sup>       |
| 0                    | h    | 3                    |

Тут мы выделяем другое значение в куче, и поэтому мы вычитаем единицу из
2<sup>30</sup>. Это выражение написать легче, чем `1,073,741,823`. В любом
случае, переменные создаются, как обычно.

В конце `bar()` вызывает `baz()`:

| Адрес                | Имя  | Значение             |
|----------------------|------|----------------------|
|  2<sup>30</sup>      |      | 20                   |
| (2<sup>30</sup>) - 1 |      | 5                    |
| ...                  | ...  | ...                  |
| 12                   | g    | 100                  |
| 11                   | f    | 4                    |
| 10                   | e    | 9                    |
| 9                    | d    | (2<sup>30</sup>) - 1 |
| 8                    | c    | 5                    |
| 7                    | b    | 4                    |
| 6                    | a    | 0                    |
| 5                    | z    | 4                    |
| 4                    | y    | 10                   |
| 3                    | x    | 0                    |
| 2                    | j    | 0                    |
| 1                    | i    | 2<sup>30</sup>       |
| 0                    | h    | 3                    |

Сейчас мы на наибольшей глубине! Поздравляем с достижением данной точки.

После завершения `baz()`, мы избавляемся от `f` и `g`:

| Адрес                | Имя  | Значение             |
|----------------------|------|----------------------|
|  2<sup>30</sup>      |      | 20                   |
| (2<sup>30</sup>) - 1 |      | 5                    |
| ...                  | ...  | ...                  |
| 10                   | e    | 9                    |
| 9                    | d    | (2<sup>30</sup>) - 1 |
| 8                    | c    | 5                    |
| 7                    | b    | 4                    |
| 6                    | a    | 0                    |
| 5                    | z    | 4                    |
| 4                    | y    | 10                   |
| 3                    | x    | 0                    |
| 2                    | j    | 0                    |
| 1                    | i    | 2<sup>30</sup>       |
| 0                    | h    | 3                    |

Далее мы выполняем возврат из `bar()`. В этом случае `d` представляет собой
`Box<T>`, поэтому он также освобождает и то, на что он указывает:
(2<sup>30</sup>) - 1.

| Адрес           | Имя  | Значение       |
|-----------------|------|----------------|
|  2<sup>30</sup> |      | 20             |
| ...             | ...  | ...            |
| 5               | z    | 4              |
| 4               | y    | 10             |
| 3               | x    | 0              |
| 2               | j    | 0              |
| 1               | i    | 2<sup>30</sup> |
| 0               | h    | 3              |

И после этого происходит возврат из `foo()`:

| Адрес           | Имя  | Значение       |
|-----------------|------|----------------|
|  2<sup>30</sup> |      | 20             |
| ...             | ...  | ...            |
| 2               | j    | 0              |
| 1               | i    | 2<sup>30</sup> |
| 0               | h    | 3              |

И вот, наконец, `main()`, которая очищает все остальное. Когда освобождается `i`
(`Drop`), будет также очищен и конец кучи.

# А что делают другие языки?

Большинство языков со сборщиком мусора по умолчанию выделяет память из кучи. Это
означает, что каждое значение будет упаковано. Есть ряд причин, почему делается
именно так, но они выходят за рамки данного руководства. Есть несколько
возможных оптимизаций, которые, правда, не достигают своей цели во всех случаях.
Вместо того чтобы полагаться на стек и `Drop` в вопросах очистки памяти, сборщик
мусора работает с кучей.

# Что использовать?

Но, если стек быстрее и проще в управлении, зачем тогда нужна куча? Весомая
причина заключается в том, что память в стеке может выделяться только по
принципу «первым пришёл — последним вышел». Таким образом, место из-под кадра
стека предыдущего вызова функции будет переиспользовано под следующий вызов.
Выделение в куче — более общая техника. Она позволяет выделение и освобождение
памяти в любом порядке. Однако, это достигается ценой увеличения сложности
реализации механизма выделения памяти.

В общем случае, следует предпочитать выделение в стеке, и поэтому, Rust
использует выделение в стеке по умолчанию. LIFO модель стека («последним
пришёл — первым вышел») фундаментально проще. Это значит, что программа быстрее
исполняется, и проще по смыслу.

## Эффективность во время выполнения

Управление памятью для стека тривиально: машина просто увеличивает или уменьшает
одно значение, так называемый «указатель стека». Управление памятью для кучи
сложнее: память, выделенная в куче, освобождается в произвольные моменты, а
каждая область выделенной в куче памяти может быть произвольного размера.
Распределителю памяти, как правило, требуется приложить гораздо больше усилий
для определения областей, которые можно использовать заново.

Если вы хотите изучить эту тему более подробно, то [эта статья][wilson] будет
отличным введением.

[wilson]: http://www.cs.northwestern.edu/~pdinda/icsclass/doc/dsa.pdf

## Простота программы

Выделение памяти в стеке воздействует как на сам язык Rust, так и на модель
мышления разработчиков. Стековая семантика — ключевое понятие Rust. Мы получаем
автоматическое управление памятью без усложнения среды исполнения. Именно этот
механизм позволяет освободить память в куче, как только её владелец вышел из
области видимости — по сути, как только схлопнулся стек кадра, на котором он
жил. К сожалению, в некоторых ситуациях стека недостаточно. Если нужна большая
гибкость во владении памятью, можно воспользоваться стётчиками ссылок `Rc<T>` и
`Arc<T>`.

Желание более удобно пользоваться памятью в куче может доходить до крайности. С
одной стороны, можно реализовать сборщик мусора — но это сильно увеличивает
сложность среды исполнения. С другой стороны, полностью ручное управление
памятью с явным вызовом процедуры освобождения часто приводит к ошибкам,
предотвратить которые компилятор Rust не в силах.
