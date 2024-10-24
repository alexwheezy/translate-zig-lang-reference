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
|Addition|`a + b`<br>`a += b`|Integers <br>Floats|Can cause overflow for integers. <br>Invokes Peer Type Resolution for the operands.<br>See also `@addWithOverflow`.| `1 + 5 == 7`
|Wrapping<br>Addition|`a +% b`<br>`a +%= b`|Integers|Twos-complement wrapping behavior.<br>Invokes Peer Type Resolution for the operands.<br>See also `@addWithOverflow`.|`@as(u32, 0xffffffff) +% 1 == 0`
|Saturating<br>Addition|`a +\| b`<br>`a +\|= b`|Integers|Invokes Peer Type Resolution for the operands.|`@as(u8, 255) +\| 1 == @as(u8, 255)`
|Subtraction|`a - b`<br>`a -= b`|Integers<br>Floats|Can cause overflow for integers.<br>Invokes Peer Type Resolution for the operands.<br>See also `@subWithOverflow`.|`2 - 5 == -3`
|Wrapping<br>Subtraction|`a -% b`<br>`a -%= b`|Integers|Twos-complement wrapping behavior.<br>Invokes Peer Type Resolution for the operands.<br>See also `@subWithOverflow`.|`@as(u8, 0) -% 1 == 255`
|Saturating<br>Subtraction|`a -\| b`<br>`a -\|= b`|Integers|Invokes Peer Type Resolution for the operands.|`@as(u32, 0) -\| 1 == 0`
|Negation|`-a`|Integers<br>Floats|Can cause overflow for integers.|`-1 == 0 - 1`
|Wrapping<br>Negation|`-%a`|Integers|Twos-complement wrapping behavior.|`-%@as(i8, -128) == -128`
|Multiplication|`a * b`<br>`a *= b`|Integers<br>Floats|Can cause overflow for integers.<br>Invokes Peer Type Resolution for the operands.<br>See also `@mulWithOverflow`.|`2 * 5 == 10`
|Wrapping<br>Multiplication|`a *% b`<br>`a *%= b`|Integers|Twos-complement wrapping behavior.<br>Invokes Peer Type Resolution for the operands.<br>See also `@mulWithOverflow`.|`@as(u8, 200) *% 2 == 144`
|Saturating<br>Multiplication|`a *\| b`<br>`a *\|= b`|Integers|Invokes Peer Type Resolution for the operands.|`@as(u8, 200) *\| 2 == 255`
|Division|`a / b`<br>`a /= b`|Integers<br>Floats|Can cause overflow for integers.<br>Can cause Division by Zero for integers.<br>Can cause Division by Zero for floats in FloatMode.Optimized Mode.<br>Signed integer operands must be comptime-known and positive. In other cases, use `@divTrunc`, `@divFloor`, or `@divExact` instead.<br>Invokes Peer Type Resolution for the operands.|`10 / 2 == 5`
|Remainder<br>Division|`a % b`<br>`a %= b`|Integers<br>Floats|Can cause Division by Zero for integers.<br>Can cause Division by Zero for floats in FloatMode.Optimized Mode.<br>Signed or floating-point operands must be comptime-known and positive. In other cases, use @rem or `@mod` instead.<br>Invokes Peer Type Resolution for the operands.|`10 % 3 == 1`
|Bit Shift Left|`a << b`<br>`a <<= b`|Integers|Moves all bits to the left, inserting new zeroes at the least-significant bit.<br>`b` must be comptime-known or have a type with log2 number of bits as `a`.<br>See also `@shlExact`.<br>See also `@shlWithOverflow`.|`0b1 << 8 == 0b100000000`
|Saturating Bit Shift Left|`a <<\| b`<br>`a <<\|= b`|Integers|See also `@shlExact`.<br>See also `@shlWithOverflow`.|`@as(u8,1) <<\| 8 == 255`
|Bit Shift Right|`a >> b`<br>`a >>= b`|Integers|Moves all bits to the right, inserting zeroes at the most-significant bit.<br>`b` must be comptime-known or have a type with log2 number of bits as `a`.<br>See also `@shrExact`.|`0b1010 >> 1 == 0b101`
|Bitwise And|`a & b`<br>`a &= b`|Integers|Invokes Peer Type Resolution for the operands.|`0b011 & 0b101 == 0b001`
|Bitwise Or|`a \| b`<br>`a \|= b`|Integers|Invokes Peer Type Resolution for the operands.|`0b010 \| 0b100 == 0b110`
|Bitwise Xor|`a ^ b`<br>`a ^= b`|Integers|Invokes Peer Type Resolution for the operands.|`0b011 ^ 0b101 == 0b110`
|Bitwise Not|`~a`|Integers||`~@as(u8, 0b10101111) == 0b01010000`
|Defaulting<br>Optional<br>Unwrap|`a orelse b`|Optionals|If `a` is `null`, returns `b` ("default value"), otherwise returns the unwrapped value of `a`. Note that `b` may be a value of type `noreturn`.|`const value: ?u32 = null;`<br>`const unwrapped = value orelse 1234;`<br>`unwrapped == 1234`
|Optional<br>Unwrap|`a.?`|Optionals|Equivalent to: `a orelse unreachable`|`const value: ?u32 = 5678;`<br>`value.? == 5678`
|Defaulting<br>Error Unwrap|`a catch b`<br>`a catch \|err\| b`|Error<br>Unions|If `a` is an `error`, returns `b` ("default value"), otherwise returns the unwrapped value of `a`. Note that `b` may be a value of type `noreturn`. `err` is the `error` and is in scope of the expression `b`.|`const value: anyerror!u32 = error.Broken;`<br>`const unwrapped = value catch 1234;`<br>`unwrapped == 1234`
|Logical And|`a and b`|bool|If `a` is `false`, returns `false` without evaluating `b`. Otherwise, returns `b`.|`(false and true) == false`
|Logical Or|`a or b`|bool|If `a` is `true`, returns `true` without evaluating `b`. Otherwise, returns `b`.|`(false or true) == true`
|Boolean Not|`!a`|bool||`!false == true`
|Equality|`a == b`|Integers<br>Floats<br>bool<br>type|Returns `true` if `a` and `b` are equal, otherwise returns `false`.<br>Invokes Peer Type Resolution for the operands.|`(1 == 1) == true`
|Null Check|`a == null`|Optionals|Returns `true` if `a` is `null`, otherwise returns `false.`|`const value: ?u32 =null;`<br>`(value == null) == true`
|Inequality|`a != b`|Integers<br>Floats<br>bool<br>type|Returns `false` if `a` and `b` are unequal, otherwise returns `true`.<br>Invokes Peer Type Resolution for the operands.|`(1 != 1) == false`
|Non-Null Check|`a != null`|Optionals|Returns `false` if `a` is `null`, otherwise returns `true.`|`const value: ?u32 = 5;`<br>`(value != null) == true`
|Greater Than|`a > b`|Integers<br>Floats|Returns `true` if `a` is greater than `b`, otherwise returns `false`.<br>Invokes Peer Type Resolution for the operands.|`(2 > 1) == true`
|Greater or Equal|`a >= b`|Integers<br>Floats|Returns `true` if `a` is greater than `b` or equal, otherwise returns `false`.<br>Invokes Peer Type Resolution for the operands.|`(2 >= 1) == true`
|Less Than|`a < b`|Integers<br>Floats|Returns `true` if `a` is less than `b`, otherwise returns `false`.<br>Invokes Peer Type Resolution for the operands.|`(1 < 2) == true`
|Lesser or Equal|`a <= b`|Integers<br>Floats|Returns `true` if `a` is less than `b` or equal, otherwise returns `false`.<br>Invokes Peer Type Resolution for the operands.|`(1 <= 2) == true`
|Array Concatenation|`a ++ b`|Arrays|Only available when the lengths of both `a` and `b` are compile-time known.|`const mem = @import("std").mem;`<br>`const array1 = [_]u32{1,2};`<br>`const array2 = [_]u32{3,4};`<br>`const together = array1 ++ array2;`<br>`mem.eql(u32, &together, &[_]u32{1,2,3,4})`
|Array Multiplication|`a ** b`|Arrays|Only available when the length of `a` and `b` are compile-time known.|`const mem = @import("std").mem;`<br>`const pattern = "ab" ** 3;`<br>`mem.eql(u8, pattern, "ababab")`
|Pointer Dereference|`a.*`|Pointers|Pointer dereference.|`const x: u32 = 1234;`<br>`const ptr = &x;`<br>`ptr.* == 1234`
|Address Of|`&a`|All types||`const x: u32 = 1234;`<br>`const ptr = &x;`<br>`ptr.* = 1234`
|Error Set Merge|`a \|\| b`|Error Set Type| Merging Error Set|`const A = error{One};`<br>`const B = error{Two};`<br>`(A \|\| B) == error{One,Two}`

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
------------
### Массивы
```zig
const expect = @import("std").testing.expect;
const assert = @import("std").debug.assert;
const mem = @import("std").mem;

// массив литералов
const message = [_]u8{ 'h', 'e', 'l', 'l', 'o' };

// альтернативная инициализация с указанием размера массива
const alt_message: [5]u8 = .{ 'h', 'e', 'l', 'l', 'o' };

comptime {
    assert(mem.eql(u8, &message, &alt_message));
}

// получить размер массива
comptime {
    assert(message.len == 5);
}

// Строковый литерал - это одноэлементный указатель на массив.
const same_message = "hello";

comptime {
    assert(mem.eql(u8, &message, same_message));
}

test "iterate over an array" {
    var sum: usize = 0;
    for (message) |byte| {
        sum += byte;
    }
    try expect(sum == 'h' + 'e' + 'l' * 2 + 'o');
}

// изменяемый массив
var some_integers: [100]i32 = undefined;

test "modify an array" {
    for (&some_integers, 0..) |*item, i| {
        item.* = @intCast(i);
    }
    try expect(some_integers[10] == 10);
    try expect(some_integers[99] == 99);
}

// объединение массивов работает, если значения известны
// во время компиляции
const part_one = [_]i32{ 1, 2, 3, 4 };
const part_two = [_]i32{ 5, 6, 7, 8 };
const all_of_it = part_one ++ part_two;
comptime {
    assert(mem.eql(i32, &all_of_it, &[_]i32{ 1, 2, 3, 4, 5, 6, 7, 8 }));
}

// помните, что строковые литералы - это массивы
const hello = "hello";
const world = "world";
const hello_world = hello ++ " " ++ world;
comptime {
    assert(mem.eql(u8, hello_world, "hello world"));
}

// ** выполняет повторяющиеся действия
const pattern = "ab" ** 3;
comptime {
    assert(mem.eql(u8, pattern, "ababab"));
}

// инициализируем массив равным нулю
const all_zero = [_]u16{0} ** 10;

comptime {
    assert(all_zero.len == 10);
    assert(all_zero[5] == 0);
}

// используйте код во время компиляции для инициализации массива
var fancy_array = init: {
    var initial_value: [10]Point = undefined;
    for (&initial_value, 0..) |*pt, i| {
        pt.* = Point{
            .x = @intCast(i),
            .y = @intCast(i * 2),
        };
    }
    break :init initial_value;
};
const Point = struct {
    x: i32,
    y: i32,
};

test "compile-time array initialization" {
    try expect(fancy_array[4].x == 4);
    try expect(fancy_array[4].y == 8);
}

// вызов функции для инициализации массива
var more_points = [_]Point{makePoint(3)} ** 10;
fn makePoint(x: i32) Point {
    return Point{
        .x = x,
        .y = x * 2,
    };
}
test "array initialization with function calls" {
    try expect(more_points[4].x == 3);
    try expect(more_points[4].y == 6);
    try expect(more_points.len == 10);
}
```
```bash
$ zig test test_arrays.zig
1/4 test_arrays.test.iterate over an array...OK
2/4 test_arrays.test.modify an array...OK
3/4 test_arrays.test.compile-time array initialization...OK
4/4 test_arrays.test.array initialization with function calls...OK
All 4 tests passed.
```

