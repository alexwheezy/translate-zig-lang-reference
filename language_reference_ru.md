### Вступление

**Zig** - это системный язык программирования общего назначения и набор инструментов для поддержки надежного,
оптимального и многократно используемого программного обеспечения.

- Надёжный
    - Поведение корректно даже в крайних случаях, таких как нехватка памяти.
- Оптимальный
    - Пишите программы так, чтобы они могли работать наилучшим образом.
- Многоразовый
    - Один и тот же код работает во многих средах с различными ограничениями.
- Поддерживаемый
    - Точно передает намерения компилятору и другим программистам.
    - Этот язык не требует больших затрат на чтение кода и устойчив к меняющимся требованиям и средам.

Часто самый эффективный способ узнать что-то новое - это посмотреть примеры, поэтому в этой документации показано как
использовать каждую из функций **Zig**. Все это находится на одной странице, так что вы можете выполнять поиск с помощью
инструмента поиска вашего браузера.

Примеры кода в этом документе скомпилированы и протестированы в рамках основного набора тестов **Zig**.

Этот HTML-документ не зависит от внешних файлов, поэтому вы можете использовать его в автономном режиме.

------------

### Стандартная библиотека **Zig**

Стандартная библиотека **Zig** имеет собственную документацию.

Стандартная библиотека **Zig** содержит часто используемые алгоритмы, структуры данных и определения, которые помогут вам
создавать программы или библиотеки. В этой документации вы увидите множество примеров стандартной библиотеки **Zig**,
используемых в этой документации. Чтобы узнать больше о стандартной библиотеке **Zig**, перейдите по ссылке выше.

------------

### Привет, Мир!

```zig
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    try stdout.print("Hello, {s}!\n", .{"World"});
}
```
```bash
$ zig build-exe hello.zig
$ ./hello_world
Hello, World!
```


В большинстве случаев более целесообразно выполнять запись в `stderr`, а не в стандартный вывод и не имеет значения
будет ли сообщение успешно записано в поток. Для этого общего случая существует более простой API:

```zig
const std = @import("std");

pub fn main() !void {
    std.debug.print("Hello, {s}!\n", .{"World"});
}
```
```bash
$ zig build-exe hello_again.zig
$ ./hello_world_again
Hello, World!
```

В этом случае символ ! может быть опущен в возвращаемом типе, поскольку функция не возвращает ошибок.

------------

### Комментарии

**Zig** поддерживает 3 типа комментариев. Обычные комментарии игнорируются, но компилятор использует комментарии `doc` и
комментарии верхнего уровня для создания документации по пакету.

Созданная документация все еще является экспериментальной и может быть создана с помощью:

```bash
$ zig test -femit-docs main.zig
```

```zig
const print = @import("std").debug.print;

pub fn main() !void {
    // Comments in Zig start with "//" and end at the next LF byte (end of line).
    // The line below is a comment and won't be executed.

    //print("Hello?", .{});
    print("Hello, World!\n", .{});
}
```
```bash
$ zig build-exe comments.zig
$ ./comments
Hello, World!
```

В **Zig** нет многострочных комментариев (например, таких как /* */ comments в C). Это позволяет **Zig** обладать
свойством позволяющим выделять каждую строку кода из контекста.


#### Комментарии к документу