#### Многомерные массивы

Многомерные массивы могут быть созданы путем вложения массивов одного в другой:
```zig
const std = @import("std");
const expect = std.testing.expect;

const mat4x4 = [4][4]f32{
    [_]f32{ 1.0, 0.0, 0.0, 0.0 },
    [_]f32{ 0.0, 1.0, 0.0, 1.0 },
    [_]f32{ 0.0, 0.0, 1.0, 0.0 },
    [_]f32{ 0.0, 0.0, 0.0, 1.0 },
};
test "multidimensional arrays" {
    // Получите доступ к двумерному массиву, проиндексировав внешний массив, а затем внутренний массив.
    try expect(mat4x4[1][1] == 1.0);

    // Здесь мы выполняем итерацию с помощью циклов for.
    for (mat4x4, 0..) |row, row_index| {
        for (row, 0..) |cell, column_index| {
            if (row_index == column_index) {
                try expect(cell == 1.0);
            }
        }
    }
}
```
```bash
$ zig test test_multidimensional_arrays.zig
1/1 test_multidimensional_arrays.test.multidimensional arrays...OK
All 1 tests passed.
```

#### Нуль терменированные массивы

Синтаксис `[N:x]T` описывает массив, который имеет контрольный элемент со значением `x` с индексом соответствующим
длине `N`.

```zig
const std = @import("std");
const expect = std.testing.expect;

test "0-terminated sentinel array" {
    const array = [_:0]u8{ 1, 2, 3, 4 };

    try expect(@TypeOf(array) == [4:0]u8);
    try expect(array.len == 4);
    try expect(array[4] == 0);
}

test "extra 0s in 0-terminated sentinel array" {
    // Контрольное значение может появиться раньше, но оно не влияет на время компиляции 'len'.
    const array = [_:0]u8{ 1, 0, 0, 4 };

    try expect(@TypeOf(array) == [4:0]u8);
    try expect(array.len == 4);
    try expect(array[4] == 0);
}
```
```bash
$ zig test test_null_terminated_array.zig
1/2 test_null_terminated_array.test.0-terminated sentinel array...OK
2/2 test_null_terminated_array.test.extra 0s in 0-terminated sentinel array...OK
All 2 tests passed.
```
------------
### Векторы

Вектор - это группа логических значений, целых чисел, чисел с плавающей точкой или указателей которые обрабатываются
параллельно, по возможности используя инструкции SIMD. Типы векторов создаются с помощью встроенной функции `@Vector`.

Векторы поддерживают те же встроенные операторы, что и их базовые типы. Эти операции выполняются поэлементно и
возвращают вектор той же длины, что и входные векторы. Это включает:
- Arithmetic (`+`, `-`, `/`, `*`, `@divFloor`, `@sqrt`, `@ceil`, `@log`, etc.)
- Bitwise operators (`>>`, `<<`, `&`, `|`, `~`, etc.)
- Comparison operators (`<`, `>`, `==`, etc.)

Запрещено использовать математический оператор для сочетания скаляров (отдельных чисел) и векторов. Zig предоставляет
встроенную функцию `@splat` для простого преобразования скаляров в векторы, а также поддерживает синтаксис `@reduce` и
индексации массива для преобразования векторов в скаляры. Векторы также поддерживают присваивание массивам фиксированной
длины и из них с известной во время вычисления длиной.

Для перестановки элементов внутри векторов и между ними **Zig** предоставляет функции `@shuffle` и `@select`.

Операции с векторами длина которых меньше чем исходный размер SIMD целевой машины обычно компилируются в одну
SIMD-инструкцию, в то время как векторы длина которых превышает исходный размер SIMD целевой машины компилируются в
несколько SIMD-инструкций. Если данная операция не поддерживает SIMD в целевой архитектуре компилятор по умолчанию
будет работать с каждым векторным элементом по очереди. **Zig** поддерживает любую известную во время вычислений длину
вектора вплоть до 2^32-1, хотя наиболее типичными являются малые степени двойки (2-64). Обратите внимание, что
чрезмерно большие длины векторов (например, 2^20) могут привести к сбоям компилятора в текущих версиях **Zig**.