Комментарий к документу `doc`начинается ровно с трех косых черт (т.е. ///, но не с ////); несколько комментариев к
документу `doc`подряд объединяются в многострочный комментарий к документу `doc`. Комментарий к документу документирует
все, что следует непосредственно за ним.

```zig
/// Структура для хранения временной метки с точностью до наносекунды (это
/// многострочный комментарий к документу).
const Timestamp = struct {
    /// Количество секунд, прошедших с момента начала эпохи (это также комментарий к документу).
    seconds: i64, // signed so we can represent pre-1970 (not a doc comment)
    /// Количество наносекунд, прошедших после второй (снова комментарий к документу).
    nanos: u32,

    /// Возвращает структуру `Timestamp`, представляющую эпоху Unix; то есть
    /// момент времени 1 января 1970 года, 1 00:00:00 UTC (это тоже комментарий к документу).
    pub fn unixEpoch() Timestamp {
        return Timestamp{
            .seconds = 0,
            .nanos = 0,
        };
    }
};
```

Комментарии `doc` разрешены только в определенных местах; наличие комментария `doc` в неожиданном месте, например, в
середине выражения или непосредственно перед комментарием не относящимся к `doc` является ошибкой компиляции.

```zig
/// doc-comment
//! top-level doc-comment
const std = @import("std");
```
```bash
$ zig build-obj invalid_doc-comment.zig
doc/langref/invalid_doc-comment.zig:1:16: error: expected type expression, found 'a document comment'
/// doc-comment
```

```zig
pub fn main() void {}

/// End of file
```
```bash
$ zig build-obj unattached_doc-comment.zig
doc/langref/unattached_doc-comment.zig:3:1: error: unattached documentation comment
/// End of file
^~~~~~~~~~~~~~~
```

Комментарии в формате `doc` могут чередоваться с обычными комментариями. В настоящее время при создании документации по
вашему вниманию общие рекомендации которые объединяются с рекомендациями в документе `doc`.

#### Комментарии к документу верхнего уровня

Комментарий к документу верхнего уровня начинается с двух косых черт и восклицательного знака: //!; он документирует
текущий модуль. Ошибка компиляции возникает, если комментарий к документу верхнего уровня не помещается в начало
контейнера перед какими-либо выражениями.

```zig
//! Этот модуль предоставляет функции для получения текущей даты и
//! времени с различной степенью точности. Он не
//! зависит от libc, но будет использовать функции из него, если они доступны.

const S = struct {
    //! Комментарии верхнего уровня разрешены внутри контейнера, отличного от модуля,
    //! но это не очень полезно.  В настоящее время при создании документации по пакету
    //! эти комментарии игнорируются.
};
```
------------

### Значения

```zig
// Объявления верхнего уровня не зависят от порядка:
const print = std.debug.print;
const std = @import("std");
const os = std.os;
const assert = std.debug.assert;

pub fn main() !void {
    // integers
    const one_plus_one: i32 = 1 + 1;
    print("1 + 1 = {d}\n", .{one_plus_one});

    // floats
    const seven_div_three: f32 = 7.0 / 3.0;
    print("7.0 / 3.0 = {any}\n", .{seven_div_three});

    // booleans
    print("{any}\n{any}\n{any}\n\n", .{
        true and false,
        true or false,
        !true,
    });

    // optional
    var optional_value: ?[]const u8 = null;
    assert(optional_value == null);

    print("optional 1\ntype: {any}\nvalue: {?s}\n\n", .{
        @TypeOf(optional_value),
        optional_value,
    });

    optional_value = "hi";
    assert(optional_value != null);

    print("optional 2\ntype: {any}\nvalue: {?s}\n\n", .{
        @TypeOf(optional_value),
        optional_value,
    });

    // error union
    var number_or_error: anyerror!i32 = error.ArgNotFound;

    print("error union 1\ntype: {any}\nvalue: {any}\n\n", .{
        @TypeOf(number_or_error),
        number_or_error,
    });

    number_or_error = 1234;

    print("error union 2\ntype: {any}\nvalue: {any}\n\n", .{
        @TypeOf(number_or_error),
        number_or_error,
    });
}
```
```bash
$ zig build-exe values.zig
$ ./values
1 + 1 = 2
7.0 / 3.0 = 2.3333333e0
false
true
false

optional 1
type: ?[]const u8
value: null

optional 2
type: ?[]const u8
value: hi

error union 1
type: anyerror!i32
value: error.ArgNotFound

error union 2
type: anyerror!i32
value: 1234
```

**<center>Primitive Types</center>**
|Type|C Equivalent|Description|
|----|------------|-----------|
| <font color="blue">`i8`</font>| `int8_t` | signed 8-bit integer
| <font color="blue">`u8`</font>| `uint8_t` | unsigned 8-bit integer
| <font color="blue">`i16`</font>| `int16_t` | signed 16-bit integer |
| <font color="blue">`u16`</font>| `uint16_t` | unsigned 16-bit integer
| <font color="blue">`i32`</font>| `int32_t` | signed 32-bit integer
| <font color="blue">`u32`</font>| `uint32_t` | unsigned 32-bit integer
| <font color="blue">`i64`</font>| `int64_t` | signed 64-bit integer
| <font color="blue">`u64`</font>| `uint64_t` | unsigned 64-bit integer
| <font color="blue">`i128`</font>| `__int128` | signed 128-bit integer
| <font color="blue">`u128`</font>| `unsigned __int128` | unsigned 128-bit integer
| <font color="blue">`isize`</font>| `intptr_t` | signed pointer sized integer
| <font color="blue">`usize`</font>| `uintptr_t`, `size_t` | unsigned pointer sized integer. Also see #5185
| <font color="blue">`c_char`</font>| `char` | for ABI compatibility with C
| <font color="blue">`c_short`</font>| `short` | for ABI compatibility with C
| <font color="blue">`c_ushort`</font>| `unsigned short` | for ABI compatibility with C
| <font color="blue">`c_int`</font> | `int` | for ABI compatibility with C
| <font color="blue">`c_uint`</font>| `unsigned int` | for ABI compatibility with C
| <font color="blue">`c_long`</font> | `long` | for ABI compatibility with C
| <font color="blue">`c_ulong`</font>| `unsigned long` | for ABI compatibility with C
| <font color="blue">`c_longlong`</font>| `long long` | for ABI compatibility with C
| <font color="blue">`c_ulonglong`</font>| `unsigned long long` | for ABI compatibility with C
| <font color="blue">`c_longdouble`</font>| `long double` | for ABI compatibility with C
| <font color="blue">`f16`</font>| `_Float16` | 16-bit floating point (10-bit mantissa) IEEE-754-2008 binary16
| <font color="blue">`f32`</font>| `float` | 32-bit floating point (23-bit mantissa) IEEE-754-2008 binary32
| <font color="blue">`f64`</font>| `double` | 64-bit floating point (52-bit mantissa) IEEE-754-2008 binary64
| <font color="blue">`f80`</font>| `double` | 80-bit floating point (64-bit mantissa) IEEE-754-2008 80-bit extended precision
| <font color="blue">`f128`</font>| `_Float128` | 128-bit floating point (112-bit mantissa) IEEE-754-2008 binary128
| <font color="blue">`bool`</font>| `bool` | true or false
| <font color="blue">`anytype`</font>| `void` | Used for type-erased pointers.
| <font color="blue">`void`</font>| `(none)` |  Always the value void{}
| <font color="blue">`noreturn`</font>| `(none)` | the type of break, continue, return, unreachable, and while (true) {}
| <font color="blue">`type`</font>| `(none)` | the type of types
| <font color="blue">`anyerror`</font>| `(none)` | an error code
| <font color="blue">`comptime_int`</font>| `(none)` | Only allowed for comptime-known values. The type of integer literals.
| <font color="blue">`comptime_float`</font>| `(none)` | Only allowed for comptime-known values. The type of float literals.

В дополнение к целочисленным типам, указанным выше, можно использовать ссылки на целые числа произвольной разрядности,
используя идентификатор `i` или `u`, за которым следуют цифры. Например, идентификатор i7 относится к 7-разрядному
целому числу со знаком. Максимально допустимая разрядность для целочисленного типа равна 65535.

**<center>Primitive Values</center>**
|Name|Description|
|---|---|
| <font color="red">`true`</font> and <font color="red">`false`</font> | <font color="blue">`bool`</font> values |
| <font color="red">`null`</font> | used to set an optional type to <font color="red">`null`</font> |
| <font color="red">`undefined`</font> | used to leave a value unspecified |


#### Строковые литералы и литералы с кодовой точкой в Юникоде

Строковые литералы - это константные одноэлементные указатели на массивы байтов заканчивающиеся нулем. Тип строковых
литералов кодирует как длину, так и тот факт, что они заканчиваются нулем и таким образом, они могут быть
преобразованы как в слайсы, так и в указатели, заканчивающиеся нулем. Разыменование строковых литералов преобразует
их в массивы.

Поскольку исходный код **Zig** закодирован в UTF-8, любые байты, отличающиеся от ASCII, появляющиеся в строковом литерале в
исходном коде, передают свое значение в формате UTF-8 в содержимое строки в программе **Zig**; байты не изменяются
компилятором. Можно вставить байты, отличные от UTF-8, в строковый литерал, используя обозначение \xNN.

При индексировании строки, содержащей байты, отличные от ASCII, возвращаются отдельные байты, независимо от того,
является ли допустимым UTF-8 или нет.

Литералы с кодовой точкой в Юникоде имеют тип `comptime_int`, такой же, как и целочисленные литералы. Все управляющие
последовательности допустимы как в строковых литералах, так и в литералах с кодовой точкой в Юникоде.

```zig
const print = @import("std").debug.print;
const mem = @import("std").mem;

pub fn main() !void {
    const bytes = "hello";
    print("{any}\n", .{@TypeOf(bytes)});
    print("{d}\n", .{bytes.len});
    print("{c}\n", .{bytes[1]});
    print("{d}\n", .{bytes[5]});
    print("{}\n", .{'e' == '\x65'});
    print("{d}\n", .{'\u{1f4a9}'});
    print("{d}\n", .{'💯'}); // 128175
    print("{u}\n", .{'⚡'});
    print("{}\n", .{mem.eql(u8, "hello", "h\x65llo")}); // true
    print("{}\n", .{mem.eql(u8, "💯", "\xf0\x9f\x92\xaf")}); // also true

    const invalid_utf8 = "\xff\xfe"; // non-UTF-8 strings are possible with \xNN notation.
    print("0x{x}\n", .{invalid_utf8[1]}); // indexing them returns individual bytes...
    print("0x{x}\n", .{"💯"[1]}); // ...as does indexing part-way through non-ASCII characters
}
```
```bash
$ zig build-exe string_literals.zig
$ ./string_literals
*const [5:0]u8
5
e
0
true
128169
128175
⚡
true
true
0xfe
0x9f
```

##### Управляющие последовательности

**<center>Escape Sequences</center>**
|Escape Sequence|Name|
|---------------|----|
|`\n` |Newline
|`\r` | Carriage Return
|`\t` | Tab
|`\\` | Backslash
|`\'` | Single Quote
|`\"` | Double Quote
|`\xNN` | hexadecimal 7-bit byte value (2 digits)
|`\u{NNNNNN}` | hexadecimal Unicode code point UTF-8 encoded (1 or more digits)

Обратите внимание, что максимально допустимая точка в Юникоде равна <font color="red">`0x10ffff`</font>.

##### Многострочные строковые литералы

Многострочные строковые литералы не имеют экранирующих элементов и могут занимать несколько строк. Чтобы начать
многострочный строковый литерал, используйте символ \\. Как и в случае с комментарием, строковый литерал продолжается до
конца строки. Конец строки не включается в строковый литерал. Однако, если следующая строка начинается с \\, то
добавляется новая строка и строковый литерал продолжается.

```zig
const hello_world_in_c =
    \\#include <stdio.h>
    \\
    \\int main(int argc, char **argv) {
    \\    printf("hello world\n");
    \\    return 0;
    \\}
;
```

#### Присваивание

Используйте ключевое слово `const` для присваивания значения идентификатору:

```zig
const x = 1234;

fn foo() void {
    // Он работает как в области действия файла, так и внутри функций.
    const y = 5678;

    // После присваивания идентификатор не может быть изменен.
    y += 1;
}

pub fn main() void {
    foo();
}
```
```bash
$ zig build-exe constant_identifier_cannot_change.zig
/home/andy/src/zig/doc/langref/constant_identifier_cannot_change.zig:8:7: error: cannot assign to constant
    y += 1;
    ~~^~~~
referenced by:
    main: /home/andy/src/zig/doc/langref/constant_identifier_cannot_change.zig:12:5
    callMain: /home/andy/src/zig/lib/std/start.zig:514:17
    remaining reference traces hidden; use '-freference-trace' to see all reference traces
```
Значение `const` применяется ко всем байтам к которым непосредственно обращается идентификатор. Указатели имеют свою
собственную константу.

Если вам нужна переменная которую вы можете изменять, используйте ключевое слово `var`:
```zig
const print = @import("std").debug.print;

pub fn main() void {
    var y: i32 = 5678;

    y += 1;

    print("{d}", .{y});
}
```
```bash
$ zig build-exe mutable_var.zig
$ ./mutable_var
5679
```
Переменные должны быть инициализированы:
```zig
pub fn main() void {
    var x: i32;

    x = 1;
}
```
```bash
$ zig build-exe var_must_be_initialized.zig
/home/andy/src/zig/doc/langref/var_must_be_initialized.zig:2:15: error: expected '=', found ';'
    var x: i32;
```

##### Неопределенный

Используйте <font color="red">`undefined`</font> чтобы оставить переменные неинициализированными:
```zig
const print = @import("std").debug.print;

pub fn main() void {
    var x: i32 = undefined;
    x = 1;
    print("{d}", .{x});
}
```
```bash
$ zig build-exe assign_undefined.zig
$ ./assign_undefined
1
```

Значение <font color="red">`undefined`</font> может быть приведено к любому типу. Как только это произойдет, уже
невозможно будет определить, что значение не определено. Значение <font color="red">`undefined`</font> может быть
любым, даже бессмысленным в соответствии с типом. В переводе на английский <font color="red">`undefined`</font>
означает "Не имеющее смысла значение". Использование этого значения может привести к ошибке. Значение будет
неиспользуемым или перезаписанным перед использованием.

В режиме отладки **Zig** записывает <font color="red">`0xaa`</font> байт в неопределенную память. Это делается для
раннего выявления ошибок и помогает обнаружить использование неопределенной памяти в отладчике. Однако такое поведение
является только особенностью реализации, а не семантикой языка, поэтому не гарантируется, что оно будет наблюдаться в
коде.

------------

### Тестирование в Zig

Код, написанный в рамках одного или нескольких тестовых объявлений может быть использован для обеспечения соответствия
поведения ожиданиям программы:

```zig
const std = @import("std");

test "expect addOne adds one to 41" {
    // Стандартная библиотека содержит полезные функции, помогающие создавать тесты.
    // `expect` - это функция, которая проверяет, что ее аргумент равен true.
    // Она вернет ошибку, если ее аргумент равен false, что указывает на сбой.
    // `try` используется для возврата ошибки тестировщику, чтобы уведомить его о том, что тест не прошел.
    try std.testing.expect(addOne(41) == 42);
}

test addOne {
    // Имя теста также может быть записано с использованием идентификатора.
    // Это тестовый документ, который служит документацией для `addOne`.
    try std.testing.expect(addOne(41) == 42);
}

/// Функция `addOne` добавляет единицу к числу, указанному в качестве ее аргумента.
fn addOne(number: i32) i32 {
    return number + 1;
}
```
```bash
$ zig test testing_introduction.zig
1/2 testing_introduction.test.expect addOne adds one to 41...OK
2/2 testing_introduction.decltest.addOne...OK
All 2 tests passed.
```

Пример кода testing_introduction.zig проверяет функцию `addOne`, чтобы убедиться, что она возвращает значение 42 при
входных данных 41. С точки зрения этого теста, функция `addOne` является тестируемым кодом.

`zig test` - это инструмент, который создает и запускает тестовую сборку. По умолчанию он создает и запускает исполняемую
программу, используя средство запуска тестов по умолчанию предоставляемое стандартной библиотекой **Zig**, в качестве
основной точки входа. Во время сборки объявления тестов найденные при разрешении данного исходного файла **Zig**,
включаются в программу тестирования по умолчанию для запуска и составления отчета.

>В этой документации рассматриваются возможности программы запуска тестов по умолчанию, предоставляемой стандартной
>библиотекой **Zig**. Ее исходный код находится в lib/test_runner.zig.

В выводе командной строки, показанном выше, после команды `zig test` отображаются две строки. Программа тестирования по
умолчанию выводит эти строки со стандартной ошибкой:
**<br>`1/2 testing_introduction.test.expect addOne adds one to 41...`<br>**
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Строки, подобные этой, указывают, какой тест из
общего числа тестов выполняется. В данном случае 1/2 означает, что выполняется первый тест из общего числа двух тестов.
Обратите внимание, что когда в терминал выводится стандартная ошибка программы test runner, эти строки очищаются при
успешном завершении теста.
**<br>`2/2 testing_introduction.decltest.addOne...`<br>**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Когда имя теста является идентификатором, программа
тестирования по умолчанию использует текст decltest вместо test. **<br>`All 2 tests passed.`<br>**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;В этой строке указано общее количество пройденных
тестов.

#### Объявления тестов

Объявления тестов содержат ключевое слово `test`, за которым следует необязательное имя, записанное в виде строкового
литерала или идентификатора, а затем блок, содержащий любой допустимый **Zig**-код, который разрешен в функции.

Тестовые блоки без имени всегда выполняются во время сборки тестов и не подлежат пропуску тестов.

Объявления тестов похожи на функции: у них есть тип возвращаемого значения и блок кода. Неявный тип возвращаемого
значения теста - это тип объединения ошибок `anyerror!void`, и его нельзя изменить. Если исходный файл **Zig** не
создается с помощью `zig test tool`, объявления тестов не включаются в сборку.

Объявления тестов могут быть записаны в том же файле, в котором записан тестируемый код, или в отдельном исходном файле
**Zig**. Поскольку тестовые объявления являются объявлениями верхнего уровня, они не зависят от порядка выполнения и
могут быть написаны до или после тестируемого кода.

##### Тестирование документации

Тестовые объявления именуемые с использованием идентификатора являются тестами doctests. Идентификатор должен
ссылаться на другое объявление в области видимости. Тест doctest, как и комментарий к документу, служит документацией
для соответствующего объявления и будет отображаться в сгенерированной документации для объявления.

Эффективное тестирование должно быть автономным и сосредоточенным на тестируемой декларации, отвечать на вопросы,
которые могут возникнуть у нового пользователя о ее интерфейсе или предполагаемом использовании, избегая при этом
ненужных или сбивающих с толку деталей. Doctest - это не замена комментария к документу, а скорее дополнение и
компаньон, предоставляющий тестируемый пример, основанный на коде, проверенный с помощью `zig test`.

#### Ошибки в тестах

Программа запуска тестов по умолчанию проверяет наличие ошибки возвращенной из теста. Когда тест возвращает ошибку,
тест считается неудачным и его трассировка возврата ошибки выводится в виде стандартной ошибки. Общее количество ошибок
будет указано после выполнения всех тестов.

```zig
const std = @import("std");

test "expect this to fail" {
    try std.testing.expect(false);
}

test "expect this to succeed" {
    try std.testing.expect(true);
}
```
```bash
$ zig test testing_failure.zig
1/2 testing_failure.test.expect this to fail...FAIL (TestUnexpectedResult)
/home/andy/src/zig/lib/std/testing.zig:540:14: 0x103ce3f in expect (test)
    if (!ok) return error.TestUnexpectedResult;
             ^
/home/andy/src/zig/doc/langref/testing_failure.zig:4:5: 0x103cf55 in test.expect this to fail (test)
    try std.testing.expect(false);
    ^
2/2 testing_failure.test.expect this to succeed...OK
1 passed; 0 skipped; 1 failed.
error: the following test command failed with exit code 1:
/home/andy/src/zig/.zig-cache/o/054f0b6f088824f384d1b6c648523593/test
```

#### Пропуск тестов

Один из способов пропустить тесты - отфильтровать их с помощью параметра командной строки `zig test --test-filter [text]`.
В результате в тестовую сборку будут включены только те тесты, название которых содержит указанный текст фильтра.
Обратите внимание, что неименованные тесты запускаются даже при использовании параметра командной строки `--test-filter
[text]`.

Чтобы программно пропустить тест, сделайте так, чтобы тест возвращал сообщение об ошибке `error.SkipZigTest` и программа
тестирования по умолчанию будут считать тест пропущенным. После выполнения всех тестов будет сообщено общее количество
пропущенных тестов.

```zig
test "this will be skipped" {
    return error.SkipZigTest;
}
```
```bash
$ zig test testing_skip.zig
1/1 testing_skip.test.this will be skipped...SKIP
0 passed; 1 skipped; 0 failed.h
```

#### Сообщение об утечках памяти

Когда код выделяет память с помощью тестового аллокатора стандартной библиотеки **Zig**, `std.testing.allocator`,
программа тестирования по умолчанию сообщает о любых утечках, обнаруженных при использовании тестового аллокатора:

```zig
const std = @import("std");

test "detect leak" {
    var list = std.ArrayList(u21).init(std.testing.allocator);
    // отсутствует `defer list.deinit();`
    try list.append('☔');

    try std.testing.expect(list.items.len == 1);
}
```
```bash
$ zig test testing_detect_leak.zig
1/1 testing_detect_leak.test.detect leak...OK
[gpa] (err): memory address 0x7fdbea69e000 leaked:
/home/andy/src/zig/lib/std/array_list.zig:457:67: 0x104f76e in ensureTotalCapacityPrecise (test)
                const new_memory = try self.allocator.alignedAlloc(T, alignment, new_capacity);
                                                                  ^
/home/andy/src/zig/lib/std/array_list.zig:434:51: 0x1045610 in ensureTotalCapacity (test)
            return self.ensureTotalCapacityPrecise(better_capacity);
                                                  ^
/home/andy/src/zig/lib/std/array_list.zig:483:41: 0x1041fe0 in addOne (test)
            try self.ensureTotalCapacity(newlen);
                                        ^
/home/andy/src/zig/lib/std/array_list.zig:262:49: 0x103ef2d in append (test)
            const new_item_ptr = try self.addOne();
                                                ^
/home/andy/src/zig/doc/langref/testing_detect_leak.zig:6:20: 0x103d172 in test.detect leak (test)
    try list.append('☔');
                   ^
/home/andy/src/zig/lib/compiler/test_runner.zig:157:25: 0x104c6a0 in mainTerminal (test)
        if (test_fn.func()) |_| {
                        ^
/home/andy/src/zig/lib/compiler/test_runner.zig:37:28: 0x10428bb in main (test)
        return mainTerminal();
                           ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x103f429 in posixCallMainAndExit (test)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x103ef91 in _start (test)
    asm volatile (switch (native_arch) {
    ^

All 1 tests passed.
1 errors were logged.
1 tests leaked memory.
error: the following test command failed with exit code 1:
/home/andy/src/zig/.zig-cache/o/4a17198138bf81bcfabd5652b0d6be24/test
```

#### Определение тестовой сборки

Используйте переменную времени компиляции `@import("builtin").is_test` для определения тестовой сборки:

```zig
const std = @import("std");
const builtin = @import("builtin");
const expect = std.testing.expect;

test "builtin.is_test" {
    try expect(isATest());
}

fn isATest() bool {
    return builtin.is_test;
}
```
```bash
$ zig test testing_detect_test.zig
1/1 testing_detect_test.test.builtin.is_test...OK
All 1 tests passed.
```

#### Тестовый вывод и ведение журнала

Программа запуска тестов по умолчанию и пространство имен тестирования стандартной библиотеки **Zig** выводят сообщения
о стандартной ошибке.

#### Пространство имен для тестирования

Пространство имен `testing` стандартной библиотеки **Zig** содержит полезные функции, которые помогут вам создавать
тесты. В дополнение к функции `expect` в этом документе используется еще несколько функций, как показано на примере
ниже:

```zig
const std = @import("std");

test "expectEqual demo" {
    const expected: i32 = 42;
    const actual = 42;

    // Первый аргумент `expectEqual` - это известный ожидаемый результат.
    // Второй аргумент - это результат некоторого выражения.
    // Тип actual преобразуется в тип expected.
    try std.testing.expectEqual(expected, actual);
}

test "expectError demo" {
    const expected_error = error.DemoError;
    const actual_error_union: anyerror!void = error.DemoError;

    // `expectError` завершится ошибкой, если фактическая ошибка отличается от
    // ожидаемой ошибки.
    try std.testing.expectError(expected_error, actual_error_union);
}
```
```bash
$ zig test testing_namespace.zig
1/2 testing_namespace.test.expectEqual demo...OK
2/2 testing_namespace.test.expectError demo...OK
All 2 tests passed.
```

Стандартная библиотека **Zig** также содержит функции для сравнения слайсов, строк и многого другого. Изучите
остальную часть пространства имен `std.testing` в стандартной библиотеке **Zig** для получения дополнительных функций.

#### Документация по инструменту тестирования

В `zig test` есть несколько параметров командной строки, которые влияют на компиляцию тестов. Полный список приведен в
разделе `zig test --help`.

------------

### Переменные

Переменная - это единица хранения в памяти.

Как правило, при объявлении переменной предпочтительнее использовать `const`, а не `var`. Это сокращает работу как
для людей так и для компьютеров при чтении кода и создает больше возможностей для оптимизации.

Ключевое слово `extern` или встроенную функцию `@extern` можно использовать для связывания с переменной, экспортируемой
из другого объекта. Ключевое слово `export` или встроенную функцию `@export` можно использовать для того, чтобы сделать
переменную доступной для других объектов во время линковки. В обоих случаях тип переменной должен быть совместим с `C
ABI`.

#### Идентификаторы

Идентификаторы переменных никогда не должны затенять идентификаторы из внешней области видимости.

Идентификаторы должны начинаться с буквенного символа или символа подчеркивания и за ними может следовать любое
количество буквенно-цифровых символов или знаков подчеркивания. Они не должны пересекаться с какими-либо ключевыми
словами.

Если требуется имя, которое не соответствует этим требованиям, например, для связи с внешними библиотеками, можно
использовать синтаксис `@""`.

```zig
const @"identifier with spaces in it" = 0xff;
const @"1SmallStep4Man" = 112358;

const c = @import("std").c;
pub extern "c" fn @"error"() void;
pub extern "c" fn @"fstat$INODE64"(fd: c.fd_t, buf: *c.Stat) c_int;

const Color = enum {
    red,
    @"really red",
};
const color: Color = .@"really red";
```

#### Переменные уровня контейнера

Переменные уровня контейнера имеют статическое время жизни, не зависят от порядка и лениво анализируются.
Значением инициализации переменных уровня контейнера неявно является `comptime`. Если переменная уровня контейнера
равна `const`, то ее значение известно во времени компиляции, в противном случае оно известно во время выполнения.

```zig
var y: i32 = add(10, x);
const x: i32 = add(12, 34);

test "container level variables" {
    try expect(x == 46);
    try expect(y == 56);
}

fn add(a: i32, b: i32) i32 {
    return a + b;
}

const std = @import("std");
const expect = std.testing.expect;
```
```bash
$ zig test test_container_level_variables.zig
1/1 test_container_level_variables.test.container level variables...OK
All 1 tests passed.
```

Переменные уровня контейнера могут быть объявлены внутри `struct`, `union`, `enum` или `opaque`:

```zig
const std = @import("std");
const expect = std.testing.expect;

test "namespaced container level variable" {
    try expect(foo() == 1235);
    try expect(foo() == 1236);
}

const S = struct {
    var x: i32 = 1234;
};

fn foo() i32 {
    S.x += 1;
    return S.x;
}
```
```bash
$ zig test test_namespaced_container_level_variable.zig
1/1 test_namespaced_container_level_variable.test.namespaced container level variable...OK
All 1 tests passed.
```

#### Статические локальные переменные

Также возможно иметь локальные переменные со статическим временем жизни, используя контейнеры внутри функций.

```zig
const std = @import("std");
const expect = std.testing.expect;

test "static local variable" {
    try expect(foo() == 1235);
    try expect(foo() == 1236);
}

fn foo() i32 {
    const S = struct {
        var x: i32 = 1234;
    };
    S.x += 1;
    return S.x;
}
```
```bash
$ zig test test_static_local_variable.zig
1/1 test_static_local_variable.test.static local variable...OK
All 1 tests passed.
```

#### Локальные переменные потока

Переменная может быть указана как локальная переменная потока с использованием ключевого слова `threadlocal`, что
позволяет каждому потоку работать с отдельным экземпляром переменной:

```zig
const std = @import("std");
const assert = std.debug.assert;

threadlocal var x: i32 = 1234;

test "thread local storage" {
    const thread1 = try std.Thread.spawn(.{}, testTls, .{});
    const thread2 = try std.Thread.spawn(.{}, testTls, .{});
    testTls();
    thread1.join();
    thread2.join();
}

fn testTls() void {
    assert(x == 1234);
    x += 1;
    assert(x == 1235);
}
```
```bash
$ zig test test_thread_local_variables.zig
1/1 test_thread_local_variables.test.thread local storage...OK
All 1 tests passed.
```
Для однопоточных сборок все локальные переменные потока обрабатываются как обычные переменные уровня контейнера.

Локальные переменные потока могут быть не `const`.

#### Локальные переменные

Локальные переменные встречаются внутри функций, блоков `comptime` и блоков `@cImport`.

Если локальная переменная имеет значение `const`, это означает, что после инициализации значение переменной не
изменится. Если значение инициализации переменной `const` известно в `comptime` контексте то эта переменная также
известна в `comptime` контексте.

Локальная переменная может быть определена с помощью ключевого слова `comptime`. Это приводит к тому, что значение
переменной становится известным во время компиляции и все загрузки и сохранения переменной происходят во время
семантического анализа программы, а не во время выполнения. Все переменные, объявленные в выражении `comptime` неявно
являются переменными `comptime` контекста.

```zig
const std = @import("std");
const expect = std.testing.expect;

test "comptime vars" {
    var x: i32 = 1;
    comptime var y: i32 = 1;

    x += 1;
    y += 1;

    try expect(x == 2);
    try expect(y == 2);

    if (y != 2) {
        // Эта ошибка компиляции никогда не срабатывает, потому что y - это переменная времени компиляции,
        // и поэтому `y != 2` - это значение времени компиляции, и это значение if вычисляется статически.
        @compileError("wrong y value");
    }
}
```
```bash
$ zig test test_comptime_variables.zig
1/1 test_comptime_variables.test.comptime vars...OK
All 1 tests passed.
```

------------

### Целые числа

#### Целочисленные литералы

```zig
const decimal_int = 98222;
const hex_int = 0xff;
const another_hex_int = 0xFF;
const octal_int = 0o755;
const binary_int = 0b11110000;

// символы подчеркивания могут быть помещены между двумя цифрами в качестве визуального разделителя
const one_billion = 1_000_000_000;
const binary_mask = 0b1_1111_1111;
const permissions = 0o7_5_5;
const big_address = 0xFF80_0000_0000_0000;
```

#### Целочисленные значения времени выполнения

Целочисленные литералы не имеют ограничений по размеру и если возникает какое-либо неопределенное поведение, компилятор
перехватывает его.

Однако, если целочисленное значение больше и не известно во время компиляции то оно должно иметь известный размер и
уязвимо для неопределенного поведения.

```zig
fn divide(a: i32, b: i32) i32 {
    return a / b;
}
```

В этой функции значения `a` и `b` известны только во время выполнения и таким образом эта операция деления уязвима для
обоих ошибок переполнение целого числа и деления на ноль.

Такие операторы как `+` и `-` вызывают неопределенное поведение при переполнении целых чисел. Предусмотрены
альтернативные операторы для переноса и насыщения арифметических данных по всем целевым объектам. `+%` и `-%` выполняют
арифметику переноса, в то время как `+|` и `-|` выполняют арифметику насыщения.

**Zig** поддерживает целые числа произвольной разрядности, для обозначения которых используется идентификатор `i` или
`u`, за которым следуют цифры. Например, идентификатор `i7` относится к 7-разрядному целому числу со знаком. Максимально
допустимая разрядность целочисленного типа равна `65535`. Для целых типов со знаком **Zig** использует представление
дополнение до двух.

### Числа с плавающей запятой

**Zig** имеет следующие типы с плавающей запятой:
- <span style="color:blue">`f16`</span> - IEEE-754-2008 binary16
- <span style="color:blue">`f32`</span> - IEEE-754-2008 binary32
- <span style="color:blue">`f64`</span> - IEEE-754-2008 binary64
- <span style="color:blue">`f80`</span>- IEEE-754-2008 80-bit extended precision
- <span style="color:blue">`f128`</span> - IEEE-754-2008 binary128
- <span style="color:blue">`c_longdouble`</span> - matches `long double` for the target C ABI

#### Литералы с плавающей запятой

Литералы с плавающей запятой имеют тип `comptime_float`, который гарантированно имеет ту же точность и операции, что и
другой самый большой тип с плавающей запятой который является f128.

Литералы с плавающей запятой приводятся к любому типу с плавающей запятой и к любому целочисленному типу, если в нем нет
дробной составляющей.

```zig
const floating_point = 123.0E+77;
const another_float = 123.0;
const yet_another = 123.0e+77;

const hex_floating_point = 0x103.70p-5;
const another_hex_float = 0x103.70;
const yet_another_hex_float = 0x103.70P-5;

// символы подчеркивания могут быть помещены между двумя цифрами в качестве визуального разделителя
const lightspeed = 299_792_458.000_000;
const nanosecond = 0.000_000_001;
const more_hex = 0x1234_5678.9ABC_CDEFp-10;
```

Не существует синтаксиса для значений NaN, infinity или отрицательной бесконечности. Для получения этих специальных
значений необходимо использовать стандартную библиотеку:
```zig
const std = @import("std");

const inf = std.math.inf(f32);
const negative_inf = -std.math.inf(f64);
const nan = std.math.nan(f128);
```

#### Операции с плавающей запятой

По умолчанию для операций с плавающей запятой используется строгий режим, но вы можете переключиться на оптимизированный
режим для каждого блока:
```zig
const std = @import("std");
const big = @as(f64, 1 << 40);

export fn foo_strict(x: f64) f64 {
    return x + big - big;
}

export fn foo_optimized(x: f64) f64 {
    @setFloatMode(.optimized);
    return x + big - big;
}
```
```bash
$ zig build-obj float_mode_obj.zig -O ReleaseFast
```

Для этого теста мы должны разделить код на два объектных файла - в противном случае оптимизатор вычисляет все значения
во время компиляции, которая работает в строгом режиме.
```zig
const print = @import("std").debug.print;

extern fn foo_strict(x: f64) f64;
extern fn foo_optimized(x: f64) f64;

pub fn main() void {
    const x = 0.001;
    print("optimized = {}\n", .{foo_optimized(x)});
    print("strict = {}\n", .{foo_strict(x)});
}
```

------------

### Операторы

Перегрузки операторов нет. Когда вы видите оператор в **Zig**, вы знаете, что он выполняет что-то из этой таблицы и
ничего больше.

**<center>Table of Operators</center>**

|Name|Syntax|Types|Remarks|Example|
|----|:------:|-----|-------|-------|
|Addition|`a + b`<br>`a += b`|Integers <br>Floats|Can cause overflow for integers. <br>Invokes Peer Type Resolution for the operands.<br>See also `@addWithOverflow.`| `1 + 5 == 7`
|Wrapping<br>Addition|`a +% b`<br>`a +%= b`|Integers|Twos-complement wrapping behavior.<br>Invokes Peer Type Resolution for the operands.<br>See also `@addWithOverflow.`|`@as(u32, 0xffffffff) +% 1 == 0`
|Saturating<br>Addition|`a +\| b`<br>`a +\|= b`|Integers|Invokes Peer Type Resolution for the operands.|`@as(u8, 255) +\| 1 == @as(u8, 255)`
|Subtraction|`a - b`<br>`a -= b`|Integers<br>Floats|Can cause overflow for integers.<br>Invokes Peer Type Resolution for the operands.<br>See also `@subWithOverflow.`|`2 - 5 == -3`


#### Приоритеты операций
```zig
x() x[] x.y x.* x.?
a!b
x{}
!x -x -%x ~x &x ?x
* / % ** *% *| ||
+ - ++ +% -% +| -|
<< >> <<|
& ^ | orelse catch
== != < > <= >=
and
or
= *= *%= *|= /= %= += +%= +|= -= -%= -|= <<= <<|= >>= &= ^= |=
```