```zig
const std = @import("std");
const expectEqual = std.testing.expectEqual;

test "Basic vector usage" {
    // Векторы имеют известную во время компиляции длину и базовый тип.
    const a = @Vector(4, i32){ 1, 2, 3, 4 };
    const b = @Vector(4, i32){ 5, 6, 7, 8 };

    // Математические операции выполняются поэтапно.
    const c = a + b;

    // Доступ к отдельным векторным элементам можно получить используя синтаксис индексации массива.
    try expectEqual(6, c[0]);
    try expectEqual(8, c[1]);
    try expectEqual(10, c[2]);
    try expectEqual(12, c[3]);
}

test "Conversion between vectors, arrays, and slices" {
    // Векторы и массивы фиксированной длины могут автоматически назначаться туда и обратно
    const arr1: [4]f32 = [_]f32{ 1.1, 3.2, 4.5, 5.6 };
    const vec: @Vector(4, f32) = arr1;
    const arr2: [4]f32 = vec;
    try expectEqual(arr1, arr2);

    // Вы также можете присвоить вектору срез с известной во время компиляции длиной используя .*
    const vec2: @Vector(2, f32) = arr1[1..3].*;

    const slice: []const f32 = &arr1;
    var offset: u32 = 1; // var, чтобы сделать его известным во время выполнения
    _ = &offset; // подавить ошибку "var никогда не мутирует"
    // Чтобы извлечь известную во время выполнения длину из известного во время выполнения смещения,
    // сначала извлеките новый фрагмент из начального смещения, а затем массив
    // известной во время компиляции длины
    const vec3: @Vector(2, f32) = slice[offset..][0..2].*;
    try expectEqual(slice[offset], vec2[0]);
    try expectEqual(slice[offset + 1], vec2[1]);
    try expectEqual(vec2, vec3);
}
```
```bash
$ zig test test_vector.zig
1/2 test_vector.test.Basic vector usage...OK
2/2 test_vector.test.Conversion between vectors, arrays, and slices...OK
All 2 tests passed.
```

TODO talk about C ABI interop<br>
TODO consider suggesting std.MultiArrayList

------------
### Указатели

В **Zig** есть два вида указателей: одноэлементные и многоэлементные:

- `*T` - одноэлементный указатель ровно на один элемент.
    - Поддерживает синтаксис разыменования: `ptr.*`
- `[*]T` - многоэлементный указатель на неизвестное количество элементов.
    - Поддерживает синтаксис индексациия: `ptr[i]`
    - Поддерживает синтаксис срезов: `ptr[start..end]` and `ptr[start..]`
    - Поддерживает арифметику указателей: `ptr + x`, `ptr - x`
    - `T` должен иметь известный размер, что означает, что он не может быть константным или каким-либо другим
    непрозрачным типом.

Эти типы тесно связаны с массивами и слайсами:
- `*[N]T` - указатель на N элементов, такой же как одноэлементный указатель на массив.
    - Поддерживает синтаксис индекса: `array_ptr[i]`
    - Поддерживает синтаксис срезов: `array_ptr[start..end]`
    - Поддерживает свойство длины: `array_ptr.len`

- `[]T` - это слайс (толстый указатель который содержит указатель типа `[*]T` и длину).
    - Поддерживает синтаксис индекса: `slice[i]`
    - Поддерживает синтаксис срезов: `slice[start..end]`
    - Поддерживает свойство длины: `slice.len`

Используйте `&x` для получения указателя на один элемент:

```zig
const expect = @import("std").testing.expect;

test "address of syntax" {
    // Получить адрес переменной:
    const x: i32 = 1234;
    const x_ptr = &x;

    // Разыменование указателя:
    try expect(x_ptr.* == 1234);

    // Когда вы получаете адрес переменной const, вы получаете указатель на один элемент const.
    try expect(@TypeOf(x_ptr) == *const i32);

    // Если вы хотите изменить значение, вам понадобится адрес изменяемой переменной:
    var y: i32 = 5678;
    const y_ptr = &y;
    try expect(@TypeOf(y_ptr) == *i32);
    y_ptr.* += 1;
    try expect(y_ptr.* == 5679);
}

test "pointer array access" {
    // Если взять адрес отдельного элемента, то получится
    // указатель на один элемент. Этот тип указателя
    // не поддерживает арифметику указателей.
    var array = [_]u8{ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
    const ptr = &array[2];
    try expect(@TypeOf(ptr) == *u8);

    try expect(array[2] == 3);
    ptr.* += 1;
    try expect(array[2] == 4);
}
```
```bash
$ zig test test_single_item_pointer.zig
1/2 test_single_item_pointer.test.address of syntax...OK
2/2 test_single_item_pointer.test.pointer array access...OK
All 2 tests passed.
```

**Zig** поддерживает арифметику с указателями. Лучше присвоить указателю значение `[*]T` и увеличить эту переменную.
Например, прямое увеличение указателя из среза приведет к его повреждению.

```zig
const expect = @import("std").testing.expect;

test "pointer arithmetic with many-item pointer" {
    const array = [_]i32{ 1, 2, 3, 4 };
    var ptr: [*]const i32 = &array;

    try expect(ptr[0] == 1);
    ptr += 1;
    try expect(ptr[0] == 2);

    // разбиение указателя на множество элементов без указания конца эквивалентно
    // арифметике указателя: `ptr[start..] == ptr + start`
    try expect(ptr[1..] == ptr + 1);
}

test "pointer arithmetic with slices" {
    var array = [_]i32{ 1, 2, 3, 4 };
    var length: usize = 0; // var, чтобы он был известен во время выполнения
    _ = &length; // подавить ошибку "var не был изменён"
    var slice = array[length..array.len];

    try expect(slice[0] == 1);
    try expect(slice.len == 4);

    slice.ptr += 1;
    // теперь срез находится в неконсистентном состоянии, так как len не обновлялся

    try expect(slice[0] == 2);
    try expect(slice.len == 4);
}
```
```bash
$ zig test test_pointer_arithmetic.zig
1/2 test_pointer_arithmetic.test.pointer arithmetic with many-item pointer...OK
2/2 test_pointer_arithmetic.test.pointer arithmetic with slices...OK
All 2 tests passed.
```

 В **Zig** мы как правило предпочитаем срезы, а не указатели заканчивающиеся кардиналом. Вы можете преобразовать
 массив или указатель в срез используя синтаксис срезов.

Срезы имеют проверку границ и следовательно защищены от такого рода неопределенного поведения. Это одна из причин по
которой мы предпочитаем срезы указателям.

```zig
const expect = @import("std").testing.expect;

test "pointer slicing" {
    var array = [_]u8{ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
    var start: usize = 2; // var, чтобы он был известен во время выполнения
    _ = &start; // подавить ошибку "var не был изменён"
    const slice = array[start..4];
    try expect(slice.len == 2);

    try expect(array[3] == 4);
    slice[1] += 1;
    try expect(array[3] == 5);
}
```
```bash
$ zig test test_slice_bounds.zig
1/1 test_slice_bounds.test.pointer slicing...OK
All 1 tests passed.
```

Указатели также работают во время компиляции если только код не зависит от неопределенного расположения памяти:
```zig
const expect = @import("std").testing.expect;

test "comptime pointers" {
    comptime {
        var x: i32 = 1;
        const ptr = &x;
        ptr.* += 1;
        x += 1;
        try expect(ptr.* == 3);
    }
}
```
```bash
$ zig test test_comptime_pointers.zig
1/1 test_comptime_pointers.test.comptime pointers...OK
All 1 tests passed.
```

Чтобы преобразовать целочисленный адрес в указатель используйте `@ptrFromInt`. Чтобы преобразовать указатель в целое
число используйте `@intFromPtr`:

```zig
const expect = @import("std").testing.expect;

test "@intFromPtr and @ptrFromInt" {
    const ptr: *i32 = @ptrFromInt(0xdeadbee0);
    const addr = @intFromPtr(ptr);
    try expect(@TypeOf(addr) == usize);
    try expect(addr == 0xdeadbee0);
}
```
```bash
$ zig test test_integer_pointer_conversion.zig
1/1 test_integer_pointer_conversion.test.@intFromPtr and @ptrFromInt...OK
All 1 tests passed.
```

**Zig** способен сохранять адреса памяти в comptime коде до тех пор пока указатель никогда не будет разыменован:
```zig
const expect = @import("std").testing.expect;

test "comptime @ptrFromInt" {
    comptime {
        // Zig может делать это во время компиляции, при условии, что
        // ptr никогда не разыменовывается.
        const ptr: *i32 = @ptrFromInt(0xdeadbee0);
        const addr = @intFromPtr(ptr);
        try expect(@TypeOf(addr) == usize);
        try expect(addr == 0xdeadbee0);
    }
}
```
```bash
$ zig test test_comptime_pointer_conversion.zig
1/1 test_comptime_pointer_conversion.test.comptime @ptrFromInt...OK
All 1 tests passed.
```

#### volatile

Предполагается, что загрузка и сохранение данных не имеют побочных эффектов. Если данная загрузка или сохранение данных
должны иметь побочные эффекты такие как ввод/вывод с отображением памяти (MMIO) используйте `volatile`. В следующем коде
гарантированно все загрузки и сохранения с помощью `mmio_ptr` выполняются в том же порядке, что и в исходном коде:

```zig
const expect = @import("std").testing.expect;

test "volatile" {
    const mmio_ptr: *volatile u8 = @ptrFromInt(0x12345678);
    try expect(@TypeOf(mmio_ptr) == *volatile u8);
}
```
```bash
$ zig test test_volatile.zig
1/1 test_volatile.test.volatile...OK
All 1 tests passed.
```

Обратите внимание, что `volatile` не связан с параллелизмом и атомарностью. Если вы видите код который использует
`volatile` для чего-то другого кроме ввода/вывода с отображением в памяти, то это вероятно ошибка.

`@ptrCast` преобразует тип элемента указателя в другой. Это создает новый указатель, который может вызвать неопределяемое
незаконное поведение в зависимости от загружаемых данных и хранилищ, которые проходят через него. Как правило, другие
виды преобразования типов предпочтительнее чем `@ptrCast`, если это возможно.

```zig
const std = @import("std");
const expect = std.testing.expect;

test "pointer casting" {
    const bytes align(@alignOf(u32)) = [_]u8{ 0x12, 0x12, 0x12, 0x12 };
    const u32_ptr: *const u32 = @ptrCast(&bytes);
    try expect(u32_ptr.* == 0x12121212);

    // Даже этот пример является надуманным - есть способы сделать это лучше, чем
    // приведение указателя. Например, используя приведение с сужением слайса:
    const u32_value = std.mem.bytesAsSlice(u32, bytes[0..])[0];
    try expect(u32_value == 0x12121212);

    // И даже другой способ, самый простой способ сделать это:
    try expect(@as(u32, @bitCast(bytes)) == 0x12121212);
}

test "pointer child type" {
    // типы указателей имеют `дочернее` поле, в котором указывается тип, на который они указывают.
    try expect(@typeInfo(*u32).Pointer.child == u32);
}
```
```bash
$ zig test test_pointer_casting.zig
1/2 test_pointer_casting.test.pointer casting...OK
2/2 test_pointer_casting.test.pointer child type...OK
All 2 tests passed.
```

#### Выравнивание

У каждого типа есть **alignment** - количество байт, так что, когда значение этого типа загружается из памяти или
сохраняется в ней, адрес памяти должен быть равномерно кратен этому числу. Вы можете использовать `@alignOf`, чтобы
узнать это значение для любого типа.

Выравнивание зависит от архитектуры процессора, но всегда имеет степень двойки и меньше `1 << 29`.

В **Zig** тип указателя имеет значение выравнивание. Если значение равно выравниванию базового типа, его можно не
указывать в типе:

```zig
const std = @import("std");
const builtin = @import("builtin");
const expect = std.testing.expect;

test "variable alignment" {
    var x: i32 = 1234;
    const align_of_i32 = @alignOf(@TypeOf(x));
    try expect(@TypeOf(&x) == *i32);
    try expect(*i32 == *align(align_of_i32) i32);
    if (builtin.target.cpu.arch == .x86_64) {
        try expect(@typeInfo(*i32).Pointer.alignment == 4);
    }
}
```
```bash
$ zig test test_variable_alignment.zig
1/1 test_variable_alignment.test.variable alignment...OK
All 1 tests passed.
```

Точно так же как a `*i32` может быть преобразован в `*const i32`, указатель с большим выравниванием может быть неявно
преобразован в указатель с меньшим выравниванием, но не наоборот.

Вы можете указать выравнивание для переменных и функций. Если вы сделаете это, то указатели на них получат указанное
выравнивание:

```zig
const expect = @import("std").testing.expect;

var foo: u8 align(4) = 100;

test "global variable alignment" {
    try expect(@typeInfo(@TypeOf(&foo)).Pointer.alignment == 4);
    try expect(@TypeOf(&foo) == *align(4) u8);
    const as_pointer_to_array: *align(4) [1]u8 = &foo;
    const as_slice: []align(4) u8 = as_pointer_to_array;
    const as_unaligned_slice: []u8 = as_slice;
    try expect(as_unaligned_slice[0] == 100);
}

fn derp() align(@sizeOf(usize) * 2) i32 {
    return 1234;
}
fn noop1() align(1) void {}
fn noop4() align(4) void {}

test "function alignment" {
    try expect(derp() == 1234);
    try expect(@TypeOf(derp) == fn () i32);
    try expect(@TypeOf(&derp) == *align(@sizeOf(usize) * 2) const fn () i32);

    noop1();
    try expect(@TypeOf(noop1) == fn () void);
    try expect(@TypeOf(&noop1) == *align(1) const fn () void);

    noop4();
    try expect(@TypeOf(noop4) == fn () void);
    try expect(@TypeOf(&noop4) == *align(4) const fn () void);
}
```
```bash
$ zig test test_variable_func_alignment.zig
1/2 test_variable_func_alignment.test.global variable alignment...OK
2/2 test_variable_func_alignment.test.function alignment...OK
All 2 tests passed.
```

Если у вас есть указатель или слайс который имеет небольшое выравнивание, но вы знаете, что на самом деле он имеет
большее выравнивание используйте `@alignCast` чтобы изменить указатель на более выровненный. Во время выполнения это не
требуется, но вставляется проверка безопасности:

```zig
const std = @import("std");

test "pointer alignment safety" {
    var array align(4) = [_]u32{ 0x11111111, 0x11111111 };
    const bytes = std.mem.sliceAsBytes(array[0..]);
    try std.testing.expect(foo(bytes) == 0x11111111);
}
fn foo(bytes: []u8) u32 {
    const slice4 = bytes[1..5];
    const int_slice = std.mem.bytesAsSlice(u32, @as([]align(4) u8, @alignCast(slice4)));
    return int_slice[0];
}
```
```bash
$ zig test test_incorrect_pointer_alignment.zig
1/1 test_incorrect_pointer_alignment.test.pointer alignment safety...thread 3568823 panic: incorrect alignment
/home/andy/src/zig/doc/langref/test_incorrect_pointer_alignment.zig:10:68: 0x103d13a in foo (test)
    const int_slice = std.mem.bytesAsSlice(u32, @as([]align(4) u8, @alignCast(slice4)));
                                                                   ^
/home/andy/src/zig/doc/langref/test_incorrect_pointer_alignment.zig:6:31: 0x103cfd7 in test.pointer alignment safety (test)
    try std.testing.expect(foo(bytes) == 0x11111111);
                              ^
/home/andy/src/zig/lib/compiler/test_runner.zig:157:25: 0x1047f10 in mainTerminal (test)
        if (test_fn.func()) |_| {
                        ^
/home/andy/src/zig/lib/compiler/test_runner.zig:37:28: 0x103e28b in main (test)
        return mainTerminal();
                           ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x103d609 in posixCallMainAndExit (test)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x103d171 in _start (test)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
error: the following test command crashed:
/home/andy/src/zig/.zig-cache/o/c771705677c0d2df24e00269a9189f97/test

```

#### allowzero

Этот атрибут pointer позволяет указателю иметь нулевой адрес. Это необходимо только для автономной целевой системы OS,
где нулевой адрес может быть отображен. Если вы хотите представить нулевые указатели используйте необязательные
указатели. Необязательные указатели с значением `allowzero` не имеют такого же размера как указатели на объекты. В этом
примере кода, если бы указатель не имел атрибута `allowzero`, это означало бы недопустимое приведение указателя к
нулевому значению.:

```zig
const std = @import("std");
const expect = std.testing.expect;

test "allowzero" {
    var zero: usize = 0; // значение параметра должно быть известно во время выполнения
    _ = &zero; // подавить ошибку "var не был изменён"
    const ptr: *allowzero i32 = @ptrFromInt(zero);
    try expect(@intFromPtr(ptr) == 0);
}
```
```bash
$ zig test test_allowzero.zig
1/1 test_allowzero.test.allowzero...OK
All 1 tests passed.
```

#### Указатели с контрольным элементом

Синтаксис `[*:x]T` описывает указатель длина которого определяется контрольным элементом. Это обеспечивает защиту от
переполнения буфера и повторным чтением.

```zig
const std = @import("std");

// Это также доступно как `std.c.printf`.
pub extern "c" fn printf(format: [*:0]const u8, ...) c_int;

pub fn main() anyerror!void {
    _ = printf("Hello, world!\n"); // OK

    const msg = "Hello, world!\n";
    const non_null_terminated_msg: [msg.len]u8 = msg.*;
    _ = printf(&non_null_terminated_msg);
}
```
```bash
$ zig build-exe sentinel-terminated_pointer.zig -lc
/home/andy/src/zig/doc/langref/sentinel-terminated_pointer.zig:11:16: error: expected type '[*:0]const u8', found '*const [14]u8'
    _ = printf(&non_null_terminated_msg);
               ^~~~~~~~~~~~~~~~~~~~~~~~
/home/andy/src/zig/doc/langref/sentinel-terminated_pointer.zig:11:16: note: destination pointer requires '0' sentinel
/home/andy/src/zig/doc/langref/sentinel-terminated_pointer.zig:4:35: note: parameter type declared here
pub extern "c" fn printf(format: [*:0]const u8, ...) c_int;
                                 ~^~~~~~~~~~~~
referenced by:
    callMain: /home/andy/src/zig/lib/std/start.zig:524:32
    callMainWithArgs: /home/andy/src/zig/lib/std/start.zig:482:12
    remaining reference traces hidden; use '-freference-trace' to see all reference traces

```

------------
### Срезы

Срез - это указатель и длина. Разница между массивом и срезом заключается в том, что длина массива является частью типа
и известна во время компиляции, тогда как длина среза известна во время выполнения. Для доступа к обоим параметрам
используется поле `len`.

```zig
const expect = @import("std").testing.expect;
const expectEqualSlices = @import("std").testing.expectEqualSlices;

test "basic slices" {
    var array = [_]i32{ 1, 2, 3, 4 };
    var known_at_runtime_zero: usize = 0;
    _ = &known_at_runtime_zero;
    const slice = array[known_at_runtime_zero..array.len];

    // альтернативная инициализация с использованием местоположения результата
    const alt_slice: []const i32 = &.{ 1, 2, 3, 4 };

    try expectEqualSlices(i32, slice, alt_slice);

    try expect(@TypeOf(slice) == []i32);
    try expect(&slice[0] == &array[0]);
    try expect(slice.len == array.len);

    // Если выполнить срез с известными начальными и конечными позициями, то результатом будет
    // указатель на массив, а не на срез.
    const array_ptr = array[0..array.len];
    try expect(@TypeOf(array_ptr) == *[array.len]i32);

    // Вы можете выполнить срез по длине выполнив срез дважды. Это позволяет компилятору
    // выполнить некоторые оптимизации например, распознать длину известную во время компиляции, когда
    // начальная позиция известна только во время выполнения.
    var runtime_start: usize = 1;
    _ = &runtime_start;
    const length = 2;
    const array_ptr_len = array[runtime_start..][0..length];
    try expect(@TypeOf(array_ptr_len) == *[length]i32);

    // Использование оператора address-of для среза дает указатель на один элемент.
    try expect(@TypeOf(&slice[0]) == *i32);
    // Использование поля `ptr` дает указатель на множество элементов.
    try expect(@TypeOf(slice.ptr) == [*]i32);
    try expect(@intFromPtr(slice.ptr) == @intFromPtr(&slice[0]));

    // Срезы проверяют границы массива.
    // Если вы попытаетесь получить доступ к чему-либо за пределами вы получите сбой проверки безопасности:
    slice[10] += 1;

    // Обратите внимание, что `slice.ptr` не вызывает проверку безопасности, в то время как `&slice[0]`
    // утверждает, что срез имеет len > 0.
}
```
```bash
$ zig test test_basic_slices.zig
1/1 test_basic_slices.test.basic slices...thread 3571722 panic: index out of bounds: index 10, len 4
/home/andy/src/zig/doc/langref/test_basic_slices.zig:41:10: 0x103f955 in test.basic slices (test)
    slice[10] += 1;
         ^
/home/andy/src/zig/lib/compiler/test_runner.zig:157:25: 0x104c800 in mainTerminal (test)
        if (test_fn.func()) |_| {
                        ^
/home/andy/src/zig/lib/compiler/test_runner.zig:37:28: 0x104210b in main (test)
        return mainTerminal();
                           ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x103fe49 in posixCallMainAndExit (test)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x103f9b1 in _start (test)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
error: the following test command crashed:
/home/andy/src/zig/.zig-cache/o/3c391d75c939ce98356b98dd812503b1/test
```

Это одна из причин по которой мы предпочитаем срезы указателям.

```zig
const std = @import("std");
const expect = std.testing.expect;
const mem = std.mem;
const fmt = std.fmt;

test "using slices for strings" {
    // Zig не имеет понятия о строках. Строковые литералы - это контантные указатели
    // на массивы u8 заканчивающиеся нулем, и по соглашению параметры
    // которые являются "строками", должны быть срезами u8 в кодировке UTF-8.
    // Здесь мы преобразуем *const [5:0]u8 и *const [6:0]u8 в []const u8
    const hello: []const u8 = "hello";
    const world: []const u8 = "世界";

    var all_together: [100]u8 = undefined;
    // Вы можете использовать синтаксис срезов содержащий по крайней мере один известный во время выполнения индекс в массиве
    // чтобы преобразовать массив в срез.
    var start: usize = 0;
    _ = &start;
    const all_together_slice = all_together[start..];
    // Пример объединения строк.
    const hello_world = try fmt.bufPrint(all_together_slice, "{s} {s}", .{ hello, world });

    // Как правило вы можете использовать UTF-8 и не беспокоиться о том, является ли что-либо строкой.
    // Если вам не нужно иметь дело с отдельными символами, нет необходимости в декодировании.
    try expect(mem.eql(u8, hello_world, "hello 世界"));
}

test "slice pointer" {
    var array: [10]u8 = undefined;
    const ptr = &array;
    try expect(@TypeOf(ptr) == *[10]u8);

    // Указатель на массив может быть разделен точно так же, как и массив:
    var start: usize = 0;
    var end: usize = 5;
    _ = .{ &start, &end };
    const slice = ptr[start..end];
    // Среза является изменяемым потому что мы вырезали изменяемый указатель.
    try expect(@TypeOf(slice) == []u8);
    slice[2] = 3;
    try expect(array[2] == 3);

    // Опять же, срез с использованием известных в comptime индексов приведет к созданию другого указателя
    // на массив:
    const ptr2 = slice[2..3];
    try expect(ptr2.len == 1);
    try expect(ptr2[0] == 3);
    try expect(@TypeOf(ptr2) == *[1]u8);
}
```
```bash
$ zig test test_slices.zig
1/2 test_slices.test.using slices for strings...OK
2/2 test_slices.test.slice pointer...OK
All 2 tests passed.
```

#### Срезы с контрольным элементом

Синтаксис `[:x]T` - это срез, длина которого известна во время выполнения, а также гарантирует наличие контрольного
элемента индексированного по длине. Тип не гарантирует, что до этого не было контрольных элементов. Срезы завершенные
кардиналом предоставляют элементу доступ к индексу `len`.

```zig
const std = @import("std");
const expect = std.testing.expect;

test "0-terminated slice" {
    const slice: [:0]const u8 = "hello";

    try expect(slice.len == 5);
    try expect(slice[5] == 0);
}
```
```bash
$ zig test test_null_terminated_slice.zig
1/1 test_null_terminated_slice.test.0-terminated slice...OK
All 1 tests passed.
```

Срезы заканчивающиеся контрольным элементом также могут быть созданы с использованием вариации синтаксиса среза
`data[start..end :x]`, где данные представляют собой указатель на множество элементов, массив или срез, а `x` - значение
контрольного элемента.

```zig
const std = @import("std");
const expect = std.testing.expect;

test "0-terminated slicing" {
    var array = [_]u8{ 3, 2, 1, 0, 3, 2, 1, 0 };
    var runtime_length: usize = 3;
    _ = &runtime_length;
    const slice = array[0..runtime_length :0];

    try expect(@TypeOf(slice) == [:0]u8);
    try expect(slice.len == 3);
}
```
```bash
$ zig test test_null_terminated_slicing.zig
1/1 test_null_terminated_slicing.test.0-terminated slicing...OK
All 1 tests passed.
```

Для среза завершенного с помощью контрольного элемента, утверждается, что элемент в позиции кардинала в исходных данных
на самом деле является значением элемента. Если это не так, возникает неопределенное поведение защищенное с помощью
safety.

```zig
const std = @import("std");
const expect = std.testing.expect;

test "sentinel mismatch" {
    var array = [_]u8{ 3, 2, 1, 0 };

    // Создание среза заканчивающегося контрольным элементом из массива длиной 2
    // приведет к тому, что значение `1` займет позицию этого элемента.
    // Это не соответствует указанному значению элемента равное `0`, что приведет
    // к панике во время выполнения.
    var runtime_length: usize = 2;
    _ = &runtime_length;
    const slice = array[0..runtime_length :0];

    _ = slice;
}
```
```bash
$ zig test test_sentinel_mismatch.zig
1/1 test_sentinel_mismatch.test.sentinel mismatch...thread 3579807 panic: sentinel mismatch: expected 0, found 1
/home/andy/src/zig/doc/langref/test_sentinel_mismatch.zig:13:24: 0x103cf16 in test.sentinel mismatch (test)
    const slice = array[0..runtime_length :0];
                       ^
/home/andy/src/zig/lib/compiler/test_runner.zig:157:25: 0x1048aa0 in mainTerminal (test)
        if (test_fn.func()) |_| {
                        ^
/home/andy/src/zig/lib/compiler/test_runner.zig:37:28: 0x103eabb in main (test)
        return mainTerminal();
                           ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x103d4f9 in posixCallMainAndExit (test)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x103d061 in _start (test)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
error: the following test command crashed:
/home/andy/src/zig/.zig-cache/o/2af8da0d34d396fbb50fa515cef10c72/test
```

------------
