### Introduction

[Zig](https://ziglang.org) - это системный язык программирования общего назначения и набор инструментов для поддержки
надежного, оптимального и многократно используемого программного обеспечения.

- Надёжность
    - Поведение корректно даже в крайних случаях, таких как нехватка памяти.
- Оптимальность
    - Пишите программы так чтобы они могли работать наилучшим образом.
- Переиспользуемость
    - Один и тот же код работает во многих средах, которые имеют различные ограничениями.
- Поддерживаемость
    - Точная передача намерений компилятору и другим программистам.
    - Этот язык не требует больших затрат на чтение кода и устойчив к меняющимся требованиям и средам.

Часто самый эффективный способ узнать что-то новое - это посмотреть примеры, поэтому в этой документации показано, как
использовать каждую из функций Zig. Все это находится на одной странице, поэтому вы можете искать информацию с помощью
поискового инструмента вашего браузера.

Примеры кода в этом документе скомпилированы и протестированы в рамках основного набора тестов Zig.

Этот HTML-документ не зависит от внешних файлов, поэтому вы можете использовать его в автономном режиме.


------------
### Zig Standard Library

[Стандартная библиотека Zig](https://ziglang.org/documentation/master/std/) имеет собственную
документацию.

Стандартная библиотека Zig содержит часто используемые алгоритмы, структуры данных и определения, которые помогут вам
создавать программы или библиотеки. В этой документации вы увидите множество примеров стандартной библиотеки Zig,
используемых в этой документации. Чтобы узнать больше о стандартной библиотеке Zig, перейдите по ссылке выше.


------------
### Hello, World!

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

В большинстве случаев более целесообразно выполнять запись в `stderr`, а не в `stdout`, и не имеет значения будет ли
сообщение успешно записано в поток. Для этого общего случая существует более простой API:

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

В этом случае символ `!` может быть не указан в возвращаемом типе, поскольку функция не возвращает ошибок.

Смотрите также:
- [Values](#values)
- [@import](#import)
- [Errors](#errors)
- [Root Source File](#root-source-file)
- [Source Encoding](#source-encoding)


------------
### Comments

Zig поддерживает 3 типа комментариев. Обычные комментарии игнорируются, но [комментарии к документу](#doc-comments) и
[комментарии к документу верхнего уровня](#top-level-doc-comments) используются компилятором для создания документации
пакета.

Созданная документация все еще является экспериментальной и может быть создана с помощью:

```bash
$ zig test -femit-docs main.zig
```
```zig
const print = @import("std").debug.print;

pub fn main() !void {
    // Комментарии в Zig начинаются с "//" и заканчиваются на следующем LF-байте (конце строки).
    // Строка ниже является комментарием и не будет выполнена.

    //print("Hello?", .{});
    print("Hello, World!\n", .{});
}
```
```bash
$ zig build-exe comments.zig
$ ./comments
Hello, World!
```

В Zig нет многострочных комментариев (например, таких как `/* */` в C). Это позволяет Zig обладать тем свойством, что
каждая строка кода может быть токенизирована вне контекста.


#### Doc Comments

Комментарий к документу начинаются ровно с трех косых черт (т.е. `///`, но не с `////`); несколько комментариев к
документам подряд объединяются в многострочный комментарий к документу. Комментарий к документу документирует все, что
следует непосредственно за ним.

```zig
/// Структура для хранения временной метки с точностью до наносекунды (это
/// многострочный комментарий к документу).
const Timestamp = struct {
    /// Количество секунд, прошедших с момента начала эпохи (это также комментарий к документу).
    seconds: i64, // подписано, чтобы мы могли представить период до 1970 года (не комментарий к документу)
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
Комментарии к документу разрешены только в определенных местах; ошибка компиляции заключается в том, что комментарий к
документу находится в неожиданном месте, например, в середине выражения или непосредственно перед не "комментарием к
документу".

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

Комментарии к документу могут чередоваться с обычными комментариями. В настоящее время при создании документации пакета
обычные комментарии объединяются с комментариями к документу.


#### Top-Level Doc Comments

Комментарий к документу верхнего уровня начинается с двух косых черт и восклицательного знака: `//!`; он документирует
текущий модуль. Ошибка компиляции возникает, если комментарий к документу верхнего уровня не помещается в начало
[контейнера](#containers) перед какими-либо выражениями.

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
### Values

```zig
// Объявления верхнего уровня не зависят от порядка:
const print = std.debug.print;
const std = @import("std");
const os = std.os;
const assert = std.debug.assert;

pub fn main() !void {
    // целые числа
    const one_plus_one: i32 = 1 + 1;
    print("1 + 1 = {d}\n", .{one_plus_one});

    // числа с плавающей запятой
    const seven_div_three: f32 = 7.0 / 3.0;
    print("7.0 / 3.0 = {any}\n", .{seven_div_three});

    // логические значения
    print("{any}\n{any}\n{any}\n\n", .{
        true and fsalse,
        true or false,
        !true,
    });

    // опциональные значения
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

    // объединения с ошибками
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


#### Primitive Types

**<p align="center">Примитивные типы</p>**

|Тип|Эквивалент C|Описание|
|---|---|---|
|`i8`|`int8_t`|8-разрядное целое число со знаком|
|`u8`|`uint8_t`|8-разрядное целое число без знака|
|`i16`|`int16_t`|16-разрядное целое число со знаком|
|`u16`|`uint16_t`|16-разрядное целое число без знака|
|`i32`|`int32_t`|32-разрядное целое число со знаком|
|`u32`|`uint32_t`|32-разрядное целое число без знака|
|`i64`|`int64_t`|64-разрядное целое число со знаком|
|`u64`|`uint64_t`| 64-разрядное целое число без знака|
|`i128`|`__int128`|128-разрядное целое число со знаком|
|`u128`|`unsigned __int128`|128-разрядное целое число без знака|
|`isize`|`intptr_t`|целое число размером со знаковый указатель|
|`usize`|`uintptr_t`, `size_t`|целое число размером с указатель без знака. Смотрите также [#5185](https://github.com/ziglang/zig/issues/5185)|
|`c_char`|`char`|для обеспечения совместимости ABI с C|
|`c_short`|`short`|для обеспечения совместимости ABI с C|
|`c_ushort`|`unsigned short`|для обеспечения совместимости ABI с C|
|`c_int`|`int`|для обеспечения совместимости ABI с C|
|`c_uint`|`unsigned int`|для обеспечения совместимости ABI с C|
|`c_long`|`long`|для обеспечения совместимости ABI с C|
|`c_ulong`|`unsigned long`|для обеспечения совместимости ABI с C|
|`c_longlong`|`long long`|для обеспечения совместимости ABI с C|
|`c_ulonglong`|`unsigned long long`|для обеспечения совместимости ABI с C|
|`c_longdouble`|`long double`|для обеспечения совместимости ABI с C|
|`f16`|`_Float16`|16-разрядное число с плавающей запятой (10-разрядная мантисса) стандарта IEEE-754-200816|
|`f32`|`float`|32-разрядное число с плавающей запятой (23-разрядная мантисса) стандарта IEEE-754-200832|
|`f64`|`double`|64-разрядное число с плавающей запятой (52-разрядная мантисса) IEEE-754-200864|
|`f80`|`double`|80-разрядное число с плавающей запятой (64-разрядная мантисса) стандарта IEEE-754-2008 80- повышенная точность в битах|
|`f128`|`_Float128`|128-разрядное число с плавающей запятой (112-разрядная мантисса) стандарта IEEE-754-2008128|
|`bool`|`bool`|`true` или `false`|
|`anytype`|`void`|Используется для указателей со стиранием типа.|
|`void`|`(none)`|Всегда значение `void{}`|
|`noreturn`|`(none)`|тип `break`, `continue`, `return`, `unreachable` и `while (true) {}`|
|`type`|`(none)`|тип типов|
|`anyerror`|`(none)`|код ошибки|
|`comptime_int`|`(none)`|Допускается только для значений известных в [comptime](#comptime). Тип целочисленных литералов.|
|`comptime_float`|`(none)`|Допускается только для значений известных в [comptime](#comptime). Тип литералов с плавающей запятой.|

В дополнение к целочисленным типам указанным выше, можно использовать ссылки на целые числа произвольной разрядности,
используя идентификатор `i` или `u` за которым следуют цифры. Например, идентификатор `i7` относится к 7-разрядному
целому числу со знаком. Максимально допустимая разрядность для целочисленного типа равна `65535`.

Смотрите также:
- [Integers](#integers)
- [Floats](#floats)
- [void](#void)
- [Errors](#errors)
- [@Type](#type)


**<p align="center">Примитивные значения</p>**

|Имя|Описание|
|---|---|
|`true` and `false`|`bool` значения|
|`null`|используется для установки опционального типа в `null`|
|`undefined`|используется для того чтобы оставить значение неопределенным|

Смотрите также:
- [Optionals](#optionals)
- [undefined](#undefined)


#### String Literals and Unicode Code Point Literals

Строковые литералы - это константные одноэлементные [указатели](#pointers) на нуль-терминированные байтовые массивы. Тип
строковых литералов кодирует как длину так и тот факт, что они являются нуль-терменированными и поэтому они могут быть
[преобразованы](#type-coercion) как в [срезы](#slices), так и в [указатели с нуль-терменированным
значением](#sentinel-terminated-pointers). Разыменование строковых литералов преобразует их в массивы.

Поскольку исходный код Zig [закодирован в UTF-8](#source-encoding), любые байты, отличающиеся от ASCII появляющиеся в
строковом литерале в исходном коде передают свое значение в формате UTF-8 в содержимое строки в программе Zig; байты не
изменяются компилятором. Можно вставить байты отличные от UTF-8 в строковый литерал, используя обозначение `\xNN`.

При индексировании строки содержащей байты отличные от ASCII, возвращаются отдельные байты независимо от того, является
ли допустимым UTF-8 или нет.

Литералы с кодовой точкой в Юникоде имеют тип `comptime_int`, такой же как и [целочисленные
литералы](#integer-literals). Все [управляющие последовательности](#escape-sequences) допустимы как в строковых
литералах, так и в литералах с кодовой точкой в Юникоде.

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

    const invalid_utf8 = "\xff\xfe"; // строки, отличные от UTF-8, могут быть записаны в формате \xNN.
    print("0x{x}\n", .{invalid_utf8[1]}); // при их индексации возвращаются отдельные байты...
    print("0x{x}\n", .{"💯"[1]}); // ...как и частичная индексация с помощью символов, отличных от ASCII
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

Смотрите также:
- [Arrays](#arrays)
- [Source Encoding](#source-encoding)


##### Escape Sequences

**<p align="center">Управляющие последовательности</p>**

|Управляющая последовательность|Имя|
|---|---|
|`\n`|Новая строка|
|`\r`|Возврат каретки|
|`\t`|Табуляция|
|`\\`|Обратная косая черта|
|`\'`|Одинарная кавычка|
|`\"`|Двойная кавычка|
|`\xNN`|шестнадцатеричное 7-разрядное байтовое значение (2 цифры)|
|`\u{NNNNNN}`|шестнадцатеричный код Юникода в кодировке UTF-8 (1 или более цифр)|

Обратите внимание, что максимально допустимая точка в Юникоде равна `0x10ffff`.


##### Multiline String Literals

Многострочные строковые литералы не имеют экранирующих элементов и могут занимать несколько строк. Чтобы начать
многострочный строковый литерал используйте символ `\\`. Как и в случае с комментарием, строковый литерал продолжается
до конца строки. Конец строки не включается в строковый литерал. Однако, если следующая строка начинается с `\\`, то
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

Смотрите также:
- [@embedFile](#embedfile)


#### Assignment

Используйте ключевое слово `const` для присваивания значения идентификатору:

```zig
const x = 1234;

fn foo() void {
    // Он работает как в области видимости файла, так и внутри функций.
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

Значение `const` применяется ко всем байтам к которым непосредственно обращается идентификатор. [Указатели](#pointers)
имеют свою собственную константу.

Если вам нужна переменная которую вы хотите изменять, используйте ключевое слово `var`:

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


##### undefined

Используйте `undefined` чтобы оставить переменные неинициализированными:

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

Значение `undefined` может быть [преобразовано](#type-coercion) к любому типу. Как только это произойдет уже невозможно
будет определить, что значение не определено. Значение `undefined` может быть любым, даже бессмысленным в соответствии с
типом. В переводе на английский `undefined` означает "Не имеющее смысла значение". Использование этого значения может
привести к багам. Значение будет неиспользуемым или перезаписанным перед использованием.

В режиме [отладки](#debug) Zig записывает `0xaa` байт в неопределенную память. Это делается для раннего выявления ошибок
и помогает обнаружить использование неопределенной памяти в отладчике. Однако такое поведение является только
особенностью реализации, а не семантикой языка, поэтому не гарантируется, что оно будет наблюдаться в коде.


------------
### Zig Test

Код написанный в рамках одного или нескольких `test` объявлений, может быть использован для проверки соответствия
поведения ожиданиям:

```zig
const std = @import("std");

test "expect addOne adds one to 41" {
    // Стандартная библиотека содержит полезные функции помогающие создавать тесты.
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

Пример кода `testing_introduction.zig` проверяет [функцию](#functions) `addOne`, чтобы убедиться, что она возвращает
значение `42` при входных данных `41`. С точки зрения этого теста, функция `addOne` является _тестируемым кодом_.

`zig test` - это инструмент, который создает и запускает тестовую сборку. По умолчанию он создает и запускает
исполняемую программу, используя средство запуска тестов по умолчанию предоставляемое [стандартной библиотекой
Zig](https://ziglang.org/documentation/0.13.0/#Zig-Standard-Library) в качестве основной точки входа. Во время сборки
объявления `test` найденные при [разрешении](#root-source-file) данного исходного файла Zig, включаются в программу
тестирования по умолчанию для запуска и составления отчета.

>В этой документации рассматриваются возможности программы запуска тестов по умолчанию, предоставляемой стандартной
>библиотекой Zig. Ее исходный код находится в `lib/test_runner.zig`.

В выводе командной строки показанном выше, после команды `zig test` отображаются две строки. Программа тестирования по
умолчанию выводит эти строки со стандартной ошибкой:<br>
**<br>`1/2 testing_introduction.test.expect addOne adds one to 41...`<br>**
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Строки, подобные этой, указывают, какой тест из
общего числа тестов выполняется. В данном случае `1/2` означает, что выполняется первый тест из общего числа двух
тестов. Обратите внимание, что когда в терминал выводится стандартная ошибка программы test runner, эти строки очищаются
при успешном завершении теста.
**<br>`2/2 testing_introduction.decltest.addOne...`<br>**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Когда имя теста является идентификатором, программа
тестирования по умолчанию использует текст decltest вместо test. **<br>`All 2 tests passed.`<br>**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;В этой строке указано общее количество пройденных
тестов.


#### Test Declarations

Объявления тестов содержат [ключевое слово](#keyword-reference) `test`, за которым следует опциональное имя, записанное
в виде [строкового литерала](#string-literals) или [идентификатора](#identifiers), а затем [блок](#blocks), содержащий
любой допустимый Zig-код, который разрешен в [функции](#functions).

Тестовые блоки без имени всегда выполняются во время сборки тестов и не подлежат [пропуску тестов](#skip-tests).

Объявления тестов похожи на [функции](#functions): у них есть тип возвращаемого значения и блок кода. Неявный тип
возвращаемого значения `test` - это [тип объединения ошибок](#error-union-type) `anyerror!void` и его нельзя изменить.
Если исходный файл Zig не создается с помощью `zig test tool`, объявления тестов не включаются в сборку.

Объявления тестов могут быть записаны в том же файле в котором записан тестируемый код или в отдельном исходном файле
Zig. Поскольку тестовые объявления являются объявлениями верхнего уровня они не зависят от порядка выполнения и могут
быть написаны до или после тестируемого кода.

Смотрите также:
- [The Global Error Set](#the-global-error-set)
- [Grammar](#grammar)


##### Doctests

Тестовые объявления именуемые с использованием идентификатора, являются тестами _doctests_. Идентификатор должен
ссылаться на другое объявление в области видимости. Документируемые тесты, как и [комментарий к
документу](#doc-comments), служат документацией для соответствующего объявления и будет отображаться в сгенерированной
документации для объявления.

Эффективное тестирование должно быть автономным и сосредоточенным на тестируемой объявлении, отвечать на вопросы которые
могут возникнуть у нового пользователя о ее интерфейсе или предполагаемом использовании, избегая при этом ненужных или
сбивающих с толку деталей. Документируемые тесты - это не замена комментария к документу, а скорее дополнение и
компаньон, предоставляющий тестируемый пример основанный на коде, проверенный с помощью `zig test`.


#### Test Failure

Программа запуска тестов по умолчанию проверяет наличие [ошибки](#errors) возвращаемой из теста. Когда тест возвращает
ошибку, тест считается неудачным и его [трассировка возврата ошибки](#error-return-traces) выводится в виде стандартной
ошибки. Общее количество ошибок будет указано после выполнения всех тестов.

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


#### Skip Tests

Один из способов пропустить тесты - отфильтровать их с помощью параметра командной строки `zig test --test-filter [text]`.
В результате в тестовую сборку будут включены только те тесты, название которых содержит указанный текст фильтра.
Обратите внимание, что неименованные тесты запускаются даже при использовании параметра командной строки `--test-filter
[text]`.

Чтобы программно пропустить тест, сделайте так чтобы `test` возвращал сообщение об ошибке `error.SkipZigTest` и
программа тестирования по умолчанию будут считать тест пропущенным. После выполнения всех тестов будет сообщено общее
количество пропущенных тестов.

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


#### Report Memory Leaks

Когда код выделяет [память](#memory) с помощью тестового аллокатора [стандартной библиотеки
Zig](https://ziglang.org/documentation/0.13.0/#Zig-Standard-Library), `std.testing.allocator`, программа тестирования по
умолчанию сообщает о любых утечках обнаруженных при использовании тестового аллокатора:

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

Смотрите также:
- [defer](#defer)
- [Memory](#memory)


#### Detecting Test Build

Используйте [переменную времени компиляции](#compile-variables) `@import("builtin").is_test` для определения тестовой
сборки:

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


#### Test Output and Logging

Программа запуска тестов по умолчанию и пространство имен тестирования стандартной библиотеки Zig выводят сообщения
о стандартной ошибке.


#### The Testing Namespace

Пространство имен `testing` стандартной библиотеки Zig содержит полезные функции, которые помогут вам создавать
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

Стандартная библиотека Zig также содержит функции для сравнения [срезов](#slices), строк и многого другого. Изучите
остальную часть пространства имен `std.testing` в [стандартной библиотеке
Zig](https://ziglang.org/documentation/0.13.0/#Zig-Standard-Library) для получения дополнительных функций.


#### Test Tool Documentation

В `zig test` есть несколько параметров командной строки, которые влияют на компиляцию тестов. Полный список приведен в
разделе `zig test --help`.


------------
### Variables

Переменная - это единица хранения данных в [памяти](#memory).

Как правило, при объявлении переменной предпочтительнее использовать `const`, а не `var`. Это сокращает работу как для
людей так и для компьютеров при чтении кода и создает больше возможностей для оптимизации.

Ключевое слово `extern` или встроенную функцию [@extern](#extern) можно использовать для связывания с переменной,
экспортируемой из другого объекта. Ключевое слово `export` или встроенную функцию [@export](#export) можно использовать
для того, чтобы сделать переменную доступной для других объектов во время линковки. В обоих случаях тип переменной
должен быть совместим с C ABI.

Смотрите также:
- [Exporting a C Library](#exporting-a-c-library)


#### Identifiers

Идентификаторы переменных никогда не должны затенять идентификаторы из внешней области видимости.

Идентификаторы должны начинаться с буквенного символа или символа подчеркивания и за ними может следовать любое
количество буквенно-цифровых символов или символов подчеркивания. Они не должны пересекаться с какими-либо ключевыми
словами. Смотрите также [Keyword Reference](#keyword-reference).

Если требуется имя которое не соответствует этим требованиям, например, для связи с внешними библиотеками, можно
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


#### Container Level Variables

Переменные уровня [контейнера](#containers) имеют статическое время жизни, не зависят от порядка и лениво анализируются.
Значение инициализации переменных уровня контейнера неявно соответствует [времени компиляции](#comptime). Если
переменная уровня контейнера равна `const`, то ее значение известно во время компиляции, в противном случае оно
известно во время выполнения.

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

Переменные уровня контейнера могут быть объявлены внутри [структур](#struct), [объединений](#union),
[перечисления](#enum) или [непрозрачного типа](#opaque):

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


#### Static Local Variables

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


#### Thread Local Variables

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

Для [однопоточных сборок](#single-threaded-builds) все локальные переменные потока обрабатываются как обычные
[переменные уровня контейнера](#container-level-variables).

Локальные переменные потока не могут быть `const`.


#### Local Variables

Локальные переменные встречаются внутри [функций](#functions), блоков [времени компиляции](#comptime) и блоков
[@cImport](#cimport).

Если локальная переменная имеет значение `const`, это означает, что после инициализации значение переменной не
изменится. Если значение инициализации переменной `const` известно во [время компиляции](#comptime) то эта переменная
также известна во время компиляции.

Локальная переменная может быть определена с помощью ключевого слова `comptime`. Это приводит к тому, что значение
переменной становится известным во время компиляции и все загрузки и сохранения переменной происходят во время
семантического анализа программы, а не во время выполнения. Все переменные объявленные в выражении `comptime` неявно
являются переменными `comptime`.

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
        // и поэтому `y != 2` - это значение времени компиляции и это значение if вычисляется статически.
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
### Integers

#### Integer Literals

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


#### Runtime Integer Values

Целочисленные литералы не имеют ограничений по размеру и если возникает какое-либо неопределенное поведение то
компилятор перехватывает его.

Однако, как только целочисленное значение перестает быть известным во время компиляции, оно должно иметь известный
размер и становится уязвимым для неопределенного поведения.

```zig
fn divide(a: i32, b: i32) i32 {
    return a / b;
}
```

В этой функции значения `a` и `b` известны только во время выполнения и таким образом эта операция деления уязвима для
обоих ошибок [переполнение целого числа](#integer-overflow) и [деления на ноль](#division-by-zero).

Операторы `+` и `-` вызывают неопределенное поведение при целочисленном переполнении. Для вписанной и насыщенной
арифметики для всех целей предусмотрены альтернативные операторы. Операторы `+%` и `-%` выполняют вписанную
арифметику, в то время как `+|` и `-|` выполняют арифметику с насыщением.

Zig поддерживает целые числа произвольной разрядности, для обозначения которых используется идентификатор `i` или `u`,
за которым следуют цифры. Например, идентификатор `i7` относится к 7-разрядному целому числу со знаком. Максимально
допустимая разрядность целочисленного типа равна `65535`. Для целых типов со знаком Zig использует представление
[дополнение до двух](https://en.wikipedia.org/wiki/Two's_complement).

Смотрите также:
- [Wrapping Operations](#wrapping-operations)


------------
### Floats

Zig имеет следующие типы с плавающей запятой:
- `f16` - IEEE-754-2008 binary16
- `f32` - IEEE-754-2008 binary32
- `f64` - IEEE-754-2008 binary64
- `f80` - IEEE-754-2008 80-битная расширенная точность
- `f128` - IEEE-754-2008 binary128
- `c_longdouble`** - соответствует `long double` для целевого C ABI


#### Float Literals

Литералы с плавающей запятой имеют тип `comptime_float`, который гарантированно имеет ту же точность и операции, что и
другой самый большой тип с плавающей запятой который является `f128`.

Литералы с плавающей запятой [преобразуются](#type-coercion) к любому типу с плавающей запятой и к любому
[целочисленному](#integers) типу если в нем нет дробной составляющей.

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

Не существует синтаксиса для значений `NaN`, `infinity` или отрицательной бесконечности. Для получения этих специальных
значений необходимо использовать стандартную библиотеку:

```zig
const std = @import("std");

const inf = std.math.inf(f32);
const negative_inf = -std.math.inf(f64);
const nan = std.math.nan(f128);
```


#### Floating Point Operations

По умолчанию для операций с плавающей запятой используется `Strict` режим, но вы можете переключиться на `Optimized`
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

Смотрите также:
- [@setFloatMode](#setFloatMode)
- [Division by Zero](#division-by-zero)


------------
### Operators

Перегрузки операторов нет. Когда вы видите оператор в Zig, вы знаете, что он выполняет что-то из этой таблицы и
ничего больше.

Деление остатка

**<p align="center">Таблица операторов</p>**

|Имя|Синтаксис|Типы|Примечания|Пример|
|----|:------:|-----|-------|-------|
|Сложение|`a + b`<br>`a += b`|[Integers](#integers) <br>[Floats](#floats)|Может вызвать [переполнение](#default-operations) для целых чисел. <br> Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.<br>Смотрите также [@addWithOverflow](#addwithoverflow).|`1 + 5 == 7`|
|Вписанное сложение|`a +% b`<br>`a +%= b`|[Integers](#integers)|Поведение переноса с дополненим до двух.<br> Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.<br>Смотрите также [@addWithOverflow](#addwithoverflow).|`@as(u32, 0xffffffff) +% 1 == 0`|
|Сложение с насыщением|`a +\| b`<br>`a +\|= b`|[Integers](#integers)|Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.|`@as(u8, 255) +\| 1 == @as(u8, 255)`|
|Вычитание|`a - b`<br>`a -= b`|[Integers](#integers)<br>[Floats](#floats)|Может вызвать [переполнение](#default-operations) для целых чисел.<br> Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.<br>Смотрите также [@subWithOverflow](#subwithoverflow).|`2 - 5 == -3`|
|Вписанное вычитание|`a -% b`<br>`a -%= b`|[Integers](#integers)|Поведение переноса с дополением до двух.<br>Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.<br>Смотрите также [@subWithOverflow](#subwithoverflow).|`@as(u8, 0) -% 1 == 255`|
|Вычитание с насыщанием|`a -\| b`<br>`a -\|= b`|[Integers](#integers)|Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.|`@as(u32, 0) -\| 1 == 0`|
|Отрицание|`-a`|[Integers](#integers)<br>[Floats](#floats)|Может вызвать [переполнение](#default-operations) для целых чисел.|`-1 == 0 - 1`|
|Вписанное отрицание|`-%a`|[Integers](#integers)|Поведение переноса с дополнением до двух.|`-%@as(i8, -128) == -128`|
|Умножение|`a * b`<br>`a *= b`|[Integers](#integers)<br>[Floats](#floats)|Может вызвать [переполнение](#default-operations) для целых чисел.<br> Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.<br>Смотрите также [@mulWithOverflow](#mulwithoverflow).|`2 * 5 == 10`|
|Вписанное умножение|`a *% b`<br>`a *%= b`|[Integers](#integers)|Поведение переноса с дополнением до двух.<br>Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.<br>Смотрите также [@mulWithOverflow](#mulwithoverflow).|`@as(u8, 200) *% 2 == 144`|
|Умножение с насыщением|`a *\| b`<br>`a *\|= b`|[Integers](#integers)|Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.|`@as(u8, 200) *\| 2 == 255`|
|Деление|`a / b`<br>`a /= b`|[Integers](#integers)<br>[Floats](#floats)|Может вызвать [переполнение](#default-operations) для целых чисел.<br> Может вызвать [деление на ноль](#division-by-zero) для целых чисел.<br> Может вызвать [деление на ноль](#division-by-zero) для чисел с плавающей запятой в [FloatMode.Optimized](#floating-point-operations) режиме.<br>Целочисленные операнды со знаком должны быть известны во время компиляции и быть положительными. В других случаях вместо этого используйте [@divTrunc](#divtrunc), [@divFloor](#divfloor) или [@divExact](#divexact).<br>Вызывает [разрешение одноранговых типов](#peer-type-resolution) для операндов.|`10 / 2 == 5`|
|Деление с остатком|`a % b`<br>`a %= b`|[Integers](#integers)<br>[Floats](#floats)|Может привести к [делению на ноль](#division-by-zero) целых чисел.<br> Может привести к [делению на ноль](#division-by-zero) чисел с плавающей запятой в [FloatMode.Optimized](#floating-point-operations) режиме.<br>Операнды со знаком или с плавающей запятой должны быть известны во время компиляции и быть положительными. В других случаях вместо этого используйте [@rem](#rem) или [@mod](#mod).<br>Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.|`10 % 3 == 1`|
|Битовый сдвиг влево|`a << b`<br>`a <<= b`|[Integers](#integers)|Перемещает все биты влево, добавляя новые нули в наименее значащий бит.<br>`b` должен быть известен во время компиляции или иметь тип с числом битов log2 равным `a`.<br>Смотрите также [@shlExact](#shlexact).<br>Смотрите также [@shlWithOverflow](#shlwithoverflow).|`0b1 << 8 == 0b1000000000`.|
|Битовый сдвиг влево с насыщением|`a <<\| b`<br>`a <<\|= b`|[Integers](#integers)|Смотрите также [shlExact](#shlexact).<br>Смотрите также [@shlWithOverflow](#shlwithoverflow).|`@as(u8,1) <<\| 8 == 255`|
|Битовый сдвиг вправо|`a >> b`<br>`a >>= b`|[Integers](#integers)|Перемещает все биты вправо, добавляя нули в старший значащий бит.<br>`b` должен быть известен во время компиляции или иметь тип c числом битов log2 равным `a`.<br>Смотрите также [@shrExact](#shrexact).|`0b1010 >> 1 == 0b101`|
|Побитовое И|`a & b`<br>`a &= b`|[Integers](#integers)|Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.|`0b011 & 0b101 == 0b001`|
|Побитовое ИЛИ|`a \| b`<br>`a \|= b`|[Integers](#integers)|Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.|`0b010 \| 0b100 == 0b110`|
|Побитовое исключающее ИЛИ|`a ^ b`<br>`a ^= b`|[Integers](#integers)|Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.|`0b011 ^ 0b101 == 0b110`|
|Побитовое НЕ|`~a`|[Integers](#integers)||`~@as(u8, 0b10101111) == 0b01010000`|
|Опциональное раскрытие со значением по умолчанию|`a orelse b`|[Optionals](#optionals)|Если `a` равно `null` возвращается `b` ("значение по умолчанию"), в противном случае возвращается раскрытое значение `a`. Обратите внимание, что `b` может быть значением типа [noreturn](#noreturn).|`const value: ?u32 = null;`<br>`const unwrapped = value orelse 1234;`<br>`unwrapped == 1234`|
|Опциональное раскрытие|`a.?`|[Optionals](#optionals)|Эквивалентно: `a orelse unreachable`|`const value: ?u32 = 5678;`<br>`value.? == 5678`
|Раскрытие ошибки со значением по умолчанию|`a catch b`<br>`a catch \|err \| b`|[Error Unions](#errors)|Если `a` это `error` возвращается `b` ("значение по умолчанию"), в противном случае возвращается раскрытое значение `a`. Обратите внимание, что `b` может быть значением типа [noreturn](#noreturn). `err` это `error` и находится в области действия выражения `b`.|`const value: anyerror!u32 = error.Broken;`<br>`const unwrapped = value catch 1234;`<br>`unwrapped == 1234`|
|Логическое И|`a and b`|[bool](#primitive-types)|Если `a` равно `false` возвращается `false` без вычисления `b`. В противном случае возвращается `b`.|`(false and true) == false`|
|Логическое ИЛИ|`a or b`|[bool](#primitive-types)|Если `a` равно `true` возвращается `true` без вычисления `b`. В противном случае возвращается `b`.|`(false or true) == true`|
|Логическое НЕ|`!a`|[bool](#primitive-types)||`!false == true`
|Равенство|`a == b`|[Integers](#integers)<br>[Floats](#floats)<br>[bool](#primitive-types)<br>[type](#primitive-types)|Возвращает значение `true`, если `a` и `b` равны, в противном случае возвращает `false`.<br> Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.|`(1 == 1) == true`|
|Проверка на нуль|`a == null`|[Optionals](#optionals)|Возвращает `true`, если `a` равно `null`, в противном случае возвращает `false`.|`const value: ?u32 =null;`<br>`(value == null) == true`|
|Неравенство|`a != b`|[Integers](#integers)<br>[Floats](#floats)<br>[bool](#primitive-types)<br>[type](#primitive-types)|Возвращает значение `false`, если значения `a` и `b` неравны, в противном случае возвращает значение `true`.Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.|`(1 != 1) == false`|
|Проверка на неравенство нулю|`a != null`|[Optionals](#optionals)|Возвращает значение `false`, если `a` равно `null`, в противном случае возвращает `true.`|`const value: ?u32 = 5;`<br>`(value != null) == true`|
|Больше чем|`a > b`|[Integers](#integers)<br>[Floats](#floats)|Возвращает значение `true`, если `a` больше чем `b`, в противном случае возвращает `false`.<br> Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.|`(2 > 1) == true`|
|Больше или Равно|`a >= b`|[Integers](#integers)<br>[Floats](#floats)|Возвращает значение `true`, если `a` больше чем `b` или равно ему, в противном случае возвращает `false`.<br>Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.|`(2 >= 1) == true`|
|Меньше чем|`a < b`|[Integers](#integers)<br>[Floats](#floats)|Возвращает значение `true`, если `a` меньше чем `b`, в противном случае возвращает `false`. Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.|`(1 < 2) == true`|
|Меньше или Равно|`a <= b`|[Integers](#integers)<br>[Floats](#floats)|Возвращает значение `true`, если `a` меньше чем `b` или равно ему, в противном случае возвращает `false`.Вызывает [Peer Type Resolution](#peer-type-resolution) для операндов.|`(1 <= 2) == true`|
|Объединение массивов|`a ++ b`|[Arrays](#arrays)|Доступно только в том случае, если длины `a` и `b` известны во время компиляции.|`const mem = @import("std").mem;`<br>`const array1 = [_]u32{1,2};`<br>`const array2 = [_]u32{3,4};`<br>`const together = array1 ++ array2;`<br>`mem.eql(u32, &together, &[_]u32{1,2,3,4})`|
|Множитель массивов|`a ** b`|[Arrays](#arrays)|Доступно только в том случае, если длины `a` и `b` известны во время компиляции.|`const mem = @import("std").mem;`<br>`const pattern = "ab" ** 3;`<br>`mem.eql(u8, pattern, "ababab")`|
|Разыменование указателя|`a.*`|[Pointers](#pointers)|Разыменование указателя.|`const x: u32 = 1234;`<br>`const ptr = &x;`<br>`ptr.* == 1234`|
|Взятие адреса|`&a`|All types||`const x: u32 = 1234;`<br>`const ptr = &x;`<br>`ptr.* = 1234`|
|Слияние наборов ошибок|`a \|\| b`|[Error Set Type](#error-set-type)|Слияние наборов ошибок|`const A = error{One};`<br>`const B = error{Two};`<br>`(A \|\| B) == error{One,Two}`|


#### Precedence

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
### Arrays

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

Смотрите также:
- [for](#for)
- [Slices](#slices)


#### Multidimensional Arrays

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


#### Sentinel-Terminated Arrays

Синтаксис `[N:x]T` описывает массив, который имеет контрольное значение `x` с индексом соответствующим длине `N`.

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

Смотрите также:
- [Sentinel-Terminated Pointers](#sentinel-terminated-pointer)
- [Sentinel-Terminated Slices](#sentinel-terminated-slices)


------------
### Vectors

Вектор - это группа логических значений, [целых чисел](#integers), [чисел с плавающей запятой](#floats) или
[указателей](#pointers) которые обрабатываются параллельно, по возможности используя инструкции SIMD. Типы векторов
создаются с помощью встроенной функции [@Vector](#vector).

Векторы поддерживают те же встроенные операторы, что и их базовые типы. Эти операции выполняются поэлементно и
возвращают вектор той же длины, что и входные векторы. Это включает:

- Арифметические операторы (`+`, `-`, `/`, `*`, `@divFloor`, `@sqrt`, `@ceil`, `@log`, и другие.)
- Побитовые операторы (`>>`, `<<`, `&`, `|`, `~`, и другие.)
- Операторы сравнения (`<`, `>`, `==`, и другие.)

Запрещено использовать математический оператор для сочетания скаляров (отдельных чисел) и векторов. Zig предоставляет
встроенную функцию [@splat](#splat) для простого преобразования скаляров в векторы, а также поддерживает синтаксис
[@reduce](#reduce) и индексации массива для преобразования векторов в скаляры. Векторы также поддерживают присваивание
массивам с фиксированной длиной и из массивов с известной длиной времени компиляции.

Для перестановки элементов внутри векторов и между ними Zig предоставляет функции [@shuffle](#shuffle) и
[@select](#select).

Операции с векторами длина которых меньше чем исходный размер SIMD целевой машины обычно компилируются в одну
SIMD-инструкцию, в то время как векторы длина которых превышает исходный размер SIMD целевой машины компилируются в
несколько SIMD-инструкций. Если данная операция не поддерживает SIMD в целевой архитектуре компилятор по умолчанию будет
работать с каждым векторным элементом по очереди. Zig поддерживает любую известную во время компиляции длину вектора
вплоть до 2^32-1, хотя наиболее типичными являются малые степени двойки (2-64). Обратите внимание, что чрезмерно большие
длины векторов (например, 2^20) могут привести к сбоям компилятора в текущих версиях Zig.

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

    // Вы также можете присвоить вектору - срез с известной длиной во время компиляции используя .*
    const vec2: @Vector(2, f32) = arr1[1..3].*;

    const slice: []const f32 = &arr1;
    var offset: u32 = 1; // var, чтобы сделать его известным во время выполнения
    _ = &offset; // подавить ошибку, что "var никогда не изменяется"
    // Чтобы извлечь известную длину во время выполнения из известного смещения во время выполнения,
    // сначала извлеките новый фрагмент из начального смещения, а затем массив
    // известной длины во время компиляции
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

Смотрите также:
- [@splat](#splat)
- [@shuffle](#shuffle)
- [@select](#select)
- [@reduce](#reduce)


------------
### Pointers

В Zig есть два вида указателей: одноэлементные и многоэлементные:
- `*T` - одноэлементный указатель ровно на один элемент.
    - Поддерживает синтаксис разыменования: `ptr.*`
- `[*]T` - многоэлементный указатель на неизвестное количество элементов.
    - Поддерживает синтаксис индексации: `ptr[i]`
    - Поддерживает синтаксис срезов: `ptr[start..end]` и `ptr[start..]`
    - Поддерживает арифметику указателей: `ptr + x`, `ptr - x`
    - `T` должен иметь известный размер, что означает, что он не может быть `anyopaque` или каким-либо другим
    [непрозрачным типом](#opaque).

Эти типы тесно связаны с [массивами](#arrays) и [срезами](#slices):
- `*[N]T` - указатель на N элементов, такой же как одноэлементный указатель на массив.
    - Поддерживает синтаксис индекса: `array_ptr[i]`
    - Поддерживает синтаксис срезов: `array_ptr[start..end]`
    - Поддерживает свойство длины: `array_ptr.len`
- `[]T` - это срез ("толстый" указатель который содержит указатель типа `[*]T` и длину).
    - Поддерживает синтаксис индексации: `slice[i]`
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

    // Когда вы получаете адрес константной переменной, вы получаете указатель на один константный элемент.
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

Zig поддерживает арифметику с указателями. Лучше присвоить указателю значение `[*]T` и увеличить эту переменную.
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
    _ = &length; // подавить ошибку, что "var никогда не изменяется"
    var slice = array[length..array.len];

    try expect(slice[0] == 1);
    try expect(slice.len == 4);

    slice.ptr += 1; // теперь срез находится в неконсистентном состоянии, так как len не обновлялся

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

В Zig мы как правило предпочитаем [срезы](#slices), а не [указатели имеющие контрольно-терминированное
значение](#sentinel-terminated-pointers). Вы можете преобразовать массив или указатель в срез используя синтаксис
срезов.

Срезы имеют проверку границ и следовательно защищены от такого рода неопределенного поведения. Это одна из причин по
которой мы предпочитаем срезы, а не указатели.

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

Zig способен сохранять адреса памяти в компилируемом коде, если указатель никогда не разыменовывают:

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

Смотрите также:
- [Optional Pointers](#optional-pointers)
- [@ptrFromInt](#ptrfromint)
- [@intFromPtr](#intfromptr)
- [C Pointers](#c-pointers)


#### volatile

Предполагается, что загрузка и сохранение данных не имеют побочных эффектов. Если данная загрузка или сохранение данных
должны иметь побочные эффекты такие как ввод/вывод с отображением памяти (MMIO) используйте `volatile`. В следующем
коде гарантированно все загрузки и сохранения с помощью `mmio_ptr` выполняются в том же порядке, что и в исходном коде:

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

Обратите внимание, что `volatile` не связан с параллелизмом и [атомарностью](#atomics). Если вы видите код который
использует `volatile` для чего-то другого кроме ввода/вывода с отображением в памяти, то это вероятно ошибка.

[@ptrCast](#ptrcast) преобразует тип элемента указателя в другой. Это создает новый указатель, который может вызвать
неопределенное поведение в зависимости от загружаемых данных и хранилищ, которые проходят через него. Как правило,
другие виды преобразования типов предпочтительнее чем `@ptrCast`, если это возможно.

```zig
const std = @import("std");
const expect = std.testing.expect;

test "pointer casting" {
    const bytes align(@alignOf(u32)) = [_]u8{ 0x12, 0x12, 0x12, 0x12 };
    const u32_ptr: *const u32 = @ptrCast(&bytes);
    try expect(u32_ptr.* == 0x12121212);

    // Даже этот пример является надуманным - есть способы сделать это лучше, чем
    // приведение указателя. Например, используя приведение с сужением среза:
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


#### Alignment

Каждый тип имеет `alignment` - количество байт, при котором значение типа загружается из памяти или
сохраняется в ней, адрес памяти должен быть равномерно кратен этому числу. Вы можете использовать [@alignOf](#alignof),
чтобы узнать это значение для любого типа.

Выравнивание зависит от архитектуры процессора, но всегда имеет степень двойки и меньше `1 << 29`.

В Zig тип указателя имеет значение выравнивание. Если значение равно выравниванию базового типа, его можно не
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

Точно так же как `*i32` может быть [преобразован](#type-coercion) в `*const i32`, указатель с большим выравниванием
может быть неявно преобразован в указатель с меньшим выравниванием, но не наоборот.

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

Если у вас есть указатель или срез который имеет небольшое выравнивание, но вы знаете, что на самом деле он имеет
большее выравнивание используйте [@alignCast](#aligncast) чтобы изменить указатель на более выровненный. Во время
выполнения это не требуется, но добавляется [проверка безопасности](#incorrect-pointer-alignment):

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

Этот атрибут указателя позволяет указателю иметь нулевой адрес. Это необходимо только для автономной целевой
операционной системы, где нулевой адрес может быть отображен. Если вы хотите представить нулевые указатели используйте
опциональные указатели. Опциональные указатели с значением `allowzero` не имеют такого же размера как указатели на
объекты. В этом примере кода, если бы указатель не имел атрибута `allowzero`, это означало бы недопустимое приведение
указателя к нулевому значению.:

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


#### Sentinel-Terminated Pointers

Синтаксис `[*:x]T` описывает указатель, длина которого определяется контрольно-терминированным значением. Это
обеспечивает защиту от переполнения буфера и перечитывания.

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

Смотрите также:
- [Sentinel-Terminated Slices](#sentinel-terminated-slices)
- [Sentinel-Terminated Arrays](#sentinel-terminated-arrays)


------------
### Slices

Срез - это указатель и длина. Разница между массивом и срезом заключается в том, что длина массива является частью
типа и известна во время компиляции, тогда как длина среза известна во время выполнения. Для доступа к обоим параметрам
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

    // Использование оператора взятие адреса для среза дает указатель на один элемент.
    try expect(@TypeOf(&slice[0]) == *i32);
    // Использование поля `ptr` дает указатель на множество элементов.
    try expect(@TypeOf(slice.ptr) == [*]i32);
    try expect(@intFromPtr(slice.ptr) == @intFromPtr(&slice[0]));

    // Срезы проверяют границы массива.
    // Если вы попытаетесь получить доступ к чему-либо выходящему за границы срезы, вы получите сбой проверки безопасности:
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

Это одна из причин по которой мы предпочитаем срезы - указателям.

```zig
const std = @import("std");
const expect = std.testing.expect;
const mem = std.mem;
const fmt = std.fmt;

test "using slices for strings" {
    // Zig не имеет понятия о строках. Строковые литералы - это константные указатели
    // на нуль-терминированные массивы u8 и по соглашению параметры
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
    // Срез является изменяемым потому что мы "вырезали" изменяемый указатель.
    try expect(@TypeOf(slice) == []u8);
    slice[2] = 3;
    try expect(array[2] == 3);

    // Опять же, срез с использованием известных во время компиляции индексов приведет к созданию другого указателя
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

Смотрите также:
- [Pointers](#pointers)
- [for](#for)
- [Arrays](#arrays)


#### Sentinel-Terminated Slices

Синтаксис `[:x]T` - это срез, который имеет известную во время выполнения длину, а также гарантирует наличие
контрольного значения в элементе, индексируемом этой длиной. Тип не гарантирует, что перед этим элементом нет
контрольных значений. Срезы имеющие контрольное значение позволяют получить доступ к элементу по индексу индексу `len`.

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

Срезы имеющие контрольное значение также могут быть созданы с использованием вариации синтаксиса среза `data[start..end
:x]`, где данные представляют собой указатель на множество элементов, массив или срез, а `x` - контрольное значение.

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

Для среза с контрольно-терминированным значением, утверждается, что элемент в позиции контрольного значения на самом
деле является контрольным значением. Если это не так, возникает [неопределенное поведение](#undefined-behavior) с
проверкой безопасности.

```zig
const std = @import("std");
const expect = std.testing.expect;

test "sentinel mismatch" {
    var array = [_]u8{ 3, 2, 1, 0 };

    // Создание среза заканчивающегося контрольным значением из массива длиной 2
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

Смотрите также:
- [Sentinel-Terminated Pointers](#sentinel-terminated-pointers)
- [Sentinel-Terminated Arrays](#sentinel-terminated-arrays)


------------
### Struct

```zig
// Объявляем структуру.
// Zig не дает никаких гарантий относительно порядка расположения полей и размера
// структуры, но поля гарантированно будут выровнены по вертикали.
const Point = struct {
    x: f32,
    y: f32,
};

// Возможно мы хотим передать это в OpenGL, поэтому мы хотим быть внимательными к тому как расположены байты.
const Point2 = packed struct {
    x: f32,
    y: f32,
};

// Объявляет экземпляр структуры.
const p = Point{
    .x = 0.12,
    .y = 0.34,
};

// Возможно мы еще не готовы заполнить некоторые поля.
var p2 = Point{
    .x = 0.12,
    .y = undefined,
};

// У структур могут быть методы
// Методы структуры не являются специальными, они просто разделены пространством имен
// функции, которые вы можете вызывать с помощью точечного синтаксиса.
const Vec3 = struct {
    x: f32,
    y: f32,
    z: f32,

    pub fn init(x: f32, y: f32, z: f32) Vec3 {
        return Vec3{
            .x = x,
            .y = y,
            .z = z,
        };
    }

    pub fn dot(self: Vec3, other: Vec3) f32 {
        return self.x * other.x + self.y * other.y + self.z * other.z;
    }
};

const expect = @import("std").testing.expect;
test "dot product" {
    const v1 = Vec3.init(1.0, 0.0, 0.0);
    const v2 = Vec3.init(0.0, 1.0, 0.0);
    try expect(v1.dot(v2) == 0.0);

    // Помимо того, что методы структуры доступны для вызова с использованием точечного синтаксиса, они не являются
    // особенными. Вы можете ссылаться на них как на любое другое объявление внутри
    // структура:
    try expect(Vec3.dot(v1, v2) == 0.0);
}

// Структуры могут содержать объявления.
// Структуры могут содержать 0 полей.
const Empty = struct {
    pub const PI = 3.14;
};
test "struct namespaced variable" {
    try expect(Empty.PI == 3.14);
    try expect(@sizeOf(Empty) == 0);

    // вы все еще можете создать экземпляр пустой структуры
    const does_nothing = Empty{};

    _ = does_nothing;
}

// порядок расположения полей структуры определяется компилятором для обеспечения оптимальной производительности.
// однако вы все равно можете вычислить базовый указатель структуры по указателю поля:
fn setYBasedOnX(x: *f32, y: f32) void {
    const point: *Point = @fieldParentPtr("x", x);
    point.y = y;
}
test "field parent pointer" {
    var point = Point{
        .x = 0.1234,
        .y = 0.5678,
    };
    setYBasedOnX(&point.x, 0.9);
    try expect(point.y == 0.9);
}

// Вы можете вернуть структуру из функции. Вот как мы создаем обобщения в Zig:
fn LinkedList(comptime T: type) type {
    return struct {
        pub const Node = struct {
            prev: ?*Node,
            next: ?*Node,
            data: T,
        };

        first: ?*Node,
        last: ?*Node,
        len: usize,
    };
}

test "linked list" {
    // Функции, вызываемые во время компиляции, сохраняются в памяти. Это означает, что вы можете
    // сделать это:
    try expect(LinkedList(i32) == LinkedList(i32));

    const list = LinkedList(i32){
        .first = null,
        .last = null,
        .len = 0,
    };
    try expect(list.len == 0);

    // Поскольку типы являются значениями первого класса, вы можете создать экземпляр типа присвоив его переменной:
    const ListOfInts = LinkedList(i32);
    try expect(ListOfInts == LinkedList(i32));

    var node = ListOfInts.Node{
        .prev = null,
        .next = null,
        .data = 1234,
    };
    const list2 = LinkedList(i32){
        .first = &node,
        .last = &node,
        .len = 1,
    };

    // При использовании указателя на структуру к полям можно обращаться напрямую без явного разыменования указателя.
    // Таким образом, вы можете сделать
    try expect(list2.first.?.data == 1234);
    // вместо try expect(list2.first.?.*.data == 1234);
}
```
```bash
$ zig test test_structs.zig
1/4 test_structs.test.dot product...OK
2/4 test_structs.test.struct namespaced variable...OK
3/4 test_structs.test.field parent pointer...OK
4/4 test_structs.test.linked list...OK
All 4 tests passed.
```


#### Default Field Values

Каждое структурное поле может содержать выражение указывающее значение поля по умолчанию. Такие выражения выполняются
во [время компиляции](#comptime) и позволяют исключить поле в структурно литеральном выражении:

```zig
const Foo = struct {
    a: i32 = 1234,
    b: i32,
};

test "default struct initialization fields" {
    const x: Foo = .{
        .b = 5,
    };
    if (x.a + x.b != 1239) {
        comptime unreachable;
    }
}
```
```bash
$ zig test struct_default_field_values.zig
1/1 struct_default_field_values.test.default struct initialization fields...OK
All 1 tests passed.
```

Значения полей по умолчанию подходят только в том случае если инварианты данных структуры не могут быть нарушены путем
исключения этого поля из инициализации.

Например, здесь приведено неправильное использование инициализации поля структуры по умолчанию:

```zig
const Threshold = struct {
    minimum: f32 = 0.25,
    maximum: f32 = 0.75,

    const Category = enum { low, medium, high };

    fn categorize(t: Threshold, value: f32) Category {
        assert(t.maximum >= t.minimum);
        if (value < t.minimum) return .low;
        if (value > t.maximum) return .high;
        return .medium;
    }
};

pub fn main() !void {
    var threshold: Threshold = .{
        .maximum = 0.20,
    };
    const category = threshold.categorize(0.90);
    try std.io.getStdOut().writeAll(@tagName(category));
}

const std = @import("std");
const assert = std.debug.assert;
```
```bash
$ zig build-exe bad_default_value.zig
$ ./bad_default_value
thread 3570319 panic: reached unreachable code
/home/andy/src/zig/lib/std/debug.zig:412:14: 0x1037a6d in assert (bad_default_value)
    if (!ok) unreachable; // assertion failure
             ^
/home/andy/src/zig/doc/langref/bad_default_value.zig:8:15: 0x1034f59 in categorize (bad_default_value)
        assert(t.maximum >= t.minimum);
              ^
/home/andy/src/zig/doc/langref/bad_default_value.zig:19:42: 0x1034e8a in main (bad_default_value)
    const category = threshold.categorize(0.90);
                                         ^
/home/andy/src/zig/lib/std/start.zig:524:37: 0x1034da5 in posixCallMainAndExit (bad_default_value)
            const result = root.main() catch |err| {
                                    ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x10348c1 in _start (bad_default_value)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)

```

Выше вы можете увидеть опасность игнорирования этого принципа. Значения полей по умолчанию привели к нарушению
инварианта данных, что привело к недопустимому поведению.

Чтобы исправить это, удалите значения по умолчанию из всех структурных полей и укажите именованное значение
по умолчанию:

```zig
const Threshold = struct {
    minimum: f32,
    maximum: f32,

    const default: Threshold = .{
        .minimum = 0.25,
        .maximum = 0.75,
    };
};
```

Если для инициализации значения структуры требуется значение известное во время выполнения, без нарушения инвариантов
данных, то используйте метод инициализации который принимает эти значения во время выполнения и заполняет остальные
поля.


#### extern struct

`extern struct` имеет расположение в памяти соответствующее C ABI для целевого объекта.

Если не требуется четко определенное расположение в памяти, [структура](#struct) - лучший выбор, поскольку он
накладывает меньше ограничений на компилятор.

Смотрите раздел [packed struct](#packed-struct) для структуры которая имеет ABI своего базового целого числа,
что может быть полезно для моделирования флагов.

Смотрите также:
- [extern union](#extern-union)
- [extern enum](#extern-enum)


#### packed struct

В отличие от обычных структур, `packed struct` имеют гарантированное расположение в памяти:

- Поля остаются в указанном порядке, от наименее значимого к наиболее значимому.
- Между полями нет промежутков.
- Zig поддерживает [целые числа](#integers) произвольной ширины, и хотя обычно целые числа содержащие менее 8 бит,
по-прежнему занимают 1 байт памяти, в упакованных структурах они используют именно свою разрядность.
- `bool` поля используют ровно 1 бит.
- Поле [перечисления](#enum) использует ровно столько же битовой ширины, сколько и его целочисленный тип тега.
- Поле [упакованного объединения](#packed-union) использует ровно столько же битовой ширины сколько поле объединения с
наибольшей битовой шириной.

Это означает, что `packed struct` может участвовать в [@bitCast](#bitcast) или [@ptrCast](#ptrcast) операциях для
реинтерпретации памяти. Это работает даже во [время компиляции](#comptime):


```zig
const std = @import("std");
const native_endian = @import("builtin").target.cpu.arch.endian();
const expect = std.testing.expect;

const Full = packed struct {
    number: u16,
};
const Divided = packed struct {
    half1: u8,
    quarter3: u4,
    quarter4: u4,
};

test "@bitCast between packed structs" {
    try doTheTest();
    try comptime doTheTest();
}

fn doTheTest() !void {
    try expect(@sizeOf(Full) == 2);
    try expect(@sizeOf(Divided) == 2);
    const full = Full{ .number = 0x1234 };
    const divided: Divided = @bitCast(full);
    try expect(divided.half1 == 0x34);
    try expect(divided.quarter3 == 0x2);
    try expect(divided.quarter4 == 0x1);

    const ordered: [2]u8 = @bitCast(full);
    switch (native_endian) {
        .big => {
            try expect(ordered[0] == 0x12);
            try expect(ordered[1] == 0x34);
        },
        .little => {
            try expect(ordered[0] == 0x34);
            try expect(ordered[1] == 0x12);
        },
    }
}
```
```bash
$ zig test test_packed_structs.zig
1/1 test_packed_structs.test.@bitCast between packed structs...OK
All 1 tests passed.
```

Базовое целое число вычисляется из общей разрядности полей. При желании оно может быть явно указано и применено во время
компиляции:

```zig
test "missized packed struct" {
    const S = packed struct(u32) { a: u16, b: u8 };
    _ = S{ .a = 4, .b = 2 };
}
```
```bash
$ zig test test_missized_packed_struct.zig
doc/langref/test_missized_packed_struct.zig:2:29: error: backing integer type 'u32' has bit size 32 but the struct fields have a total bit size of 24
    const S = packed struct(u32) { a: u16, b: u8 };
                            ^~~
```

Zig позволяет использовать адрес поля не выровненного по байтам:

```zig
const std = @import("std");
const expect = std.testing.expect;

const BitField = packed struct {
    a: u3,
    b: u3,
    c: u2,
};

var foo = BitField{
    .a = 1,
    .b = 2,
    .c = 3,
};

test "pointer to non-byte-aligned field" {
    const ptr = &foo.b;
    try expect(ptr.* == 2);
```
```bash
$ zig test test_pointer_to_non-byte_aligned_field.zig
1/1 test_pointer_to_non-byte_aligned_field.test.pointer to non-byte-aligned field...OK
All 1 tests passed.
```

Однако указатель на поле не выровненное по байтам, обладает особыми свойствами и не может быть передан, когда ожидается
обычный указатель:

```zig
const std = @import("std");
const expect = std.testing.expect;

const BitField = packed struct {
    a: u3,
    b: u3,
    c: u2,
};

var bit_field = BitField{
    .a = 1,
    .b = 2,
    .c = 3,
};

test "pointer to non-byte-aligned field" {
    try expect(bar(&bit_field.b) == 2);
}

fn bar(x: *const u3) u3 {
    return x.*;
}
```
```bash
$ zig test test_misaligned_pointer.zig
doc/langref/test_misaligned_pointer.zig:17:20: error: expected type '*const u3', found '*align(1:3:1) u3'
    try expect(bar(&bit_field.b) == 2);
                   ^~~~~~~~~~~~
doc/langref/test_misaligned_pointer.zig:17:20: note: pointer host size '1' cannot cast into pointer host size '0'
doc/langref/test_misaligned_pointer.zig:17:20: note: pointer bit offset '3' cannot cast into pointer bit offset '0'
doc/langref/test_misaligned_pointer.zig:20:11: note: parameter type declared here
fn bar(x: *const u3) u3 {
          ^~~~~~~~~
```

В этом случае функция `bar` не может быть вызвана, поскольку указатель на поле не выровненное по ABI указывает битовое
смещение, а функция ожидает указатель выровненный по ABI.

Указатели на поля не выровненные по ABI имеют тот же адрес, что и другие поля в пределах их основного целого:

```zig
const std = @import("std");
const expect = std.testing.expect;

const BitField = packed struct {
    a: u3,
    b: u3,
    c: u2,
};

var bit_field = BitField{
    .a = 1,
    .b = 2,
    .c = 3,
};

test "pointers of sub-byte-aligned fields share addresses" {
    try expect(@intFromPtr(&bit_field.a) == @intFromPtr(&bit_field.b));
    try expect(@intFromPtr(&bit_field.a) == @intFromPtr(&bit_field.c));
}
```
```bash
$ zig test test_packed_struct_field_address.zig
1/1 test_packed_struct_field_address.test.pointers of sub-byte-aligned fields share addresses...OK
All 1 tests passed.
```

Это можно наблюдать с помощью [@bitOffsetOf](#bitoffsetof) и [offsetOf](#offsetof):

```zig
const std = @import("std");
const expect = std.testing.expect;

const BitField = packed struct {
    a: u3,
    b: u3,
    c: u2,
};

test "offsets of non-byte-aligned fields" {
    comptime {
        try expect(@bitOffsetOf(BitField, "a") == 0);
        try expect(@bitOffsetOf(BitField, "b") == 3);
        try expect(@bitOffsetOf(BitField, "c") == 6);

        try expect(@offsetOf(BitField, "a") == 0);
        try expect(@offsetOf(BitField, "b") == 0);
        try expect(@offsetOf(BitField, "c") == 0);
    }
}
```
```bash
$ zig test test_bitOffsetOf_offsetOf.zig
1/1 test_bitOffsetOf_offsetOf.test.offsets of non-byte-aligned fields...OK
All 1 tests passed.
```

Упакованные структуры имеют то же выравнивание, что и их базовое целое число, однако выровненные указатели на
упакованные структуры могут это переопределить:

```zig
const std = @import("std");
const expect = std.testing.expect;

const S = packed struct {
    a: u32,
    b: u32,
};
test "overaligned pointer to packed struct" {
    var foo: S align(4) = .{ .a = 1, .b = 2 };
    const ptr: *align(4) S = &foo;
    const ptr_to_b: *u32 = &ptr.b;
    try expect(ptr_to_b.* == 2);
}
```
```bash
$ zig test test_overaligned_packed_struct.zig
1/1 test_overaligned_packed_struct.test.overaligned pointer to packed struct...OK
All 1 tests passed.
```

Также можно задать выравнивание полей структуры:

```zig
const std = @import("std");
const expectEqual = std.testing.expectEqual;

test "aligned struct fields" {
    const S = struct {
        a: u32 align(2),
        b: u32 align(64),
    };
    var foo = S{ .a = 1, .b = 2 };

    try expectEqual(64, @alignOf(S));
    try expectEqual(*align(2) u32, @TypeOf(&foo.a));
    try expectEqual(*align(64) u32, @TypeOf(&foo.b));
}
```
```bash
$ zig test test_aligned_struct_fields.zig
1/1 test_aligned_struct_fields.test.aligned struct fields...OK
All 1 tests passed.
```

Использование упакованных структур с [volatile](#volatile) проблематично, и может привести к ошибке компиляции в
будущем. Для получения подробной информации об этом подпишитесь на [этот вопрос](https://github.com/ziglang/zig/issues/1761).

<br>TODO После устранения этой проблемы обновите эти документы, добавив в них рекомендации по использованию упакованных
структур с помощью MMIO (пример использования для изменчивых упакованных структур).
Не волнуйтесь, в Zig будет хорошее решение для этого варианта использования.


#### Struct Naming

Поскольку все структуры анонимны, Zig выводит имя типа на основе нескольких правил.

- Если структура находится в выражении инициализации переменной, то она получает имя этой переменной.
- Если структура находится в выражении `return`, она получает имя функции из которой возвращается с сериализованными
значениями параметров.
- В противном случае структура получает такое имя как (`filename.funcname.__struct_ID`).
- Если структура объявлена внутри другой структуры, она получает имя как родительской структуры так и имя выведенное по
предыдущим правилам разделенное точкой.

```zig
const std = @import("std");

pub fn main() void {
    const Foo = struct {};
    std.debug.print("variable: {s}\n", .{@typeName(Foo)});
    std.debug.print("anonymous: {s}\n", .{@typeName(struct {})});
    std.debug.print("function: {s}\n", .{@typeName(List(i32))});
}

fn List(comptime T: type) type {
    return struct {
        x: T,
    };
}
```
```bash
$ zig build-exe struct_name.zig
$ ./struct_name
variable: struct_name.main.Foo
anonymous: struct_name.main__struct_3331
function: struct_name.List(i32)
```


#### Anonymous Struct Literals

Zig позволяет не указывать структурный тип литерала. При принудительном [преобразовании](#type-coercion) результата
структурный литерал непосредственно создаст экземпляр [местоположения результата](#result-location-semantics) без
копирования:

```zig
const std = @import("std");
const expect = std.testing.expect;

const Point = struct { x: i32, y: i32 };

test "anonymous struct literal" {
    const pt: Point = .{
        .x = 13,
        .y = 67,
    };
    try expect(pt.x == 13);
    try expect(pt.y == 67);
}
```
```bash
$ zig test test_struct_result.zig
1/1 test_struct_result.test.anonymous struct literal...OK
All 1 tests passed.
```

Тип структуры может быть выведен. В данном случае [местоположение результата](#result-location-semantics) не содержит
типа, поэтому Zig выводит тип:

```zig
const std = @import("std");
const expect = std.testing.expect;

test "fully anonymous struct" {
    try check(.{
        .int = @as(u32, 1234),
        .float = @as(f64, 12.34),
        .b = true,
        .s = "hi",
    });
}

fn check(args: anytype) !void {
    try expect(args.int == 1234);
    try expect(args.float == 12.34);
    try expect(args.b);
    try expect(args.s[0] == 'h');
    try expect(args.s[1] == 'i');
}
```
```bash
$ zig test test_anonymous_struct.zig
1/1 test_anonymous_struct.test.fully anonymous struct...OK
All 1 tests passed.
```


#### Tuples

Анонимные структуры могут создаваться без указания имен полей и называются "кортежами".

Поля неявно именуются числами, начинающимися с 0. Поскольку их имена являются целыми числами, к ним нельзя получить
доступ с помощью `.` синтаксиса который не включает в себя `@""`. Имена внутри `@""` всегда распознаются как
[идентификаторы](#identifiers).

Как и массивы, кортежи имеют поле `.len` и могут индексироваться (при условии, что индекс известен во время компиляции)
и может работать с операторами `++` и `**`. По кортежам также можно итерироваться с помощью [inline for](#inline-for).

```zig
const std = @import("std");
const expect = std.testing.expect;

test "tuple" {
    const values = .{
        @as(u32, 1234),
        @as(f64, 12.34),
        true,
        "hi",
    } ++ .{false} ** 2;
    try expect(values[0] == 1234);
    try expect(values[4] == false);
    inline for (values, 0..) |v, i| {
        if (i != 2) continue;
        try expect(v);
    }
    try expect(values.len == 6);
    try expect(values.@"3"[0] == 'h');
}
```
```bash
$ zig test test_tuples.zig
1/1 test_tuples.test.tuple...OK
All 1 tests passed.
```

Смотрите также:
- [comptime](#comptime)
- [@fieldParentPtr](#fieldparentptr)


------------
### enum

```zig
const expect = @import("std").testing.expect;
const mem = @import("std").mem;

// Объявление  перечисления.
const Type = enum {
    ok,
    not_ok,
};

// Объявление конкретного поля перечисления.
const c = Type.ok;

// Если вы хотите получить доступ к порядковому значению перечисления вы можете указать тип тега.
const Value = enum(u2) {
    zero,
    one,
    two,
};
// Теперь вы можете выбирать между u2 и значением.
// Порядковое значение начинается с 0, увеличиваясь на 1 от предыдущего элемента.
test "enum ordinal value" {
    try expect(@intFromEnum(Value.zero) == 0);
    try expect(@intFromEnum(Value.one) == 1);
    try expect(@intFromEnum(Value.two) == 2);
}

// Вы можете переопределить порядковое значение для перечисления.
const Value2 = enum(u32) {
    hundred = 100,
    thousand = 1000,
    million = 1000000,
};
test "set enum ordinal value" {
    try expect(@intFromEnum(Value2.hundred) == 100);
    try expect(@intFromEnum(Value2.thousand) == 1000);
    try expect(@intFromEnum(Value2.million) == 1000000);
}

// Вы также можете переопределить только некоторые значения.
const Value3 = enum(u4) {
    a,
    b = 8,
    c,
    d = 4,
    e,
};
test "enum implicit ordinal values and overridden values" {
    try expect(@intFromEnum(Value3.a) == 0);
    try expect(@intFromEnum(Value3.b) == 8);
    try expect(@intFromEnum(Value3.c) == 9);
    try expect(@intFromEnum(Value3.d) == 4);
    try expect(@intFromEnum(Value3.e) == 5);
}

// У перечислений могут быть методы, такие же как у структур и объединений.
// Перечислимые методы не являются особыми, они просто расположены в пространстве имен
// функции которые вы можете вызывать с помощью точечного синтаксиса.
const Suit = enum {
    clubs,
    spades,
    diamonds,
    hearts,

    pub fn isClubs(self: Suit) bool {
        return self == Suit.clubs;
    }
};
test "enum method" {
    const p = Suit.spades;
    try expect(!p.isClubs());
}

// Перечисление можно переключать.
const Foo = enum {
    string,
    number,
    none,
};
test "enum switch" {
    const p = Foo.number;
    const what_is_it = switch (p) {
        Foo.string => "this is a string",
        Foo.number => "this is a number",
        Foo.none => "this is a none",
    };
    try expect(mem.eql(u8, what_is_it, "this is a number"));
}

// @typeInfo может использоваться для доступа к целочисленному типу тега перечисления.
const Small = enum {
    one,
    two,
    three,
    four,
};
test "std.meta.Tag" {
    try expect(@typeInfo(Small).Enum.tag_type == u2);
}

// @typeInfo сообщает нам количество полей и их названия:
test "@typeInfo" {
    try expect(@typeInfo(Small).Enum.fields.len == 4);
    try expect(mem.eql(u8, @typeInfo(Small).Enum.fields[1].name, "two"));
}

// @tagName дает представление [:0]const u8 для значения перечисления:
test "@tagName" {
    try expect(mem.eql(u8, @tagName(Small.three), "three"));
}
```
```bash
$ zig test test_enums.zig
1/8 test_enums.test.enum ordinal value...OK
2/8 test_enums.test.set enum ordinal value...OK
3/8 test_enums.test.enum implicit ordinal values and overridden values...OK
4/8 test_enums.test.enum method...OK
5/8 test_enums.test.enum switch...OK
6/8 test_enums.test.std.meta.Tag...OK
7/8 test_enums.test.@typeInfo...OK
8/8 test_enums.test.@tagName...OK
All 8 tests passed.
```

Смотрите также:
- [@typeInfo](#typeinfo)
- [@tagName](#tagname)
- [@sizeOf](#sizeof)


#### extern enum

По умолчанию совместимость перечислений с C ABI не гарантируется:

```zig
const Foo = enum { a, b, c };
export fn entry(foo: Foo) void {
    _ = foo;
}
```
```bash
$ zig build-obj enum_export_error.zig
doc/langref/enum_export_error.zig:2:17: error: parameter of type 'enum_export_error.Foo' not allowed in function with calling convention 'C'
export fn entry(foo: Foo) void {
                ^~~~~~~~
doc/langref/enum_export_error.zig:2:17: note: enum tag type 'u2' is not extern compatible
doc/langref/enum_export_error.zig:2:17: note: only integers with 0, 8, 16, 32, 64 and 128 bits are extern compatible
doc/langref/enum_export_error.zig:1:13: note: enum declared here
const Foo = enum { a, b, c };
            ^~~~~~~~~~~~~~~~
```

Для перечисления совместимого с C ABI укажите явный тип тега для перечисления:

```zig
const Foo = enum(c_int) { a, b, c };
export fn entry(foo: Foo) void {
    _ = foo;
}
```
```bash
$ zig build-obj enum_export.zig
```


#### Enum Literals

Литералы перечисления позволяют указывать имя поля перечисления без указания типа перечисления:

```zig
const std = @import("std");
const expect = std.testing.expect;

const Color = enum {
    auto,
    off,
    on,
};

test "enum literals" {
    const color1: Color = .auto;
    const color2 = Color.auto;
    try expect(color1 == color2);
}

test "switch using enum literals" {
    const color = Color.on;
    const result = switch (color) {
        .auto => false,
        .on => true,
        .off => false,
    };
    try expect(result);
}
```
```bash
$ zig test test_enum_literals.zig
1/2 test_enum_literals.test.enum literals...OK
2/2 test_enum_literals.test.switch using enum literals...OK
All 2 tests passed.
```


#### Non-exhaustive enum

Неисчерпывающее перечисление можно создать добавив завершающее поле `_`. Перечисление должно указывать тип тега и не
может использовать все значения перечисления.

[@enumFromInt](#enumfromint) для неисчерпывающего перечисления включает в себя безопасную семантику
[@intCast](#intcast) к целочисленному типу тега, но помимо этого всегда приводит к четко определенному значению
перечисления.

`switch` в неисчерпывающем перечислении может включать символ `_` в качестве альтернативы `else`. С помощью символа
`_` компилятор выдает ошибку если все известные имена тегов не обрабатываются с помощью `switch`.

```zig
const std = @import("std");
const expect = std.testing.expect;

const Number = enum(u8) {
    one,
    two,
    three,
    _,
};

test "switch on non-exhaustive enum" {
    const number = Number.one;
    const result = switch (number) {
        .one => true,
        .two, .three => false,
        _ => false,
    };
    try expect(result);
    const is_one = switch (number) {
        .one => true,
        else => false,
    };
    try expect(is_one);
}
```
```bash
$ zig test test_switch_non-exhaustive.zig
1/1 test_switch_non-exhaustive.test.switch on non-exhaustive enum...OK
All 1 tests passed.
```


------------
### Union

`union` определяет набор возможных типов значений в виде списка полей. Одновременно может быть активным только одно
поле. Представление в памяти "голых" объединений не гарантируется. Объединения нельзя использовать для реинтерпретации
памяти. Для этого используйте [@ptrCast](#ptrcast) или используйте [внешнее объединение](#extern-union) или [упакованное
объединение](#packed-union) которые гарантируют расположение в памяти. [Доступ к неактивному
полю](#wrong-union-field-access) это [неопределенное поведение](#undefined-behavior) с проверкой безопасности:

```zig
const Payload = union {
    int: i64,
    float: f64,
    boolean: bool,
};
test "simple union" {
    var payload = Payload{ .int = 1234 };
    payload.float = 12.34;
}
```
```bash
$ zig test test_wrong_union_access.zig
1/1 test_wrong_union_access.test.simple union...thread 3579408 panic: access of union field 'float' while field 'int' is active
/home/andy/src/zig/doc/langref/test_wrong_union_access.zig:8:12: 0x103ce87 in test.simple union (test)
    payload.float = 12.34;
           ^
/home/andy/src/zig/lib/compiler/test_runner.zig:157:25: 0x1048070 in mainTerminal (test)
        if (test_fn.func()) |_| {
                        ^
/home/andy/src/zig/lib/compiler/test_runner.zig:37:28: 0x103e08b in main (test)
        return mainTerminal();
                           ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x103d419 in posixCallMainAndExit (test)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x103cf81 in _start (test)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
error: the following test command crashed:
/home/andy/src/zig/.zig-cache/o/bb9968225995fac0bbc9f2116e8583c2/test
```

Вы можете активировать другое поле, присвоив ему всё объединение:

```zig
const std = @import("std");
const expect = std.testing.expect;

const Payload = union {
    int: i64,
    float: f64,
    boolean: bool,
};
test "simple union" {
    var payload = Payload{ .int = 1234 };
    try expect(payload.int == 1234);
    payload = Payload{ .float = 12.34 };
    try expect(payload.float == 12.34);
}
```
```bash
$ zig test test_simple_union.zig
1/1 test_simple_union.test.simple union...OK
All 1 tests passed.
```

Чтобы использовать [switch](#switch) с объединением это должен быть [tagged union](#tagged-union).

Чтобы инициализировать объединение когда тег является именем известным во [время компиляции](#comptime), смотрите
[@unionInit](#unioninit).


#### Tagged union

Объединения могут быть объявлены с типом тега перечисления. Это превращает объединение в _тэгированное_ объединение, что
позволяет использовать его в выражениях [switch](#switch). Тэгированное объединение преобразуются к своему типу тега:
[Type Coercion: Unions and Enums](#type-coercion-unions-and-enums).

```zig
const std = @import("std");
const expect = std.testing.expect;

const ComplexTypeTag = enum {
    ok,
    not_ok,
};
const ComplexType = union(ComplexTypeTag) {
    ok: u8,
    not_ok: void,
};

test "switch on tagged union" {
    const c = ComplexType{ .ok = 42 };
    try expect(@as(ComplexTypeTag, c) == ComplexTypeTag.ok);

    switch (c) {
        ComplexTypeTag.ok => |value| try expect(value == 42),
        ComplexTypeTag.not_ok => unreachable,
    }
}

test "get tag type" {
    try expect(std.meta.Tag(ComplexType) == ComplexTypeTag);
}
```
```bash
$ zig test test_tagged_union.zig
1/2 test_tagged_union.test.switch on tagged union...OK
2/2 test_tagged_union.test.get tag type...OK
All 2 tests passed.
```

Чтобы изменить полезную нагрузку тэгированного объединения в выражении `switch`, поместите символ `*` перед именем
переменной чтобы сделать его указателем:

```zig
const std = @import("std");
const expect = std.testing.expect;

const ComplexTypeTag = enum {
    ok,
    not_ok,
};
const ComplexType = union(ComplexTypeTag) {
    ok: u8,
    not_ok: void,
};

test "modify tagged union in switch" {
    var c = ComplexType{ .ok = 42 };

    switch (c) {
        ComplexTypeTag.ok => |*value| value.* += 1,
        ComplexTypeTag.not_ok => unreachable,
    }

    try expect(c.ok == 43);
}
```
```bash
$ zig test test_switch_modify_tagged_union.zig
1/1 test_switch_modify_tagged_union.test.modify tagged union in switch...OK
All 1 tests passed.
```

Объединения могут быть созданы для определения типа тега перечисления. Кроме того у объединений могут быть такие же
методы как у структур и перечислений.

```zig
const std = @import("std");
const expect = std.testing.expect;

const Variant = union(enum) {
    int: i32,
    boolean: bool,

    // void может быть не указан при выводе типа тега enum.
    none,

    fn truthy(self: Variant) bool {
        return switch (self) {
            Variant.int => |x_int| x_int != 0,
            Variant.boolean => |x_bool| x_bool,
            Variant.none => false,
        };
    }
};

test "union method" {
    var v1 = Variant{ .int = 1 };
    var v2 = Variant{ .boolean = false };

    try expect(v1.truthy());
    try expect(!v2.truthy());
}
```
```bash
$ zig test test_union_method.zig
1/1 test_union_method.test.union method...OK
All 1 tests passed.
```

[@tagName](#tagnam) может использоваться для возврата значения [времени компиляции](#comptime) `[:0]const u8`,
представляющего имя поля:

```zig
const std = @import("std");
const expect = std.testing.expect;

const Small2 = union(enum) {
    a: i32,
    b: bool,
    c: u8,
};
test "@tagName" {
    try expect(std.mem.eql(u8, @tagName(Small2.a), "a"));
}
```
```bash
$ zig test test_tagName.zig
1/1 test_tagName.test.@tagName...OK
All 1 tests passed.
```


#### extern union

`extern union` имеет расположение памяти гарантированно совместимое с целевым C ABI.

Смотрите также:
- [extern struct](#extern-struct)


#### packed union

`packed union` имеет четко определенное расположение в памяти и имеет право находиться в [упакованной
структуре](#packed-struct).


#### Anonymous Union Literals

Синтаксис [анонимных структурных литералов](#anonymous-struct-literals) может использоваться для инициализации
объединений без указания типа:

```zig
const std = @import("std");
const expect = std.testing.expect;

const Number = union {
    int: i32,
    float: f64,
};

test "anonymous union literal syntax" {
    const i: Number = .{ .int = 42 };
    const f = makeNumber();
    try expect(i.int == 42);
    try expect(f.float == 12.34);
}

fn makeNumber() Number {
    return .{ .float = 12.34 };
}
```
```bash
 zig test test_anonymous_union.zig
1/1 test_anonymous_union.test.anonymous union literal syntax...OK
All 1 tests passed.
```


------------
### Opaque

`opaque {}` объявляет новый тип с неизвестным (но ненулевым) размером и выравниванием. Он может содержать такие же
объявления, как структуры, объединения и перечисления.

Обычно это используется для обеспечения безопасности типов при взаимодействии с кодом на C, который не раскрывает детали
структуры.

Пример:

```zig
const Derp = opaque {};
const Wat = opaque {};

extern fn bar(d: *Derp) void;
fn foo(w: *Wat) callconv(.C) void {
    bar(w);
}

test "call foo" {
    foo(undefined);
}
```
```bash
$ zig test test_opaque.zig
doc/langref/test_opaque.zig:6:9: error: expected type '*test_opaque.Derp', found '*test_opaque.Wat'
    bar(w);
        ^
doc/langref/test_opaque.zig:6:9: note: pointer type child 'test_opaque.Wat' cannot cast into pointer type child 'test_opaque.Derp'
doc/langref/test_opaque.zig:2:13: note: opaque declared here
const Wat = opaque {};
            ^~~~~~~~~
doc/langref/test_opaque.zig:1:14: note: opaque declared here
const Derp = opaque {};
             ^~~~~~~~~
doc/langref/test_opaque.zig:4:18: note: parameter type declared here
extern fn bar(d: *Derp) void;
                 ^~~~~
referenced by:
    test.call foo: doc/langref/test_opaque.zig:10:5
    remaining reference traces hidden; use '-freference-trace' to see all reference traces

```


------------
### Blocks

Блоки используются для ограничения области действия объявлений переменных:

```zig
test "access variable after block scope" {
    {
        var x: i32 = 1;
        _ = &x;
    }
    x += 1;
}
```
```bash
$ zig test test_blocks.zig
doc/langref/test_blocks.zig:6:5: error: use of undeclared identifier 'x'
    x += 1;
    ^
```

Блоки являются выражениями. Метку `break` можно использовать для возврата значения из блока:

```zig
const std = @import("std");
const expect = std.testing.expect;

test "labeled break from labeled block expression" {
    var y: i32 = 123;

    const x = blk: {
        y += 1;
        break :blk y;
    };
    try expect(x == 124);
    try expect(y == 124);
}
```
```bash
$ zig test test_labeled_break.zig
1/1 test_labeled_break.test.labeled break from labeled block expression...OK
All 1 tests passed.
```

Здесь `blk` может быть любым именем.

Смотрите также:
- [Labeled while](#labeled-while)
- [Labeled for](#labeled-for)


#### Shadowing

[Идентификаторам](#identifiers) никогда не разрешается "скрывать" другие идентификаторы используя одно и то же имя:

```zig
const pi = 3.14;

test "inside test block" {
    // Давайте даже зайдем в другой блок
    {
        var pi: i32 = 1234;
    }
}
```
```bash
$ zig test test_shadowing.zig
doc/langref/test_shadowing.zig:6:13: error: local variable shadows declaration of 'pi'
        var pi: i32 = 1234;
            ^~
doc/langref/test_shadowing.zig:1:1: note: declared here
const pi = 3.14;
^~~~~~~~~~~~~~~
```

Поэтому, читая код Zig, вы всегда можете рассчитывать на то, что идентификатор будет означать одно и то же в той
области в которой он определен. Обратите внимание, что вы можете использовать одно и то же имя, если области
видимости разделены:

```zig
test "separate scopes" {
    {
        const pi = 3.14;
        _ = pi;
    }
    {
        var pi: bool = true;
        _ = &pi;
    }
}
```
```bash
$ zig test test_scopes.zig
1/1 test_scopes.test.separate scopes...OK
All 1 tests passed.
```


#### Empty Blocks

Пустой блок эквивалентен `void{}`:

```zig
const std = @import("std");
const expect = std.testing.expect;

test {
    const a = {};
    const b = void{};
    try expect(@TypeOf(a) == void);
    try expect(@TypeOf(b) == void);
    try expect(a == b);
}
```
```bash
$ zig test test_empty_block.zig
1/1 test_empty_block.test_0...OK
All 1 tests passed.
```

------------
### Switch

```zig
const std = @import("std");
const builtin = @import("builtin");
const expect = std.testing.expect;

test "switch simple" {
    const a: u64 = 10;
    const zz: u64 = 103;

    // Все ветви выражения switch должны быть преобразованы к общему типу.
    // Переходы между ветвями не могут быть завершены с ошибками. Если требуется выполнение с ошибками, объедините
    // варианты и используйте if.
    const b = switch (a) {
        // Несколько обращений могут быть объединены с помощью ','
        1, 2, 3 => 0,

        // Диапазоны могут быть заданы с помощью "..." синтаксиса. Они являются всеобъемлющими с обоих концов.
        5...100 => 1,

        // Ветви могут быть сколь угодно сложными.
        101 => blk: {
            const c: u64 = 5;
            break :blk c * 2 + 1;
        },

        // Включение произвольных выражений допускается до тех пор, пока
        // выражение известно во время компиляции.
        zz => zz,
        blk: {
            const d: u32 = 5;
            const e: u32 = 100;
            break :blk d + e;
        } => 107,

        // Ветвь else перехватывает все, что еще не захвачено.
        // Ветви else являются обязательными, если не обрабатывается весь диапазон значений.
        else => 9,
    };

    try expect(b == 1);
}

// Выражения switch могут использоваться вне функции:
const os_msg = switch (builtin.target.os.tag) {
    .linux => "we found a linux user",
    else => "not a linux user",
};

// Внутри функции операторы switch неявно выполняются во время компиляции
// вычисляются если целевое выражение известно во время компиляции.
test "switch inside function" {
    switch (builtin.target.os.tag) {
        .fuchsia => {
            // В операционной системе, отличной от fuchsia блок даже не анализируется,
            // поэтому эта ошибка компиляции не запускается.
            // В fuchsia эта ошибка компиляции была бы запущена.
            @compileError("fuchsia not supported");
        },
        else => {},
    }
}
```
```bash
$ zig test test_switch.zig
1/2 test_switch.test.switch simple...OK
2/2 test_switch.test.switch inside function...OK
All 2 tests passed.
```

`switch` можно использовать для записи значений полей в тэгированное объединение. Для изменения значений полей перед
именем переменной записи можно поместить символ `*`, сделав его указателем.

```zig
const expect = @import("std").testing.expect;

test "switch on tagged union" {
    const Point = struct {
        x: u8,
        y: u8,
    };
    const Item = union(enum) {
        a: u32,
        c: Point,
        d,
        e: u32,
    };

    var a = Item{ .c = Point{ .x = 1, .y = 2 } };

    // Допускается включение более сложных перечислений.
    const b = switch (a) {
        // Группа захвата разрешена для совпадения и вернет перечисление
        // значение совпадает. Если типы полезной нагрузки в обоих случаях одинаковы
        // их можно поместить в одну и ту же ветку сопоставления.
        Item.a, Item.e => |item| item,

        // Ссылка на сопоставленное значение может быть получена с помощью синтаксиса `*`.
        Item.c => |*item| blk: {
            item.*.x += 1;
            break :blk 6;
        },

        // Больше ничего не требуется, если все случаи сопоставлений были обработаны исчерпывающим образом
        Item.d => 8,
    };

    try expect(b == 6);
    try expect(a.c.x == 2);
}
```
```bash
$ zig test test_switch_tagged_union.zig
1/1 test_switch_tagged_union.test.switch on tagged union...OK
All 1 tests passed.
```

Смотрите также:
- [comptime](#comptime)
- [enum](#enum)
- [@compileError](#compileerror)
- [Compile Variables](#compile-variables)


#### Exhaustive Switching

Если выражение `switch` не содержит условия `else`, то оно должно содержать исчерпывающий список всех возможных
значений. Невыполнение этого требования является ошибкой компиляции:

```zig
const Color = enum {
    auto,
    off,
    on,
};

test "exhaustive switching" {
    const color = Color.off;
    switch (color) {
        Color.auto => {},
        Color.on => {},
    }
}
```
```bash
$ zig test test_unhandled_enumeration_value.zig
doc/langref/test_unhandled_enumeration_value.zig:9:5: error: switch must handle all possibilities
    switch (color) {
    ^~~~~~
doc/langref/test_unhandled_enumeration_value.zig:3:5: note: unhandled enumeration value: 'off'
    off,
    ^~~
doc/langref/test_unhandled_enumeration_value.zig:1:15: note: enum 'test_unhandled_enumeration_value.Color' declared here
const Color = enum {
              ^~~~
```


#### Switching with Enum Literals

[Литералы перечисления](#enum-literals) полезно использовать со `switch`, чтобы избежать повторного указания типов
перечислений или объединений:

```zig
const std = @import("std");
const expect = std.testing.expect;

const Color = enum {
    auto,
    off,
    on,
};

test "enum literals with switch" {
    const color = Color.off;
    const result = switch (color) {
        .auto => false,
        .on => false,
        .off => true,
    };
    try expect(result);
}
```
```bash
$ zig test test_exhaustive_switch.zig
1/1 test_exhaustive_switch.test.enum literals with switch...OK
All 1 tests passed.
```


#### Inline Switch Prongs

Элементы `switch` могут быть помечены как `inline` чтобы сгенерировать тело элемента для каждого возможного значения
которое оно может иметь, что делает захваченное значение известным во [время компиляции](#comptime).

```zig
const std = @import("std");
const expect = std.testing.expect;
const expectError = std.testing.expectError;

fn isFieldOptional(comptime T: type, field_index: usize) !bool {
    const fields = @typeInfo(T).Struct.fields;
    return switch (field_index) {
        // Этот параметр анализируется дважды, причем каждый раз `idx` является
        // значением известным по время компиляции.
        inline 0, 1 => |idx| @typeInfo(fields[idx].type) == .Optional,
        else => return error.IndexOutOfBounds,
    };
}

const Struct1 = struct { a: u32, b: ?u32 };

test "using @typeInfo with runtime values" {
    var index: usize = 0;
    try expect(!try isFieldOptional(Struct1, index));
    index += 1;
    try expect(try isFieldOptional(Struct1, index));
    index += 1;
    try expectError(error.IndexOutOfBounds, isFieldOptional(Struct1, index));
}

// Вызовы `isFieldOptional` в `Struct1` преобразуются в эквивалент
// этой функции:
fn isFieldOptionalUnrolled(field_index: usize) !bool {
    return switch (field_index) {
        0 => false,
        1 => true,
        else => return error.IndexOutOfBounds,
    };
}
```
```bash
$ zig test test_inline_switch.zig
1/1 test_inline_switch.test.using @typeInfo with runtime values...OK
All 1 tests passed.
```

Ключевое слово `inline` также может сочетаться с диапазонами:

```zig
fn isFieldOptional(comptime T: type, field_index: usize) !bool {
    const fields = @typeInfo(T).Struct.fields;
    return switch (field_index) {
        inline 0...fields.len - 1 => |idx| @typeInfo(fields[idx].type) == .Optional,
        else => return error.IndexOutOfBounds,
    };
}
```

Элементы `inline else` можно использовать в качестве типобезопасной альтернативы циклам `inline for`:

```zig
const std = @import("std");
const expect = std.testing.expect;

const SliceTypeA = extern struct {
    len: usize,
    ptr: [*]u32,
};
const SliceTypeB = extern struct {
    ptr: [*]SliceTypeA,
    len: usize,
};
const AnySlice = union(enum) {
    a: SliceTypeA,
    b: SliceTypeB,
    c: []const u8,
    d: []AnySlice,
};

fn withFor(any: AnySlice) usize {
    const Tag = @typeInfo(AnySlice).Union.tag_type.?;
    inline for (@typeInfo(Tag).Enum.fields) |field| {
        // При использовании `inline for` функция генерируется как
        // серия инструкций `if`, зависящих от оптимизатора
        // для преобразования ее в switch.
        if (field.value == @intFromEnum(any)) {
            return @field(any, field.name).len;
        }
    }
    // При использовании `inline for` компилятор не знает, что был обработан каждый
    // возможный случай требующий явного указания `unreachable`.
    unreachable;
}

fn withSwitch(any: AnySlice) usize {
    return switch (any) {
        // При использовании `inline else` функция генерируется явно
        // в качестве желаемого параметра и компилятор может проверить, что
        // обработаны все возможные варианты.
        inline else => |slice| slice.len,
    };
}

test "inline for and inline else similarity" {
    const any = AnySlice{ .c = "hello" };
    try expect(withFor(any) == 5);
    try expect(withSwitch(any) == 5);
}
```
```bash
$ zig test test_inline_else.zig
1/1 test_inline_else.test.inline for and inline else similarity...OK
All 1 tests passed.
```

При использовании `inline switch` на объединении можно использовать дополнительный захват для получения значения тега
перечисления объединения.

```zig
const std = @import("std");
const expect = std.testing.expect;

const U = union(enum) {
    a: u32,
    b: f32,
};

fn getNum(u: U) u32 {
    switch (u) {
        // Здесь "num" - это известное во время выполнения значение, которое может быть либо
        // `u.a", либо "u.b", а "tag" - это значение тега известное во время компиляции.
        inline else => |num, tag| {
            if (tag == .b) {
                return @intFromFloat(num);
            }
            return num;
        },
    }
}

test "test" {
    const u = U{ .b = 42 };
    try expect(getNum(u) == 42);
}
```
```bash
$ zig test test_inline_switch_union_tag.zig
1/1 test_inline_switch_union_tag.test.test...OK
All 1 tests passed.
```

Смотрите также:
- [inline while](#inline-while)
- [inline for](#inline-for)


------------
### While

Цикл `while` используется для многократного выполнения выражения до тех пор пока какое-либо условие не перестанет быть
истинным.

```zig
const expect = @import("std").testing.expect;

test "while basic" {
    var i: usize = 0;
    while (i < 10) {
        i += 1;
    }
    try expect(i == 10);
}
```
```bash
$ zig test test_while.zig
1/1 test_while.test.while basic...OK
All 1 tests passed.
```

Используйте `break` для раннего выхода из цикла `while`.

```zig
test_while_break.zig

const expect = @import("std").testing.expect;

test "while break" {
    var i: usize = 0;
    while (true) {
        if (i == 10)
            break;
        i += 1;
    }
    try expect(i == 10);
}
```
```bash
$ zig test test_while_break.zig
1/1 test_while_break.test.while break...OK
All 1 tests passed.
```

Используйте `continue` чтобы вернуться к началу цикла.

```zig
const expect = @import("std").testing.expect;

test "while continue" {
    var i: usize = 0;
    while (true) {
        i += 1;
        if (i < 10)
            continue;
        break;
    }
    try expect(i == 10);
}
```
```bash
$ zig test test_while_continue.zig
1/1 test_while_continue.test.while continue...OK
All 1 tests passed.
```

Циклы `while` поддерживают выражение `continue` которое выполняется при продолжении цикла. Ключевое слово `continue`
соответствует этому выражению.

```zig
const expect = @import("std").testing.expect;

test "while loop continue expression" {
    var i: usize = 0;
    while (i < 10) : (i += 1) {}
    try expect(i == 10);
}

test "while loop continue expression, more complicated" {
    var i: usize = 1;
    var j: usize = 1;
    while (i * j < 2000) : ({
        i *= 2;
        j *= 3;
    }) {
        const my_ij = i * j;
        try expect(my_ij < 2000);
    }
}
```
```bash
$ zig test test_while_continue_expression.zig
1/2 test_while_continue_expression.test.while loop continue expression...OK
2/2 test_while_continue_expression.test.while loop continue expression, more complicated...OK
All 2 tests passed.
```

Циклы `while` являются выражениями. Результатом выражения является результат предложения `else` цикла `while` который
выполняется когда условие цикла `while` проверяется как ложное.

`break` как и `return` принимает значение параметра. Это результат выражения `while`. Когда вы выходите из цикла
`while`, ветвь `else` не вычисляется.

```zig
const expect = @import("std").testing.expect;

test "while else" {
    try expect(rangeHasNumber(0, 10, 5));
    try expect(!rangeHasNumber(0, 10, 15));
}

fn rangeHasNumber(begin: usize, end: usize, number: usize) bool {
    var i = begin;
    return while (i < end) : (i += 1) {
        if (i == number) {
            break true;
        }
    } else false;
}
```
```bash
$ zig test test_while_else.zig
1/1 test_while_else.test.while else...OK
All 1 tests passed.
```

#### Labeled while

Когда цикл [while](#while) имеет метку, то на неё можно ссылаться с помощью операторов `break` или `continue` из
вложенного цикла:

```zig
test "nested break" {
    outer: while (true) {
        while (true) {
            break :outer;
        }
    }
}

test "nested continue" {
    var i: usize = 0;
    outer: while (i < 10) : (i += 1) {
        while (true) {
            continue :outer;
        }
    }
}
```
```bash
$ zig test test_while_nested_break.zig
1/2 test_while_nested_break.test.nested break...OK
2/2 test_while_nested_break.test.nested continue...OK
All 2 tests passed.
```

#### while with Optionals

Так же, как и в выражениях [if](#if), циклы [while](#while) могут принимать опциональное значение в качестве условия и
получать полезную нагрузку. При обнаружении значения [null](#null) цикл завершается.

Когда в выражении [while](#while) присутствует синтаксис `|x|`, условие `while` должно иметь [опциональный
тип](#optional-type).

Ветвь `else` разрешена для опциональной итерации. В этом случае он будет выполнен при первом обнаруженном нулевом
значении.

```zig
const expect = @import("std").testing.expect;

test "while null capture" {
    var sum1: u32 = 0;
    numbers_left = 3;
    while (eventuallyNullSequence()) |value| {
        sum1 += value;
    }
    try expect(sum1 == 3);

    // захват нулевого значения с помощью блока else
    var sum2: u32 = 0;
    numbers_left = 3;
    while (eventuallyNullSequence()) |value| {
        sum2 += value;
    } else {
        try expect(sum2 == 3);
    }

    // захват нулевого значения с помощью выражения continue
    var i: u32 = 0;
    var sum3: u32 = 0;
    numbers_left = 3;
    while (eventuallyNullSequence()) |value| : (i += 1) {
        sum3 += value;
    }
    try expect(i == 3);
}

var numbers_left: u32 = undefined;
fn eventuallyNullSequence() ?u32 {
    return if (numbers_left == 0) null else blk: {
        numbers_left -= 1;
        break :blk numbers_left;
    };
}
```
```bash
$ zig test test_while_null_capture.zig
1/1 test_while_null_capture.test.while null capture...OK
All 1 tests passed.
```

#### while with Error Unions

Как и в случае с выражениями [if](#if), циклы [while](#while) могут принимать объединение с ошибкой в качестве условия и
получать полезную нагрузку или код ошибки. Когда результатом выполнения условия является код ошибки, вычисляется ветвь
`else` и цикл завершается.

Когда синтаксис `else |x|` присутствует в выражении [while](#while), условие `while` должно иметь [тип объединения
ошибки](#error-union-type).

```zig
const expect = @import("std").testing.expect;

test "while error union capture" {
    var sum1: u32 = 0;
    numbers_left = 3;
    while (eventuallyErrorSequence()) |value| {
        sum1 += value;
    } else |err| {
        try expect(err == error.ReachedZero);
    }
}

var numbers_left: u32 = undefined;

fn eventuallyErrorSequence() anyerror!u32 {
    return if (numbers_left == 0) error.ReachedZero else blk: {
        numbers_left -= 1;
        break :blk numbers_left;
    };
}
```
```bash
$ zig test test_while_error_capture.zig
1/1 test_while_error_capture.test.while error union capture...OK
All 1 tests passed.
```


#### inline while

Циклы [while](#while) можно встраивать. Это приводит к развертыванию цикла, что позволяет коду выполнять некоторые
действия которые работают только во время компиляции, например, использовать типы в качестве значений первого класса.

```zig
const expect = @import("std").testing.expect;

test "inline while loop" {
    comptime var i = 0;
    var sum: usize = 0;
    inline while (i < 3) : (i += 1) {
        const T = switch (i) {
            0 => f32,
            1 => i8,
            2 => bool,
            else => unreachable,
        };
        sum += typeNameLength(T);
    }
    try expect(sum == 9);
}

fn typeNameLength(comptime T: type) usize {
    return @typeName(T).len;
}
```
```bash
$ zig test test_inline_while.zig
1/1 test_inline_while.test.inline while loop...OK
All 1 tests passed.
```

Рекомендуется использовать `inline` циклы только по одной из этих причин:
- Для того чтобы семантика работала необходимо чтобы цикл выполнялся во время компиляции.
- У вас есть тест который докажет, что принудительное развертывание цикла таким образом значительно быстрее.


------------
### for

```zig
const expect = @import("std").testing.expect;

test "for basics" {
    const items = [_]i32{ 4, 5, 3, 4, 0 };
    var sum: i32 = 0;

    // Циклы for выполняют итерацию по срезам и массивам.
    for (items) |value| {
        // Поддерживает break и continue.
        if (value == 0) {
            continue;
        }
        sum += value;
    }
    try expect(sum == 16);

    // Чтобы выполнить итерацию по части среза выполните повторную итерацию.
    for (items[0..1]) |value| {
        sum += value;
    }
    try expect(sum == 20);

    // Чтобы получить доступ к индексу итерации, укажите также второе условие
    // в качестве второго значения для записи.
    var sum2: i32 = 0;
    for (items, 0..) |_, i| {
        try expect(@TypeOf(i) == usize);
        sum2 += @as(i32, @intCast(i));
    }
    try expect(sum2 == 10);

    // Для перебора последовательных целых чисел используйте синтаксис диапазонов.
    // Неограниченный диапазон всегда является ошибкой компиляции.
    var sum3: usize = 0;
    for (0..5) |i| {
        sum3 += i;
    }
    try expect(sum3 == 10);
}

test "multi object for" {
    const items = [_]usize{ 1, 2, 3 };
    const items2 = [_]usize{ 4, 5, 6 };
    var count: usize = 0;

    // Выполнить итерацию по нескольким объектам.
    // В начале цикла все длины должны быть равны, в противном случае будет обнаружено недопустимое поведение.
    for (items, items2) |i, j| {
        count += i + j;
    }

    try expect(count == 21);
}

test "for reference" {
    var items = [_]i32{ 3, 4, 2 };

    // Выполните итерацию по срезу по ссылке, указав, что значение захвата является указателем.
    for (&items) |*value| {
        value.* += 1;
    }

    try expect(items[0] == 4);
    try expect(items[1] == 5);
    try expect(items[2] == 3);
}

test "for else" {
    // for допускает присоединение к нему else, аналогично циклу while.
    const items = [_]?i32{ 3, 4, null, 5 };

    // Циклы for также могут использоваться в качестве выражений.
    // Аналогично циклам while, когда вы прерываете цикл for ветвь else не вычисляется.
    var sum: i32 = 0;
    const result = for (items) |value| {
        if (value != null) {
            sum += value.?;
        }
    } else blk: {
        try expect(sum == 12);
        break :blk sum;
    };
    try expect(result == 12);
}
```
```bash
$ zig test test_for.zig
1/4 test_for.test.for basics...OK
2/4 test_for.test.multi object for...OK
3/4 test_for.test.for reference...OK
4/4 test_for.test.for else...OK
All 4 tests passed.
```


#### Labeled for

Когда цикл [for](#for) отмечен как блок на него можно ссылаться из `break` или `continue` из вложенного цикла:

```zig
const std = @import("std");
const expect = std.testing.expect;

test "nested break" {
    var count: usize = 0;
    outer: for (1..6) |_| {
        for (1..6) |_| {
            count += 1;
            break :outer;
        }
    }
    try expect(count == 1);
}

test "nested continue" {
    var count: usize = 0;
    outer: for (1..9) |_| {
        for (1..6) |_| {
            count += 1;
            continue :outer;
        }
    }

    try expect(count == 8);
}
```
```bash
$ zig test test_for_nested_break.zig
1/2 test_for_nested_break.test.nested break...OK
2/2 test_for_nested_break.test.nested continue...OK
All 2 tests passed.
```


#### inline for

Циклы [for](#for) можно встраивать. Это приводит к развертыванию цикла, что позволяет коду выполнять некоторые действия,
которые работают только во время компиляции, например, использовать типы в качестве значений первого класса. Значение
захвата и значение итератора встроенных циклов `for` известны во время компиляции.

```zig
const expect = @import("std").testing.expect;

test "inline for loop" {
    const nums = [_]i32{ 2, 4, 6 };
    var sum: usize = 0;
    inline for (nums) |i| {
        const T = switch (i) {
            2 => f32,
            4 => i8,
            6 => bool,
            else => unreachable,
        };
        sum += typeNameLength(T);
    }
    try expect(sum == 9);
}

fn typeNameLength(comptime T: type) usize {
    return @typeName(T).len;
}
```
```bash
$ zig test test_inline_for.zig
1/1 test_inline_for.test.inline for loop...OK
All 1 tests passed.
```

Рекомендуется использовать `inline` циклы только по одной из этих причин:
- Для того чтобы семантика работала необходимо чтобы цикл выполнялся во время компиляции.
- У вас есть тест который докажет, что принудительное развертывание цикла таким образом значительно быстрее.


------------
### if

```zig
// У выражений if есть три варианта использования соответствующие трем типам:
// * bool
// * ?T
// * anyerror!T

const expect = @import("std").testing.expect;

test "if expression" {
    // Вместо if выражения можно использовать тернарное выражение
    const a: u32 = 5;
    const b: u32 = 4;
    const result = if (a != b) 47 else 3089;
    try expect(result == 47);
}

test "if boolean" {
    // Выражения if проверяют логические условия.
    const a: u32 = 5;
    const b: u32 = 4;
    if (a != b) {
        try expect(true);
    } else if (a == 9) {
        unreachable;
    } else {
        unreachable;
    }
}

test "if error union" {
    // Если выражения проверяются на наличие ошибок.
    // Обратите внимание на запись |err| в else.

    const a: anyerror!u32 = 0;
    if (a) |value| {
        try expect(value == 0);
    } else |err| {
        _ = err;
        unreachable;
    }

    const b: anyerror!u32 = error.BadValue;
    if (b) |value| {
        _ = value;
        unreachable;
    } else |err| {
        try expect(err == error.BadValue);
    }

    // else и |err| захват строго обязательны.
    if (a) |value| {
        try expect(value == 0);
    } else |_| {}

    // Чтобы проверить только значение ошибки, используйте пустое блочное выражение.
    if (b) |_| {} else |err| {
        try expect(err == error.BadValue);
    }

    // Доступ к значению осуществляется по ссылке с помощью захвата указателя.
    var c: anyerror!u32 = 3;
    if (c) |*value| {
        value.* = 9;
    } else |_| {
        unreachable;
    }

    if (c) |value| {
        try expect(value == 9);
    } else |_| {
        unreachable;
    }
}
```
```bash
$ zig test test_if.zig
1/3 test_if.test.if expression...OK
2/3 test_if.test.if boolean...OK
3/3 test_if.test.if error union...OK
All 3 tests passed.
```


#### if with Optionals

```zig
const expect = @import("std").testing.expect;

test "if optional" {
    // if выражения проверяются на наличие null.

    const a: ?u32 = 0;
    if (a) |value| {
        try expect(value == 0);
    } else {
        unreachable;
    }

    const b: ?u32 = null;
    if (b) |_| {
        unreachable;
    } else {
        try expect(true);
    }

    // Остальное не требуется.
    if (a) |value| {
        try expect(value == 0);
    }

    // Чтобы проверить только значение null используйте оператор двойного равенства.
    if (b == null) {
        try expect(true);
    }

    // Доступ к значению осуществляется по ссылке с помощью захвата указателя.
    var c: ?u32 = 3;
    if (c) |*value| {
        value.* = 2;
    }

    if (c) |value| {
        try expect(value == 2);
    } else {
        unreachable;
    }
}

test "if error union with optional" {
    // Выражения if проверяются на наличие ошибок перед развертыванием параметров.
    // Тип |optional_value| capture равен ?u32.

    const a: anyerror!?u32 = 0;
    if (a) |optional_value| {
        try expect(optional_value.? == 0);
    } else |err| {
        _ = err;
        unreachable;
    }

    const b: anyerror!?u32 = null;
    if (b) |optional_value| {
        try expect(optional_value == null);
    } else |_| {
        unreachable;
    }

    const c: anyerror!?u32 = error.BadValue;
    if (c) |optional_value| {
        _ = optional_value;
        unreachable;
    } else |err| {
        try expect(err == error.BadValue);
    }

    // Доступ к значению осуществляется по ссылке, каждый раз используя захват указателя.
    var d: anyerror!?u32 = 3;
    if (d) |*optional_value| {
        if (optional_value.*) |*value| {
            value.* = 9;
        }
    } else |_| {
        unreachable;
    }

    if (d) |optional_value| {
        try expect(optional_value.? == 9);
    } else |_| {
        unreachable;
    }
}
```
```bash
$ zig test test_if_optionals.zig
1/2 test_if_optionals.test.if optional...OK
2/2 test_if_optionals.test.if error union with optional...OK
All 2 tests passed.
```

Смотрите также:
- [Optionals](#optionals)
- [Errors](#errors)


------------
### defer

Выполняет выражение безусловно при выходе из области видимости.

```zig
const std = @import("std");
const expect = std.testing.expect;
const print = std.debug.print;

fn deferExample() !usize {
    var a: usize = 1;

    {
        defer a = 2;
        a = 1;
    }
    try expect(a == 2);

    a = 5;
    return a;
}

test "defer basics" {
    try expect((try deferExample()) == 5);
}
```
```bash
$ zig test test_defer.zig
1/1 test_defer.test.defer basics...OK
All 1 tests passed.
```

Отложенные выражения вычисляются в обратном порядке.

```zig
const std = @import("std");
const expect = std.testing.expect;
const print = std.debug.print;

test "defer unwinding" {
    print("\n", .{});

    defer {
        print("1 ", .{});
    }
    defer {
        print("2 ", .{});
    }
    if (false) {
        // отложенные запросы не запускаются, если они никогда не выполняются.
        defer {
            print("3 ", .{});
        }
    }
}
```
```bash
$ zig test defer_unwind.zig
1/1 defer_unwind.test.defer unwinding...
2 1 OK
All 1 tests passed.
```

Внутри выражения `defer` оператор `return` не допустим.

```zig
fn deferInvalidExample() !void {
    defer {
        return error.DeferError;
    }

    return error.DeferError;
}
```
```bash
$ zig test test_invalid_defer.zig
doc/langref/test_invalid_defer.zig:3:9: error: cannot return from defer expression
        return error.DeferError;
        ^~~~~~~~~~~~~~~~~~~~~~~
doc/langref/test_invalid_defer.zig:2:5: note: defer expression here
    defer {
    ^~~~~
```

Смотрите также:
- [Errors](#errors)


------------
### unreachable

В режимах [Debug](#debug) и [ReleaseSafe](#releasesafe) функция `unreachable` вызывает `panic` с сообщением `reached
unreachable code`.

В режимах [ReleaseFast](#releasefast) и [ReleaseSmall](#releasesmall) оптимизатор использует предположение о том, что
недоступный код никогда не будет запущен для выполнения оптимизаций.


#### Basics

```zig
// unreachable используется для утверждения, что поток управления никогда не достигнет
// определенного местоположения:
test "basic math" {
    const x = 1;
    const y = 2;
    if (x + y != 3) {
        unreachable;
    }
}
```
```bash
$ zig test test_unreachable.zig
1/1 test_unreachable.test.basic math...OK
All 1 tests passed.
```

Фактически, именно так реализован `std.debug.assert`:

```zig
// Вот как реализован std.debug.assert
fn assert(ok: bool) void {
    if (!ok) unreachable; // assertion failure
}

// Этот тест завершится неудачей, потому что мы попали в недостижимую область.
test "this will fail" {
    assert(false);
}
```
```bash
$ zig test test_assertion_failure.zig
1/1 test_assertion_failure.test.this will fail...thread 3571599 panic: reached unreachable code
/home/andy/src/zig/doc/langref/test_assertion_failure.zig:3:14: 0x103cd9d in assert (test)
    if (!ok) unreachable; // assertion failure
             ^
/home/andy/src/zig/doc/langref/test_assertion_failure.zig:8:11: 0x103cd5a in test.this will fail (test)
    assert(false);
          ^
/home/andy/src/zig/lib/compiler/test_runner.zig:157:25: 0x10479a0 in mainTerminal (test)
        if (test_fn.func()) |_| {
                        ^
/home/andy/src/zig/lib/compiler/test_runner.zig:37:28: 0x103dbbb in main (test)
        return mainTerminal();
                           ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x103d249 in posixCallMainAndExit (test)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x103cdb1 in _start (test)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
error: the following test command crashed:
/home/andy/src/zig/.zig-cache/o/a6b3ce5875a9e285c15739b2a1b30733/test
```


#### At Compile-Time

```zig
const assert = @import("std").debug.assert;

test "type of unreachable" {
    comptime {
        // Тип unreachable - noreturn.

        // Однако это утверждение все равно не удастся скомпилировать, поскольку
        // Выражения unreachable являются ошибками компиляции.

        assert(@TypeOf(unreachable) == noreturn);
    }
}
```
```bash
$ zig test test_comptime_unreachable.zig
doc/langref/test_comptime_unreachable.zig:10:16: error: unreachable code
        assert(@TypeOf(unreachable) == noreturn);
               ^~~~~~~~~~~~~~~~~~~~
doc/langref/test_comptime_unreachable.zig:10:24: note: control flow is diverted here
        assert(@TypeOf(unreachable) == noreturn);
                       ^~~~~~~~~~~
```

Смотрите также:
- [Zig Test](#zig-test)
- [Build Mode](#build-mode)
- [comptime](#comptime)


------------
### noreturn

`noreturn` - это тип:
- `break`
- `continue`
- `return`
- `unreachable`
- `while (true) {}`

При совместном разрешении типов в выражениях `if` или `switch`, тип `noreturn` совместим со всеми другими типами.
Рассмотрим:

```zig
fn foo(condition: bool, b: u32) void {
    const a = if (condition) b else return;
    _ = a;
    @panic("do something with a");
}
test "noreturn" {
    foo(false, 1);
}
```
```bash
$ zig test test_noreturn.zig
1/1 test_noreturn.test.noreturn...OK
All 1 tests passed.
```

Другим вариантом использования `noreturn` является функция `exit`:

```zig
const std = @import("std");
const builtin = @import("builtin");
const native_arch = builtin.cpu.arch;
const expect = std.testing.expect;

const WINAPI: std.builtin.CallingConvention = if (native_arch == .x86) .Stdcall else .C;
extern "kernel32" fn ExitProcess(exit_code: c_uint) callconv(WINAPI) noreturn;

test "foo" {
    const value = bar() catch ExitProcess(1);
    try expect(value == 1234);
}

fn bar() anyerror!u32 {
    return 1234;
}
```
```bash
$ zig test test_noreturn_from_exit.zig -target x86_64-windows --test-no-exec
```


------------
### Functions

```zig
const std = @import("std");
const builtin = @import("builtin");
const native_arch = builtin.cpu.arch;
const expect = std.testing.expect;

// Функции объявляются следующим образом
fn add(a: i8, b: i8) i8 {
    if (a == 0) {
        return b;
    }

    return a + b;
}

// Спецификатор экспорта делает функцию внешне видимой в сгенерированном
// объектном файле и заставляет ее использовать C ABI.
export fn sub(a: i8, b: i8) i8 {
    return a - b;
}

// Спецификатор extern используется для объявления функции, которая будет разрешена
// во время линковки при статической связывании или во время выполнения при динамическом связывании.
// Идентификатор заключенный в кавычки после ключевого слова extern указывает
// библиотеку в которой есть функция. (например, "c" -> libc.so)
// Спецификатор callconv изменяет соглашение о вызове функции.
const WINAPI: std.builtin.CallingConvention = if (native_arch == .x86) .Stdcall else .C;
extern "kernel32" fn ExitProcess(exit_code: u32) callconv(WINAPI) noreturn;
extern "c" fn atan2(a: f64, b: f64) f64;

// Встроенная функция @setCold сообщает оптимизатору, что функция вызывается редко.
fn abort() noreturn {
    @setCold(true);
    while (true) {}
}

// Из-за соглашения о голом вызове функция не имеет ни пролога, ни эпилога.
// Это может быть полезно при интеграции с assembly.
fn _start() callconv(.Naked) noreturn {
    abort();
}

// Соглашение о встроенном вызове требует, чтобы функция была встроена во все места вызова.
// Если функция не может быть встроена, это ошибка времени компиляции.
inline fn shiftLeftOne(a: u32) u32 {
    return a << 1;
}

// Спецификатор pub позволяет отображать функцию при импорте.
// В другом файле можно использовать @import и вызвать sub2
pub fn sub2(a: i8, b: i8) i8 {
    return a - b;
}

// Указатели на функции имеют префикс `*const `.
const Call2Op = *const fn (a: i8, b: i8) i8;
fn doOp(fnCall: Call2Op, op1: i8, op2: i8) i8 {
    return fnCall(op1, op2);
}

test "function" {
    try expect(doOp(add, 5, 6) == 11);
    try expect(doOp(sub2, 5, 6) == -1);
}
```
```bash
$ zig test test_functions.zig
1/1 test_functions.test.function...OK
All 1 tests passed.
```

Существует разница между телом функции и указателем на функцию. Тела функций - это типы, доступные только во [время
компиляции](#comptime), в то время как [указатели](#pointers) на функции могут быть известны во время выполнения.


#### Pass-by-value Parameters

Примитивные типы, такие как [целые числа](#integers) и [числа с плавающей запятой](#floats) передаваемые в качестве
параметров копируются, а затем копия будет доступна в теле функции. Это называется "передачей по значению". Копирование
примитивного типа по сути является бесплатным и обычно не требует ничего, кроме установки регистра.

Структуры, объединения и массивы иногда могут быть более эффективно переданы в качестве ссылки, поскольку копия может
быть сколь угодно дорогой в зависимости от размера. Когда эти типы передаются в качестве параметров, Zig может выбрать
копирование и передачу по значению или по ссылке в зависимости от того, какой способ по мнению Zig будет быстрее.
Отчасти это стало возможным благодаря тому, что параметры неизменяемы.

```zig
const Point = struct {
    x: i32,
    y: i32,
};

fn foo(point: Point) i32 {
    // Здесь "точка" может быть ссылкой или копией. Тело функции
    // может игнорировать разницу и рассматривать ее как значение. Будьте очень осторожны
    // принимая адрес параметра - он должен обрабатываться так, как если бы
    // адрес станет недействительным при возврате функции.
    return point.x + point.y;
}

const expect = @import("std").testing.expect;

test "pass struct to function" {
    try expect(foo(Point{ .x = 1, .y = 2 }) == 3);
}
```
```bash
$ zig test test_pass_by_reference_or_value.zig
1/1 test_pass_by_reference_or_value.test.pass struct to function...OK
All 1 tests passed.
```

Для внешних функций Zig следует C ABI для передачи структур и объединений по значению.


#### Function Parameter Type Inference

Параметры функции могут быть объявлены с использованием `anytype` вместо какого-то типа. В этом случае типы параметров
будут определены при вызове функции. Используйте [@TypeOf](#typeof) и [@TypeInfo](#typeinfo) для получения информации
о предполагаемом типе.

```zig
const expect = @import("std").testing.expect;

fn addFortyTwo(x: anytype) @TypeOf(x) {
    return x + 42;
}

test "fn type inference" {
    try expect(addFortyTwo(1) == 43);
    try expect(@TypeOf(addFortyTwo(1)) == comptime_int);
    const y: i64 = 2;
    try expect(addFortyTwo(y) == 44);
    try expect(@TypeOf(addFortyTwo(y)) == i64);
}
```
```bash
$ zig test test_fn_type_inference.zig
1/1 test_fn_type_inference.test.fn type inference...OK
All 1 tests passed.
```


#### inline fn

Добавление ключевого слова `inline` к определению функции приводит к тому, что функция становится семантически
встроенной в место вызова. Это не является намеком на то, что это может быть замечено при выполнении этапов оптимизации,
но влияет на типы и значения, задействованные в вызове функции.

В отличие от обычных вызовов функций, аргументы в месте вызова встроенной функции, известные во время компиляции,
обрабатываются как [параметры времени компиляции](#compile-time-parameters). Это потенциально может распространяться на
все возвращаемое значение:

```zig
test "inline function call" {
    if (foo(1200, 34) != 1234) {
        @compileError("bad");
    }
}

inline fn foo(a: i32, b: i32) i32 {
    return a + b;
}
```
```bash
$ zig test inline_call.zig
1/1 inline_call.test.inline function call...OK
All 1 tests passed.
```

Если функция `inline` удалена, тест завершается с ошибкой компиляции, а не с передачей.

Обычно лучше позволить компилятору решать, когда встраивать функцию, за исключением этих сценариев:
- Для изменения количества стековых "фреймов" в стеке вызовов в целях отладки.
- Чтобы время выполнения аргументов передавалось на возвращаемое значение функции, как в приведенном выше примере.
- Этого требуют измерения производительности в реальном мире.

Обратите внимание, что `inline` фактически ограничивает возможности компилятора. Это может негативно сказаться на
размере двоичного файла, скорости компиляции и даже производительности во время выполнения.


#### Function Reflection

```zig
const std = @import("std");
const math = std.math;
const testing = std.testing;

test "fn reflection" {
    try testing.expect(@typeInfo(@TypeOf(testing.expect)).Fn.params[0].type.? == bool);
    try testing.expect(@typeInfo(@TypeOf(testing.tmpDir)).Fn.return_type.? == testing.TmpDir);

    try testing.expect(@typeInfo(@TypeOf(math.Log2Int)).Fn.is_generic);
}
```
```bash
$ zig test test_fn_reflection.zig
1/1 test_fn_reflection.test.fn reflection...OK
All 1 tests passed.
```


------------
### Errors

#### Error Set Type

Набор ошибок похож на [перечисление](#enum). Однако каждому имени ошибки во всей компиляции присваивается целое число
без знака, большее 0. Вам разрешено объявлять одно и то же имя ошибки более одного раза, и если вы это сделаете ему
будет присвоено одно и то же целочисленное значение.

Тип набора ошибок по умолчанию равен `u16`, хотя, если максимальное количество различных значений ошибок указано с
помощью параметра командной строки `--error-limit [num]`, будет использоваться целочисленный тип с минимальным
количеством битов необходимым для представления всех значений ошибок.

Вы можете [преобразовать](#type-coercion) ошибку из подмножества в надмножество:

```zig
const std = @import("std");

const FileOpenError = error{
    AccessDenied,
    OutOfMemory,
    FileNotFound,
};

const AllocationError = error{
    OutOfMemory,
};

test "coerce subset to superset" {
    const err = foo(AllocationError.OutOfMemory);
    try std.testing.expect(err == FileOpenError.OutOfMemory);
}

fn foo(err: AllocationError) FileOpenError {
    return err;
}
```
```bash
$ zig test test_coerce_error_subset_to_superset.zig
1/1 test_coerce_error_subset_to_superset.test.coerce subset to superset...OK
All 1 tests passed.
```

Но вы не можете [преобразовать](#type-coercion) ошибку из надмножества в подмножество:

```zig
const FileOpenError = error{
    AccessDenied,
    OutOfMemory,
    FileNotFound,
};

const AllocationError = error{
    OutOfMemory,
};

test "coerce superset to subset" {
    foo(FileOpenError.OutOfMemory) catch {};
}

fn foo(err: FileOpenError) AllocationError {
    return err;
}
```
```bash

$ zig test test_coerce_error_superset_to_subset.zig
doc/langref/test_coerce_error_superset_to_subset.zig:16:12: error: expected type 'error{OutOfMemory}', found 'error{AccessDenied,OutOfMemory,FileNotFound}'
    return err;
           ^~~
doc/langref/test_coerce_error_superset_to_subset.zig:16:12: note: 'error.AccessDenied' not a member of destination error set
doc/langref/test_coerce_error_superset_to_subset.zig:16:12: note: 'error.FileNotFound' not a member of destination error set
doc/langref/test_coerce_error_superset_to_subset.zig:15:28: note: function return type declared here
fn foo(err: FileOpenError) AllocationError {
                           ^~~~~~~~~~~~~~~
referenced by:
    test.coerce superset to subset: doc/langref/test_coerce_error_superset_to_subset.zig:12:5
    remaining reference traces hidden; use '-freference-trace' to see all reference traces
```

Существует быстрый способ объявить набор ошибок, содержащий только одно значение, а затем получить это значение:

```zig
const err = error.FileNotFound;
```

Это эквивалентно:

```zig
const err = (error{FileNotFound}).FileNotFound;
```

Это становится полезным при использовании [предполагаемых наборов ошибок](#inferred-error-sets).


#### The Global Error Set

`anyerror` относится к глобальному набору ошибок. Это набор ошибок, который содержит все ошибки во всем модуле
компиляции. Это надмножество всех других наборов ошибок и не является подмножеством ни одного из них.

Вы можете [преобразовать](#type-coercion) любой набор ошибок к глобальному, а также явно преобразовать ошибку из
глобального набора ошибок к неглобальному. При этом добавляется утверждение на уровне языка, чтобы убедиться, что
значение ошибки действительно находится в целевом наборе ошибок.

Как правило, следует избегать глобального набора ошибок, поскольку он не позволяет компилятору узнать какие ошибки
возможны во время компиляции. Знание набора ошибок во время компиляции лучше для создания документации и полезных
сообщений об ошибках таких как забывание значения возможной ошибки в [switch](#switch).


#### Error Union Type

Тип набора ошибок и обычный тип могут быть объединены с помощью бинарного оператора `!` для создания типа объединения
ошибок. Скорее всего, вы будете использовать тип объединения ошибок чаще, чем тип набора ошибок сам по себе.

Вот функция для преобразования строки в 64-разрядное целое число:

```zig
const std = @import("std");
const maxInt = std.math.maxInt;

pub fn parseU64(buf: []const u8, radix: u8) !u64 {
    var x: u64 = 0;

    for (buf) |c| {
        const digit = charToDigit(c);

        if (digit >= radix) {
            return error.InvalidChar;
        }

        // x *= radix
        var ov = @mulWithOverflow(x, radix);
        if (ov[1] != 0) return error.OverFlow;

        // x += digit
        ov = @addWithOverflow(ov[0], digit);
        if (ov[1] != 0) return error.OverFlow;
        x = ov[0];
    }

    return x;
}

fn charToDigit(c: u8) u8 {
    return switch (c) {
        '0'...'9' => c - '0',
        'A'...'Z' => c - 'A' + 10,
        'a'...'z' => c - 'a' + 10,
        else => maxInt(u8),
    };
}

test "parse u64" {
    const result = try parseU64("1234", 10);
    try std.testing.expect(result == 1234);
}
```
```bash
$ zig test error_union_parsing_u64.zig
1/1 error_union_parsing_u64.test.parse u64...OK
All 1 tests passed.
```

Обратите внимание, что тип возвращаемого значения - `!u64`. Это означает, что функция возвращает либо 64-разрядное
целое число без знака, либо ошибку. Мы не указали значение ошибки слева от `!` поэтому выводится значение ошибки.

В определении функции вы можете увидеть несколько инструкций которые возвращают ошибку, а внизу оператор возврата,
который возвращает `u64`. Оба типа [преобразуются](#type-coercion) к `anyerror!u64`.

Как выглядит использование этой функции, зависит от того, что вы пытаетесь сделать.
Одно из следующих действий:
- Вы хотите указать значение по умолчанию если оно вернуло ошибку.
- Если оно вернуло ошибку вы хотите вернуть ту же ошибку.
- Вы знаете с полной уверенностью, что оно не вернет ошибку, поэтому хотите безоговорочно отменить ее.
- Вы хотите предпринять различные действия для каждой возможной ошибки.


##### catch

Если вы хотите указать значение по умолчанию, вы можете использовать двоичный оператор `catch`:

```zig
const parseU64 = @import("error_union_parsing_u64.zig").parseU64;

fn doAThing(str: []u8) void {
    const number = parseU64(str, 10) catch 13;
    _ = number; // ...
}
```

В этом коде `number` будет равно успешно обработанной строке или значение по умолчанию равно 13. Тип правой части
бинарного оператора `catch` должен совпадать с типом разворачиваемого объединения ошибок или иметь тип `noreturn`.

Если вы хотите предоставить значение по умолчанию с помощью `catch` после выполнения некоторой логики, вы можете
объединить `catch` с именованными [блоками](#blocks):

```zig
const parseU64 = @import("error_union_parsing_u64.zig").parseU64;

fn doAThing(str: []u8) void {
    const number = parseU64(str, 10) catch blk: {
        // делать что-то
        break :blk 13;
    };
    _ = number; // теперь number инициализирован
}
```


##### try

Допустим, вы хотели вернуть ошибку если она у вас появилась, в противном случае продолжайте использовать логику
функции:

```zig
const parseU64 = @import("error_union_parsing_u64.zig").parseU64;

fn doAThing(str: []u8) !void {
    const number = parseU64(str, 10) catch |err| return err;
    _ = number; // ...
}
```

Для этого есть короткий путь. Выражение `try`:

```zig
const parseU64 = @import("error_union_parsing_u64.zig").parseU64;

fn doAThing(str: []u8) !void {
    const number = try parseU64(str, 10);
    _ = number; // ...
}
```

`try` вычисляет выражение объединения с ошибкой. Если это ошибка то текущая функция возвращает результат с той же
ошибкой. В противном случае результатом выражения будет развернутое значение.

Возможно, вы с полной уверенностью знаете, что выражение никогда не будет ошибкой. В этом случае вы можете поступить
следующим образом:

```zig
const number = parseU64("1234", 10) catch unreachable;
```

Здесь мы точно знаем, что "1234" будет успешно обработано. Поэтому мы помещаем значение `unreachable` в правую часть.
`unreachable` вызывает панику в режимах [Debug](#debug) и [ReleaseSafe](#releasesafe) и неопределенное поведение в
режимах [ReleaseFast](#releasefast) и [ReleaseSmall](#releasesmall). Итак, во время отладки приложения если бы произошла
непредвиденная ошибка, приложение завершило бы работу соответствующим образом.

Возможно, вы захотите выполнить разные действия в каждой ситуации. Для этого мы объединяем выражения [if](#if) и
[switch](#switch):

```zig
fn doAThing(str: []u8) void {
    if (parseU64(str, 10)) |number| {
        doSomethingWithNumber(number);
    } else |err| switch (err) {
        error.Overflow => {
            // обработать переполнение...
        },
        // мы обещаем, что InvalidChar не произойдет (или произойдет сбой в режиме отладки, если это произойдет)
        error.InvalidChar => unreachable,
    }
}
```

Наконец, возможно вам захочется обработать только некоторые ошибки. Для этого вы можете отобразить необработанные
ошибки в случае `else`, который теперь содержит более узкий набор ошибок:

```zig
fn doAnotherThing(str: []u8) error{InvalidChar}!void {
    if (parseU64(str, 10)) |number| {
        doSomethingWithNumber(number);
    } else |err| switch (err) {
        error.Overflow => {
            // обработать переполнение...
        },
        else => |leftover_err| return leftover_err,
    }
}
```

Вы должны использовать синтаксис захвата переменной. Если переменная вам не нужна, вы можете захватить ее с помощью `_`
и избежать `switch`.

```zig
fn doADifferentThing(str: []u8) void {
    if (parseU64(str, 10)) |number| {
        doSomethingWithNumber(number);
    } else |_| {
        // делай, как тебе хочется
    }
}
```


##### errdefer

Другим компонентом обработки ошибок являются операторы `defer`. В дополнение к безусловному [defer](#defer), в Zig есть
`errdefer`, который вычисляет "отложенное" выражение на пути выхода из блока тогда и только тогда, когда функция
возвращает сообщение об ошибке из блока.

Пример:
```zig
fn createFoo(param: i32) !Foo {
    const foo = try tryToAllocateFoo();
    // теперь мы выделили foo, нам нужно освободить его если функция завершится ошибкой.
    // но мы хотим вернуть его, если функция завершится успешно.
    errdefer deallocateFoo(foo);

    const tmp_buf = allocateTmpBuffer() orelse return error.OutOfMemory;
    // tmp_buf - это действительно временный ресурс, и мы, безусловно, хотим его очистить
    // до того, как этот блок покинет область видимости
    defer deallocateTmpBuffer(tmp_buf);

    if (param > 1337) return error.InvalidParam;

    // здесь функция errdefer не будет запущена, так как мы возвращаем результат успешной работы функции.
    // но функция defer будет запущена!
    return foo;
}
```

Самое приятное в этом то, что вы получаете надежную обработку ошибок без многословия и когнитивных затрат, связанных с
попытками убедиться, что пройден каждый путь к выходу. Код освобождения всегда следует непосредственно за кодом
выделения.


##### Common errdefer Slip-Ups

Следует отметить, что операторы `errdefer` действуют только до конца блока в котором они записаны и следовательно не
выполняются если ошибка возвращается за пределами этого блока:

```zig
onst std = @import("std");
const Allocator = std.mem.Allocator;

const Foo = struct {
    data: u32,
};

fn tryToAllocateFoo(allocator: Allocator) !*Foo {
    return allocator.create(Foo);
}

fn deallocateFoo(allocator: Allocator, foo: *Foo) void {
    allocator.destroy(foo);
}

fn getFooData() !u32 {
    return 666;
}

fn createFoo(allocator: Allocator, param: i32) !*Foo {
    const foo = getFoo: {
        var foo = try tryToAllocateFoo(allocator);
        errdefer deallocateFoo(allocator, foo); // Действует только до конца getFoo

        // Вызывает deallocateFoo при ошибке
        foo.data = try getFooData();

        break :getFoo foo;
    };

    // Выходит за рамки определения ошибки, поэтому
    // deallocateFoo здесь вызываться не будет
    if (param > 1337) return error.InvalidParam;

    return foo;
}

test "createFoo" {
    try std.testing.expectError(error.InvalidParam, createFoo(std.testing.allocator, 2468));
}
```
```bash
$ zig test test_errdefer_slip_ups.zig
1/1 test_errdefer_slip_ups.test.createFoo...OK
[gpa] (err): memory address 0x7f11f521a000 leaked:
/home/andy/src/zig/doc/langref/test_errdefer_slip_ups.zig:9:28: 0x103d3cf in tryToAllocateFoo (test)
    return allocator.create(Foo);
                           ^
/home/andy/src/zig/doc/langref/test_errdefer_slip_ups.zig:22:39: 0x103d5e5 in createFoo (test)
        var foo = try tryToAllocateFoo(allocator);
                                      ^
/home/andy/src/zig/doc/langref/test_errdefer_slip_ups.zig:39:62: 0x103d82d in test.createFoo (test)
    try std.testing.expectError(error.InvalidParam, createFoo(std.testing.allocator, 2468));
                                                             ^
/home/andy/src/zig/lib/compiler/test_runner.zig:157:25: 0x104d2a0 in mainTerminal (test)
        if (test_fn.func()) |_| {
                        ^
/home/andy/src/zig/lib/compiler/test_runner.zig:37:28: 0x104340b in main (test)
        return mainTerminal();
                           ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x103f9f9 in posixCallMainAndExit (test)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x103f561 in _start (test)
    asm volatile (switch (native_arch) {
    ^

All 1 tests passed.
1 errors were logged.
1 tests leaked memory.
error: the following test command failed with exit code 1:
/home/andy/src/zig/.zig-cache/o/58c079cb550addefaa354f72d736afd7/test
```

Чтобы гарантировать, что `deallocateFoo` правильно вызывается при возврате ошибки, вы должны добавить `errdefer` вне
блока:

```zig
const std = @import("std");
const Allocator = std.mem.Allocator;

const Foo = struct {
    data: u32,
};

fn tryToAllocateFoo(allocator: Allocator) !*Foo {
    return allocator.create(Foo);
}

fn deallocateFoo(allocator: Allocator, foo: *Foo) void {
    allocator.destroy(foo);
}

fn getFooData() !u32 {
    return 666;
}

fn createFoo(allocator: Allocator, param: i32) !*Foo {
    const foo = getFoo: {
        var foo = try tryToAllocateFoo(allocator);
        errdefer deallocateFoo(allocator, foo);

        foo.data = try getFooData();

        break :getFoo foo;
    };
    // Это продолжается до конца выполнения функции
    errdefer deallocateFoo(allocator, foo);

    // Ошибка теперь корректно обрабатывается errdefer
    if (param > 1337) return error.InvalidParam;

    return foo;
}

test "createFoo" {
    try std.testing.expectError(error.InvalidParam, createFoo(std.testing.allocator, 2468));
}
```
```bash
$ zig test test_errdefer_block.zig
1/1 test_errdefer_block.test.createFoo...OK
All 1 tests passed.
```

Тот факт, что определения ошибок сохраняются только для того блока, в котором они объявлены, особенно важен при
использовании циклов:

```zig
const std = @import("std");
const Allocator = std.mem.Allocator;

const Foo = struct { data: *u32 };

fn getData() !u32 {
    return 666;
}

fn genFoos(allocator: Allocator, num: usize) ![]Foo {
    const foos = try allocator.alloc(Foo, num);
    errdefer allocator.free(foos);

    for (foos, 0..) |*foo, i| {
        foo.data = try allocator.create(u32);
        // Эта ошибка не сохраняется между итерациями
        errdefer allocator.destroy(foo.data);

        // Данные за первые 3 месяца будут утечены
        if (i >= 3) return error.TooManyFoos;

        foo.data.* = try getData();
    }

    return foos;
}

test "genFoos" {
    try std.testing.expectError(error.TooManyFoos, genFoos(std.testing.allocator, 5));
}
```
```bash
$ zig test test_errdefer_loop_leak.zig
1/1 test_errdefer_loop_leak.test.genFoos...OK
[gpa] (err): memory address 0x7f57c7578000 leaked:
/home/andy/src/zig/doc/langref/test_errdefer_loop_leak.zig:15:40: 0x103d7a6 in genFoos (test)
        foo.data = try allocator.create(u32);
                                       ^
/home/andy/src/zig/doc/langref/test_errdefer_loop_leak.zig:29:59: 0x103e0dd in test.genFoos (test)
    try std.testing.expectError(error.TooManyFoos, genFoos(std.testing.allocator, 5));
                                                          ^
/home/andy/src/zig/lib/compiler/test_runner.zig:157:25: 0x104e010 in mainTerminal (test)
        if (test_fn.func()) |_| {
                        ^
/home/andy/src/zig/lib/compiler/test_runner.zig:37:28: 0x1043eab in main (test)
        return mainTerminal();
                           ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x10402a9 in posixCallMainAndExit (test)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x103fe11 in _start (test)
    asm volatile (switch (native_arch) {
    ^

[gpa] (err): memory address 0x7f57c7578004 leaked:
/home/andy/src/zig/doc/langref/test_errdefer_loop_leak.zig:15:40: 0x103d7a6 in genFoos (test)
        foo.data = try allocator.create(u32);
                                       ^
/home/andy/src/zig/doc/langref/test_errdefer_loop_leak.zig:29:59: 0x103e0dd in test.genFoos (test)
    try std.testing.expectError(error.TooManyFoos, genFoos(std.testing.allocator, 5));
                                                          ^
/home/andy/src/zig/lib/compiler/test_runner.zig:157:25: 0x104e010 in mainTerminal (test)
        if (test_fn.func()) |_| {
                        ^
/home/andy/src/zig/lib/compiler/test_runner.zig:37:28: 0x1043eab in main (test)
        return mainTerminal();
                           ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x10402a9 in posixCallMainAndExit (test)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x103fe11 in _start (test)
    asm volatile (switch (native_arch) {
    ^

[gpa] (err): memory address 0x7f57c7578008 leaked:
/home/andy/src/zig/doc/langref/test_errdefer_loop_leak.zig:15:40: 0x103d7a6 in genFoos (test)
        foo.data = try allocator.create(u32);
                                       ^
/home/andy/src/zig/doc/langref/test_errdefer_loop_leak.zig:29:59: 0x103e0dd in test.genFoos (test)
    try std.testing.expectError(error.TooManyFoos, genFoos(std.testing.allocator, 5));
                                                          ^
/home/andy/src/zig/lib/compiler/test_runner.zig:157:25: 0x104e010 in mainTerminal (test)
        if (test_fn.func()) |_| {
                        ^
/home/andy/src/zig/lib/compiler/test_runner.zig:37:28: 0x1043eab in main (test)
        return mainTerminal();
                           ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x10402a9 in posixCallMainAndExit (test)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x103fe11 in _start (test)
    asm volatile (switch (native_arch) {
    ^

All 1 tests passed.
3 errors were logged.
1 tests leaked memory.
error: the following test command failed with exit code 1:
/home/andy/src/zig/.zig-cache/o/29fcda275b7c426418534b679850fa2e/test
```

С кодом, который выделяет данные в цикле необходимо соблюдать особую осторожность, чтобы убедиться в отсутствии утечки
памяти при возврате ошибки:

```zig
const std = @import("std");
const Allocator = std.mem.Allocator;

const Foo = struct { data: *u32 };

fn getData() !u32 {
    return 666;
}

fn genFoos(allocator: Allocator, num: usize) ![]Foo {
    const foos = try allocator.alloc(Foo, num);
    errdefer allocator.free(foos);

    // Используется для отслеживания того, сколько foo было инициализировано
    // (включая распределение их данных)
    var num_allocated: usize = 0;
    errdefer for (foos[0..num_allocated]) |foo| {
        allocator.destroy(foo.data);
    };
    for (foos, 0..) |*foo, i| {
        foo.data = try allocator.create(u32);
        num_allocated += 1;

        if (i >= 3) return error.TooManyFoos;

        foo.data.* = try getData();
    }

    return foos;
}

test "genFoos" {
    try std.testing.expectError(error.TooManyFoos, genFoos(std.testing.allocator, 5));
}
```
```bash
$ zig test test_errdefer_loop.zig
1/1 test_errdefer_loop.test.genFoos...OK
All 1 tests passed.
```

Еще пара лакомых кусочков об обработке ошибок:
- Эти примитивы обеспечивают достаточную выразительность, так что вполне практично если ошибка проверки на наличие
ошибки является ошибкой компиляции. Если вы действительно хотите проигнорировать ошибку, вы можете добавить `catch`
`unreachable` и получить дополнительное преимущество от сбоя в режимах [Debug](#debug) и [ReleaseSafe](#releasesafe),
если ваше предположение было неверным.
- Поскольку Zig понимает типы ошибок, он может предварительно взвесить ветви в пользу того, что ошибки не возникают.
Просто небольшое преимущество в оптимизации, недоступное на других языках.

Смотрите также:
- [defer](#defer)
- [if](#if)
- [switch](#switch)


Объединение с ошибкой создается с помощью бинарного оператора `!`. Вы можете использовать "рефлексию" во время
компиляции для доступа к дочернему типу объединения с ошибкой:

```zig
const expect = @import("std").testing.expect;

test "error union" {
    var foo: anyerror!i32 = undefined;

    // Извлекать из дочернего типа объединения ошибок:
    foo = 1234;

    // Извлекать из набора ошибок:
    foo = error.SomeError;

    // Используйте рефлексию во время компиляции для доступа к типу полезной нагрузки объединения ошибок:
    try comptime expect(@typeInfo(@TypeOf(foo)).ErrorUnion.payload == i32);

    // Используйте рефлексию во время компиляции для доступа к типу набора объединения ошибок:
    try comptime expect(@typeInfo(@TypeOf(foo)).ErrorUnion.error_set == anyerror);
}
```
```bash
$ zig test test_error_union.zig
1/1 test_error_union.test.error union...OK
All 1 tests passed.
```

##### Merging Error Sets

Используйте оператор `||` чтобы объединить два набора ошибок. Результирующий набор ошибок содержит ошибки из обоих
наборов ошибок. Комментарии к документам в левой части переопределяют комментарии к документам в правой части. В этом
примере комментарием к документу для `C.PathNotFound` является `комментарий к документу A`.

Это особенно полезно для функций, которые возвращают разные наборы ошибок в зависимости от ветвей [времени
компиляции](#comptime). Например, стандартная библиотека Zig использует `LinuxFileOpenError || WindowsFileOpenError` для
набора ошибок при открытии файлов.

```zig
const A = error{
    NotDir,

    /// A комментарий к документу
    PathNotFound,
};
const B = error{
    OutOfMemory,

    /// B комментарий к документу
    PathNotFound,
};

const C = A || B;

fn foo() C!void {
    return error.NotDir;
}

test "merge error sets" {
    if (foo()) {
        @panic("unexpected");
    } else |err| switch (err) {
        error.OutOfMemory => @panic("unexpected"),
        error.PathNotFound => @panic("unexpected"),
        error.NotDir => {},
    }
}
```
```bash
$ zig test test_merging_error_sets.zig
1/1 test_merging_error_sets.test.merge error sets...OK
All 1 tests passed.
```


##### Inferred Error Sets

Поскольку многие функции в Zig возвращают возможную ошибку, Zig поддерживает вывод набора ошибок. Чтобы вывести набор
ошибок для функции, добавьте оператор `!` к типу возвращаемого значения функции, например `!T`:

```zig
// С установленным выводом об ошибке
pub fn add_inferred(comptime T: type, a: T, b: T) !T {
    const ov = @addWithOverflow(a, b);
    if (ov[1] != 0) return error.Overflow;
    return ov[0];
}

// С явно заданной ошибкой
pub fn add_explicit(comptime T: type, a: T, b: T) Error!T {
    const ov = @addWithOverflow(a, b);
    if (ov[1] != 0) return error.Overflow;
    return ov[0];
}

const Error = error{
    Overflow,
};

const std = @import("std");

test "inferred error set" {
    if (add_inferred(u8, 255, 1)) |_| unreachable else |err| switch (err) {
        error.Overflow => {}, // ok
    }
}
```
```bash
$ zig test test_inferred_error_sets.zig
1/1 test_inferred_error_sets.test.inferred error set...OK
All 1 tests passed.
```

Когда функция содержит набор выводимых ошибок, эта функция становится универсальной и следовательно, с ней становится
сложнее выполнять определенные действия, такие как получение указателя на функцию или создание набора ошибок который
был бы согласован для разных целей сборки. Кроме того, выводимые наборы ошибок несовместимы с рекурсией.

В таких ситуациях рекомендуется использовать явный набор ошибок. Как правило, вы можете начать с пустого набора ошибок и
позволить ошибкам компиляции направлять вас к завершению набора.

Эти ограничения могут быть устранены в будущей версии Zig.


#### Error Return Traces

Трассировка возврата ошибок показывает все точки в коде, в которых вызывающая функция вернула ошибку. Это позволяет
использовать [try](#try) повсеместно и при этом знать, что произошло, если ошибка "вышла" за пределы вашего приложения.

```zig
pub fn main() !void {
    try foo(12);
}

fn foo(x: i32) !void {
    if (x >= 5) {
        try bar();
    } else {
        try bang2();
    }
}

fn bar() !void {
    if (baz()) {
        try quux();
    } else |err| switch (err) {
        error.FileNotFound => try hello(),
    }
}

fn baz() !void {
    try bang1();
}

fn quux() !void {
    try bang2();
}

fn hello() !void {
    try bang2();
}

fn bang1() !void {
    return error.FileNotFound;
}

fn bang2() !void {
    return error.PermissionDenied;
}
```
```bash
$ zig build-exe error_return_trace.zig
$ ./error_return_trace
error: PermissionDenied
/home/andy/src/zig/doc/langref/error_return_trace.zig:34:5: 0x1034e08 in bang1 (error_return_trace)
    return error.FileNotFound;
    ^
/home/andy/src/zig/doc/langref/error_return_trace.zig:22:5: 0x1034f13 in baz (error_return_trace)
    try bang1();
    ^
/home/andy/src/zig/doc/langref/error_return_trace.zig:38:5: 0x1034f38 in bang2 (error_return_trace)
    return error.PermissionDenied;
    ^
/home/andy/src/zig/doc/langref/error_return_trace.zig:30:5: 0x1034fa3 in hello (error_return_trace)
    try bang2();
    ^
/home/andy/src/zig/doc/langref/error_return_trace.zig:17:31: 0x103505a in bar (error_return_trace)
        error.FileNotFound => try hello(),
                              ^
/home/andy/src/zig/doc/langref/error_return_trace.zig:7:9: 0x1035140 in foo (error_return_trace)
        try bar();
        ^
/home/andy/src/zig/doc/langref/error_return_trace.zig:2:5: 0x1035198 in main (error_return_trace)
    try foo(12);
    ^
```

Посмотрите внимательно на этот пример. Это не трассировка стека.

Вы можете видеть, что последняя ошибка которая возникла, была `PermissionDenied`, но исходной ошибкой, из-за которой все
это началось был `FileNotFound`. В функции `bar` код обрабатывает исходный код ошибки, а затем возвращает другой, из
оператора `switch`. Трассировка возврата ошибок показывает это наглядно, в то время как трассировка стека выглядела бы
следующим образом:

```zig
pub fn main() void {
    foo(12);
}

fn foo(x: i32) void {
    if (x >= 5) {
        bar();
    } else {
        bang2();
    }
}

fn bar() void {
    if (baz()) {
        quux();
    } else {
        hello();
    }
}

fn baz() bool {
    return bang1();
}

fn quux() void {
    bang2();
}

fn hello() void {
    bang2();
}

fn bang1() bool {
    return false;
}

fn bang2() void {
    @panic("PermissionDenied");
}
```
```bash
$ zig build-exe stack_trace.zig
$ ./stack_trace
thread 3570764 panic: PermissionDenied
/home/andy/src/zig/doc/langref/stack_trace.zig:38:5: 0x1039320 in bang2 (stack_trace)
    @panic("PermissionDenied");
    ^
/home/andy/src/zig/doc/langref/stack_trace.zig:30:10: 0x1068bd8 in hello (stack_trace)
    bang2();
         ^
/home/andy/src/zig/doc/langref/stack_trace.zig:17:14: 0x10392fc in bar (stack_trace)
        hello();
             ^
/home/andy/src/zig/doc/langref/stack_trace.zig:7:12: 0x103721c in foo (stack_trace)
        bar();
           ^
/home/andy/src/zig/doc/langref/stack_trace.zig:2:8: 0x103519d in main (stack_trace)
    foo(12);
       ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034a49 in posixCallMainAndExit (stack_trace)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x10345b1 in _start (stack_trace)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```

Здесь трассировка стека не объясняет, как поток управления в `bar` дошел до вызова функции `hello()`. Чтобы выяснить
это нужно было бы открыть отладчик или дополнительно протестировать приложение. С другой стороны, трассировка возврата
ошибки показывает как именно возникла ошибка.

Эта функция отладки упрощает быструю итерацию кода который надежно обрабатывает все условия ошибки. Это означает, что
разработчикам Zig, естественно придется писать корректный, надежный код чтобы ускорить процесс разработки.

Трассировка возвращаемых ошибок включена по умолчанию в сборках [Debug](#debug) и [ReleaseSafe](#releasesafe) и
отключена по умолчанию в сборках [ReleaseFast](#releasefast) и сборках [ReleaseSmall](#releasesmall).

Существует несколько способов активировать эту функцию отслеживания возврата ошибок:
- Возврат ошибки из `main`
- Ошибка попадает в `catch unreachable`, и вы не переопределили стандартный обработчик паники
- Используйте [errorReturnTrace](#errorreturntrace) для доступа к текущей возвращаемой трассировке. Вы можете
использовать `std.debug.dumpStackTrace`, чтобы распечатать его. Эта функция возвращает значение [null](#null), известное
по времени компиляции, при сборке без поддержки трассировки возврата ошибок.


##### Implementation Details

Чтобы проанализировать затраты на производительность, рассмотрим два случая:
- когда ошибки не возвращаются
- когда ошибки возвращаются

Для случая, когда ошибки не возвращаются, стоимость составляет одну операцию записи в память, только в первой
недоступной функции в графе вызовов, которая вызывает недоступную функцию, т.е. когда функция возвращающая `void`,
вызывает функцию возвращающую ошибку. Это необходимо для инициализации данной структуры в стековой памяти:

```zig
pub const StackTrace = struct {
    index: usize,
    instruction_addresses: [N]usize,
};
```

Здесь N - максимальная глубина вызова функции, определенная с помощью анализа графа вызовов. Рекурсия игнорируется и
засчитывается за 2.

Указатель на `StackTrace` передается в качестве секретного параметра каждой функции, которая может возвращать ошибку, но
это всегда первый параметр, поэтому он, скорее всего, может находиться в регистре и оставаться там.

Вот и все для пути когда ошибок не возникает. Это практически бесплатно с точки зрения производительности.

При генерации кода для функции, возвращающей ошибку, непосредственно перед оператором `return` (только для операторов
`return`, возвращающих ошибки) Zig генерирует вызов этой функции:

```zig
// помечен как "не встроенный" в LLVM IR
fn __zig_return_error(stack_trace: *StackTrace) void {
    stack_trace.instruction_addresses[stack_trace.index] = @returnAddress();
    stack_trace.index = (stack_trace.index + 1) % N;
}
```

Стоимость составляет 2 математические операции плюс некоторые операции чтения и записи в память. Объем памяти к которой
осуществляется доступ ограничен и должен оставаться в кэше на время возврата ошибки.

Что касается размера кода, то 1 вызов функции перед оператором `return` не представляет большой проблемы. Несмотря на
это, у меня есть [план](https://github.com/ziglang/zig/issues/690) сделать вызов `__zig_return_error` хвостовым вызовом,
что фактически сведет стоимость размера кода к нулю. То, что является оператором `return` в коде без отслеживания
возврата ошибки может стать инструкцией перехода в коде с отслеживанием возврата ошибки.


------------
### Optionals

Одна из областей в которой Zig обеспечивает безопасность без ущерба для эффективности или удобочитаемости - это
опциональный тип.

Вопросительный знак символизирует опциональный тип. Вы можете преобразовать тип в опциональный, поставив перед ним знак
вопроса, вот так:

```zig
// обычное целое число
const normal_int: i32 = 1234;

// опциональное целое число
const optional_int: ?i32 = 5678;
```

Теперь переменная `optional_int` может быть `i32` или `null`.

Вместо целых чисел давайте поговорим об указателях. Ссылки на `Null` являются источником многих исключений во время
выполнения, и их даже обвиняют в том, что они являются [худшей ошибкой в
информатике](https://www.lucidchart.com/techblog/2015/08/31/the-worst-mistake-of-computer-science).

В Zig их нет.

Вместо этого вы можете использовать опциональный указатель. Это скрытно преобразуется в обычный указатель, поскольку мы
знаем, что можем использовать 0 в качестве нулевого значения для опционального типа. Но компилятор может проверить вашу
работу и убедиться, что вы не присваиваете `null` тому, что не может быть `null`.

Обычно недостатком отсутствия `null` является то, что это делает написание кода более многословным. Но давайте сравним
эквивалентный код на C и Zig-код.

Задача: вызвать `malloc`, если результат равен `null`, то верните `null`.

```c
// прототип malloc включен для справки
void *malloc(size_t size);

struct Foo *do_a_thing(void) {
    char *ptr = malloc(1234);
    if (!ptr) return NULL;
    // ...
}
```
```zig
// прототип malloc включен для справки
extern fn malloc(size: usize) ?[*]u8;

fn doAThing() ?*Foo {
    const ptr = malloc(1234) orelse return null;
    _ = ptr; // ...
}
```

Здесь Zig по крайней мере так же удобен, если не больше чем C. И тип "ptr" - это `[*]u8`, а не `?[*]u8`. Ключевое слово
`orelse` разворачивает опциональный тип и поэтому значение `ptr` гарантированно будет ненулевым везде, где он
используется в функции.

Другая форма проверки на значение `NULL`, которую вы можете увидеть, выглядит следующим образом:

```c
void do_a_thing(struct Foo *foo) {
    // сделать кое-что еще

    if (foo) {
        do_something_with_foo(foo);
    }

    // сделать кое-что еще
}
```

В Zig вы можете сделать то же самое:

```zig
const Foo = struct {};
fn doSomethingWithFoo(foo: *Foo) void {
    _ = foo;
}

fn doAThing(optional_foo: ?*Foo) void {
    // сделать кое-что еще

    if (optional_foo) |foo| {
        doSomethingWithFoo(foo);
    }

    // сделать кое-что еще
}
```

Еще раз отметим, что здесь примечательно то, что внутри блока [if](#if), `foo` больше не является опциональный
указателем, это указатель который не может быть нулевым.

Одним из преимуществ этого является то, что функции, которые принимают указатели в качестве аргументов, могут быть
помечены атрибутом "nonnull" - `__attribute__((nonnull))` в
[GCC](https://gcc.gnu.org/onlinedocs/gcc-4.0.0/gcc/Function-Attributes.html). Оптимизатор иногда может принимать более
правильные решения, зная, что аргументы указателя не могут быть нулевыми.


#### Optional Type

Опциональный тип создается путем указания `?` перед типом. Вы можете использовать "рефлексию" во время компиляции для
доступа к дочернему типу опционального типа:

```zig
const expect = @import("std").testing.expect;

test "optional type" {
    // Объявляйте опциональное значение и преобразуйте его к null:
    var foo: ?i32 = null;

    // Преобразование из дочернего опционального типа
    foo = 1234;

    // Используйте "рефлексию" во время компиляции для доступа к дочернему опциональному типу:
    try comptime expect(@typeInfo(@TypeOf(foo)).Optional.child == i32);
}
```
```bash
$ zig test test_optional_type.zig
1/1 test_optional_type.test.optional type...OK
All 1 tests passed.
```


#### null

Как и [undefined](#undefined), `null` имеет свой собственный тип и единственный способ его использовать - преобразовать
к другому типу:

```zig
const optional_value: ?i32 = null;
```


#### Optional Pointers

Опциональный указатель гарантированно будет иметь тот же размер, что и указатель. Нулевым адресом опционального
указателя гарантированно является адрес 0.

```zig
const expect = @import("std").testing.expect;

test "optional pointers" {
    // Указатели не могут быть нулевыми. Если вам нужен нулевой указатель, используйте опциональный
    // префикс `?`, чтобы сделать тип указателя опциональным.
    var ptr: ?*i32 = null;

    var x: i32 = 1;
    ptr = &x;

    try expect(ptr.?.* == 1);

    // Опциональные указатели имеют тот же размер, что и обычные указатели,
    // поскольку значение указателя 0 используется в качестве нулевого значения.
    try expect(@sizeOf(?*i32) == @sizeOf(*i32));
}
```
```bash
$ zig test test_optional_pointer.zig
1/1 test_optional_pointer.test.optional pointers...OK
All 1 tests passed.
```

Смотрите также:
- [while with Optionals](#while-with-optionals)
- [if with Optionals](#if-with-optionals)


------------
### Casting

Приведение типа преобразует значение одного типа в другой. В Zig есть [приведение типа](#type-coercion) для
преобразований которые как известно абсолютно безопасны и однозначны и [явное приведение](#explicit-cast) для
преобразований которые не хотелось бы чтобы произошли случайно. Существует также третий вид преобразования типов
называемый [разрешением одноранговых типов](#peer-type-resolution) для случая когда тип результата должен быть определен
с учетом нескольких типов операндов.


#### Type Coercion

Приведение типа происходит когда ожидается один тип, но предоставляется другой тип:

```zig
test "type coercion - variable declaration" {
    const a: u8 = 1;
    const b: u16 = a;
    _ = b;
}

test "type coercion - function call" {
    const a: u8 = 1;
    foo(a);
}

fn foo(b: u16) void {
    _ = b;
}

test "type coercion - @as builtin" {
    const a: u8 = 1;
    const b = @as(u16, a);
    _ = b;
}
```
```bash
$ zig test test_type_coercion.zig
1/3 test_type_coercion.test.type coercion - variable declaration...OK
2/3 test_type_coercion.test.type coercion - function call...OK
3/3 test_type_coercion.test.type coercion - @as builtin...OK
All 3 tests passed.
```

Приведение к типу допустимо только в том случае, когда совершенно однозначно как перейти от одного типа к другому и
гарантируется безопасность преобразования. Есть одно исключение - [указатели на языке C](#c-pointers).


##### Type Coercion: Stricter Qualification

Значения которые имеют одинаковое представление во время выполнения могут быть приведены для повышения строгости
квалификаторов, независимо от того насколько они вложены:
- `const` - разрешено приведение от неконстантных к константным
- `volatile` - разрешено приведение от неволатильных к волатильным
- `align` - разрешено выравнивание от большего к меньшему
- разрешается приведение наборов ошибок к супер-сетам

Во время выполнения эти приведения не выполняются, поскольку представление значения не меняется.

```zig
test "type coercion - const qualification" {
    var a: i32 = 1;
    const b: *i32 = &a;
    foo(b);
}

fn foo(_: *const i32) void {}
```
```bash
$ zig test test_no_op_casts.zig
1/1 test_no_op_casts.test.type coercion - const qualification...OK
All 1 tests passed.
```

Кроме того, указатели преобразуются к опциональным константным указателям:

```zig
const std = @import("std");
const expect = std.testing.expect;
const mem = std.mem;

test "cast *[1][*]const u8 to [*]const ?[*]const u8" {
    const window_name = [1][*]const u8{"window name"};
    const x: [*]const ?[*]const u8 = &window_name;
    try expect(mem.eql(u8, std.mem.sliceTo(@as([*:0]const u8, @ptrCast(x[0].?)), 0), "window name"));
}
```
```bash
$ zig test test_pointer_coerce_const_optional.zig
1/1 test_pointer_coerce_const_optional.test.cast *[1][*]const u8 to [*]const ?[*]const u8...OK
All 1 tests passed.
```


##### Type Coercion: Integer and Float Widening

[Целые числа](#integers) преобразуются к целочисленным типам которые могут представлять каждое значение старого типа и
аналогично значения с [плавающей запятой](#floats) преобразуются к типам с плавающей запятой которые могут представлять
каждое значение старого типа.

```zig
const std = @import("std");
const builtin = @import("builtin");
const expect = std.testing.expect;
const mem = std.mem;

test "integer widening" {
    const a: u8 = 250;
    const b: u16 = a;
    const c: u32 = b;
    const d: u64 = c;
    const e: u64 = d;
    const f: u128 = e;
    try expect(f == a);
}

test "implicit unsigned integer to signed integer" {
    const a: u8 = 250;
    const b: i16 = a;
    try expect(b == 250);
}

test "float widening" {
    const a: f16 = 12.34;
    const b: f32 = a;
    const c: f64 = b;
    const d: f128 = c;
    try expect(d == a);
}
```
```bash
$ zig test test_integer_widening.zig
1/3 test_integer_widening.test.integer widening...OK
2/3 test_integer_widening.test.implicit unsigned integer to signed integer...OK
3/3 test_integer_widening.test.float widening...OK
All 3 tests passed.
```


##### Type Coercion: Float to Int

Ошибка компилятора уместна, поскольку это неоднозначное выражение оставляет компилятору два варианта приведения в
действие.

- Преобразование значения `54.0` к `comptime_int` в результате приводит к `@as(comptime_int, 10)`, которое преобразуется
в `@as(f32, 10)`
- Преобразование значение `5` в `comptime_float`, в результате приводит к `@as(comptime_float, 10.8)`, который
преобразуется в `@as(f32, 10.8)`

```zig
// Приведение значения float во время компиляции к int
test "implicit cast to comptime_int" {
    const f: f32 = 54.0 / 5;
    _ = f;
}
```
```bash
$ zig test test_ambiguous_coercion.zig
doc/langref/test_ambiguous_coercion.zig:3:25: error: ambiguous coercion of division operands 'comptime_float' and 'comptime_int'; non-zero remainder '4'
    const f: f32 = 54.0 / 5;
                   ~~~~~^~~
```


##### Type Coercion: Slices, Arrays and Pointers

```zig
const std = @import("std");
const expect = std.testing.expect;

// Вы можете назначить константные указатели на массивы, среза с помощью модификатора
// const в зависимости от типа элемента. Полезно, в частности, для
// строковых литералов.
test "*const [N]T to []const T" {
    const x1: []const u8 = "hello";
    const x2: []const u8 = &[5]u8{ 'h', 'e', 'l', 'l', 111 };
    try expect(std.mem.eql(u8, x1, x2));

    const y: []const f32 = &[2]f32{ 1.2, 3.4 };
    try expect(y[0] == 1.2);
}

// Аналогично, это работает, когда типом назначения является объединение с ошибкой.
test "*const [N]T to E![]const T" {
    const x1: anyerror![]const u8 = "hello";
    const x2: anyerror![]const u8 = &[5]u8{ 'h', 'e', 'l', 'l', 111 };
    try expect(std.mem.eql(u8, try x1, try x2));

    const y: anyerror![]const f32 = &[2]f32{ 1.2, 3.4 };
    try expect((try y)[0] == 1.2);
}

// Аналогично, это работает, когда тип назначения является необязательным.
test "*const [N]T to ?[]const T" {
    const x1: ?[]const u8 = "hello";
    const x2: ?[]const u8 = &[5]u8{ 'h', 'e', 'l', 'l', 111 };
    try expect(std.mem.eql(u8, x1.?, x2.?));

    const y: ?[]const f32 = &[2]f32{ 1.2, 3.4 };
    try expect(y.?[0] == 1.2);
}

// При таком приведении длина массива становится длиной среза.
test "*[N]T to []T" {
    var buf: [5]u8 = "hello".*;
    const x: []u8 = &buf;
    try expect(std.mem.eql(u8, x, "hello"));

    const buf2 = [2]f32{ 1.2, 3.4 };
    const x2: []const f32 = &buf2;
    try expect(std.mem.eql(f32, x2, &[2]f32{ 1.2, 3.4 }));
}

// Указатели на массивы состоящие из одного элемента могут быть преобразованы в указатели на множество элементов.
test "*[N]T to [*]T" {
    var buf: [5]u8 = "hello".*;
    const x: [*]u8 = &buf;
    try expect(x[4] == 'o');
    // x[5] было бы неперехваченным разыменованием указателя за пределы допустимых значений!
}

// Аналогично, это работает, когда тип назначения является необязательным.
test "*[N]T to ?[*]T" {
    var buf: [5]u8 = "hello".*;
    const x: ?[*]u8 = &buf;
    try expect(x.?[4] == 'o');
}

// Одноэлементные указатели могут быть преобразованы в одноэлементные массивы len-1.
test "*T to *[1]T" {
    var x: i32 = 1234;
    const y: *[1]i32 = &x;
    const z: [*]i32 = y;
    try expect(z[0] == 1234);
}
```
```bash
$ zig test test_coerce_slices_arrays_and_pointers.zig
1/7 test_coerce_slices_arrays_and_pointers.test.*const [N]T to []const T...OK
2/7 test_coerce_slices_arrays_and_pointers.test.*const [N]T to E![]const T...OK
3/7 test_coerce_slices_arrays_and_pointers.test.*const [N]T to ?[]const T...OK
4/7 test_coerce_slices_arrays_and_pointers.test.*[N]T to []T...OK
5/7 test_coerce_slices_arrays_and_pointers.test.*[N]T to [*]T...OK
6/7 test_coerce_slices_arrays_and_pointers.test.*[N]T to ?[*]T...OK
7/7 test_coerce_slices_arrays_and_pointers.test.*T to *[1]T...OK
All 7 tests passed.
```

Смотрите также:
- [C Pointers](#c-pointers)


##### Type Coercion: Optionals

Тип полезной нагрузки [опциональных](#optionals) типов, так же как и [null](#null), преобразуются к опциональному типу.

```zig
const std = @import("std");
const expect = std.testing.expect;

test "coerce to optionals" {
    const x: ?i32 = 1234;
    const y: ?i32 = null;

    try expect(x.? == 1234);
    try expect(y == null);
}
```
```bash
$ zig test test_coerce_optionals.zig
1/1 test_coerce_optionals.test.coerce to optionals...OK
All 1 tests passed.
```

Опциональные типы также работают вложенными в [тип объединения ошибок](#error-union-type):

```zig
const std = @import("std");
const expect = std.testing.expect;

test "coerce to optionals wrapped in error union" {
    const x: anyerror!?i32 = 1234;
    const y: anyerror!?i32 = null;

    try expect((try x).? == 1234);
    try expect((try y) == null);
}
```
```bash
$ zig test test_coerce_optional_wrapped_error_union.zig
1/1 test_coerce_optional_wrapped_error_union.test.coerce to optionals wrapped in error union...OK
All 1 tests passed.
```


##### Type Coercion: Error Unions

Тип полезной нагрузки [типа объединения ошибок](#error-union-type), а также [тип набора ошибок](#error-set-type)
определяют тип объединения ошибок:

```zig
const std = @import("std");
const expect = std.testing.expect;

test "coercion to error unions" {
    const x: anyerror!i32 = 1234;
    const y: anyerror!i32 = error.Failure;

    try expect((try x) == 1234);
    try std.testing.expectError(error.Failure, y);
}
```
```bash
$ zig test test_coerce_to_error_union.zig
1/1 test_coerce_to_error_union.test.coercion to error unions...OK
All 1 tests passed.
```


##### Type Coercion: Compile-Time Known Numbers

Если известно, что число во [время компиляции](#comptime) может быть представлено в целевом типе, оно может быть
принудительно преобразовано:

```zig
const std = @import("std");
const expect = std.testing.expect;

test "coercing large integer type to smaller one when value is comptime-known to fit" {
    const x: u64 = 255;
    const y: u8 = x;
    try expect(y == 255);
}
```
```bash
$ zig test test_coerce_large_to_small.zig
1/1 test_coerce_large_to_small.test.coercing large integer type to smaller one when value is comptime-known to fit...OK
All 1 tests passed.
```


##### Type Coercion: Unions and Enums

Тегированные объединения могут быть преобразованы в перечисления, а перечисления могут быть преобразованы в тэгированные
объединения если известно во [время компиляции](#comptime), что они являются полем объединения имеющим только одно
возможное значение, например [void](#void):

```zig
const std = @import("std");
const expect = std.testing.expect;

const E = enum {
    one,
    two,
    three,
};

const U = union(E) {
    one: i32,
    two: f32,
    three,
};

const U2 = union(enum) {
    a: void,
    b: f32,

    fn tag(self: U2) usize {
        switch (self) {
            .a => return 1,
            .b => return 2,
        }
    }
};

test "coercion between unions and enums" {
    const u = U{ .two = 12.34 };
    const e: E = u; // приведение union к enum
    try expect(e == E.two);

    const three = E.three;
    const u_2: U = three; // приведение enum к union
    try expect(u_2 == E.three);

    const u_3: U = .three; // приведение enum literal к union
    try expect(u_3 == E.three);

    const u_4: U2 = .a; // приведение enum literal к union с предпологаемым enum tag type.
    try expect(u_4.tag() == 1);

    // Следующий пример неверен.
    // ошибка: приведение из enum '@TypeOf(.enum_literal)' к union 'test_coerce_unions_enum.U2' должен инициализировать поле 'b' 'f32'.
    //var u_5: U2 = .b;
    //try expect(u_5.tag() == 2);
}
```
```bash
$ zig test test_coerce_unions_enums.zig
1/1 test_coerce_unions_enums.test.coercion between unions and enums...OK
All 1 tests passed.
```

Смотрите также:
- [union](#union)
- [enum](#enum)


##### Type Coercion: undefined

Значение [undefined](#undefined) может быть приведено к любому типу.


##### Type Coercion: Tuples to Arrays

[Кортежи](#Tuple) могут быть преобразованы в массивы, если все поля имеют одинаковый тип.

```zig
const std = @import("std");
const expect = std.testing.expect;

const Tuple = struct { u8, u8 };
test "coercion from homogenous tuple to array" {
    const tuple: Tuple = .{ 5, 6 };
    const array: [2]u8 = tuple;
    _ = array;
}
```
```bash
$ zig test test_coerce_tuples_arrays.zig
1/1 test_coerce_tuples_arrays.test.coercion from homogenous tuple to array...OK
All 1 tests passed.
```

#### Explicit Casts

Явные приведения выполняются с помощью [встроенных функций](#builtin-functions). Некоторые явные приведения безопасны,
некоторые - нет. Некоторые явные приведения выполняют утверждения на уровне языка, некоторые - нет. Некоторые явные
приведения не выполняются во время выполнения, некоторые - нет.

- [@bitCast](#bitcast) - изменить тип, но сохранить битовое представление
- [@alignCast](#aligncast) - сделать указатель более выровненным
- [@enumFromInt](#enumfromint) - получить значение перечисления на основе его целочисленного тега.
- [@errorFromInt](#errorfromint) - получение кода ошибки на основе его целочисленного значения
- [@errorCast](#errorcast) - преобразование в меньший набор ошибок
- [@floatCast](#floatcast) - преобразование большего числа с плавающей запятой в меньшее число с плавающей точкой
- [@floatFromInt](#floatfromint) - преобразует целое число в значение с плавающей запятой
- [@intCast](#intcast) - преобразует значения между целыми типами
- [@intFromBool](#intfrombool) - преобразует значение `true` в `1,` а значение `false` в `0`
- [@intFromEnum](#intfromenum) - получение целочисленного значения тега перечисления или объединения с тегами
- [@intFromError](#intfromerror) - получение целочисленного значения кода ошибки
- [@intFromFloat](#intfromfloat) - получение целой части значения с плавающей запятой
- [@intFromPtr](#intfromptr) - получение адреса указателя
- [@ptrFromInt](#ptrfromint) - преобразование адреса в указатель
- [@ptrCast](#ptrcast) - преобразование между типами указателей
- [@truncate](#truncate) - преобразование между целыми типами, отсекая биты


#### Peer Type Resolution

В этих местах происходит разрешение одноранговых типов:

- [switch](#switch) выражения
- [if](#if) выражения
- [while](#while) выражения
- [for](#for) выражения
- Несколько операторов `break` в блоке
- Некоторые [бинарные операции](#operators)

При таком разрешении типов выбирается тип к которому могут быть привязаны все одноранговые типы. Вот несколько
примеров:

```zig
const std = @import("std");
const expect = std.testing.expect;
const mem = std.mem;

test "peer resolve int widening" {
    const a: i8 = 12;
    const b: i16 = 34;
    const c = a + b;
    try expect(c == 46);
    try expect(@TypeOf(c) == i16);
}

test "peer resolve arrays of different size to const slice" {
    try expect(mem.eql(u8, boolToStr(true), "true"));
    try expect(mem.eql(u8, boolToStr(false), "false"));
    try comptime expect(mem.eql(u8, boolToStr(true), "true"));
    try comptime expect(mem.eql(u8, boolToStr(false), "false"));
}
fn boolToStr(b: bool) []const u8 {
    return if (b) "true" else "false";
}

test "peer resolve array and const slice" {
    try testPeerResolveArrayConstSlice(true);
    try comptime testPeerResolveArrayConstSlice(true);
}
fn testPeerResolveArrayConstSlice(b: bool) !void {
    const value1 = if (b) "aoeu" else @as([]const u8, "zz");
    const value2 = if (b) @as([]const u8, "zz") else "aoeu";
    try expect(mem.eql(u8, value1, "aoeu"));
    try expect(mem.eql(u8, value2, "zz"));
}

test "peer type resolution: ?T and T" {
    try expect(peerTypeTAndOptionalT(true, false).? == 0);
    try expect(peerTypeTAndOptionalT(false, false).? == 3);
    comptime {
        try expect(peerTypeTAndOptionalT(true, false).? == 0);
        try expect(peerTypeTAndOptionalT(false, false).? == 3);
    }
}
fn peerTypeTAndOptionalT(c: bool, b: bool) ?usize {
    if (c) {
        return if (b) null else @as(usize, 0);
    }

    return @as(usize, 3);
}

test "peer type resolution: *[0]u8 and []const u8" {
    try expect(peerTypeEmptyArrayAndSlice(true, "hi").len == 0);
    try expect(peerTypeEmptyArrayAndSlice(false, "hi").len == 1);
    comptime {
        try expect(peerTypeEmptyArrayAndSlice(true, "hi").len == 0);
        try expect(peerTypeEmptyArrayAndSlice(false, "hi").len == 1);
    }
}
fn peerTypeEmptyArrayAndSlice(a: bool, slice: []const u8) []const u8 {
    if (a) {
        return &[_]u8{};
    }

    return slice[0..1];
}
test "peer type resolution: *[0]u8, []const u8, and anyerror![]u8" {
    {
        var data = "hi".*;
        const slice = data[0..];
        try expect((try peerTypeEmptyArrayAndSliceAndError(true, slice)).len == 0);
        try expect((try peerTypeEmptyArrayAndSliceAndError(false, slice)).len == 1);
    }
    comptime {
        var data = "hi".*;
        const slice = data[0..];
        try expect((try peerTypeEmptyArrayAndSliceAndError(true, slice)).len == 0);
        try expect((try peerTypeEmptyArrayAndSliceAndError(false, slice)).len == 1);
    }
}
fn peerTypeEmptyArrayAndSliceAndError(a: bool, slice: []u8) anyerror![]u8 {
    if (a) {
        return &[_]u8{};
    }

    return slice[0..1];
}

test "peer type resolution: *const T and ?*T" {
    const a: *const usize = @ptrFromInt(0x123456780);
    const b: ?*usize = @ptrFromInt(0x123456780);
    try expect(a == b);
    try expect(b == a);
}

test "peer type resolution: error union switch" {
    // Случаи без ошибок и error являются одинаковыми только в том случае, если случай ошибки является просто выражением switch;
    // шаблон "if (x) {...} else |err| blk: { switch (err) {...} }" не рассматривает
    // случай без ошибок и случай с ошибкой как равноправные.
    var a: error{ A, B, C }!u32 = 0;
    _ = &a;
    const b = if (a) |x|
        x + 3
    else |err| switch (err) {
        error.A => 0,
        error.B => 1,
        error.C => null,
    };
    try expect(@TypeOf(b) == ?u32);

    // Варианты без ошибок и error являются одинаковыми только в том случае, если случай ошибки является просто выражением switch;
    // шаблон "x catch |err| blk: { switch (err) {...} }" не учитывает развернутый `x`
    // и случай ошибки должны быть одинаковыми.
    const c = a catch |err| switch (err) {
        error.A => 0,
        error.B => 1,
        error.C => null,
    };
    try expect(@TypeOf(c) == ?u32);
}
```
```bash
$ zig test test_peer_type_resolution.zig
1/8 test_peer_type_resolution.test.peer resolve int widening...OK
2/8 test_peer_type_resolution.test.peer resolve arrays of different size to const slice...OK
3/8 test_peer_type_resolution.test.peer resolve array and const slice...OK
4/8 test_peer_type_resolution.test.peer type resolution: ?T and T...OK
5/8 test_peer_type_resolution.test.peer type resolution: *[0]u8 and []const u8...OK
6/8 test_peer_type_resolution.test.peer type resolution: *[0]u8, []const u8, and anyerror![]u8...OK
7/8 test_peer_type_resolution.test.peer type resolution: *const T and ?*T...OK
8/8 test_peer_type_resolution.test.peer type resolution: error union switch...OK
All 8 tests passed.
```


------------
### Zero Bit Types

Для некоторых типов [@sizeOf](#sizeof) равно 0:
- [void](#void)
- [Целые числа](#integers) `u0` и `i0`.
- [Массивы](#arrays) и [векторы](#vectors) с нулевой длиной или с типом элемента который является нулевым разрядным типом.
- [Перечисление](#enum) только с 1 тегом.
- [Структура](#structs), все поля которой имеют нулевые разрядные типы.
- [Объединения](#union) только с 1 полем которое имеет нулевой разрядный тип.

Эти типы могут иметь только одно возможное значение и следовательно для их представления требуется 0 бит. Код
использующий эти типы не включается в окончательный сгенерированный код:

```zig
export fn entry() void {
    var x: void = {};
    var y: void = {};
    x = y;
    y = x;
}
```

Когда это превращается в машинный код в теле ввода не генерируется код, даже в режиме [отладки](#debug). Например, на
`x86_64`:

```asm
0000000000000010 <entry>:
  10:	55                   	push   %rbp
  11:	48 89 e5             	mov    %rsp,%rbp
  14:	5d                   	pop    %rbp
  15:	c3                   	retq
```

Эти инструкции по сборке не содержат никакого кода связанного со значениями `void` - они выполняют только пролог и
эпилог вызова функции.


#### void

`void` может быть полезен для создания экземпляров универсальных типов. Например, если задана `Map(Key, Value)`, можно
передать `void` для типа значения, чтобы превратить его в набор:

```zig
const std = @import("std");
const expect = std.testing.expect;

test "turn HashMap into a set with void" {
    var map = std.AutoHashMap(i32, void).init(std.testing.allocator);
    defer map.deinit();

    try map.put(1, {});
    try map.put(2, {});

    try expect(map.contains(2));
    try expect(!map.contains(3));

    _ = map.remove(2);
    try expect(!map.contains(2));
}
```
```bash
$ zig test test_void_in_hashmap.zig
1/1 test_void_in_hashmap.test.turn HashMap into a set with void...OK
All 1 tests passed.
```

Обратите внимание, что это отличается от использования фиктивного значения для значения хэш-таблица. При использовании
`void` в качестве типа значения тип записи хэш-таблица не имеет поля значения и таким образом хэш-таблица занимает
меньше места. Далее, весь код, который связан с сохранением и загрузкой значения, удаляется, как показано выше.

`void` отличается от `anyopaque`. Известный размер `void` равен 0 байтам, а `anyopaque` имеет неизвестный, но ненулевой
размер.

Выражения типа `void` - единственные значение которых можно игнорировать. Например, игнорирование выражения отличного
от `void` приводит к ошибке компиляции:

```zig
test "ignoring expression value" {
    foo();
}

fn foo() i32 {
    return 1234;
}
```
```bash

$ zig test test_expression_ignored.zig
doc/langref/test_expression_ignored.zig:2:8: error: value of type 'i32' ignored
    foo();
    ~~~^~
doc/langref/test_expression_ignored.zig:2:8: note: all non-void values must be used
doc/langref/test_expression_ignored.zig:2:8: note: to discard the value, assign it to '_'
```

Однако, если выражение имеет тип `void` ошибки не будет. Результаты выражения можно явно проигнорировать, присвоив им
значение _.

```zig
test "void is ignored" {
    returnsVoid();
}

test "explicitly ignoring expression value" {
    _ = foo();
}

fn returnsVoid() void {}

fn foo() i32 {
    return 1234;
}
```
```bash
$ zig test test_void_ignored.zig
1/2 test_void_ignored.test.void is ignored...OK
2/2 test_void_ignored.test.explicitly ignoring expression value...OK
All 2 tests passed.
```


------------
### Result Location Semantics

Во время компиляции каждому Zig-выражению и подвыражению присваивается необязательная информация о местоположении
результата. Эта информация определяет какой тип должно иметь выражение (его тип результата) и куда следует поместить
результирующее значение в памяти (его местоположение результата). Эта информация является необязательной в том смысле,
что не каждое выражение содержит эту информацию: присвоение значения `_`, например, не предоставляет никакой информации
о типе выражения и не указывает конкретную ячейку памяти для его размещения.

В качестве мотивирующего примера рассмотрим утверждение `const x: u32 = 42;`. Приведенная здесь аннотация к типу
предоставляет результирующий тип `u32` для выражения инициализации `42`, давая указание компилятору преобразовать это
целое число (изначально типа `comptime_int`) в этот тип. Вскоре мы увидим больше примеров.

Это не относится к деталям реализации: логика описанная выше кодифицирована в спецификации языка Zig и является
основным механизмом вывода типов в языке. Эта система в совокупности называется "Семантика расположения результатов".


#### Result Types

Типы результатов, по возможности, рекурсивно передаются через выражения. Например, если выражение `&e` имеет тип
результата `*u32`, то `e` присваивается тип результата `u32`, что позволяет языку выполнить это приведение перед
использованием ссылки.

Механизм определения типа результата используется с помощью встроенных функций, таких как `@intCast`. Вместо того, чтобы
принимать в качестве аргумента тип к которому нужно привести, эти встроенные функции используют свой тип результата для
определения этой информации. Тип результата часто известен из контекста; если это не так, то для явного указания типа
результата можно использовать встроенную опцию `@as`.

Мы можем разбить типы результатов для каждого компонента простого выражения следующим образом:

```zig
const expectEqual = @import("std").testing.expectEqual;
test "result type propagates through struct initializer" {
    const S = struct { x: u32 };
    const val: u64 = 123;
    const s: S = .{ .x = @intCast(val) };
    // .{ .x = @intCast(val) } имеет тип результата "S" из-за аннотации типа
    //         @intCast(val) имеет тип результата "u32" из-за типа поля `S.x`
    //         val не имеет результирующего типа, так как он может быть любого целочисленного типа
    try expectEqual(@as(u32, 123), s.x);
}
```
```bash
$ zig test result_type_propagation.zig
1/1 result_type_propagation.test.result type propagates through struct initializer...OK
All 1 tests passed.
```

Эта информация о типе результата полезна для вышеупомянутых встроенных функций приведения, а также для того чтобы
избежать создания значений заданных до приведения и для того чтобы в некоторых случаях избежать необходимости в явном
приведении типов. В следующей таблице подробно описано как некоторые распространенные выражения передают типы
результатов, где `x и` `y являются` произвольными подвыражениями.

|Выражение|Родительский тип результата|Тип результата вложенного выражения|
|----|------------|-----------|
|`const val: T = x`| - |`x` - это `T`|
|`var val: T = x`| - |`x` - это `T`|
|`val = x`| - |`x` - это `@TypeOf(val)`|
|`@as(T, x)`| - |`x` - это `T`|
|`&x`| `*T` |`x` - это `T`|
|`&x`| `[]T` |`x` - это некоторый массив `T`|
|`f(x)`| - |`x` имеет тип первого параметра `f`|
|`.{x}`| `T` |`x` - это `std.meta.FieldType(T, .@"0")`|
|`.{ .a = x }`| `T` |`x` - это `std.meta.FieldType(T, .a)`|
|`T{x}`| - |`x` - это `std.meta.FieldType(T, .@"0")`|
|`T{ .a = x }`| - |`x` - это `std.meta.FieldType(T, .a)`|
|`@Type(x)`| - |`x` это`std.builtin.Type`|
|`@typeInfo(x)`| - |`x` - это `type`|
|`x << y`| - |`y` - это `std.math.Log1IntCeil(@TypeOf(x))`|


#### Result Locations

В дополнение к информации о типе результата, каждому выражению может быть дополнительно присвоено местоположение
результата: указатель, в который должно быть непосредственно записано значение. Эта система может быть использована для
предотвращения промежуточных копий при инициализации структур данных, что может быть важно для типов, которые должны
иметь фиксированный адрес в памяти ("закрепленные" типы).

При компиляции простого выражения присваивания `x = e` многие языки создали бы временное значение `e` в стеке, а затем
присвоили бы его `x`, потенциально выполняя приведение к типу в процессе. Zig подходит к этому по-другому. Выражению `e`
присваивается тип результата, соответствующий типу `x`, и местоположение результата - `&x`. Для многих синтаксических
форм `e` это не имеет практического значения. Однако это может иметь важные семантические последствия при работе с более
сложными синтаксическими формами.

Например, если выражение `.{ .a = x, .b = y }` имеет результирующее местоположение `ptr`, затем `x` присваивается
результирующее местоположение `&ptr.a`, а `y` - результирующее местоположение `&ptr.b`. Без этой системы это выражение
создало бы временное структурное значение полностью в стеке, и только после этого скопируйте его на адрес назначения. По
сути, Zig заменяет присваивание `foo = .{ .a = x, .b = y }` двум операторам `foo.a = x; foo.b = y;`.

Иногда это может быть важно при присвоении агрегатного значения когда выражение инициализации зависит от предыдущего
значения агрегата. Самый простой способ продемонстрировать это - попытаться поменять местами поля структуры или массива
- следующая логика выглядит разумной, но на самом деле таковой не является:

```zig
const expect = @import("std").testing.expect;
test "attempt to swap array elements with array initializer" {
    var arr: [2]u32 = .{ 1, 2 };
    arr = .{ arr[1], arr[0] };
    // Предыдущая строка эквивалентна двум следующим строкам:
    // arr[0] = arr[1];
    // arr[1] = arr[0];
    // Таким образом, это не работает!
    try expect(arr[0] == 2); // succeeds
    try expect(arr[1] == 1); // fails
}
```
```bash
$ zig test result_location_interfering_with_swap.zig
1/1 result_location_interfering_with_swap.test.attempt to swap array elements with array initializer...FAIL (TestUnexpectedResult)
/home/andy/src/zig/lib/std/testing.zig:540:14: 0x103ce1f in expect (test)
    if (!ok) return error.TestUnexpectedResult;
             ^
/home/andy/src/zig/doc/langref/result_location_interfering_with_swap.zig:10:5: 0x103cf85 in test.attempt to swap array elements with array initializer (test)
    try expect(arr[1] == 1); // fails
    ^
0 passed; 0 skipped; 1 failed.
error: the following test command failed with exit code 1:
/home/andy/src/zig/.zig-cache/o/c42c6019fdf548f70655aafe3673a46e/test
```

 В следующей таблице подробно описано, как некоторые распространенные выражения передают местоположения результатов, где
 `x` и `y` являются произвольными подвыражениями. Обратите внимание, что некоторые выражения не могут предоставить
 значимые местоположения результатов для подвыражений даже если у них самих есть местоположение результата.

|Выражение|Родительский тип результата|Тип результата вложенного выражения|
|----|------------|-----------|
|`const val: T = x`| - |`x` имеет результирующее местоположение `&val`|
|`var val: T = x`| - |`x` имеет результирующее местоположение `&val`|
|`val = x` | - |`x` имеет результирующее местоположение `&val`|
|`@as(T, x)`| `ptr` |`x` не имеет местоположения результата|
|`&x`| `ptr` |`x` не имеет результирующего местоположения|
|`f(x)`| `ptr` |`x` не имеет результирующего местоположения|
|`.{x}`| `ptr` |`x` имеет местоположение результата `&ptr[0]`|
|`.{ .a = x }`| `ptr` |`x` имеет результирующее местоположение `&ptr.a`|
|`T{x}`| `ptr` |`x` не имеет результирующего местоположения (типизированные инициализаторы не передают результирующие местоположения)|
|`T{ .a = x }`| `ptr` |`x` не имеет результирующего местоположения (типизированные инициализаторы не передают результирующие местоположения)|
|`@Type(x)`| `ptr` |`x` не имеет результирующего местоположения|
|`@TypeInfo(x)`| `ptr` |`x` не имеет местоположения результата|
|`x << y`| `ptr` |`x` и `y` не имеют результирующих местоположений|


------------
### usingnamespace

`usingnamespace` - это объявление, которое объединяет все публичные объявления операнда, который должен быть
[структурой](#struct), [объединением](#union), [перечислением](#enum) или [непрозрачным типом](#opaque), в пространство имен:

```zig
test "using std namespace" {
    const S = struct {
        usingnamespace @import("std");
    };
    try S.testing.expect(true);
}
```
```bash
$ zig test test_usingnamespace.zig
1/1 test_usingnamespace.test.using std namespace...OK
All 1 tests passed.
```

`usingnamespace` имеет важное значение при организации общедоступного API для файла или пакета. Например, можно
использовать `c.zig` со всеми [импортами языка C](#import-from-c-header-file):

```zig
pub usingnamespace @cImport({
    @cInclude("epoxy/gl.h");
    @cInclude("GLFW/glfw3.h");
    @cDefine("STBI_ONLY_PNG", "");
    @cDefine("STBI_NO_STDIO", "");
    @cInclude("stb_image.h");
});
```

В приведенном выше примере показано, что использование `pub` для определения пространства `usingnamespace` дополнительно
делает импортируемые объявления общедоступными. Это может использоваться для пересылки объявлений, предоставляя точный
контроль над тем какие объявления предоставляет данный файл.


------------
### comptime

Zig придает большое значение тому известно ли выражение во время компиляции. Существует несколько различных областей
применения этой концепции и эти строительные блоки используются для того чтобы язык был компактным, удобочитаемым и
мощным.


#### Introducing the Compile-Time Concept

##### Compile-Time Parameters

Параметры времени компиляции - это то как Zig реализует обобщения. Это утиная типизация во время компиляции.

```zig
fn max(comptime T: type, a: T, b: T) T {
    return if (a > b) a else b;
}
fn gimmeTheBiggerFloat(a: f32, b: f32) f32 {
    return max(f32, a, b);
}
fn gimmeTheBiggerInteger(a: u64, b: u64) u64 {
    return max(u64, a, b);
}
```

В Zig, типы являются пользователями первого класса. Их можно присваивать переменным, передавать в качестве параметров
функциям и возвращать из функций. Однако они могут использоваться только в выражениях, которые известны во время
компиляции, поэтому параметр `T` в приведенном выше фрагменте должен быть помечен как `comptime`.

Параметр `comptime` означает, что:
- В месте вызова значение должно быть известно во время компиляции, в противном случае это ошибка компиляции.
- В определении функции значение известно во время компиляции.

Например, если бы мы ввели другую функцию в приведенный выше фрагмент:

```zig
fn max(comptime T: type, a: T, b: T) T {
    return if (a > b) a else b;
}
test "try to pass a runtime type" {
    foo(false);
}
fn foo(condition: bool) void {
    const result = max(if (condition) f32 else u64, 1234, 5678);
    _ = result;
}
```
```bash
$ zig test test_unresolved_comptime_value.zig
doc/langref/test_unresolved_comptime_value.zig:8:28: error: unable to resolve comptime value
    const result = max(if (condition) f32 else u64, 1234, 5678);
                           ^~~~~~~~~
doc/langref/test_unresolved_comptime_value.zig:8:28: note: condition in comptime branch must be comptime-known
referenced by:
    test.try to pass a runtime type: doc/langref/test_unresolved_comptime_value.zig:5:5
    remaining reference traces hidden; use '-freference-trace' to see all reference traces
```

Это ошибка, потому что программист попытался передать значение известное только во время выполнения, функции которая
ожидает значение известное во время компиляции.

Другой способ получить ошибку - это передать тип, который нарушает проверку типов при анализе функции. Вот что значит
использовать утиную типизацию во время компиляции.

Например:

```zig
fn max(comptime T: type, a: T, b: T) T {
    return if (a > b) a else b;
}
test "try to compare bools" {
    _ = max(bool, true, false);
}
```
```bash
$ zig test test_comptime_mismatched_type.zig
doc/langref/test_comptime_mismatched_type.zig:2:18: error: operator > not allowed for type 'bool'
    return if (a > b) a else b;
               ~~^~~
referenced by:
    test.try to compare bools: doc/langref/test_comptime_mismatched_type.zig:5:12
    remaining reference traces hidden; use '-freference-trace' to see all reference traces
```

С другой стороны, внутри определения функции с параметром `comptime` значение известно во время компиляции. Это
означает, что мы действительно могли бы заставить это работать для типа `bool`, если бы захотели:

```zig
fn max(comptime T: type, a: T, b: T) T {
    if (T == bool) {
        return a or b;
    } else if (a > b) {
        return a;
    } else {
        return b;
    }
}
test "try to compare bools" {
    try @import("std").testing.expect(max(bool, false, true) == true);
}
```
```bash
$ zig test test_comptime_max_with_bool.zig
1/1 test_comptime_max_with_bool.test.try to compare bools...OK
All 1 tests passed.
```

Это работает, потому что Zig неявно встраивает выражения `if` когда условие известно во время компиляции и компилятор
гарантирует, что он пропустит анализ неиспользованной ветви.

Это означает, что фактическая функция сгенерированная для `max` в этой ситуации выглядит следующим образом:

```zig
fn max(a: bool, b: bool) bool {
    {
        return a or b;
    }
}
```

Весь код который имел дело с известными во время компиляции значениями удаляется и у нас остается только необходимый
для выполнения задачи код во время выполнения.

Это работает аналогично для выражений `switch` - они неявно встраиваются когда целевое выражение известно во время
компиляции.


##### Compile-Time Variables

В Zig программист может помечать переменные как `comptime`. Это гарантирует компилятору, что каждая загрузка и
сохранение переменной выполняется во время компиляции. Любое нарушение этого правила приводит к ошибке компиляции.

Это в сочетании с тем фактом, что мы можем встраивать циклы позволяет нам написать функцию, которая частично
вычисляется во время компиляции и частично во время выполнения.

Например:

```zig
const expect = @import("std").testing.expect;

const CmdFn = struct {
    name: []const u8,
    func: fn (i32) i32,
};

const cmd_fns = [_]CmdFn{
    CmdFn{ .name = "one", .func = one },
    CmdFn{ .name = "two", .func = two },
    CmdFn{ .name = "three", .func = three },
};
fn one(value: i32) i32 {
    return value + 1;
}
fn two(value: i32) i32 {
    return value + 2;
}
fn three(value: i32) i32 {
    return value + 3;
}

fn performFn(comptime prefix_char: u8, start_value: i32) i32 {
    var result: i32 = start_value;
    comptime var i = 0;
    inline while (i < cmd_fns.len) : (i += 1) {
        if (cmd_fns[i].name[0] == prefix_char) {
            result = cmd_fns[i].func(result);
        }
    }
    return result;
}

test "perform fn" {
    try expect(performFn('t', 1) == 6);
    try expect(performFn('o', 0) == 1);
    try expect(performFn('w', 99) == 99);
}
```
```bash
$ zig test test_comptime_evaluation.zig
1/1 test_comptime_evaluation.test.perform fn...OK
All 1 tests passed.
```

Этот пример немного надуманный потому что компонент оценки во время компиляции не нужен; этот код работал бы нормально,
если бы все это выполнялось во время выполнения. Но в конечном итоге он генерирует другой код. В этом примере функция
`performFn` генерируется три раза подряд для разных значений параметра `prefix_char`:

```zig
// Из строки:
// expect(performFn('t', 1) == 6);
fn performFn(start_value: i32) i32 {
    var result: i32 = start_value;
    result = two(result);
    result = three(result);
    return result;
}
```
```zig
// Из строки:
// expect(performFn('o', 0) == 1);
fn performFn(start_value: i32) i32 {
    var result: i32 = start_value;
    result = one(result);
    return result;
}
```
```zig
// Из строки:
// expect(performFn('w', 99) == 99);
fn performFn(start_value: i32) i32 {
    var result: i32 = start_value;
    _ = &result;
    return result;
}
```

Обратите внимание, что это происходит даже при отладочной сборке. Это не способ написать более оптимизированный код, но
это способ убедиться, что то, что должно произойти во время компиляции действительно происходит во время компиляции.
Это позволяет выявлять больше ошибок и обеспечивает выразительность для достижения которой в других языках требуется
использование макросов, сгенерированного кода или препроцессора.


##### Compile-Time Expressions

В Zig имеет значение, известно ли данное выражение во время компиляции или во время выполнения. Программист может
использовать выражение `comptime`, чтобы гарантировать, что выражение будет вычислено во время компиляции. Если это не
может быть выполнено, компилятор выдаст ошибку. Например:

```zig
test_comptime_call_extern_function.zig

extern fn exit() noreturn;

test "foo" {
    comptime {
        exit();
    }
}
```
```bash
$ zig test test_comptime_call_extern_function.zig
doc/langref/test_comptime_call_extern_function.zig:5:13: error: comptime call of extern function
        exit();
        ~~~~^~
```

Не имеет смысла чтобы программа могла вызвать `exit()` (или любую другую внешнюю функцию) во время компиляции поэтому
это ошибка компиляции. Однако выражение `comptime` делает гораздо больше, чем просто иногда вызывает ошибку компиляции.

Внутри выражения `comptime`:
- Все переменные являются переменными времени компиляции.
- Все выражения `if`, `while`, `for` и `switch` вычисляются во время компиляции или выдают ошибку компиляции если это
невозможно.
- Все выражения `return` и `try` недопустимы (если только сама функция не вызывается во время компиляции).
- Весь код имеющий побочные эффекты во время выполнения или зависящий от значений во время выполнения выдает ошибку
компиляции.
- Все вызовы функций приводят к тому, что компилятор интерпретирует функцию во время компиляции выдавая ошибку
компиляции если функция пытается выполнить что-то, что приводит к глобальным побочным эффектам во время выполнения.

Это означает, что программист может создать функцию, которая вызывается как во время компиляции, так и во время
выполнения без каких-либо изменений в самой функции.

Давайте рассмотрим пример:

```zig
const expect = @import("std").testing.expect;

fn fibonacci(index: u32) u32 {
    if (index < 2) return index;
    return fibonacci(index - 1) + fibonacci(index - 2);
}

test "fibonacci" {
    // тестируем fibonacci во время выполнения
    try expect(fibonacci(7) == 13);

    // тестируем fibonacci во время компиляции
    try comptime expect(fibonacci(7) == 13);
}
```
```bash
$ zig test test_fibonacci_recursion.zig
1/1 test_fibonacci_recursion.test.fibonacci...OK
All 1 tests passed.
```

Представьте, что мы забыли базовый вариант рекурсивной функции и попытались запустить тесты:

```zig
const expect = @import("std").testing.expect;

fn fibonacci(index: u32) u32 {
    //if (index < 2) return index;
    return fibonacci(index - 1) + fibonacci(index - 2);
}

test "fibonacci" {
    try comptime expect(fibonacci(7) == 13);
}
```
```bash
$ zig test test_fibonacci_comptime_overflow.zig
doc/langref/test_fibonacci_comptime_overflow.zig:5:28: error: overflow of integer type 'u32' with value '-1'
    return fibonacci(index - 1) + fibonacci(index - 2);
                     ~~~~~~^~~
doc/langref/test_fibonacci_comptime_overflow.zig:5:21: note: called from here (7 times)
    return fibonacci(index - 1) + fibonacci(index - 2);
           ~~~~~~~~~^~~~~~~~~~~
doc/langref/test_fibonacci_comptime_overflow.zig:9:34: note: called from here
    try comptime expect(fibonacci(7) == 13);
                        ~~~~~~~~~^~~
```

Компилятор выдает ошибку которая является трассировкой стека при попытке оценить функцию во время компиляции.

К счастью, мы использовали целое число без знака и поэтому когда мы попытались вычесть 1 из 0 это вызвало
неопределенное поведение которое всегда является ошибкой компиляции если компилятор знает, что это произошло. Но что
бы произошло если бы мы использовали целое число со знаком?

```zig
const assert = @import("std").debug.assert;

fn fibonacci(index: i32) i32 {
    //if (index < 2) return index;
    return fibonacci(index - 1) + fibonacci(index - 2);
}

test "fibonacci" {
    try comptime assert(fibonacci(7) == 13);
}
```

Предполагается, что компилятор замечает, что для вычисления этой функции во время компиляции потребовалось более 1000
переходов и таким образом выдает ошибку и прекращает работу. Если программист хочет увеличить бюджет на вычисления во
время компиляции он может использовать встроенную функцию под названием [@setEvalBranchQuota](#setevalbranchquote) чтобы
изменить число по умолчанию 1000 на что-то другое.

Однако в [компиляторе есть конструктивный недостаток](https://github.com/ziglang/zig/issues/13724) из-за которого он
приводит к переполнению стека вместо надлежащего поведения. Я очень сожалею об этом. Я надеюсь, что это будет устранено
до следующего выпуска.

Что, если мы исправим базовый вариант, но введем неправильное значение в строку ожидания?

```zig
const assert = @import("std").debug.assert;

fn fibonacci(index: i32) i32 {
    if (index < 2) return index;
    return fibonacci(index - 1) + fibonacci(index - 2);
}

test "fibonacci" {
    try comptime assert(fibonacci(7) == 99999);
}
```
```bash
$ zig test test_fibonacci_comptime_unreachable.zig
lib/std/debug.zig:412:14: error: reached unreachable code
    if (!ok) unreachable; // assertion failure
             ^~~~~~~~~~~
doc/langref/test_fibonacci_comptime_unreachable.zig:9:24: note: called from here
    try comptime assert(fibonacci(7) == 99999);
                 ~~~~~~^~~~~~~~~~~~~~~~~~~~~~~
```

На уровне [контейнера](#containers) (вне какой-либо функции) все выражения неявно являются выражениями `comptime`. Это
означает, что мы можем использовать функции для инициализации сложных статических данных. Например:

```zig
const first_25_primes = firstNPrimes(25);
const sum_of_first_25_primes = sum(&first_25_primes);

fn firstNPrimes(comptime n: usize) [n]i32 {
    var prime_list: [n]i32 = undefined;
    var next_index: usize = 0;
    var test_number: i32 = 2;
    while (next_index < prime_list.len) : (test_number += 1) {
        var test_prime_index: usize = 0;
        var is_prime = true;
        while (test_prime_index < next_index) : (test_prime_index += 1) {
            if (test_number % prime_list[test_prime_index] == 0) {
                is_prime = false;
                break;
            }
        }
        if (is_prime) {
            prime_list[next_index] = test_number;
            next_index += 1;
        }
    }
    return prime_list;
}

fn sum(numbers: []const i32) i32 {
    var result: i32 = 0;
    for (numbers) |x| {
        result += x;
    }
    return result;
}

test "variable values" {
    try @import("std").testing.expect(sum_of_first_25_primes == 1060);
}
```
```bash
$ zig test test_container-level_comptime_expressions.zig
1/1 test_container-level_comptime_expressions.test.variable values...OK
All 1 tests passed.
```

Когда мы компилируем эту программу Zig генерирует константы с заранее вычисленным ответом. Вот строки из сгенерированного
LLVM IR:

```llvm
@0 = internal unnamed_addr constant [25 x i32] [i32 2, i32 3, i32 5, i32 7, i32 11, i32 13, i32 17, i32 19, i32 23, i32 29, i32 31, i32 37, i32 41, i32 43, i32 47, i32 53, i32 59, i32 61, i32 67, i32 71, i32 73, i32 79, i32 83, i32 89, i32 97]
@1 = internal unnamed_addr constant i32 1060
```

Обратите внимание, что нам не пришлось делать ничего особенного с синтаксисом этих функций. Например, мы могли бы
вызвать функцию `sum` как есть с фрагментом чисел, длина и значения которых были известны только во время выполнения.


#### Generic Data Structures

Zig использует возможности `comptime` для реализации общих структур данных без использования специального
синтаксиса.

Вот пример общей структуры данных списка.

```zig
fn List(comptime T: type) type {
    return struct {
        items: []T,
        len: usize,
    };
}

// Общая структура данных списка может быть создана путем передачи типа:
var buffer: [10]i32 = undefined;
var list = List(i32){
    .items = &buffer,
    .len = 0,
};
```

Вот и все. Это функция которая возвращает анонимную структуру. В целях получения сообщений об ошибках и отладки Zig
выводит имя `"List(i32)"` из имени функции и параметров вызываемых при создании анонимной структуры.

Чтобы явно присвоить типу имя, мы присваиваем его константе.

```zig
const Node = struct {
    next: ?*Node,
    name: []const u8,
};

var node_a = Node{
    .next = null,
    .name = "Node A",
};

var node_b = Node{
    .next = &node_a,
    .name = "Node B",
};
```

В этом примере структура `Node` ссылается на саму себя. Это работает, потому что все объявления верхнего уровня не
зависят от порядка. Пока компилятор может определять размер структуры, он может свободно ссылаться на себя. В этом
случае `Node` ссылается на себя как на указатель который имеет четко определенный размер во время компиляции поэтому он
работает нормально.


#### Case Study: print in Zig

Собрав все это вместе, давайте посмотрим как работает печать в Zig.

```zig
const print = @import("std").debug.print;

const a_number: i32 = 1234;
const a_string = "foobar";

pub fn main() void {
    print("here is a string: '{s}' here is a number: {}\n", .{ a_string, a_number });
}
```
```bash
$ zig build-exe print.zig
$ ./print
here is a string: 'foobar' here is a number: 1234
```

Давайте разберемся с реализацией этого и посмотрим, как это работает:

```zig
const Writer = struct {
    /// Вызывает функцию print и затем очищает буфер.
    pub fn print(self: *Writer, comptime format: []const u8, args: anytype) anyerror!void {
        const State = enum {
            start,
            open_brace,
            close_brace,
        };

        comptime var start_index: usize = 0;
        comptime var state = State.start;
        comptime var next_arg: usize = 0;

        inline for (format, 0..) |c, i| {
            switch (state) {
                State.start => switch (c) {
                    '{' => {
                        if (start_index < i) try self.write(format[start_index..i]);
                        state = State.open_brace;
                    },
                    '}' => {
                        if (start_index < i) try self.write(format[start_index..i]);
                        state = State.close_brace;
                    },
                    else => {},
                },
                State.open_brace => switch (c) {
                    '{' => {
                        state = State.start;
                        start_index = i;
                    },
                    '}' => {
                        try self.printValue(args[next_arg]);
                        next_arg += 1;
                        state = State.start;
                        start_index = i + 1;
                    },
                    's' => {
                        continue;
                    },
                    else => @compileError("Unknown format character: " ++ [1]u8{c}),
                },
                State.close_brace => switch (c) {
                    '}' => {
                        state = State.start;
                        start_index = i;
                    },
                    else => @compileError("Single '}' encountered in format string"),
                },
            }
        }
        comptime {
            if (args.len != next_arg) {
                @compileError("Unused arguments");
            }
            if (state != State.start) {
                @compileError("Incomplete format string: " ++ format);
            }
        }
        if (start_index < format.len) {
            try self.write(format[start_index..format.len]);
        }
        try self.flush();
    }

    fn write(self: *Writer, value: []const u8) !void {
        _ = self;
        _ = value;
    }
    pub fn printValue(self: *Writer, value: anytype) !void {
        _ = self;
        _ = value;
    }
    fn flush(self: *Writer) !void {
        _ = self;
    }
};
```

Это доказательство реализации концепции; реальная функция в стандартной библиотеке обладает большими возможностями
форматирования.

Обратите внимание, что это не запрограммировано жестко в компиляторе Zig; это пользовательский код в стандартной
библиотеке.

Когда эта функция анализируется из нашего примера кода приведенного выше, Zig частично вычисляет функцию и выдает
функцию которая на самом деле выглядит следующим образом:

```zig
pub fn print(self: *Writer, arg0: []const u8, arg1: i32) !void {
    try self.write("here is a string: '");
    try self.printValue(arg0);
    try self.write("' here is a number: ");
    try self.printValue(arg1);
    try self.write("\n");
    try self.flush();
}
```

`printValue` - это функция, которая принимает параметр любого типа и выполняет разные действия в зависимости от типа:

```zig
const Writer = struct {
    pub fn printValue(self: *Writer, value: anytype) !void {
        switch (@typeInfo(@TypeOf(value))) {
            .Int => {
                return self.writeInt(value);
            },
            .Float => {
                return self.writeFloat(value);
            },
            .Pointer => {
                return self.write(value);
            },
            else => {
                @compileError("Unable to print type '" ++ @typeName(@TypeOf(value)) ++ "'");
            },
        }
    }

    fn write(self: *Writer, value: []const u8) !void {
        _ = self;
        _ = value;
    }
    fn writeInt(self: *Writer, value: anytype) !void {
        _ = self;
        _ = value;
    }
    fn writeFloat(self: *Writer, value: anytype) !void {
        _ = self;
        _ = value;
    }
};
```

А теперь, что произойдет, если мы введем слишком много аргументов для печати?

```zig
const print = @import("std").debug.print;

const a_number: i32 = 1234;
const a_string = "foobar";

test "print too many arguments" {
    print("here is a string: '{s}' here is a number: {}\n", .{
        a_string,
        a_number,
        a_number,
    });
}
```
```bash
$ zig test test_print_too_many_args.zig
lib/std/fmt.zig:203:18: error: unused argument in 'here is a string: '{s}' here is a number: {}
                               '
            1 => @compileError("unused argument in '" ++ fmt ++ "'"),
                 ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
referenced by:
    print__anon_2377: lib/std/io/Writer.zig:24:26
    print: lib/std/io.zig:324:47
    remaining reference traces hidden; use '-freference-trace' to see all reference traces

```

Zig предоставляет программистам инструменты необходимые для защиты от их собственных ошибок.

Zig не волнует является ли аргумент `format` строковым литералом, важно только, что это известное во время
компиляции значение которое может быть преобразовано в `[]const u8`:

```zig
const print = @import("std").debug.print;

const a_number: i32 = 1234;
const a_string = "foobar";
const fmt = "here is a string: '{s}' here is a number: {}\n";

pub fn main() void {
    print(fmt, .{ a_string, a_number });
}
```
```bash
$ zig build-exe print_comptime-known_format.zig
$ ./print_comptime-known_format
here is a string: 'foobar' here is a number: 1234
```

Это работает нормально.

Zig не использует форматирование строк в специальном регистре в компиляторе и вместо этого предоставляет достаточно
возможностей для выполнения этой задачи в пользовательской среде. Это достигается без использования другого языка поверх
Zig, такого как макроязык или язык препроцессора. Это Zig на протяжении всего пути.


Смотрите также:
- [inline while](#inline-while)
- [inline for](#inline_for)


------------
### Assembly

В некоторых случаях может потребоваться прямое управление машинным кодом, генерируемым программами Zig вместо того
чтобы полагаться на генерацию Zig-кода. В таких случаях можно использовать встроенную сборку. Вот пример реализации
Hello, World в x86_64 Linux с использованием inline assembly:

```zig
pub fn main() noreturn {
    const msg = "hello world\n";
    _ = syscall3(SYS_write, STDOUT_FILENO, @intFromPtr(msg), msg.len);
    _ = syscall1(SYS_exit, 0);
    unreachable;
}

pub const SYS_write = 1;
pub const SYS_exit = 60;

pub const STDOUT_FILENO = 1;

pub fn syscall1(number: usize, arg1: usize) usize {
    return asm volatile ("syscall"
        : [ret] "={rax}" (-> usize),
        : [number] "{rax}" (number),
          [arg1] "{rdi}" (arg1),
        : "rcx", "r11"
    );
}

pub fn syscall3(number: usize, arg1: usize, arg2: usize, arg3: usize) usize {
    return asm volatile ("syscall"
        : [ret] "={rax}" (-> usize),
        : [number] "{rax}" (number),
          [arg1] "{rdi}" (arg1),
          [arg2] "{rsi}" (arg2),
          [arg3] "{rdx}" (arg3),
        : "rcx", "r11"
    );
}
```
```bash
$ zig build-exe inline_assembly.zig -target x86_64-linux
$ ./inline_assembly
hello world
```

Анализ синтаксиса:

```zig
pub fn syscall1(number: usize, arg1: usize) usize {
    // Inline assembly - это выражение, которое возвращает значение.
    // выражение начинается с ключевого слова `asm`.
    return asm
    // `volatile` - это необязательный модификатор, который сообщает Zig об этом
    // встроенное выражение assembly имеет побочные эффекты. Без
    // `volatile`, Zig разрешено удалять встроенную сборку
    // code, если результат не используется.
    volatile (
    // Далее следует строка comptime, представляющая собой ассемблерный код.
    // Внутри этой строки можно использовать `%[ret]`, `%[number]`,
    // или `%[arg1]`, где ожидается регистр чтобы указать
    // регистр, который Zig использует в качестве аргумента или возвращаемого значения,
    // если используются строки ограничения register. Однако в
    // приведенном ниже коде это не используется. Литерал `%` может быть
    // получен путем экранирования его двойным процентом: `%%`.
    // Здесь часто пригодится многострочный синтаксис строк.
        \\syscall
        // Далее следует вывод. Вполне возможно, что в будущем Zig будет
        // поддержка нескольких выходов в зависимости от того как
        // https://github.com/ziglang/zig/issues/215 решено.
        // Допускается, чтобы выходных данных не было и в этом случае
        // за этим двоеточием непосредственно следует двоеточие для входных данных.
        :
        // Это указывает имя, которое будет использоваться в синтаксисе `%[ret]` в
        // приведенной выше строке сборки. В этом примере оно не используется,
        // но синтаксис обязателен.
          [ret]
          // Далее следует строка ограничения вывода. Эта функция по-прежнему
          // считается нестабильной в Zig, поэтому для понимания семантики необходимо использовать документацию LLVM/GCC.
          // http://releases.llvm.org/10.0.0/docs/LangRef.html#inline-asm-constraint-string
          // https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html
          // В этом примере строка ограничения означает "результирующее значение
          // этой встроенной инструкции по сборке равно тому, что указано в $rax".
          "={rax}"
          // Далее следует либо привязка к значению, либо `->`, а затем тип.

          // Тип - это тип результата выражения встроенной сборки.
          // Если это привязка к значению, то будет использоваться синтаксис `%[ret]`
          // для ссылки на регистр, привязанный к данному значению.
          (-> usize),
          // Далее приведен список входных данных.
          // Ограничение для этих входных данных означает, что "при выполнении ассемблерного кода
          // $rax должен иметь значение `number`, а $rdi -
          // значение `arg1`". Допускается любое количество входных параметров,
          // включая ни один из них.
        : [number] "{rax}" (number),
          [arg1] "{rdi}" (arg1),
          // Далее приведен список клобберов. Они объявляют набор регистров,
          // значения которых не будут сохранены при выполнении этого ассемблерного кода.
          // Они не включают регистры вывода или ввода. Специальный клоббер
          // значение "memory" означает, что сборка выполняет запись в произвольные необъявленные ячейки
          // памяти, а не только в память на которую указывает объявленный косвенный
          // вывод. В этом примере мы перечисляем $rcx и $r11, поскольку известно
          //, что системный вызов // ядра не сохраняет эти регистры.
        : "rcx", "r11"
    );
}
```

Для целевых систем `x86` и `x86_64` используется синтаксис AT&T, а не более популярный синтаксис Intel. Это связано с
техническими ограничениями; синтаксический анализ сборки обеспечивается LLVM, а его поддержка синтаксиса Intel дает сбои
и плохо протестирована.

Когда-нибудь у Zig может появиться свой собственный ассемблер. Это позволило бы ему более легко интегрироваться в
язык, а также быть совместимым с популярным синтаксисом NASM. Этот раздел документации будет обновлен до выхода версии
1.0.0, и в нем будет содержаться подробное описание состояния синтаксиса AT&T и Intel/NASM.


#### Output Constraints

Входные ограничения по-прежнему считаются нестабильными в Zig, и поэтому для понимания семантики необходимо использовать
документацию [LLVM](http://releases.llvm.org/10.0.0/docs/LangRef.html#inline-asm-constraint-string) и
[GCC](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html).

Обратите внимание, что в [выпуске #215](https://github.com/ziglang/zig/issues/215) запланированы некоторые кардинальные
изменения входных ограничений.


#### Input Constraints

Входные ограничения по-прежнему считаются нестабильными в Zig, и поэтому для понимания семантики необходимо использовать
документацию [LLVM](http://releases.llvm.org/10.0.0/docs/LangRef.html#inline-asm-constraint-string) и
[GCC](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html).

Обратите внимание, что в [выпуске #215](https://github.com/ziglang/zig/issues/215) запланированы некоторые кардинальные
изменения входных ограничений.


#### Clobbers

"Клобберы" - это набор регистров, значения которых не будут сохранены при выполнении ассемблерного кода. Они не включают
регистры вывода или ввода. Специальное значение клобберов "памяти" означает, что при сборке выполняется запись в
произвольные необъявленные ячейки памяти, а не только в память на которую указывает объявленный косвенный вывод.

Неспособность объявить полный набор блокировок для данного встроенного выражения `assembly` является неконтролируемым
[неопределенным поведением](#undefined-behavior).


#### Global Assembly

Когда выражение `assembly` встречается в блоке `comptime` на уровне [контейнера](#containers), это означает **глобальную
сборку**.

Для сборки такого типа применяются другие правила чем для встроенной сборки. Во-первых, значение `volatile` недопустимо,
поскольку все глобальные сборки включаются безоговорочно. Во-вторых, нет входных данных, выходных данных или блокирующих
элементов. Вся глобальная сборка дословно объединяется в одну длинную строку и собирается вместе. В отношении `%` нет
правил подстановки шаблонов которые есть во встроенных выражениях сборки.

```zig
const std = @import("std");
const expect = std.testing.expect;

comptime {
    asm (
        \\.global my_func;
        \\.type my_func, @function;
        \\my_func:
        \\  lea (%rdi,%rsi,1),%eax
        \\  retq
    );
}

extern fn my_func(a: i32, b: i32) i32;

test "global assembly" {
    try expect(my_func(12, 34) == 46);
}
```
```bash
$ zig test test_global_assembly.zig -target x86_64-linux
1/1 test_global_assembly.test.global assembly...OK
All 1 tests passed.
```

------------
### Atomics

TODO: @fence()

TODO: @atomic rmw

TODO: builtin atomic memory ordering enum


Смотрите также:
- [@atomicLoad](#atomicload)
- [@atomicStore](#atomicstore)
- [@atomicRmw](#atomicrmw)
- [@fence](#fence)
- [@cmpxchgWeak](#cmpxchgweak)
- [@cmpxchgStrong](#cmpxchgstrong)


------------
### Async Functions

Асинхронные функции регрессировали с выходом версии 0.11.0. Их будущее в языке Zig неясно из-за множества нерешенных
проблем:

- Отсутствие возможности у LLVM оптимизировать их.
- Отсутствие возможности у сторонних отладчиков отлаживать их.
- [Проблема отмены](https://github.com/ziglang/zig/issues/5913).
- Указатели на асинхронные функции не позволяют узнать размер стека.

Эти проблемы преодолимы, но на это потребуется время. В настоящее время команда Zig сосредоточена на других
приоритетах.


------------
### Builtin Functions

Встроенные функции предоставляются компилятором и имеют префикс `@`. Ключевое слово `comptime` в параметре означает, что
этот параметр должен быть известен во время компиляции.

###### `addrSpaceCast`

```zig
@addrSpaceCast(ptr: anytype) anytype
```

Преобразует указатель из одного адресного пространства в другое. Новое адресное пространство определяется на основе типа
результата. В зависимости от текущего целевого и адресного пространств это преобразование может быть бесполезным,
сложным или недопустимым. Если приведение допустимо, то результирующий указатель указывает на ту же ячейку памяти что и
операнд указателя. Всегда допустимо приводить указатель между одними и теми же адресными пространствами.


###### `addWithOverflow`

```zig
@addWithOverflow(a: anytype, b: anytype) struct { @TypeOf(a, b), u1 }
```

Выполняет `a + b` и возвращает кортеж с результатом и возможным битом переполнения.


###### `alignCast`

```zig
@alignCast(ptr: anytype) anytype
```

`ptr` может быть `*T`, `?*T` или `[]T`. Изменяет выравнивание указателя. Используемое выравнивание определяется на
основе типа результата.

В генерируемый код добавляется [проверка безопасности выравнивания указателя](#incorrect-pointer-alignment), чтобы
убедиться, что указатель выровнен так как было обещано.


###### `alignOf`

```zig
@alignOf(comptime T: type) comptime_int
```

Эта функция возвращает количество байт на которое должен быть выровнен этот тип чтобы текущий целевой объект
соответствовал C ABI. Когда дочерний тип указателя имеет такое выравнивание, то выравнивание типа может быть опущено.

```zig
const assert = @import("std").debug.assert;
comptime {
    assert(*u32 == *align(@alignOf(u32)) u32);
}
```

Результатом является константа времени компиляции зависящая от конкретной цели. Гарантируется, что она будет меньше или
равна [@sizeOf(T)](#sizeof).

Смотрите также:
- [Alignment](#alignment)


###### `as`

```zig
@as(comptime T: type, expression) T
```

Выполняет [преобразование](#type-coercion) к типу. Это приведение разрешено если преобразование является однозначным и
безопасным и является предпочтительным способом преобразования между типами когда это возможно.


###### `atomicLoad`

```zig
@atomicLoad(comptime T: type, ptr: *const T, comptime ordering: AtomicOrder) T
```

Эта встроенная функция атомарно разыменовывает указатель на `T` и возвращает значение.

`T` должно быть указателем, логическим значением, значением с плавающей запятой, целым числом или перечислением.

`AtomicOrder` можно найти с помощью `@import("std").builtin.AtomicOrder`.

Смотрите также:
- [@atomicStore](#atomicstore)
- [@atomicRmw](#atomicrmw)
- [@fence](#fence)
- [@cmpxchgWeak](#cmpxchgweak)
- [@cmpxchgStrong](#cmpxchgstrong)


###### `atomicRmw`

```zig
@atomicRmw(comptime T: type, ptr: *T, comptime op: AtomicRmwOp, operand: T, comptime ordering: AtomicOrder) T
```

Эта встроенная функция разыменовывает указатель на `T` и атомарно изменяет значение и возвращает предыдущее значение.

`T` должно быть указателем, логическим значением, значением с плавающей запятой, целым числом или перечислением.

`AtomicOrder` можно найти с помощью `@import("std").builtin.AtomicOrder`.

`AtomicRmwOp` можно найти с помощью `@import("std").builtin.AtomicRmwOp`.

Смотрите также:
- [@atomicStore](#atomicstore)
- [@atomicLoad](#atomicload)
- [@fence](#fence)
- [@cmpxchgWeak](#cmpxchgweak)
- [@cmpxchgStrong](#cmpxchgstrong)


###### `atomicStore`

```zig
@atomicStore(comptime T: type, ptr: *T, value: T, comptime ordering: AtomicOrder) void
```

Эта встроенная функция разыменовывает указатель на `T` и атомарно сохраняет заданное значение.

`T` должно быть указателем, логическим значением, значением с плавающей запятой, целым числом или перечислением.

`AtomicOrder` можно найти с помощью `@import("std").builtin.AtomicOrder`.

Смотрите также:
- [@atomicLoad](#atomicload)
- [@atomicRmw](#atomicrmw)
- [@fence](#fence)
- [@cmpxchgWeak](#cmpxchgweak)
- [@cmpxchgStrong](#cmpxchgstrong)


###### `bitCast`

```zig
@bitCast(value: anytype) anytype
```

Преобразует значение одного типа в значение другого типа. Возвращаемый тип - это предполагаемый тип результата.

Утверждает, что `@sizeOf(@TypeOf(value)) == @sizeOf(DestType)`.

Утверждает, что `@typeInfo(DestType) != .Pointer`. Используйте `@ptrCast` или `@ptrFromInt` если вам это нужно.

Может быть использован, например, для этих целей:
- Преобразовать `f32` в `u32` бита
- Преобразовать `i32` в `u32` сохранив двоичное дополнение

Работает во время компиляции если `значение` известно во время компиляции. Передача значения с неопределенной компоновкой
является ошибкой компиляции; это означает, что помимо ограничений на типы которые имеют специальные встроенные функции
приведения (перечисления, указатели, наборы ошибок) в этой операции также нельзя использовать простые структуры,
объединением ошибок, срезы, опциональные и любые другие типы без четко определенного расположения памяти.


###### `bitOffsetOf`

```zig
@bitOffsetOf(comptime T: type, comptime field_name: []const u8) comptime_int
```

Возвращает битовое смещение поля относительно содержащей его структуры.

Для [неупакованных структур](#packed-struct) это значение всегда будет кратно `8`. Для упакованных структур поля не
выровненные по байтам будут иметь общее байтовое смещение, но они будут иметь разные битовые смещения.

Смотрите также:
- [@offsetOf](#offsetof)


###### `bitSizeOf`

```zig
@bitSizeOf(comptime T: type) comptime_int
```

Эта функция возвращает количество бит необходимое для сохранения `T` в памяти если бы тип был полем в упакованной
структуре/объединении. Результатом является константа времени компиляции зависящая от конкретной цели.

Эта функция измеряет размер во время выполнения. Для типов которые запрещены во время выполнения, таких как
`comptime_int` и `type`, результат равен 0.

Смотрите также:
- [@sizeOf](#sizeof)
- [@typeInfo](#typeinfo)


###### `breakpoint`

```zig
@breakpoint() void
```

Эта функция вставляет специфичную для платформы инструкцию "отладочной ловушки", которая вызывает сбой в работе
отладчиков. В отличие от функции `@trap()`, выполнение может быть продолжено после этого момента если программа
возобновлена.

Эта функция действительна только в пределах области действия функции.

Смотрите также:
- [@trap](#trap)


###### `mulAdd`

```zig
@mulAdd(comptime T: type, a: T, b: T, c: T) T
```

Комбинированное умножение-сложение, аналогично `(a * b) + c`, за исключением того, что оно выполняется только один раз и
следовательно, является более точным.

Поддерживает значения с [плавающей запятой](#floats) и [векторы](#vectors) значений с плавающей запятой.


###### `byteSwap`

```zig
@byteSwap(operand: anytype) T
```

`@TypeOf(operand)` должен быть целочисленного типа или целочисленного векторного типа с количеством битов равным 8.

Операнд может быть [целым числом](#integers) или [вектором](#vectors).

Изменяет порядок байтов целого числа. При этом целое число с большим порядком байтов преобразуется в целое число с
меньшим порядком байтов, а целое число с меньшим порядком байтов преобразуется в целое число с большим порядком байтов.

Обратите внимание, что для целей компоновки памяти с учетом порядка байтов, целочисленный тип должен быть связан с
количеством байт сообщаемым [@sizeOf](#sizeof). Это демонстрируется с помощью `u24`. `@sizeOf(u24) == 4`, что означает,
что `u24` хранящийся в памяти занимает 4 байта и эти 4 байта меняются местами в системе с малым и большим порядком
байтов. С другой стороны, если для параметра `T` указано значение `u24`, то меняются местами только 3 байта.


###### `bitReverse`

```zig
@bitReverse(integer: anytype) T
```

`@TypeOf(anytype)` принимает любой целочисленный тип или целочисленный векторный тип.

Изменяет битовую структуру целого значения на противоположную, включая знаковый бит если применимо.

Например, `0b10110110 (u8 = 182, i8 = -74)` становится `0b01101101 (u8 = 109, i8 = 109)`.


###### `offsetOf`

```zig
@offsetOf(comptime T: type, comptime field_name: const u8) comptime_int
```

Возвращает байтовое смещение поля относительно содержащей его структуры.

Смотрите также:
- [@bitOffsetOf](#bitoffsetof)


###### `call`

```zig
@call(modifier: std.builtin.CallModifier, function: anytype, args: anytype) anytype
```

Вызывает функцию таким же образом, как и вызов выражения в круглых скобках:

```zig
const expect = @import("std").testing.expect;

test "noinline function call" {
    try expect(@call(.auto, add, .{ 3, 9 }) == 12);
}

fn add(a: i32, b: i32) i32 {
    return a + b;
}
```
```bash
$ zig test test_call_builtin.zig
1/1 test_call_builtin.test.noinline function call...OK
All 1 tests passed.
```


`@call` обеспечивает большую гибкость, чем обычный синтаксис вызова функции. Здесь воспроизводится перечисление
`CallModifier`:

```zig
pub const CallModifier = enum {
    /// Эквивалентно синтаксису вызова функции.
    auto,

    /// Эквивалентно ключевому слову async, используемому в синтаксисе вызова функции.
    async_kw,
    /// Предотвращает оптимизацию конечного вызова.
    /// Это гарантирует, что адрес возврата будет указывать на вызывающий узел, а не на вызывающий узел вызываемого узла.
    /// Если в противном случае требуется чтобы вызов был конечным или встроенным, вместо этого выдается ошибка компиляции.
    never_tail,
    /// Гарантирует, что вызов не будет встроенным.
    /// Если в противном случае требуется чтобы вызов был встроенным, вместо этого выдается ошибка компиляции.
    never_inline,

    /// Утверждает, что вызов функции не будет приостановлен.
    /// Это позволяет несинхронной функции вызывать асинхронную функцию.
    no_async,

    /// Гарантирует, что вызов будет сгенерирован с оптимизацией конечного вызова.
    /// Если это невозможно, то вместо этого выдается ошибка компиляции.
    always_tail,

    /// Гарантирует, что вызов будет выполнен в месте вызова.
    /// Если это невозможно, то вместо этого выдается ошибка компиляции.
    always_inline,

    /// Вычисляет вызов во время компиляции.
    /// Если вызов не может быть завершен во время компиляции, вместо этого выдается ошибка компиляции.
    compile_time,
};
```


###### `cDefine`

```zig
@cDefine(comptime name: []const u8, value) void
```

Эта функция может выполняться только внутри `@cImport`.

Это добавит `#define $name $value` во временный буфер `@cImport`.

Чтобы определить без значения, вот так:

`#define _GNU_SOURCE`

Используйте значение void, вот так:

```zig
@cDefine("_GNU_SOURCE", {})
```

Смотрите также:
- [Import from C Header File](#import-from-c-header-file)
- [@cImport](#cimport)
- [@cDefine](#cdefine)
- [@cUndef](#cundef)
- [void](#void)


###### `cImport`

```zig
@cImport(expression) type
```

Эта функция анализирует код на C и импортирует функции, типы, переменные и совместимые определения макросов в новый
пустой структурный тип, а затем возвращает этот тип.

Выражения интерпретируется во время компиляции. Встроенные функции `@cInclude`, `@cDefine` и `@cUndef` работают с этим
выражением, добавляя его во временный буфер который затем анализируется как код на C.

Обычно во всем вашем приложении должен быть только один `@cImport`, потому что это избавляет компилятор от
многократного вызова `clang` и предотвращает дублирование встроенных функций.

Причинами наличия нескольких выражений `@cImport` могут быть:

- Чтобы избежать коллизии символов, например, если `foo.h` и `bar.h` имеют значения `#define CONNECTION_COUNT`
- Для анализа кода на C с различными определениями препроцессора

Смотрите также:
- [Import from C Header File](#import-from-c-header-file)
- [@cImport](#cimport)
- [@cDefine](#cdefine)
- [@cUndef](#cundef)


###### `cInclude`

```zig
@cInclude(comptime path: []const u8) void
```

Эта функция может выполняться только внутри `@cImport`.

При этом `#include <$path>\n` добавляется во временный буфер `c_import`.

Смотрите также:
- [Import from C Header File](#import-from-c-header-file)
- [@cImport](#cimport)
- [@cDefine](#cdefine)
- [@cUndef](#cundef)


###### `clz`

```zig
@clz(operand: anytype) anytype
```

`@TypeOf(operand)` должен быть целочисленного типа или целочисленного векторного типа.

Операнд может быть [целым числом](#integer) или [вектором](#vectors).

Подсчитывает количество наиболее значимых (ведущих в порядке возрастания) нулей в целом числе - "подсчитывает ведущие
нули".

Если операнд является целым числом известным по [время компиляции](#comptime) тип возвращаемого значения -
`comptime_int`. В противном случае тип возвращаемого значения - целое число без знака или вектор целых чисел без знака с
минимальным количеством битов которое может представлять количество битов целочисленного типа.

Если операнд равен нулю, `@clz` возвращает разрядность целого типа `T`.

Смотрите также:
- [@ctz](#ctz)
- [@popCount](#popcount)


###### `cmpxchgStrong`

```zig
@cmpxchgStrong(comptime T: type, ptr: *T, expected_value: T, new_value: T, success_order: AtomicOrder, fail_order: AtomicOrder) ?T
```

Эта функция выполняет строгую атомарную операцию сравнения и обмена, возвращая значение `null`, если текущее значение не
соответствует заданному ожидаемому значению. Это эквивалент этого кода, за исключением атомарного:

```zig
fn cmpxchgStrongButNotAtomic(comptime T: type, ptr: *T, expected_value: T, new_value: T) ?T {
    const old_value = ptr.*;
    if (old_value == expected_value) {
        ptr.* = new_value;
        return null;
    } else {
        return old_value;
    }
}
```

Если вы используете `cmpxchg` в цикле, [@cmpxchgWeak](#cmpxchgweak) - лучший выбор, поскольку он может быть более
эффективно реализован в машинных инструкциях.

`T` должно быть указателем, логическим значением, значением с плавающей запятой, целым числом или перечислением.

`@typeInfo(@TypeOf(ptr)).Pointer.alignment` должен быть `>= @sizeOf(T)`.

`AtomicOrder` можно найти с помощью `@import("std").builtin.AtomicOrder`.

Смотрите также:
- [@atomicStore](#atomicstore)
- [@atomicLoad](#atomicload)
- [@atomicRmw](#atomicrmw)
- [@fence](#fence)
- [@cmpxchgWeak](#cmpxchgweak)


###### `cmpxchgWeak`

```zig
@cmpxchgWeak(comptime T: type, ptr: *T, expected_value: T, new_value: T, success_order: AtomicOrder, fail_order: AtomicOrder) ?T
```

Эта функция выполняет простую атомарную операцию сравнения и обмена, возвращая значение `null`, если текущее значение не
соответствует заданному ожидаемому значению. Это эквивалент этого кода, за исключением атомарного:

```zig
fn cmpxchgWeakButNotAtomic(comptime T: type, ptr: *T, expected_value: T, new_value: T) ?T {
    const old_value = ptr.*;
    if (old_value == expected_value and usuallyTrueButSometimesFalse()) {
        ptr.* = new_value;
        return null;
    } else {
        return old_value;
    }
}
```

Если вы используете `cmpxchg` в цикле, спорадический сбой не будет проблемой и `cmpxchgWeak` - лучший выбор, поскольку
он может быть более эффективно реализован в машинных инструкциях. Однако, если вам нужна более надежная гарантия,
используйте [@cmpxchgStrong](#cmpxchgstrong).

`T` должно быть указателем, логическим значением, значением с плавающей запятой, целым числом или перечислением.

`@typeInfo(@TypeOf(ptr)).Pointer.alignment` должен быть `>= @sizeOf(T)`.

`AtomicOrder` можно найти с помощью `@import("std").builtin.AtomicOrder`.

Смотрите также:
- [@atomicStore](#atomicstore)
- [@atomicLoad](#atomicload)
- [@atomicRmw](#atomicrmw)
- [@fence](#fence)
- [@cmpxchgStrong](#cmpxchgstrong)


###### `compileError`

```zig
@compileError(comptime msg: []const u8) noreturn
```

Эта функция при семантическом анализе вызывает ошибку компиляции с сообщением `msg`.

Существует несколько способов избежать семантической проверки кода, таких как использование `if` или `switch` с
константами времени компиляции и функциями `comptime`.


###### `compileLog`

```zig
@compileLog(args: ...) void
```

Эта функция выводит аргументы, переданные ей во время компиляции.

Чтобы случайно не оставить в кодовой базе сообщения журнала компиляции, в сборку добавляется ошибка компиляции,
указывающая на сообщение журнала компиляции. Эта ошибка предотвращает генерацию кода, но не мешает анализу.

Эта функция может быть использована для выполнения "printf debugging" при выполнении кода во время компиляции.

```zig
const print = @import("std").debug.print;

const num1 = blk: {
    var val1: i32 = 99;
    @compileLog("comptime val1 = ", val1);
    val1 = val1 + 1;
    break :blk val1;
};

test "main" {
    @compileLog("comptime in main");

    print("Runtime in main, num1 = {}.\n", .{num1});
}
```
```bash
$ zig test test_compileLog_builtin.zig
doc/langref/test_compileLog_builtin.zig:11:5: error: found compile log statement
    @compileLog("comptime in main");
    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
doc/langref/test_compileLog_builtin.zig:5:5: note: also here
    @compileLog("comptime val1 = ", val1);
    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Compile Log Output:
@as(*const [16:0]u8, "comptime in main")
@as(*const [16:0]u8, "comptime val1 = "), @as(i32, 99)
```


###### `constCast`

```zig
@constCast(value: anytype) DestType
```

Удаляет квалификатор `const` из указателя.


###### `ctz`

```zig
@ctz(operand: anytype) anytype
```

`@TypeOf(operand)` должен быть целочисленного типа или целочисленного векторного типа.

Операнд может быть [целым числом](#integers) или [вектором](#vectors).

Подсчитывает количество наименее значимых (конечных в порядке возрастания) нулей в целом числе - "подсчитывает конечные
нули".

Если операнд является целым числом известным по [время компиляции](#comptime) тип возвращаемого значения -
`comptime_int`. В противном случае возвращаемым типом будет целое число без знака или вектор целых чисел без знака с
минимальным количеством битов, которое может представлять количество битов целочисленного типа.

Если операнд равен нулю, `@ctz` возвращает разрядность целого типа `T`.

Смотрите также:
- [@clz](#clz)
- [@popCount](#popcount)


###### `cUndef`

```zig
@cUndef(comptime name: []const u8) void
```

Эта функция может выполняться только внутри `@cImport`.

Это добавит `#undef $name` во временный буфер `@cImport`.

Смотрите также:
- [Import from C Header File](#import-from-c-header-file)
- [@cImport](#cimport)
- [@cDefine](#cdefine)
- [@cInclude](#cinclude)


###### `cVaArg`

```zig
@cVaArg(operand: *std.builtin.VaList, comptime T: type) T
```

Реализует макрос C `va_arg`.

Смотрите также:
- [@cVaArg](#cvaarg)
- [@cVaCopy](#cvacopy)
- [@cVaEnd](#cvaend)


###### `cVaCopy`

```zig
@cVaCopy(src: *std.builtin.VaList) std.builtin.VaList
```

Реализует макрос C `va_copy`.

Смотрите также:
- [@cVaArg](#cvaarg)
- [@cVaCopy](#cvacopy)
- [@cVaEnd](#cvaend)


###### `cVaEnd`

```zig
@cVaEnd(src: *std.builtin.VaList) void
```

Реализует макрос C `va_end`.

Смотрите также:
- [@cVaArg](#cvaarg)
- [@cVaCopy](#cvacopy)
- [@cVaEnd](#cvaend)


###### `cVaStart`

```zig
@cVaStart() std.builtin.VaList
```

Реализует макрос C `va_start`. Действителен только внутри переменной функции.

Смотрите также:
- [@cVaArg](#cvaarg)
- [@cVaCopy](#cvacopy)
- [@cVaEnd](#cvaend)


###### `divExact`

```zig
@divExact(numerator: T, denominator: T) T

```

Точное деление. Вызывающий гарантирует `denominator != 0` и `@divTrunc(numerator, denominator) * denominator ==
numerator`.

```zig
- @divExact(6, 3) == 2
- @divExact(a, b) * b == a
```

Для функции, возвращающей возможный код ошибки, используйте `@import("std").math.divExact`.

Смотрите также:
- [@divTrunc](#divexact)
- [@divFloor](#divfloor)


###### `divFloor`

```zig
@divFloor(numerator: T, denominator: T) T
```

Деление с минимумом. Округление до отрицательной бесконечности. Для целых чисел без знака это то же самое, что
`numerator / denominator`.

Вызывающий гарантирует `denominator != 0` и `!(@typeInfo(T) == .Int` и `T.is_signed` и `numerator == std.math.minInt(T)
and denominator == -1)`.

```zig
- @divFloor(-5, 3) == -2
- (@divFloor(a, b) * b) + @mod(a, b) == a
```

Для функции, возвращающей возможный код ошибки, используйте `@import("std").math.divFloor`.

Смотрите также:
- [@divTrunc](#divtrunc)
- [@divExact](#divexact)


###### `divTrunc`

```zig
@divTrunc(numerator: T, denominator: T) T
```

Усеченное деление. Округляется до нуля. Для целых чисел без знака это то же самое, что `numerator / denominator`.

Вызывающий гарантирует `denominator != 0` и `!(@typeInfo(T) == .Int` и `T.is_signed` и`numerator == std.math.minInt(T)
and denominator == -1)`.

```zig
- @divTrunc(-5, 3) == -1
- (@divTrunc(a, b) * b) + @rem(a, b) == a
```

Для функции, которая возвращает возможный код ошибки, используйте `@import("std").math.divTrunc`.

Смотрите также:
- [@divFloor](#divfloor)
- [@divExact](#divexact)


###### `embedFile`

```zig
@embedFile(comptime path: []const u8) *const [N:0]u8
```

Эта функция возвращает постоянный во время компиляции указатель на нуль-терминированный массив фиксированного размера,
длина которого равна количеству байт в файле, указанном в `path`. Содержимое массива соответствует содержимому файла.
Это эквивалентно [строковому литералу](#string-literals-and-unicode-code-point-literals) с содержимым файла.

Путь к текущему файлу может быть абсолютным или относительным, как и `@import`.

Смотрите также:
- [@import](#import)


###### `enumFromInt`

```zig
@enumFromInt(integer: anytype) anytype
```

Преобразует целое число в значение [перечисления](#enum). Возвращаемый тип - это тип предполагаемого результата.

Попытка преобразовать целое число которое не представляет значения в выбранном типе перечисления приводит к
[неопределенному поведению](#undefined-behavior) с проверкой безопасности.

Смотрите также:
- [@intFromEnum](#intfromenum)



###### `errorFromInt`

```zig
@errorFromInt(value: std.meta.Int(.unsigned, @bitSizeOf(anyerror))) anyerror
```

Преобразует из целочисленного представления ошибки в [глобальный тип набора ошибок](#the-global-error-set).

Обычно рекомендуется избегать такого преобразования, поскольку целочисленное представление ошибки нестабильно при
изменении исходного кода.

Попытка преобразовать целое число которое не соответствует какой-либо ошибке приводит к неопределенному поведению с
проверкой безопасности.

Смотрите также:
- [@intFromError](#intfromerror)


###### `errorName`

```zig
@errorName(err: anyerror) [:0]const u8
```

Эта функция возвращает строковое представление ошибки. Строковое представление `error.OutOfMem` - это `"OutOfMem"`.

Если во всем приложении нет вызовов функции `@errorName` или все вызовы имеют известное во время компиляции значение
`err`, то таблица имен ошибок сгенерирована не будет.


###### `errorReturnTrace`

```zig
@errorReturnTrace() ?*builtin.StackTrace
```

Если двоичный файл создан с использованием трассировки возврата ошибки и эта функция вызывается в функции, которая
вызывает функцию с типом возврата `error` или `error union`, возвращает объект трассировки стека. В противном случае
возвращает значение [null](#null).


###### `errorCast`

```zig
@errorCast(value: anytype) anytype
```

Преобразует набор ошибок или значение объединения ошибок из одного набора ошибок в другой набор ошибок. Тип
возвращаемого значения - это тип предполагаемого результата. Попытка преобразовать ошибку которой нет в целевом наборе
ошибок приводит неопределенному поведению с проверкой безопасности.


###### `export`

```zig
@export(declaration, comptime options: std.builtin.ExportOptions) void
```

Создает символ в выходном объектном файле.

Декларация должна быть одной из двух вещей:
- Идентификатор (`x`), идентифицирующий [функцию](#functions) или [переменную](#variables).
- Доступ к полю (`x.y`) для поиска [функции](#functions) или [переменной](#variables).

Этот встроенный модуль может быть вызван из блока `comptime` для условного экспорта символов. Если объявлением является
функция с соглашением о вызове C и `options.linkage` является строгой то это эквивалентно ключевому слову `export`,
используемому в функции:

```zig
comptime {
    @export(internalName, .{ .name = "foo", .linkage = .strong });
}

fn internalName() callconv(.C) void {}
```
```bash
$ zig build-obj export_builtin.zig
```

Это эквивалентно:

```zig
export fn foo() void {}
```
```bash
$ zig build-obj export_builtin_equivalent_code.zig
```

Обратите внимание, что даже при использовании `export` синтаксис `@"foo"` для [идентификаторов](#identifiers) можно
использовать для выбора любой строки в качестве имени символа:

```zig
export fn @"A function name that is a complete sentence."() void {}
```
```zig
$ zig build-obj export_any_symbol_name.zig
```

Глядя на полученный объект, вы можете увидеть, что символ используется дословно:

`00000000000001f0 T Имя функции, представляющее собой законченное предложение.`

Смотрите также:
- [Exporting a C Library](#exporting-a-c-library)


###### `extern`

```zig
@extern(T: type, comptime options: std.builtin.ExternOptions) T
```

Создает ссылку на внешний символ в выходном объектном файле. `T` должен быть типом указателя.

Смотрите также:
- [@export](#export)


###### `fence`

```zig
@fence(order: AtomicOrder) void
```

Функция `fence` используется для введения промежуточных границ между операциями.

`AtomicOrder` можно найти с помощью `@import("std").builtin.AtomicOrder`.

Смотрите также:
- [@atomicStore](#atomicstore)
- [@atomicLoad](#atomicload)
- [@atomicRmw](#atomicrmw)
- [@cmpxchgWeak](#cmpxchgweak)
- [@cmpxchgStrong](#cmpxchgstrong)


###### `field`

```zig
@field(lhs: anytype, comptime field_name: []const u8) (field)
```

Выполняет доступ к полю с помощью строки во время компиляции. Работает как с полями так и с объявлениями.

```zig
const std = @import("std");

const Point = struct {
    x: u32,
    y: u32,

    pub var z: u32 = 1;
};

test "field access by string" {
    const expect = std.testing.expect;
    var p = Point{ .x = 0, .y = 0 };

    @field(p, "x") = 4;
    @field(p, "y") = @field(p, "x") + 1;

    try expect(@field(p, "x") == 4);
    try expect(@field(p, "y") == 5);
}

test "decl access by string" {
    const expect = std.testing.expect;

    try expect(@field(Point, "z") == 1);

    @field(Point, "z") = 2;
    try expect(@field(Point, "z") == 2);
}
```
```bash
$ zig test test_field_builtin.zig
1/2 test_field_builtin.test.field access by string...OK
2/2 test_field_builtin.test.decl access by string...OK
All 2 tests passed.
```


###### `fieldParentPtr`

```zig
@fieldParentPtr(comptime field_name: []const u8, field_ptr: *T) anytype
```

Заданный указатель на поле возвращает базовый указатель структуры.


###### `floatCast`

```zig
@floatCast(value: anytype) anytype
```

Преобразование из одного типа с плавающей запятой в другой. Это приведение безопасно, но может привести к потере точности
числового значения. Возвращаемый тип - это предполагаемый тип результата.


###### `floatFromInt`

```zig
@floatFromInt(int: anytype) anytype
```

Преобразует целое число в наиболее близкое представление с плавающей запятой. Возвращаемый тип - это предполагаемый тип
результата. Для преобразования другим способом используйте [@intFromFloat](#intfromfloat). Эта операция допустима для
всех значений всех целочисленных типов.


###### `frameAddress`

```zig
@frameAddress() usize
```

Эта функция возвращает базовый указатель текущего кадра стека.

Последствия этого зависят от конкретной цели и не согласованы на всех платформах. Адрес кадра стека может быть
недоступен в режиме релиза из-за агрессивной оптимизации.

Эта функция действительна только в пределах области действия функции.


`@hasDecl`

```zig
@hasDecl(comptime Container: type, comptime name: []const u8) bool
```

Возвращает логическое значение, если [контейнер](#containers) имеет объявление соответствующее имени или нет.

```zig
const std = @import("std");
const expect = std.testing.expect;

const Foo = struct {
    nope: i32,

    pub var blah = "xxx";
    const hi = 1;
};

test "@hasDecl" {
    try expect(@hasDecl(Foo, "blah"));

    // Несмотря на то, что `hi` является закрытым, @hasDecl возвращает значение true, потому что этот тест
    // находится в той же области файла, что и Foo. Он вернул бы значение false, если бы Foo был объявлен
    // в другом файле.
    try expect(@hasDecl(Foo, "hi"));

    // @hasDecl предназначен для объявлений, а не для полей.
    try expect(!@hasDecl(Foo, "nope"));
    try expect(!@hasDecl(Foo, "nope1234"));
}
```
```bash
$ zig test test_hasDecl_builtin.zig
1/1 test_hasDecl_builtin.test.@hasDecl...OK
All 1 tests passed.
```

Смотрите также:
- [@hasField](#hasfield)


###### `hasField`

```zig
@hasField(comptime Container: type, comptime name: []const u8) bool
```

Возвращает логическое значение, если существует имя поля структуры, объединения или перечисления.

Результатом является константа времени компиляции.

Не содержит функций, переменных или констант.

Смотрите также:
- [@hasDecl](#hasdecl)


###### `import`

```zig
@import(comptime path: []const u8) type
```

Эта функция находит zig-файл, соответствующий `path` и добавляет его в сборку, если он еще не добавлен.

Исходные файлы Zig - это неявные структуры с именем равным базовому имени файла с усеченным расширением. `@import`
возвращает тип структуры соответствующий файлу.

Объявления содержащие ключевое слово `pub` могут ссылаться на другой исходный файл, отличный от того в котором они
объявлены.

`path` может быть относительным путем или именем пакета. Если это относительный путь, то он относится к файлу, который
содержит вызов функции `@import`.

Следующие пакеты всегда доступны:
- `@import("std")` - Стандартная библиотека Zig
- `@import("builtin")` - Информация о конкретной цели, команда `zig build-exe --show-builtin` выводит исходный код в
стандартный вывод для справки.
- `@import("root")` - Корневой исходный файл, обычно это `src/main.zig`, но зависит от того какой файл собирается.

Смотрите также:
- [Compile Variables](#compile-variables)
- [@embedFile](#embedfile)


###### `inComptime`

```zig
@inComptime() bool
```

Возвращает логическое значение если встроенная функция была запущена в контексте `comptime`. Результатом является
константа времени компиляции.

Это может быть использовано для предоставления альтернативных реализаций функций, совместимых с `comptime`. Ее не
следует использовать, например, для исключения определенных функций из вычисления во время компиляции.

Смотрите также:
- [comptime](#comptime)


###### `intCast`

```zig
@intCast(int: anytype) anytype
```

Преобразует целое число в другое целое число, сохраняя при этом то же числовое значение. Возвращаемый тип - это
предполагаемый тип результата. Попытка преобразовать число которое находится за пределами диапазона целевого типа
приводит [неопределенному поведению](#undefined-behavior) с проверкой безопасности.

```zig
test "integer cast panic" {
    var a: u16 = 0xabcd; // известный во время выполнения
    _ = &a;
    const b: u8 = @intCast(a);
    _ = b;
}
```
```bash
$ zig test test_intCast_builtin.zig
1/1 test_intCast_builtin.test.integer cast panic...thread 3573820 panic: integer cast truncated bits
/home/andy/src/zig/doc/langref/test_intCast_builtin.zig:4:19: 0x103ce6b in test.integer cast panic (test)
    const b: u8 = @intCast(a);
                  ^
/home/andy/src/zig/lib/compiler/test_runner.zig:157:25: 0x1048290 in mainTerminal (test)
        if (test_fn.func()) |_| {
                        ^
/home/andy/src/zig/lib/compiler/test_runner.zig:37:28: 0x103e24b in main (test)
        return mainTerminal();
                           ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x103d389 in posixCallMainAndExit (test)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x103cef1 in _start (test)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
error: the following test command crashed:
/home/andy/src/zig/.zig-cache/o/a3b6539437d55800e891e62090700351/test
```

Чтобы урезать значащие биты числа вне диапазона целевого типа, используйте `@truncate`.

Если `T` равно `comptime_int` то это семантически эквивалентно [преобразованию к типу](#type-coercion).


###### `intFromBool`

```zig
@intFromBool(value: bool) u1
```

Преобразует значение `true` в `@as(u1, 1)` и значение `false` в `@as(u1, 0)`.


###### `intFromEnum`

```zig
@intFromEnum(enum_or_tagged_union: anytype) anytype
```

Преобразует значение перечисления в его целочисленный тип тега. Когда передается объединение с тегом, значение тега
используется в качестве значения перечисления.

Если существует только одно возможное значение перечисления, результатом будет значение `comptime_int`, известное во
[время компиляции](#comptime).

Смотрите также:
- [@enumFromInt](#enumfromint)


###### `intFromError`

```zig
@intFromError(err: anytype) std.meta.Int(.unsigned, @bitSizeOf(anyerror))
```

Поддерживает следующие типы:
- [Набор глобальных ошибок](#the-global-error-set)
- [Тип набора ошибок](#error-set-type)
- [Тип объединения ошибок](#error-union-type)

Преобразует ошибку в целочисленное представление `error`.

Обычно рекомендуется избегать такого преобразования, поскольку целочисленное представление `error` нестабильно при
изменении исходного кода.

Смотрите также:
- [@errorFromInt](#errorfromint)


###### `intFromFloat`

```zig
@intFromFloat(float: anytype) anytype
```

Преобразует целую часть числа с плавающей запятой в выводимый тип результата.

Если целая часть числа с плавающей запятой не может соответствовать целевому типу это приводит неопределенному поведению
с проверкой безопасности.

Смотрите также:
- [@floatFromInt](#floatfromint)


###### `intFromPtr`

```zig
@intFromPtr(value: anytype) usize
```

Преобразует значение в `usize`, который является адресом указателя. Значение может быть `*T` или `?*T`.

Для преобразования другим способом используйте [@ptrFromInt](#ptrfromint).


###### `max`

```zig
@max(a: T, b: T) T
```

Возвращает максимальное значение `a` и `b`. Эта встроенная функция принимает целые числа, значения с плавающей запятой и
векторы любого из них. В последнем случае операция выполняется поэлементно.

`NaNs` обрабатываются следующим образом: если один из операндов (парной) операции равен `NaN`, возвращается другой
операнд.
Если оба операнда равны `NaN`, возвращается значение `NaN`.

Смотрите также:
- [@min](#min)
- [Vectors](#vectors)


###### `memcpy`

```zig
@memcpy(noalias dest, noalias source) void
```

Эта функция копирует байты из одной области памяти в другую.

`desk` должен быть изменяемым срезом, изменяемым [указателем](#pointers) на массив или изменяемым указателем на
множество элементов. Он может иметь любое выравнивание и любой тип элемента.

`source` должен быть срез, указатель на массив или [указатель](#pointers) на множество элементов. Он может иметь любое
выравнивание и любой тип элемента.

Тип исходного элемента должен поддерживать [преобразование типа](#type-coercion) к типу элемента `dest`. Однако типы
элементов могут иметь разный размер ABI, что может привести к снижению производительности.

Как и в случае с циклами [for](#for), по крайней мере один из `source` и `dest` должен указывать длину, а если указаны
две длины, они должны быть равны.

Наконец, две области памяти не должны перекрываться.


###### `memset`

```zig
@memset(dest, elem) void
```

Эта функция присваивает всем элементам области памяти значение `elem`.

`dest` должен быть изменяемым срезом или изменяемым указателем на массив. Он может иметь любое выравнивание и любой
тип элемента.

`elem` привязывается к типу элемента `dest`.

Для безопасного удаления конфиденциального содержимого из памяти вам следует использовать `std.crypto.utils.secureZero`.


###### `min`

```zig
@min(a: T, b: T) T
```

Возвращает минимальное значение `a` и `b`. Эта встроенная функция принимает целые числа, значения с плавающей запятой и
векторы любого из них. В последнем случае операция выполняется поэлементно.

`NaNs` обрабатываются следующим образом: если один из операндов (парной) операции равен `NaN`, возвращается другой
операнд.
Если оба операнда равны `NaN`, возвращается значение `NaN`.

Смотрите также:
- [@max](#max)
- [Vectors](#vectors)


###### `wasmMemorySize`

```zig
@wasmMemorySize(index: u32) usize
```

Эта функция возвращает размер памяти `Wasm`, определенный по индексу как значение без знака в единицах страниц `Wasm`.
Обратите внимание, что размер каждой страницы `Wasm` составляет 64 Кбайт.

Эта функция является встроенной на низком уровне и не содержит механизмов безопасности, которые обычно используются
разработчиками аллокаторов ориентированных на `Wasm`. Поэтому, если вы не пишете новый аллокатор с нуля, вам
следует использовать что-то вроде `@import("std").heap.WasmPageAllocator`.

Смотрите также:
- [@wasmMemoryGrow](#wasmmemorygrow)


###### `wasmMemoryGrow`

```zig
@wasmMemoryGrow(index: u32, delta: usize) isize
```

Эта функция увеличивает объем памяти `Wasm`, определяемый по индексу с разницей в единицах беззнакового количества
страниц `Wasm`. Обратите внимание, что размер каждой страницы `Wasm` составляет 64 Кбайт. В случае успеха возвращает
предыдущий размер памяти; в случае неудачи, если выделение не удалось, возвращает значение -1.

Эта функция является встроенной на низком уровне и не содержит механизмов безопасности, которые обычно используются
разработчиками аллокаторов, ориентированных на `Wasm`. Поэтому, если вы не пишете новый аллокатор с нуля, вам
следует использовать что-то вроде `@import("std").heap.WasmPageAllocator`.

```zig
const std = @import("std");
const native_arch = @import("builtin").target.cpu.arch;
const expect = std.testing.expect;

test "@wasmMemoryGrow" {
    if (native_arch != .wasm32) return error.SkipZigTest;

    const prev = @wasmMemorySize(0);
    try expect(prev == @wasmMemoryGrow(0, 1));
    try expect(prev + 1 == @wasmMemorySize(0));
}
```
```bash
$ zig test test_wasmMemoryGrow_builtin.zig
1/1 test_wasmMemoryGrow_builtin.test.@wasmMemoryGrow...SKIP
0 passed; 1 skipped; 0 failed.
```

Смотрите также:
- [@wasmMemorySize](#wasmmemorysize)


###### `mod`

```zig
@mod(numerator: T, denominator: T) T
```
Деление по модулю. Для целых чисел без знака это то же самое, что `numerator % denominator`. Вызывающий гарантирует
`denominator > 0`, в противном случае операция приведет к [делению остатка на ноль](#remainder-division-by-zero) при
включенной проверке безопасности во время выполнения.

```zig
- @mod(-5, 3) == 1
- (@divFloor(a, b) * b) + @mod(a, b) == a
```

Информацию о функции возвращающей код ошибки, смотрите в разделе `@import("std").math.mod`.

Смотрите также:
- [@rem](#rem)


###### `mulWithOverflow`

```zig
@mulWithOverflow(a: anytype, b: anytype) struct { @TypeOf(a, b), u1 }
```

Выполняет `a * b` и возвращает кортеж с результатом и возможным битом переполнения.


###### `panic`

```zig
@panic(message: []const u8) noreturn
```

Вызывает функцию обработчика паники. По умолчанию функция обработчика паники вызывает общедоступную функцию паники,
доступную в корневом исходном файле или если она не указана функцию `std.builtin.default_panic` из `std/builtin.zig`.

Как правило, лучше использовать `@import("std").debug.panic`. Однако `@panic` может быть полезен для двух сценариев:
- Из библиотечного кода, вызывая функцию паники программиста, если она была доступна в корневом исходном файле.
- При смешивании кода на C и Zig, вызывайте каноническую реализацию паники в нескольких файлах `.o`.

Смотрите также:
- [Root Source File](#root-source-file)


###### `popCount`

```zig
@popCount(operand: anytype) anytype
```

`@TypeOf(operand)` должен быть целочисленного типа.

Операнд может быть [целым числом](#integers) или [вектором](#vectors).

Подсчитывает количество битов заданных в целом числе - "количество населения".

Если операнд является целым числом известным во [время компиляции](#comptime) тип возвращаемого значения -
`comptime_int`. В противном случае тип возвращаемого значения - целое число без знака или вектор целых чисел без знака с
минимальным количеством битов которое может представлять количество битов целочисленного типа.

Смотрите также:
- [@ctz](#ctz)
- [@clz](#clz)


###### `prefetch`

```zig
@prefetch(ptr: anytype, comptime options: PrefetchOptions) void
```

Эта встроенная функция сообщает компилятору, что он должен выдать инструкцию предварительной выборки если она
поддерживается целевым процессором. Если целевой процессор не поддерживает запрошенную инструкцию предварительной
выборки эта встроенная функция не работает. Эта функция не влияет на поведение программы, а только на характеристики
производительности.

Аргумент `ptr` может иметь любой тип указателя и определяет адрес памяти для предварительной выборки. Эта функция не
разыменовывает указатель, совершенно законно передавать этой функции указатель на недопустимую память и это не приведет
к неопределенному поведению.

`PrefetchOptions` можно найти с помощью `@import("std").builtin.PrefetchOptions`.


###### `ptrCast`

```zig
@ptrCast(value: anytype) anytype
```

Преобразует указатель одного типа в указатель другого типа. Возвращаемый тип - это предполагаемый тип результата.

[Опциональные указатели](#optional-pointers) разрешены. Приведение опционального указателя который не является
[null](#null) к неопциональному указателю вызывает неопределенное поведение с проверкой безопасности.

`@ptrCast` не может быть использован для:
- Удаляет квалификатор `const`, используйте [@constCast](#constcast).
- Удаляет квалификатор `volatile`, используйте [@volatileCast](#volatilecast).
- Изменяет адресное пространство указателя, используйте [@addrSpaceCast](#addrspacecast).
- Увеличивает выравнивание указателя, используйте [@alignCast](#aligncast).
- Для приведения указателя не являющимся срезом к срезу, используйте синтаксис среза `ptr[start..end]`.


###### `ptrFromInt`

```zig
@ptrFromInt(address: usize) anytype
```

Преобразует целое число в [указатель]. Возвращаемый тип - это выводимый тип результата. Для преобразования другим
способом используйте `[@intFromPtr]`. Приведение адреса равного 0 к типу назначения который не является [опциональным] и
не имеет атрибута `allowzero` приведет к [недопустимому сбою при приведении указателя](#pointer-cast-invalid-null) когда
включены проверки безопасности во время выполнения.

Если тип указателя назначения не допускает нулевого адреса, а адрес равен нулю это вызывает [неопределенное
поведение](#undefined-behavior) с проверкой безопасности.


###### `rem`

```zig
@rem(numerator: T, denominator: T) T
```

Деление остатка. Для целых чисел без знака это то же самое, что `numerator % denominator`. Вызывающий гарантирует
`denominator > 0` в противном случае операция приведет к [делению остатка на ноль](#remainder-division-by-zero) при
включенной проверке безопасности во время выполнения.

```zig
- @rem(-5, 3) == -2
- (@divTrunc(a, b) * b) + @rem(a, b) == a
```

Информацию о функции, возвращающей код ошибки, смотрите в разделе `@import("std").math.rem`.

Смотрите также:
- [@mod](#mod)


###### `returnAddress`

```zig
@returnAddress() usize
```

Эта функция возвращает адрес следующей команды машинного кода которая будет выполнена при возврате текущей функции.

Последствия этого зависят от конкретной цели и не согласованы на всех платформах.

Эта функция действительна только в пределах области действия функции. Если функция встроена в вызывающую функцию,
возвращаемый адрес будет применяться к вызывающей функции.


###### `select`

```zig
@select(comptime T: type, pred: @Vector(len, bool), a: @Vector(len, T), b: @Vector(len, T)) @Vector(len, T)
```

Выбирает значения по элементам из `a` или `b` на основе `pred`. Если значение `pred[i]` равно `true` соответствующим
элементом в результате будет `a[i]`, а в противном случае `b[i]`.

Смотрите также:
- [Vectors](#vectors)


###### `setAlignStack`

```zig
@setAlignStack(comptime alignment: u29) void
```

Гарантирует, что функция будет иметь выравнивание стека как минимум в байтах выравнивания.


###### `setCold`

```zig
@setCold(comptime is_cold: bool) void
```

Сообщает оптимизатору, что текущая функция вызывается (или не вызывается) редко. Эта функция действительна только в
пределах области действия функции.


###### `setEvalBranchQuota`

```zig
@setEvalBranchQuota(comptime new_quota: u32) void
```

Увеличьте максимальное количество обратных переходов которые могут быть использованы при выполнении кода во время
компиляции прежде чем прекратить выполнение и вызвать ошибку компиляции.

Если значение `new_quota` меньше квоты по умолчанию (1000) или ранее явно установленной квоты оно игнорируется.

Пример:

```zig
test "foo" {
    comptime {
        var i = 0;
        while (i < 1001) : (i += 1) {}
    }
}
```
```bash
$ zig test test_without_setEvalBranchQuota_builtin.zig
doc/langref/test_without_setEvalBranchQuota_builtin.zig:4:9: error: evaluation exceeded 1000 backwards branches
        while (i < 1001) : (i += 1) {}
        ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
doc/langref/test_without_setEvalBranchQuota_builtin.zig:4:9: note: use @setEvalBranchQuota() to raise the branch limit from 1000
```

Теперь мы используем `@setEvalBranchQuota`:

```zig
test "foo" {
    comptime {
        @setEvalBranchQuota(1001);
        var i = 0;
        while (i < 1001) : (i += 1) {}
    }
}
```
```bash
$ zig test test_setEvalBranchQuota_builtin.zig
1/1 test_setEvalBranchQuota_builtin.test.foo...OK
All 1 tests passed.
```

Смотрите также:
- [comptime](#comptime)


###### `setFloatMode`

```zig
@setFloatMode(comptime mode: FloatMode) void
```

Изменяет правила текущей области видимости о том как определяются операции с плавающей запятой.

- Строгий (по умолчанию) - операции с плавающей запятой выполняются в строгом соответствии с требованиями стандарта IEEE.
- Оптимизированный - операции с плавающей запятой могут выполнять все перечисленные ниже действия:
  - Предположим, что аргументы и результат не являются NaN. Для сохранения заданного поведения в NAN требуется
  оптимизация, но значение результата не определено.
  - Предположим, что аргументы и результат не равны +/-Inf. Для сохранения заданного поведения по сравнению с +/-Inf
  требуется оптимизация, но значение результата не определено.
  - Считайте знак нулевого аргумента или результата несущественным.
  - Используйте значение обратное значению аргумента, а не выполняйте деление.
  - Выполняйте сокращение с плавающей запятой (например, объединяйте умножение с последующим сложением в слитное
  умножение-сложение).
  - Выполните алгебраически эквивалентные преобразования, которые могут изменить результаты с плавающей запятой
  (например, переассоциировать).
- Это эквивалентно `-ffast-math` в GCC.

Режим с плавающей запятой наследуется дочерними областями и может быть переопределен в любой области. Вы можете
установить режим с плавающей запятой в области структуры или модуля с помощью блока `comptime`.

`FloatMode` можно найти с помощью `@import("std").builtin.FloatMode`.

Смотрите также:
- [Floating Point Operations](#floating-point-operations)


###### `setRuntimeSafety`

```zig
@setRuntimeSafety(comptime safety_on: bool) void
```

Определяет, включены ли проверки безопасности во время выполнения для области содержащей вызов функции.

```zig
test "@setRuntimeSafety" {
    // Встроенный модуль применяется к области, в которой он вызывается. Таким образом, в данном случае целочисленное переполнение
    // не будет перехвачено в режимах ReleaseFast и ReleaseSmall:
    // var x: u8 = 255;
    // x += 1; // неопределенное поведение в режимах ReleaseFast/ReleaseSmall.
    {
        // Однако в этом блоке включена защита, поэтому здесь выполняются проверки безопасности,
        // даже в режимах ReleaseFast и ReleaseSmall.
        @setRuntimeSafety(true);
        var x: u8 = 255;
        x += 1;

        {
            // Значение может быть переопределено в любой области видимости. Таким образом, здесь переполнение целого числа
            // не было бы обнаружено ни в одном режиме сборки.
            @setRuntimeSafety(false);
            // var x: u8 = 255;
            // x += 1; // неопределенное поведение во всех режимах сборки.
        }
    }
}
```
```bash
$ zig test test_setRuntimeSafety_builtin.zig -OReleaseFast
1/1 test_setRuntimeSafety_builtin.test.@setRuntimeSafety...thread 3579742 panic: integer overflow
/home/andy/src/zig/doc/langref/test_setRuntimeSafety_builtin.zig:11:11: 0x100ae54 in test.@setRuntimeSafety (test)
        x += 1;
          ^
/home/andy/src/zig/lib/compiler/test_runner.zig:157:25: 0x100c9b0 in main (test)
        if (test_fn.func()) |_| {
                        ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x100af44 in posixCallMainAndExit (test)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x100ae71 in _start (test)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
error: the following test command crashed:
/home/andy/src/zig/.zig-cache/o/7c74a2939d84ddd0f45519d7a9f1c0f7/test
```

Примечание: [планируется](https://github.com/ziglang/zig/issues/978) заменить `@setRuntimeSafety` с `@optimizeFor`.


###### `shlExact`

```zig
@shlExact(value: T, shift_amt: Log2T) T
```

Выполняет операцию сдвига влево (`<<`). Для целых чисел без знака результат не определен, если смещены какие-либо биты
на 1. Для целых чисел со знаком результат не определен, если смещены какие-либо биты которые не совпадают с
результирующим битом знака.

Тип `shift_amt` - это целое число без знака с битами `log2(@TypeInfo(T).Int.bits)`. Это происходит потому, что
`shift_amt >= @TypeInfo(T).Int.bits` - это неопределенное поведение.

`comptime_int` моделируется как целое число с бесконечным числом битов, что означает, что в таком случае `@shlExact`
всегда выдает результат и не может вызвать ошибку компиляции.

Смотрите также:
- [@shrExact](#shrexact)
- [@shlWithOverflow](#shlwithoverflow)


###### `shlWithOverflow`

```zig
@shlWithOverflow(a: anytype, shift_amt: Log2T) struct { @TypeOf(a), u1 }
```

Выполняет `a << b` и возвращает кортеж с результатом и возможным битом переполнения.

Тип `shift_amt` - это целое число без знака с битами `log2(@TypeInfo(@TypeOf(a)).Int.bits)`. Это происходит потому, что
`shift_amt >= @TypeInfo(@TypeOf(a)).Int.bits` - это неопределенное поведение.

Смотрите также:
- [@shlExact](#shlexact)
- [@shrExact](#shrexact)


###### `shrExact`

```zig
@shrExact(value: T, shift_amt: Log2T) T
```

Выполняет операцию сдвига вправо (`>>`). Вызывающий объект гарантирует, что сдвиг не приведет к смещению ни на 1 бит.

Тип `shift_amt` - это целое число без знака с битами `log2(@TypeInfo(T).Int.bits)`. Это происходит потому, что
`shift_amt >= @TypeInfo(T).Int.bits` - это неопределенное поведение.

Смотрите также:
- [@shlExact](#shlexact)
- [@shlWithOverflow](#shlwithoverflow)


###### `shuffle`

```zig
@shuffle(comptime E: type, a: @Vector(a_len, E), b: @Vector(b_len, E), comptime mask: @Vector(mask_len, i32)) @Vector(mask_len, E)
```

Создает новый [вектор](#vectors), выбирая элементы из `a` и `b` на основе маски.

Каждый элемент в маске выбирает элемент либо из `a`, либо из `b`. Положительные числа выбираются из `a`, начиная с 0.
Отрицательные значения выбираются из `b`, начиная с `-1` и далее вниз. Рекомендуется использовать оператор `~` для
индексов, начиная с `b` чтобы оба индекса могли начинаться с 0 (т.е. `~@as(i32, 0)` равно `-1`).

Для каждого элемента маски, если он или выбранное значение из `a` или `a` не определено то результирующий элемент не
определен.

`a_len` и `b_len` могут отличаться по длине. Индексы элементов в маске, выходящие за границы, приводят к ошибкам компиляции.

Если значение `a` или `b` не определено это эквивалентно вектору всех неопределенных значений той же длины, что и у
другого вектора. Если оба вектора не определены, `@shuffle` возвращает вектор со всеми неопределенными элементами.

`E` должно быть [целым числом](#integers), с [плавающей запятой](#floats), [указателем](#pointers) или
[логическим](#primitive-types) значением. Маска может быть любой длины и ее длина определяет длину результата.

```zig
const std = @import("std");
const expect = std.testing.expect;

test "vector @shuffle" {
    const a = @Vector(7, u8){ 'o', 'l', 'h', 'e', 'r', 'z', 'w' };
    const b = @Vector(4, u8){ 'w', 'd', '!', 'x' };

    // Для перетасовки в пределах одного вектора передайте undefined в качестве второго аргумента.
    // Обратите внимание, что мы можем изменять порядок, дублировать или пропускать элементы входного вектора
    const mask1 = @Vector(5, i32){ 2, 3, 1, 1, 0 };
    const res1: @Vector(5, u8) = @shuffle(u8, a, undefined, mask1);
    try expect(std.mem.eql(u8, &@as([5]u8, res1), "hello"));

    // Объединение двух векторов
    const mask2 = @Vector(6, i32){ -1, 0, 4, 1, -2, -3 };
    const res2: @Vector(6, u8) = @shuffle(u8, a, b, mask2);
    try expect(std.mem.eql(u8, &@as([6]u8, res2), "world!"));
}
```
```bash
$ zig test test_shuffle_builtin.zig
1/1 test_shuffle_builtin.test.vector @shuffle...OK
All 1 tests passed.
```

Смотрите также:
- [Vectors](#vectors)


###### `sizeOf`

```zig
@sizeOf(comptime T: type) comptime_int
```

Эта функция возвращает количество байт необходимое для сохранения `T` в памяти. Результатом является константа времени
компиляции, зависящая от конкретной цели.

Этот размер может содержать дополнительные байты. Если бы в памяти было два последовательных `T` то дополнением было бы
смещение в байтах между элементом с индексом 0 и элементом с индексом 1. Для [целого числа](#integers) подумайте, хотите
ли вы использовать `@sizeOf(T)` или `@TypeInfo(T).Int.bits`.

Эта функция измеряет размер во время выполнения. Для типов которые запрещены во время выполнения, таких как
`comptime_int` и `type` результат равен 0.

Смотрите также:
- [@bitSizeOf](#bitsizeof)
- [@typeInfo](#typeinfo)


###### `splat`

```zig
@splat(scalar: anytype) anytype
```

Создает вектор в котором каждый элемент является скалярным значением. Выводится тип возвращаемого значения и
следовательно, длина вектора.

```zig
const std = @import("std");
const expect = std.testing.expect;

test "vector @splat" {
    const scalar: u32 = 5;
    const result: @Vector(4, u32) = @splat(scalar);
    try expect(std.mem.eql(u32, &@as([4]u32, result), &[_]u32{ 5, 5, 5, 5 }));
}
```
```bash
$ zig test test_splat_builtin.zig
1/1 test_splat_builtin.test.vector @splat...OK
All 1 tests passed.
```

Скаляр должен быть [целым числом](#integers), [логическим](#primitive-types), с [плавающей запятой](#floats) или
[указателем](#pointers).

Смотрите также:
- [Vectors](#vectors)
- [@shuffle](#shuffle)


###### `reduce`

```zig
@reduce(comptime op: std.builtin.ReduceOp, value: anytype) E
```

Преобразует [вектор](#vectors) в скалярное значение (типа `E`), выполняя последовательное уменьшение его элементов по
горизонтали с помощью указанного оператора `op`.

Не каждый оператор доступен для каждого типа векторных элементов:

- Для [целочисленных](#integers) векторов доступны все операторы.
- `.And`, `.Or`, `.Xor` дополнительно доступны для логических векторов,
- `.Min`, `.Max`, `.Add`, `.Mul` дополнительно доступны для векторов с [плавающей запятой](#floats),

Обратите внимание, что сокращения `.Add` и `.Mul` для целых типов являются завершающими; при применении к типам с
плавающей запятой ассоциативность операций сохраняется, если только для режима с плавающей запятой не установлено
значение `Optimized`.

```zig
const std = @import("std");
const expect = std.testing.expect;

test "vector @reduce" {
    const V = @Vector(4, i32);
    const value = V{ 1, -1, 1, -1 };
    const result = value > @as(V, @splat(0));
    // результат таков { true, false, true, false };
    try comptime expect(@TypeOf(result) == @Vector(4, bool));
    const is_all_true = @reduce(.And, result);
    try comptime expect(@TypeOf(is_all_true) == bool);
    try expect(is_all_true == false);
}
```
```bash
$ zig test test_reduce_builtin.zig
1/1 test_reduce_builtin.test.vector @reduce...OK
All 1 tests passed.
```

Смотрите также:
- [Vectors](#vectors)
- [@setFloatMode](#setfloatmode)


###### `src`

```zig
@src() std.builtin.SourceLocation
```

Возвращает структуру `SourceLocation` представляющую имя функции и ее местоположение в исходном коде. Это должно быть
вызвано в функции.

```zig
const std = @import("std");
const expect = std.testing.expect;

test "@src" {
    try doTheTest();
}

fn doTheTest() !void {
    const src = @src();

    try expect(src.line == 9);
    try expect(src.column == 17);
    try expect(std.mem.endsWith(u8, src.fn_name, "doTheTest"));
    try expect(std.mem.endsWith(u8, src.file, "test_src_builtin.zig"));
}
```
```bash
$ zig test test_src_builtin.zig
1/1 test_src_builtin.test.@src...OK
All 1 tests passed.
```


###### `sqrt`

```zig
@sqrt(value: anytype) @TypeOf(value)
```

Извлекает квадратный корень из числа с плавающей запятой. Использует специальную аппаратную инструкцию, если она
доступна.

Поддерживает значения с [плавающей запятой](#floats) и [векторы](#vectors) значений с плавающей запятой.


###### `sin`

```zig
@sin(value: anytype) @TypeOf(value)
```

Тригонометрическая функция синуса для числа с плавающей запятой в радианах. Использует специальную аппаратную
инструкцию, если она доступна.

Поддерживает значения с [плавающей запятой](#floats) и [векторы](#vectors) значений с плавающей запятой.


###### `tan`

```zig
@tan(value: anytype) @TypeOf(value)
```

Тригонометрическая функция тангенса для числа с плавающей запятой в радианах. Использует специальную аппаратную
инструкцию, если она доступна.

Поддерживает значения с [плавающей запятой](#floats) и [векторы](#vectors) значений с плавающей запятой.


###### `exp`

```zig
@exp(value: anytype) @TypeOf(value)
```

Базовая экспоненциальная функция для числа с плавающей запятой. Использует специальную аппаратную инструкцию, если она
доступна.

Поддерживает значения с [плавающей запятой](#floats) и [векторы](#vectors) значений с плавающей запятой.


###### `exp2`

```zig
@exp2(value: anytype) @TypeOf(value)
```

Экспоненциальная функция с основанием 2 для числа с плавающей запятой. Использует специальную аппаратную инструкцию,
если она доступна.

Поддерживает значения с [плавающей запятой](#floats) и [векторы](#vectors) значений с плавающей запятой.


###### `log`

```zig
@log(value: anytype) @TypeOf(value)
```

Возвращает натуральный логарифм числа с плавающей запятой. Использует специальную аппаратную инструкцию, если она
доступна.

Поддерживает значения с [плавающей запятой](#floats) и [векторы](#vectors) значений с плавающей запятой.


###### `log2`

```zig
@log2(value: anytype) @TypeOf(value)
```

Возвращает логарифм числа с плавающей запятой по основанию 2. При его наличии используется специальная аппаратная
инструкция.

Поддерживает значения с [плавающей запятой](#floats) и [векторы](#vectors) значений с плавающей запятой.


###### `log10`

```zig
@log10(value: anytype) @TypeOf(value)
```

Возвращает логарифм числа с плавающей запятой по основанию 10. При его наличии используется специальная аппаратная
инструкция.

Поддерживает значения с [плавающей запятой](#floats) и [векторы](#vectors) значений с плавающей запятой.


###### `abs`

```zig
@abs(value: anytype) anytype
```

Возвращает абсолютное значение целого числа или числа с плавающей запятой. Используется специальная аппаратная
инструкция, если она доступна. Возвращаемый тип всегда представляет собой целое число без знака той же разрядности, что
и операнд, если операнд является целым числом. Поддерживаются целые операнды без знака. Встроенный модуль не может
переполняться для целых операндов со знаком.

Поддерживает значения с [плавающей запятой](#floats), [целые числа] и [векторы](#vectors) с плавающей запятой или целых
чисел значений.


###### `floor`

```zig
@floor(value: anytype) @TypeOf(value)
```

Возвращает наибольшее целое значение, не превышающее заданное число с плавающей запятой. Использует специальную
аппаратную инструкцию, если она доступна.

Поддерживает значения с [плавающей запятой](#floats) и [векторы](#vectors) значений с плавающей запятой.


###### `ceil`

```zig
@ceil(value: anytype) @TypeOf(value)
```

Возвращает наименьшее целое значение, не меньшее заданного числа с плавающей запятой. Использует специальную аппаратную
инструкцию, если она доступна.

Поддерживает значения с [плавающей запятой](#floats) и [векторы](#vectors) значений с плавающей запятой.


###### `trunc`

```zig
@trunc(value: anytype) @TypeOf(value)
```

Округляет заданное число с плавающей запятой до целого числа приближая его к нулю. Использует специальную аппаратную
инструкцию, если она доступна.

Поддерживает значения с [плавающей запятой](#floats) и [векторы](#vectors) значений с плавающей запятой.


###### `round`

```zig
@round(value: anytype) @TypeOf(value)
```

Округляет заданное число с плавающей запятой до целого числа отличного от нуля. Использует специальную аппаратную
инструкцию, если она доступна.

Поддерживает значения с [плавающей запятой](#floats) и [векторы](#vectors) значений с плавающей запятой.


###### `subWithOverflow`

```zig
@subWithOverflow(a: anytype, b: anytype) struct { @TypeOf(a, b), u1 }
```

Выполняет операции `a` - `b` и возвращает кортеж с результатом и возможным битом переполнения.


###### `tagName`

```zig
@tagName(value: anytype) [:0]const u8
```

Преобразует значение перечисления или значение объединения в строковый литерал представляющий имя.

Если перечисление не является исчерпывающим, а значение тега не сопоставляется с именем то это вызывает [неопределенное
поведение](#undefined-behavior) с проверкой безопасности.


###### `This`

```zig
@This() type
```

Возвращает самую внутреннюю структуру, перечисление или объединения, внутри которой находится вызов функции. Это может
быть полезно для анонимной структуры которой необходимо ссылаться на саму себя:

```zig
const std = @import("std");
const expect = std.testing.expect;

test "@This()" {
    var items = [_]i32{ 1, 2, 3, 4 };
    const list = List(i32){ .items = items[0..] };
    try expect(list.length() == 4);
}

fn List(comptime T: type) type {
    return struct {
        const Self = @This();

        items: []T,

        fn length(self: Self) usize {
            return self.items.len;
        }
    };
}
```
```bash
$ zig test test_this_builtin.zig
1/1 test_this_builtin.test.@This()...OK
All 1 tests passed.
```

Когда `@This()` используется в области действия файла он возвращает ссылку на структуру, соответствующую текущему
файлу.


###### `trap`

```zig
@trap() noreturn
```

Эта функция вставляет специфичную для платформы инструкцию `trap/jam`, которая может быть использована для аварийного
завершения работы программы. Это может быть реализовано путем явного ввода недопустимой инструкции, которая может
вызвать какое-либо недопустимое исключение из инструкции. В отличие от `@breakpoint()`, выполнение не продолжается после
этой точки.

За пределами области действия функции эта встроенная функция вызывает ошибку компиляции.

Смотрите также:
- [@breakpoint](#breakpoint)


###### `truncate`

```zig
@truncate(integer: anytype) anytype
```

Эта функция усекает разряды из целочисленного типа, в результате чего получается целочисленный тип меньшего или того же
размера. Возвращаемый тип - это тип предполагаемого результата.

Эта функция всегда усекает значащие разряды целого числа, независимо от того, является ли оно конечным на целевой
платформе.

Вызов `@truncate` для числа находящегося вне диапазона типа назначения является четко определенным и работающим
кодом:

```zig
const std = @import("std");
const expect = std.testing.expect;

test "integer truncation" {
    const a: u16 = 0xabcd;
    const b: u8 = @truncate(a);
    try expect(b == 0xcd);
}
```
```bash
$ zig test test_truncate_builtin.zig
1/1 test_truncate_builtin.test.integer truncation...OK
All 1 tests passed.
```

Используйте [@intCast](#intcast) для преобразования чисел, гарантированно соответствующих типу получателя.


###### `Type`

```zig
@Type(comptime info: std.builtin.Type) type
```

Эта функция является обратной функции [@typeInfo](#typeinfo). Она преобразует информацию о типе в тип.

Он доступен для следующих типов:

- `type`
- `noreturn`
- `void`
- `bool`
- [Integers](#integers) - Максимальное количество битов для целочисленного типа равно `65535`.
- [Floats](#floats)
- [Pointers](#pointers)
- `comptime_int`
- `comptime_float`
- `@TypeOf(undefined)`
- `@TypeOf(null)`
- [Arrays](#arrays)
- [Optionals](#optionals)
- [Error Set Type](#error-set-type)
- [Error Union Type](#error-union-type)
- [Vectors](#vectors)
- [opaque](#opaque)
- `anyframe`
- [struct](#struct)
- [enum](#enum)
- [Enum Literals](#enum-literals)
- [union](#union)
- [Functions](#functions)


###### `typeInfo`

```zig
@typeInfo(comptime T: type) std.builtin.Type
```

Обеспечивает "рефлексию" типа.

Информация о типах [структур](#struct), [объединения](#union), [перечислений](#enum) и [наборов ошибок](#error-set-type)
содержит поля которые гарантированно будут располагаться в том же порядке, что и в исходном файле.

Информация о типах [структур](#struct), [объединения](#union), [перечислений](#enum) и [непрозрачных типов](#opaque)
содержит объявления которые также гарантированно будут находиться в том же порядке, что и в исходном файле.


###### `typeName`

```zig
@typeName(T: type) *const [N:0]u8
```

Эта функция возвращает строковое представление типа в виде массива. Это эквивалентно строковому литералу имени типа.
Возвращаемое имя типа является полным, а родительское пространство имен добавляется как часть имени типа с помощью ряда
точек.


###### `TypeOf`

```zig
@TypeOf(...) type
```

`@TypeOf` - это специальная встроенная функция, которая принимает любое (ненулевое) количество выражений в качестве
параметров и возвращает тип результата, используя [одноранговое разрешение типов](#peer-type-resolution).

Выражения вычисляются, однако гарантируется, что они не будут иметь побочных эффектов во _время выполнения_:

```zig
const std = @import("std");
const expect = std.testing.expect;

test "no runtime side effects" {
    var data: i32 = 0;
    const T = @TypeOf(foo(i32, &data));
    try comptime expect(T == i32);
    try expect(data == 0);
}

fn foo(comptime T: type, ptr: *T) T {
    ptr.* += 1;
    return ptr.*;
}
```
```bash
$ zig test test_TypeOf_builtin.zig
1/1 test_TypeOf_builtin.test.no runtime side effects...OK
All 1 tests passed.
```


###### `unionInit`

```zig
@unionInit(comptime Union: type, comptime active_field_name: []const u8, init_expr) Union
```

Это то же самое, что и синтаксис инициализации объединения, за исключением того, что имя поля является известным
значением [времени компиляции](#comptime), а не токеном идентификатора.

`@unionInit` передает [свое местоположение](#result-location-semantics) результата в `init_expr`.


###### `Vector`

```zig
@Vector(len: comptime_int, Element: type) type
```

Создает векторы.


###### `volatileCast`

```zig
@volatileCast(value: anytype) DestType
```

Удаляет спецификатор `volatile` из указателя.


###### `workGroupId`

```zig
@workGroupId(comptime dimension: u32) u32
```

Возвращает индекс рабочей группы при текущем вызове ядра в измерении `dimension`.


###### `workGroupSize`

```zig
@workGroupSize(comptime dimension: u32) u32
```

Возвращает количество рабочих элементов, имеющихся у рабочей группы в измерении `dimension`.


###### `workItemId`

```zig
@workItemId(comptime dimension: u32) u32
```

Возвращает индекс рабочего элемента в рабочей группе в измерении `dimension`. Эта функция возвращает значения от `0`
(включительно) до `@workGroupSize(dimension)` (исключительно).


------------
### Build Mode

Zig имеет четыре режима сборки:

- [Debug (default)](#debug)
- [ReleaseFast](#releasefast)
- [ReleaseSafe](#releasesafe)
- [ReleaseSmall](#releasesmall)

Чтобы добавить стандартные параметры сборки в файл `build.zig`, выполните следующие действия:

```zig
const std = @import("std");

pub fn build(b: *std.Build) void {
    const optimize = b.standardOptimizeOption(.{});
    const exe = b.addExecutable(.{
        .name = "example",
        .root_source_file = b.path("example.zig"),
        .optimize = optimize,
    });
    b.default_step.dependOn(&exe.step);
}
```

 Это делает доступными следующие параметры:

- `-Doptimize=Debug`
    - Оптимизация отключена, а безопасность включена (по умолчанию)
- `-Doptimize=ReleaseSafe`
    - Оптимизация и безопасность
- `-Doptimize=ReleaseFast`
    - Включение оптимизации и отключение безопасности
- `-Doptimize=ReleaseSmall`
    - Включена оптимизация размеров и отключена защита


#### Debug

```bash
$ zig build-exe example.zig
```

- Высокая скорость компиляции
- Включены проверки безопасности
- Низкая производительность во время выполнения
- Большой размер двоичного файла
- Нет требований к воспроизводимой сборке

#### ReleaseFast

```bash
$ zig build-exe example.zig -O ReleaseFast
```

- Высокая производительность во время выполнения
- Отключены проверки безопасности
- Низкая скорость компиляции
- Большой размер двоичного файла
- Воспроизводимая сборка

#### ReleaseSafe

```bash
$ zig build-exe example.zig -O ReleaseSafe
```

- Средняя производительность во время выполнения
- Включены проверки безопасности
- Низкая скорость компиляции
- Большой размер двоичного файла
- Воспроизводимая сборка

#### ReleaseSmall

```bash
$ zig build-exe example.zig -O ReleaseSmall
```

- Средняя производительность во время выполнения
- Отключены проверки безопасности
- Низкая скорость компиляции
- Небольшой размер двоичного файла
- Воспроизводимая сборка

Смотрите также:
- [Compile Variables](#compile-variables)
- [Zig Build System](#zig-build-system)
- [Undefined Behavior](#undefined-behavior)


------------
### Single Threaded Builds

В Zig есть опция компиляции `-fsingle-threaded`, которая имеет следующие эффекты:

- Все [локальные переменные потока](#thread-local-variables) обрабатываются как обычные [переменные уровня
контейнера](#container-level-variables).
- Накладные расходы на [асинхронные функции](#async-functions) становятся эквивалентными накладным расходам на вызов
функций.
- `@import("builtin").single_threaded` становится `true` и следовательно различные пользовательские API которые
считывают эту переменную становятся более эффективными. Например, `std.Mutex` становится пустой структурой данных и все
ее функции становятся нулевыми.


------------
### Undefined Behavior

В Zig есть много случаев неопределенного поведения. Если во время компиляции обнаруживается неопределенное поведение,
Zig выдает ошибку компиляции и отказывается продолжать работу. Большинство неопределенных действий которые не могут быть
обнаружены во время компиляции могут быть обнаружены во время выполнения. В таких случаях в Zig предусмотрены проверки
безопасности. Проверки безопасности можно отключить для каждого блока с помощью [@setRuntimeSafety](#setruntimesafety).
Режимы сборки [ReleaseFast](#releasefast) и [ReleaseSmall](#releasesmall) отключают все проверки безопасности (за
исключением тех, которые переопределяются [@setRuntimeSafety](#setruntimesafety), чтобы облегчить оптимизацию.

При сбое проверки безопасности Zig завершает работу с трассировкой стека, например, следующим образом:

```zig
test "safety check" {
    unreachable;
}
```
```bash
$ zig test test_undefined_behavior.zig
1/1 test_undefined_behavior.test.safety check...thread 3571226 panic: reached unreachable code
/home/andy/src/zig/doc/langref/test_undefined_behavior.zig:2:5: 0x103ce30 in test.safety check (test)
    unreachable;
    ^
/home/andy/src/zig/lib/compiler/test_runner.zig:157:25: 0x1048260 in mainTerminal (test)
        if (test_fn.func()) |_| {
                        ^
/home/andy/src/zig/lib/compiler/test_runner.zig:37:28: 0x103e21b in main (test)
        return mainTerminal();
                           ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x103d359 in posixCallMainAndExit (test)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x103cec1 in _start (test)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
error: the following test command crashed:
/home/andy/src/zig/.zig-cache/o/a71f9f37afd5b3c603d2b9ea72373fa7/test
```


#### Reaching Unreachable Code

Во время компиляции:

```zig
comptime {
    assert(false);
}
fn assert(ok: bool) void {
    if (!ok) unreachable; // ошибка утверждения
}
```
```bash
$ zig test test_comptime_reaching_unreachable.zig
doc/langref/test_comptime_reaching_unreachable.zig:5:14: error: reached unreachable code
    if (!ok) unreachable; // assertion failure
             ^~~~~~~~~~~
doc/langref/test_comptime_reaching_unreachable.zig:2:11: note: called from here
    assert(false);
    ~~~~~~^~~~~~~
```

Во время выполнения:

```zig
const std = @import("std");

pub fn main() void {
    std.debug.assert(false);
}
```
```zig
$ zig build-exe runtime_reaching_unreachable.zig
$ ./runtime_reaching_unreachable
thread 3575642 panic: reached unreachable code
/home/andy/src/zig/lib/std/debug.zig:412:14: 0x1036cdd in assert (runtime_reaching_unreachable)
    if (!ok) unreachable; // ошибка утверждения
             ^
/home/andy/src/zig/doc/langref/runtime_reaching_unreachable.zig:4:21: 0x103507a in main (runtime_reaching_unreachable)
    std.debug.assert(false);
                    ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034929 in posixCallMainAndExit (runtime_reaching_unreachable)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x1034491 in _start (runtime_reaching_unreachable)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```


#### Index out of Bounds

Во время компиляции:

```zig
comptime {
    const array: [5]u8 = "hello".*;
    const garbage = array[5];
    _ = garbage;
}
```
```bash
$ zig test test_comptime_index_out_of_bounds.zig
doc/langref/test_comptime_index_out_of_bounds.zig:3:27: error: index 5 outside array of length 5
    const garbage = array[5];
                          ^
```

Во время выполнения:

```zig
pub fn main() void {
    const x = foo("hello");
    _ = x;
}

fn foo(x: []const u8) u8 {
    return x[5];
}
```
```bash
$ zig build-exe runtime_index_out_of_bounds.zig
$ ./runtime_index_out_of_bounds
thread 3567952 panic: index out of bounds: index 5, len 5
/home/andy/src/zig/doc/langref/runtime_index_out_of_bounds.zig:7:13: 0x1037169 in foo (runtime_index_out_of_bounds)
    return x[5];
            ^
/home/andy/src/zig/doc/langref/runtime_index_out_of_bounds.zig:2:18: 0x10350b6 in main (runtime_index_out_of_bounds)
    const x = foo("hello");
                 ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034959 in posixCallMainAndExit (runtime_index_out_of_bounds)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x10344c1 in _start (runtime_index_out_of_bounds)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```


#### Cast Negative Number to Unsigned Integer

Во время компиляции:

```zig
comptime {
    const value: i32 = -1;
    const unsigned: u32 = @intCast(value);
    _ = unsigned;
}
```
```zig
$ zig test test_comptime_invalid_cast.zig
doc/langref/test_comptime_invalid_cast.zig:3:36: error: type 'u32' cannot represent integer value '-1'
    const unsigned: u32 = @intCast(value);
                                   ^~~~~
```

Во время выполнения:

```zig
const std = @import("std");

pub fn main() void {
    var value: i32 = -1; // известный во время выполнения
    _ = &value;
    const unsigned: u32 = @intCast(value);
    std.debug.print("value: {}\n", .{unsigned});
}
```
```bash

$ zig build-exe runtime_invalid_cast.zig
$ ./runtime_invalid_cast
thread 3567914 panic: attempt to cast negative value to unsigned integer
/home/andy/src/zig/doc/langref/runtime_invalid_cast.zig:6:27: 0x10351e2 in main (runtime_invalid_cast)
    const unsigned: u32 = @intCast(value);
                          ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034a49 in posixCallMainAndExit (runtime_invalid_cast)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x10345b1 in _start (runtime_invalid_cast)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```

Чтобы получить максимальное значение целого числа без знака, используйте `std.math.maxInt`.


#### Cast Truncates Data

Во время компиляции:

```zig
comptime {
    const spartan_count: u16 = 300;
    const byte: u8 = @intCast(spartan_count);
    _ = byte;
}
```
```bash
$ zig test test_comptime_invalid_cast_truncate.zig
doc/langref/test_comptime_invalid_cast_truncate.zig:3:31: error: type 'u8' cannot represent integer value '300'
    const byte: u8 = @intCast(spartan_count);
                              ^~~~~~~~~~~~~

```

Во время выполнения:

```zig
const std = @import("std");

pub fn main() void {
    var spartan_count: u16 = 300; // известный во время выполнения
    _ = &spartan_count;
    const byte: u8 = @intCast(spartan_count);
    std.debug.print("value: {}\n", .{byte});
}
```
```bash
$ zig build-exe runtime_invalid_cast_truncate.zig
$ ./runtime_invalid_cast_truncate
thread 3576020 panic: integer cast truncated bits
/home/andy/src/zig/doc/langref/runtime_invalid_cast_truncate.zig:6:22: 0x1035275 in main (runtime_invalid_cast_truncate)
    const byte: u8 = @intCast(spartan_count);
                     ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034ad9 in posixCallMainAndExit (runtime_invalid_cast_truncate)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x1034641 in _start (runtime_invalid_cast_truncate)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```

Чтобы усечь биты, используйте [@truncate](#truncate).


#### Integer Overflow

##### Default Operations

Следующие операторы могут вызвать переполнение целых чисел:

- `+` (сложение)
- `-` (вычитание)
- `-` (отрицание)
- `*` (умножение)
- `/` (деление)
- [@divTrunc](#divtrunc) (деление)
- [@divFloor](#divfloor) (деление)
- [@divExact](#divexact) (деление)

Пример со сложением во время компиляции:

```zig
comptime {
    var byte: u8 = 255;
    byte += 1;
}
```
```bash
$ zig test test_comptime_overflow.zig
doc/langref/test_comptime_overflow.zig:3:10: error: overflow of integer type 'u8' with value '256'
    byte += 1;
    ~~~~~^~~~
```

Во время выполнения:

```zig
const std = @import("std");

pub fn main() void {
    var byte: u8 = 255;
    byte += 1;
    std.debug.print("value: {}\n", .{byte});
}
```
```bash
$ zig build-exe runtime_overflow.zig
$ ./runtime_overflow
thread 3574209 panic: integer overflow
/home/andy/src/zig/doc/langref/runtime_overflow.zig:5:10: 0x103525e in main (runtime_overflow)
    byte += 1;
         ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034ad9 in posixCallMainAndExit (runtime_overflow)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x1034641 in _start (runtime_overflow)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```


##### Standard Library Math Functions

Эти функции предоставляемые стандартной библиотекой возвращают возможные ошибки.

- `@import("std").math.add`
- `@import("std").math.sub`
- `@import("std").math.mul`
- `@import("std").math.divTrunc`
- `@import("std").math.divFloor`
- `@import("std").math.divExact`
- `@import("std").math.shl`

Пример перехвата переполнения для сложения:

```zig
const math = @import("std").math;
const print = @import("std").debug.print;
pub fn main() !void {
    var byte: u8 = 255;

    byte = if (math.add(u8, byte, 1)) |result| result else |err| {
        print("unable to add one: {s}\n", .{@errorName(err)});
        return err;
    };

    print("result: {}\n", .{byte});
}
```
```bash
$ zig build-exe math_add.zig
$ ./math_add
unable to add one: Overflow
error: Overflow
/home/andy/src/zig/lib/std/math.zig:565:21: 0x10352a5 in add__anon_2592 (math_add)
    if (ov[1] != 0) return error.Overflow;
                    ^
/home/andy/src/zig/doc/langref/math_add.zig:8:9: 0x1035243 in main (math_add)
        return err;
        ^
```


##### Builtin Overflow Functions

Эти встроенные модули возвращают кортеж, содержащий информацию о том, было ли переполнение (в виде `u1`) и возможные
биты переполнения в операции:

- [@addWithOverflow](#addwithoverflow)
- [@subWithOverflow](#subwithoverflow)
- [@mulWithOverflow](#mulwithoverflow)
- [@shlWithOverflow](#shlwithoverflow)

Пример использования `@addWithOverflow`:

```zig
const print = @import("std").debug.print;
pub fn main() void {
    const byte: u8 = 255;

    const ov = @addWithOverflow(byte, 10);
    if (ov[1] != 0) {
        print("overflowed result: {}\n", .{ov[0]});
    } else {
        print("result: {}\n", .{ov[0]});
    }
}
```
```bash
$ zig build-exe addWithOverflow_builtin.zig
$ ./addWithOverflow_builtin
overflowed result: 9

```


##### Wrapping Operations

Эти операции имеют гарантированную "безопасную" семантику.

- `+%` (безопасное сложение)
- `-%` (безопасное вычитание)
- `-%` (безопасное отрицание)
- `*%` (безопасное умножение)

```zig
const std = @import("std");
const expect = std.testing.expect;
const minInt = std.math.minInt;
const maxInt = std.math.maxInt;

test "wraparound addition and subtraction" {
    const x: i32 = maxInt(i32);
    const min_val = x +% 1;
    try expect(min_val == minInt(i32));
    const max_val = min_val -% 1;
    try expect(max_val == maxInt(i32));
}
```
```bash
$ zig test test_wraparound_semantics.zig
1/1 test_wraparound_semantics.test.wraparound addition and subtraction...OK
All 1 tests passed.
```


#### Exact Left Shift Overflow

Во время компиляции:

```zig
comptime {
    const x = @shlExact(@as(u8, 0b01010101), 2);
    _ = x;
}
```
```bash
$ zig test test_comptime_shlExact_overwlow.zig
doc/langref/test_comptime_shlExact_overwlow.zig:2:15: error: operation caused overflow
    const x = @shlExact(@as(u8, 0b01010101), 2);
              ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

```

Во время выполнения:

```zig
const std = @import("std");

pub fn main() void {
    var x: u8 = 0b01010101; // известный во время выполнения
    _ = &x;
    const y = @shlExact(x, 2);
    std.debug.print("value: {}\n", .{y});
}
```
```bash
$ zig build-exe runtime_shlExact_overflow.zig
$ ./runtime_shlExact_overflow
thread 3577156 panic: left shift overflowed bits
/home/andy/src/zig/doc/langref/runtime_shlExact_overflow.zig:6:5: 0x10352bd in main (runtime_shlExact_overflow)
    const y = @shlExact(x, 2);
    ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034b19 in posixCallMainAndExit (runtime_shlExact_overflow)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x1034681 in _start (runtime_shlExact_overflow)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```


#### Exact Right Shift Overflow

Во время компиляции:

```zig
comptime {
    const x = @shrExact(@as(u8, 0b10101010), 2);
    _ = x;
}
```
```bash
$ zig test test_comptime_shrExact_overflow.zig
doc/langref/test_comptime_shrExact_overflow.zig:2:15: error: exact shift shifted out 1 bits
    const x = @shrExact(@as(u8, 0b10101010), 2);
              ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

Во время выполнения:

```zig
const std = @import("std");

pub fn main() void {
    var x: u8 = 0b10101010; // runtime-known
    _ = &x;
    const y = @shrExact(x, 2);
    std.debug.print("value: {}\n", .{y});
}
```
```bash
$ zig build-exe runtime_shrExact_overflow.zig
$ ./runtime_shrExact_overflow
thread 3579049 panic: right shift overflowed bits
/home/andy/src/zig/doc/langref/runtime_shrExact_overflow.zig:6:5: 0x10352b9 in main (runtime_shrExact_overflow)
    const y = @shrExact(x, 2);
    ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034b19 in posixCallMainAndExit (runtime_shrExact_overflow)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x1034681 in _start (runtime_shrExact_overflow)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```


#### Division by Zero

Во время компиляции:

```zig
comptime {
    const a: i32 = 1;
    const b: i32 = 0;
    const c = a / b;
    _ = c;
}
```
```bash
$ zig test test_comptime_division_by_zero.zig
doc/langref/test_comptime_division_by_zero.zig:4:19: error: division by zero here causes undefined behavior
    const c = a / b;
                  ^
```

Во время выполнения:

```zig
const std = @import("std");

pub fn main() void {
    var a: u32 = 1;
    var b: u32 = 0;
    _ = .{ &a, &b };
    const c = a / b;
    std.debug.print("value: {}\n", .{c});
}
```
```bash
$ zig build-exe runtime_division_by_zero.zig
$ ./runtime_division_by_zero
thread 3575479 panic: division by zero
/home/andy/src/zig/doc/langref/runtime_division_by_zero.zig:7:17: 0x10351f6 in main (runtime_division_by_zero)
    const c = a / b;
                ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034a49 in posixCallMainAndExit (runtime_division_by_zero)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x10345b1 in _start (runtime_division_by_zero)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```


#### Remainder Division by Zero

Во время компиляции:

```zig
comptime {
    const a: i32 = 10;
    const b: i32 = 0;
    const c = a % b;
    _ = c;
}
```
```bash
$ zig test test_comptime_remainder_division_by_zero.zig
doc/langref/test_comptime_remainder_division_by_zero.zig:4:19: error: division by zero here causes undefined behavior
    const c = a % b;
                  ^
```

Во время выполнения:

```zig
const std = @import("std");

pub fn main() void {
    var a: u32 = 10;
    var b: u32 = 0;
    _ = .{ &a, &b };
    const c = a % b;
    std.debug.print("value: {}\n", .{c});
}
```
```bash
$ zig build-exe runtime_remainder_division_by_zero.zig
$ ./runtime_remainder_division_by_zero
thread 3570535 panic: division by zero
/home/andy/src/zig/doc/langref/runtime_remainder_division_by_zero.zig:7:17: 0x10351f6 in main (runtime_remainder_division_by_zero)
    const c = a % b;
                ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034a49 in posixCallMainAndExit (runtime_remainder_division_by_zero)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x10345b1 in _start (runtime_remainder_division_by_zero)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```


#### Exact Division Remainder

Во время компиляции:

```zig
comptime {
    const a: u32 = 10;
    const b: u32 = 3;
    const c = @divExact(a, b);
    _ = c;
}
```
```bash
$ zig test test_comptime_divExact_remainder.zig
doc/langref/test_comptime_divExact_remainder.zig:4:15: error: exact division produced remainder
    const c = @divExact(a, b);
              ^~~~~~~~~~~~~~~
```

Во время выполнения:

```zig
const std = @import("std");

pub fn main() void {
    var a: u32 = 10;
    var b: u32 = 3;
    _ = .{ &a, &b };
    const c = @divExact(a, b);
    std.debug.print("value: {}\n", .{c});
}
```
```bash
$ zig build-exe runtime_divExact_remainder.zig
$ ./runtime_divExact_remainder
thread 3570103 panic: exact division produced remainder
/home/andy/src/zig/doc/langref/runtime_divExact_remainder.zig:7:15: 0x103526b in main (runtime_divExact_remainder)
    const c = @divExact(a, b);
              ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034a89 in posixCallMainAndExit (runtime_divExact_remainder)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x10345f1 in _start (runtime_divExact_remainder)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```

#### Attempt to Unwrap Null

Во время компиляции:

```zig
comptime {
    const optional_number: ?i32 = null;
    const number = optional_number.?;
    _ = number;
}
```
```bash
$ zig test test_comptime_unwrap_null.zig
doc/langref/test_comptime_unwrap_null.zig:3:35: error: unable to unwrap null
    const number = optional_number.?;
                   ~~~~~~~~~~~~~~~^~
```

Во время выполнения:

```zig
const std = @import("std");

pub fn main() void {
    var optional_number: ?i32 = null;
    _ = &optional_number;
    const number = optional_number.?;
    std.debug.print("value: {}\n", .{number});
}
```
```bash
$ zig build-exe runtime_unwrap_null.zig
$ ./runtime_unwrap_null
thread 3570514 panic: attempt to use null value
/home/andy/src/zig/doc/langref/runtime_unwrap_null.zig:6:35: 0x1035272 in main (runtime_unwrap_null)
const number = optional_number.?;
^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034ad9 in posixCallMainAndExit (runtime_unwrap_null)
root.main();
^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x1034641 in _start (runtime_unwrap_null)
asm volatile (switch (native_arch) {
^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```

Один из способов избежать этого сбоя - проверить выражение `if` на `null` вместо предположения, что оно не является
нулевым:

```zig
const print = @import("std").debug.print;
pub fn main() void {
    const optional_number: ?i32 = null;

    if (optional_number) |number| {
        print("got number: {}\n", .{number});
    } else {
        print("it's null\n", .{});
    }
}
```
```bash
$ zig build-exe testing_null_with_if.zig
$ ./testing_null_with_if
it's null
```

Смотрите также:
- [Optionals](#optionals)


#### Attempt to Unwrap Error

Во время компиляции:

```zig
comptime {
    const number = getNumberOrFail() catch unreachable;
    _ = number;
}

fn getNumberOrFail() !i32 {
    return error.UnableToReturnNumber;
}
```
```bash
$ zig test test_comptime_unwrap_error.zig
doc/langref/test_comptime_unwrap_error.zig:2:44: error: caught unexpected error 'UnableToReturnNumber'
    const number = getNumberOrFail() catch unreachable;
                                           ^~~~~~~~~~~
doc/langref/test_comptime_unwrap_error.zig:7:18: note: error returned here
    return error.UnableToReturnNumber;
                 ^~~~~~~~~~~~~~~~~~~~
```

Во время выполнения:

```zig
const std = @import("std");

pub fn main() void {
    const number = getNumberOrFail() catch unreachable;
    std.debug.print("value: {}\n", .{number});
}

fn getNumberOrFail() !i32 {
    return error.UnableToReturnNumber;
}
```
```bash
$ zig build-exe runtime_unwrap_error.zig
$ ./runtime_unwrap_error
thread 3577877 panic: attempt to unwrap error: UnableToReturnNumber
/home/andy/src/zig/doc/langref/runtime_unwrap_error.zig:9:5: 0x103738f in getNumberOrFail (runtime_unwrap_error)
    return error.UnableToReturnNumber;
    ^
/home/andy/src/zig/doc/langref/runtime_unwrap_error.zig:4:44: 0x1035301 in main (runtime_unwrap_error)
    const number = getNumberOrFail() catch unreachable;
                                           ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034b49 in posixCallMainAndExit (runtime_unwrap_error)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x10346b1 in _start (runtime_unwrap_error)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```

Один из способов избежать этого сбоя - проверить наличие ошибки вместо того чтобы предполагать успешный результат с
помощью выражения `if`:

```zig
const print = @import("std").debug.print;

pub fn main() void {
    const result = getNumberOrFail();

    if (result) |number| {
        print("got number: {}\n", .{number});
    } else |err| {
        print("got error: {s}\n", .{@errorName(err)});
    }
}

fn getNumberOrFail() !i32 {
    return error.UnableToReturnNumber;
}
```
```bash
$ zig build-exe testing_error_with_if.zig
$ ./testing_error_with_if
got error: UnableToReturnNumber
```

Смотрите также:
- [Errors](#errors)


#### Invalid Error Code

Во время компиляции:

```zig
comptime {
    const err = error.AnError;
    const number = @intFromError(err) + 10;
    const invalid_err = @errorFromInt(number);
    _ = invalid_err;
}
```
```bash
$ zig test test_comptime_invalid_error_code.zig
doc/langref/test_comptime_invalid_error_code.zig:4:39: error: integer value '11' represents no error
    const invalid_err = @errorFromInt(number);
                                      ^~~~~~
```

Во время выполнения:

```zig
const std = @import("std");

pub fn main() void {
    const err = error.AnError;
    var number = @intFromError(err) + 500;
    _ = &number;
    const invalid_err = @errorFromInt(number);
    std.debug.print("value: {}\n", .{invalid_err});
}
```
```bash
$ zig build-exe runtime_invalid_error_code.zig
$ ./runtime_invalid_error_code
thread 3570441 panic: invalid error code
/home/andy/src/zig/doc/langref/runtime_invalid_error_code.zig:7:5: 0x10352a0 in main (runtime_invalid_error_code)
    const invalid_err = @errorFromInt(number);
    ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034ae9 in posixCallMainAndExit (runtime_invalid_error_code)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x1034651 in _start (runtime_invalid_error_code)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```


#### Invalid Enum Cast

Во время компиляции:

```zig
const Foo = enum {
    a,
    b,
    c,
};
comptime {
    const a: u2 = 3;
    const b: Foo = @enumFromInt(a);
    _ = b;
}
```
```bash
$ zig test test_comptime_invalid_enum_cast.zig
doc/langref/test_comptime_invalid_enum_cast.zig:8:20: error: enum 'test_comptime_invalid_enum_cast.Foo' has no tag with value '3'
    const b: Foo = @enumFromInt(a);
                   ^~~~~~~~~~~~~~~
doc/langref/test_comptime_invalid_enum_cast.zig:1:13: note: enum declared here
const Foo = enum {
            ^~~~
```

Во время выполнения:

```zig
const std = @import("std");

const Foo = enum {
    a,
    b,
    c,
};

pub fn main() void {
    var a: u2 = 3;
    _ = &a;
    const b: Foo = @enumFromInt(a);
    std.debug.print("value: {s}\n", .{@tagName(b)});
}
```
```bash
$ zig build-exe runtime_invalid_enum_cast.zig
$ ./runtime_invalid_enum_cast
thread 3568027 panic: invalid enum value
/home/andy/src/zig/doc/langref/runtime_invalid_enum_cast.zig:12:20: 0x1035297 in main (runtime_invalid_enum_cast)
    const b: Foo = @enumFromInt(a);
                   ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034af9 in posixCallMainAndExit (runtime_invalid_enum_cast)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x1034661 in _start (runtime_invalid_enum_cast)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```


#### Invalid Error Set Cast

Во время компиляции:

```zig
const Set1 = error{
    A,
    B,
};
const Set2 = error{
    A,
    C,
};
comptime {
    _ = @as(Set2, @errorCast(Set1.B));
}
```
```bash
$ zig test test_comptime_invalid_error_set_cast.zig
doc/langref/test_comptime_invalid_error_set_cast.zig:10:19: error: 'error.B' not a member of error set 'error{A,C}'
    _ = @as(Set2, @errorCast(Set1.B));
                  ^~~~~~~~~~~~~~~~~~
```

Во время выполнения:

```zig
const std = @import("std");

const Set1 = error{
    A,
    B,
};
const Set2 = error{
    A,
    C,
};
pub fn main() void {
    foo(Set1.B);
}
fn foo(set1: Set1) void {
    const x: Set2 = @errorCast(set1);
    std.debug.print("value: {}\n", .{x});
}
```
```bash
$ zig build-exe runtime_invalid_error_set_cast.zig
$ ./runtime_invalid_error_set_cast
thread 3568026 panic: invalid error code
/home/andy/src/zig/doc/langref/runtime_invalid_error_set_cast.zig:15:21: 0x1037317 in foo (runtime_invalid_error_set_cast)
    const x: Set2 = @errorCast(set1);
                    ^
/home/andy/src/zig/doc/langref/runtime_invalid_error_set_cast.zig:12:8: 0x103523d in main (runtime_invalid_error_set_cast)
    foo(Set1.B);
       ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034ae9 in posixCallMainAndExit (runtime_invalid_error_set_cast)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x1034651 in _start (runtime_invalid_error_set_cast)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```


#### Incorrect Pointer Alignment

Во время компиляции:

```zig
comptime {
    const ptr: *align(1) i32 = @ptrFromInt(0x1);
    const aligned: *align(4) i32 = @alignCast(ptr);
    _ = aligned;
}
```
```bash
$ zig test test_comptime_incorrect_pointer_alignment.zig
doc/langref/test_comptime_incorrect_pointer_alignment.zig:3:47: error: pointer address 0x1 is not aligned to 4 bytes
    const aligned: *align(4) i32 = @alignCast(ptr);
                                              ^~~
```

Во время выполнения:

```zig
const mem = @import("std").mem;
pub fn main() !void {
    var array align(4) = [_]u32{ 0x11111111, 0x11111111 };
    const bytes = mem.sliceAsBytes(array[0..]);
    if (foo(bytes) != 0x11111111) return error.Wrong;
}
fn foo(bytes: []u8) u32 {
    const slice4 = bytes[1..5];
    const int_slice = mem.bytesAsSlice(u32, @as([]align(4) u8, @alignCast(slice4)));
    return int_slice[0];
}
```
```bash
$ zig build-exe runtime_incorrect_pointer_alignment.zig
$ ./runtime_incorrect_pointer_alignment
thread 3570122 panic: incorrect alignment
/home/andy/src/zig/doc/langref/runtime_incorrect_pointer_alignment.zig:9:64: 0x1034f0a in foo (runtime_incorrect_pointer_alignment)
    const int_slice = mem.bytesAsSlice(u32, @as([]align(4) u8, @alignCast(slice4)));
                                                               ^
/home/andy/src/zig/doc/langref/runtime_incorrect_pointer_alignment.zig:5:12: 0x1034dc7 in main (runtime_incorrect_pointer_alignment)
    if (foo(bytes) != 0x11111111) return error.Wrong;
           ^
/home/andy/src/zig/lib/std/start.zig:524:37: 0x1034cc5 in posixCallMainAndExit (runtime_incorrect_pointer_alignment)
            const result = root.main() catch |err| {
                                    ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x10347e1 in _start (runtime_incorrect_pointer_alignment)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```


#### Wrong Union Field Access

Во время компиляции:

```zig
comptime {
    var f = Foo{ .int = 42 };
    f.float = 12.34;
}

const Foo = union {
    float: f32,
    int: u32,
};
```
```bash
$ zig test test_comptime_wrong_union_field_access.zig
doc/langref/test_comptime_wrong_union_field_access.zig:3:6: error: access of union field 'float' while field 'int' is active
    f.float = 12.34;
    ~^~~~~~
doc/langref/test_comptime_wrong_union_field_access.zig:6:13: note: union declared here
const Foo = union {
            ^~~~~
```

Во время выполнения:

```zig
const std = @import("std");

const Foo = union {
    float: f32,
    int: u32,
};

pub fn main() void {
    var f = Foo{ .int = 42 };
    bar(&f);
}

fn bar(f: *Foo) void {
    f.float = 12.34;
    std.debug.print("value: {}\n", .{f.float});
}
```
```bash
$ zig build-exe runtime_wrong_union_field_access.zig
$ ./runtime_wrong_union_field_access
thread 3578787 panic: access of union field 'float' while field 'int' is active
/home/andy/src/zig/doc/langref/runtime_wrong_union_field_access.zig:14:6: 0x103cd20 in bar (runtime_wrong_union_field_access)
    f.float = 12.34;
     ^
/home/andy/src/zig/doc/langref/runtime_wrong_union_field_access.zig:10:8: 0x103ac5c in main (runtime_wrong_union_field_access)
    bar(&f);
       ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x103a4f9 in posixCallMainAndExit (runtime_wrong_union_field_access)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x103a061 in _start (runtime_wrong_union_field_access)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```

Эта система безопасности недоступна для внешних или упакованных объединений.

Чтобы изменить активное поле объединения, назначьте объединение следующим образом:

```zig
const std = @import("std");

const Foo = union {
    float: f32,
    int: u32,
};

pub fn main() void {
    var f = Foo{ .int = 42 };
    bar(&f);
}

fn bar(f: *Foo) void {
    f.* = Foo{ .float = 12.34 };
    std.debug.print("value: {}\n", .{f.float});
}
```
```bash
$ zig build-exe change_active_union_field.zig
$ ./change_active_union_field
value: 1.234e1
```

Чтобы изменить активное поле объединения, если значимое значение для этого поля неизвестно, используйте
[undefined](#undefined), например, следующим образом:

```zig
const std = @import("std");

const Foo = union {
    float: f32,
    int: u32,
};

pub fn main() void {
    var f = Foo{ .int = 42 };
    f = Foo{ .float = undefined };
    bar(&f);
    std.debug.print("value: {}\n", .{f.float});
}

fn bar(f: *Foo) void {
    f.float = 12.34;
}
```
```bash
$ zig build-exe undefined_active_union_field.zig
$ ./undefined_active_union_field
value: 1.234e1
```


#### Out of Bounds Float to Integer Cast

Это происходит при преобразовании значения с плавающей запятой в целое число, где значение с плавающей запятой находится
за пределами диапазона целочисленного типа.

Во время компиляции:

```zig
comptime {
    const float: f32 = 4294967296;
    const int: i32 = @intFromFloat(float);
    _ = int;
}
```
```bash
$ zig test test_comptime_out_of_bounds_float_to_integer_cast.zig
doc/langref/test_comptime_out_of_bounds_float_to_integer_cast.zig:3:36: error: float value '4294967296' cannot be stored in integer type 'i32'
    const int: i32 = @intFromFloat(float);
                                   ^~~~~
```

Во время выполнения:

```zig
pub fn main() void {
    var float: f32 = 4294967296; // runtime-known
    _ = &float;
    const int: i32 = @intFromFloat(float);
    _ = int;
}
```
```bash
$ zig build-exe runtime_out_of_bounds_float_to_integer_cast.zig
$ ./runtime_out_of_bounds_float_to_integer_cast
thread 3570320 panic: integer part of floating point value out of bounds
/home/andy/src/zig/doc/langref/runtime_out_of_bounds_float_to_integer_cast.zig:4:22: 0x1035169 in main (runtime_out_of_bounds_float_to_integer_cast)
    const int: i32 = @intFromFloat(float);
                     ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x10349a9 in posixCallMainAndExit (runtime_out_of_bounds_float_to_integer_cast)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x1034511 in _start (runtime_out_of_bounds_float_to_integer_cast)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```


#### Pointer Cast Invalid Null

Это происходит при преобразовании указателя с адресом 0 в указатель который может не иметь адреса 0. Например,
[C-указатели](#c-pointers), [опциональные указатели](#optional-pointers) и указатели [allowzero](#allowzero) допускают
нулевой адрес, а обычные указатели - нет.

Во время компиляции:

```zig
comptime {
    const opt_ptr: ?*i32 = null;
    const ptr: *i32 = @ptrCast(opt_ptr);
    _ = ptr;
}
```
```bash
$ zig test test_comptime_invalid_null_pointer_cast.zig
doc/langref/test_comptime_invalid_null_pointer_cast.zig:3:32: error: null pointer casted to type '*i32'
    const ptr: *i32 = @ptrCast(opt_ptr);
                               ^~~~~~~
```

Во время выполнения:

```zig
pub fn main() void {
    var opt_ptr: ?*i32 = null;
    _ = &opt_ptr;
    const ptr: *i32 = @ptrCast(opt_ptr);
    _ = ptr;
}
```
```bash
$ zig build-exe runtime_invalid_null_pointer_cast.zig
$ ./runtime_invalid_null_pointer_cast
thread 3572791 panic: cast causes pointer to be null
/home/andy/src/zig/doc/langref/runtime_invalid_null_pointer_cast.zig:4:23: 0x10350bc in main (runtime_invalid_null_pointer_cast)
    const ptr: *i32 = @ptrCast(opt_ptr);
                      ^
/home/andy/src/zig/lib/std/start.zig:514:22: 0x1034929 in posixCallMainAndExit (runtime_invalid_null_pointer_cast)
            root.main();
                     ^
/home/andy/src/zig/lib/std/start.zig:266:5: 0x1034491 in _start (runtime_invalid_null_pointer_cast)
    asm volatile (switch (native_arch) {
    ^
???:?:?: 0x0 in ??? (???)
(process terminated by signal)
```


------------
### Memory

Язык Zig не выполняет управление памятью от имени программиста. Вот почему у Zig нет среды выполнения и почему код Zig
без проблем работает во многих средах, включая программное обеспечение реального времени, ядра операционных систем,
встроенные устройства и серверы с низкой задержкой. Как следствие, программисты Zig всегда должны быть в состоянии
ответить на этот вопрос:

[Где байты?](#where-are-the-bytes)

Как и Zig, язык программирования C поддерживает ручное управление памятью. Однако, в отличие от Zig, C имеет аллокатор
по умолчанию - `malloc`, `realloc` и `free`. При связывании с `libc` Zig предоставляет этот аллокатор с помощью
`std.heap.c_allocator`. Однако, по соглашению в Zig нет аллокатора по умолчанию. Вместо этого функции которые должны
выделять память принимают параметр `Allocator`. Аналогично структуры данных, такие как `std.ArrayList`, принимают
параметр `Allocator` в своих функциях инициализации:

```zig
const std = @import("std");
const Allocator = std.mem.Allocator;
const expect = std.testing.expect;

test "using an allocator" {
    var buffer: [100]u8 = undefined;
    var fba = std.heap.FixedBufferAllocator.init(&buffer);
    const allocator = fba.allocator();
    const result = try concat(allocator, "foo", "bar");
    try expect(std.mem.eql(u8, "foobar", result));
}

fn concat(allocator: Allocator, a: []const u8, b: []const u8) ![]u8 {
    const result = try allocator.alloc(u8, a.len + b.len);
    @memcpy(result[0..a.len], a);
    @memcpy(result[a.len..], b);
    return result;
}
```
```bash
$ zig test test_allocator.zig
1/1 test_allocator.test.using an allocator...OK
All 1 tests passed.
```

В приведенном выше примере 100 байт стековой памяти используются для инициализации `FixedBufferAllocator`, который затем
передается функции. Для удобства в `std.testing.allocator` есть `FixedBufferAllocator`, доступный для быстрого
тестирования который также выполняет базовое обнаружение утечек памяти.

В Zig есть аллокатор общего назначения который можно импортировать с помощью `std.heap.GeneralPurposeAllocator`. Однако,
все же рекомендуется следовать [руководству по выбору аллокатора](#choosing-an-allocator).


#### Choosing an Allocator

Какой аллокатор использовать зависит от ряда факторов. Вот блок-схема которая поможет вам принять решение:

1. Вы создаете библиотеку? В этом случае лучше всего принять аллокатор в качестве параметра и позволить пользователям
   вашей библиотеки решать, какой аллокатор использовать.
2. Вы линкуете `libc`? В этом случае `std.heap.c_allocator` вероятно, является правильным выбором, по крайней мере для
   вашего основного аллокатора.
3. Ограничено ли максимальное количество байт которое вам понадобится, числом, известным во [время
   компиляции](#comptime)? В этом случае используйте `std.heap.FixedBufferAllocator` или
`std.heap.ThreadSafeFixedBufferAllocator` в зависимости от того нужна ли вам потокобезопасность или нет.
4. Является ли ваша программа приложением командной строки которое запускается от начала до конца без какой-либо
   фундаментальной цикличности (например, основного цикла видеоигры или обработчика запросов веб-сервера), так что было
бы разумно освободить все сразу в конце? В этом случае рекомендуется следовать следующей схеме:

    ```zig
    const std = @import("std");

    pub fn main() !void {
        var arena = std.heap.ArenaAllocator.init(std.heap.page_allocator);
        defer arena.deinit();

        const allocator = arena.allocator();

        const ptr = try allocator.create(i32);
        std.debug.print("ptr={*}\n", .{ptr});
    }
    ```
    ```bash
    $ zig build-exe cli_allocation.zig
    $ ./cli_allocation
    ptr=i32@7f989ee8e010
    ```

    При использовании такого аллокатора нет необходимости освобождать что-либо вручную. Все освобождается сразу при вызове
    функции `arena.deinit()`.

5. Являются ли выделения частью циклической схемы, такой как основной цикл видеоигры или обработчик запросов
   веб-сервера? Если все выделения могут быть освобождены сразу, в конце цикла, например, после того как кадр
видеоигры будет полностью отрисован или запрос веб-сервера будет обработан то `std.heap.ArenaAllocator` - отличный
кандидат. Как было показано в предыдущем разделе это позволяет вам освобождать целые арены за один раз. Также обратите
внимание, что если можно установить верхнюю границу объема памяти то `std.heap.FixedBufferAllocator` может быть
использован в качестве дополнительной оптимизации.
6. Вы пишете тест и хотите убедиться, что `error.OutOfMemory` обрабатывается правильно? В этом случае используйте
   `std.testing.FailingAllocator`.
7. Вы пишете тест? В этом случае используйте `std.testing.allocator`.
8. Наконец, если ничего из вышеперечисленного не применимо вам понадобится аллокатор общего назначения. Аллокатор общего
   назначения Zig доступен в виде функции которая принимает структуру параметров конфигурации `comptime` и возвращает
тип. Как правило, вы настраиваете один из них `std.heap.GeneralPurposeAllocator` в вашей основной функции, а затем
передайте ее или субаллокаторы различным частям вашего приложения.
9. Вы также можете рассмотреть возможность имплементации аллокатора.


#### Where are the bytes

Строковые литералы, такие как `"hello"` находятся в разделе глобальных константных данных. Вот почему передача строкового
литерала в изменяемый срез, например, таким образом, приводит к ошибке:

```zig
fn foo(s: []u8) void {
    _ = s;
}

test "string literal to mutable slice" {
    foo("hello");
}
```
```bash
$ zig test test_string_literal_to_slice.zig
doc/langref/test_string_literal_to_slice.zig:6:9: error: expected type '[]u8', found '*const [5:0]u8'
    foo("hello");
        ^~~~~~~
doc/langref/test_string_literal_to_slice.zig:6:9: note: cast discards const qualifier
doc/langref/test_string_literal_to_slice.zig:1:11: note: parameter type declared here
fn foo(s: []u8) void {
          ^~~~
```

Однако, если вы сделаете срез константным то это сработает:

```zig
fn foo(s: []const u8) void {
    _ = s;
}

test "string literal to constant slice" {
    foo("hello");
}
```
```bash
$ zig test test_string_literal_to_const_slice.zig
1/1 test_string_literal_to_const_slice.test.string literal to constant slice...OK
All 1 tests passed.
```

Так же, как и строковые литералы, объявления `const` если значение известно во [время компиляции](#comptime) хранятся в
разделе глобальных константных данных. [Переменные времени компиляции](#compile-time-variables) также хранятся в разделе
глобальных константных данных.

Объявления `var` внутри функций хранятся в стековом "фрэйме" функции. Как только функция возвращает результат любые
[указатели](#pointers) на переменные в стековом "фрэйме" функции становятся недопустимыми ссылками и их разыменование
становится неконтролируемым и [неопределенным поведением](#undefined-behavior).

Объявления `var` на верхнем уровне или в объявлениях [структуры](#struct) хранятся в разделе глобальных данных.

Расположение памяти выделяемой с помощью `allocator.alloc` или `allocator.create` определяется реализацией аллокатора.

TODO: thread local variables


#### Implementing an Allocator

Программисты Zig могут реализовать свои собственные аллокаторы используя интерфейс аллокатора. Для этого необходимо
внимательно прочитать комментарии к документации в `std/mem.zig`, а затем указать `allocFn` и `resizeFn`.

Существует множество примеров аллокаторов на которые стоит обратить внимание для вдохновения. Посмотрите на
`std/heap.zig` и `std.heap.GeneralPurposeAllocator`.


#### Heap Allocation Failure

Многие языки программирования решают справиться с возможностью сбоя при выделении кучи путем безусловного сбоя. По
общему мнению, программисты Zig не считают это удовлетворительным решением. Вместо, `error.OutOfMemory` представляет
собой ошибку выделения кучи и библиотеки Zig возвращают этот код ошибки всякий раз когда ошибка выделения кучи не
позволяет успешно завершить операцию.

Некоторые утверждают, что из-за того, что в некоторых операционных системах таких как Linux, по умолчанию включена
функции `overcommit` памяти, обрабатывать сбой выделения кучи бессмысленно. С этим рассуждением связано много проблем:
- Только в некоторых операционных системах есть функция `overcommit`.
    - В Linux эта функция включена по умолчанию, но ее можно настроить.
    - Windows не выполняет `overcommit`.
    - Во встроенных системах нет `overcommit`.
    - В операционных системах для хобби может быть `overcommit`, а может и не быть.
- В системах реального времени не только отсутствует `overcommit` нагрузка, но и как правило, максимальный объем памяти
для каждого приложения определяется заранее.
- При написании библиотеки одной из главных целей является повторное использование кода. Если заставить код корректно
обрабатывать ошибки выделения, библиотека становится пригодной для повторного использования в большем количестве
контекстов.
- Несмотря на то, что некоторые программы стали зависеть от включенной функции `overcommit`, ее существование является
источником бесчисленных проблем с работой пользователей. Когда система с включенной функцией `overcommit`, например
Linux с настройками по умолчанию близка к исчерпанию памяти, система блокируется и становится непригодной для
использования. На этом этапе OOM-killer выбирает приложение для уничтожения на основе эвристики. Такое
недетерминированное решение часто приводит к остановке важного процесса и часто не позволяет вернуть систему в рабочее
состояние.


#### Recursion

Рекурсия является фундаментальным инструментом в программном обеспечении для моделирования. Однако у нее есть проблема,
которую часто упускают из виду: неограниченное выделение памяти.

Рекурсия - это область активных экспериментов в Zig, и поэтому приведенная здесь документация не является окончательной.
Вы можете ознакомиться с [краткой информацией о состоянии рекурсии в примечаниях к выпуску
0.3.0](https://ziglang.org/download/0.3.0/release-notes.html#recursion).

Вкратце можно сказать, что в настоящее время рекурсия работает нормально, как и следовало ожидать. Хотя код Zig еще не
защищен от переполнения стека, планируется, что будущая версия Zig обеспечит такую защиту, при этом потребуется
определенная степень сотрудничества со стороны Zig-кода.


#### Lifetime and Ownership

Ответственность программиста Zig заключается в том, чтобы гарантировать, что доступ к [указателю](#pointers) невозможен
когда память на которую он указывает больше недоступна. Обратите внимание, что [срез](#slices) - это форма указателя,
поскольку он ссылается на другую память.

Чтобы избежать ошибок при работе с указателями необходимо соблюдать некоторые полезные соглашения. В общем случае,
когда функция возвращает указатель в документации к функции должно быть указано кому "принадлежит" указатель. Эта
концепция помогает программисту решить когда целесообразно освободить указатель, если это вообще возможно.

Например, в документации к функции может быть указано, что "вызывающий объект владеет возвращаемой памятью" и в этом
случае код вызывающий функцию должен иметь план когда освободить эту память. Вероятно, в этой ситуации функция примет
параметр `Allocator`.

Иногда время жизни указателя может быть более сложным. Например, срез `std.ArrayList(T).items` имеет время жизни которое
остается действительным до следующего изменения размера списка, например, путем добавления новых элементов.

В документации API для функций и структур данных следует уделить особое внимание объяснению семантики владения
указателями и времени их существования. Владелец определяет чья ответственность заключается в освобождении памяти, на
которую ссылается указатель, а время жизни определяет точку в которой память становится недоступной (чтобы не произошло
неопределенного поведения).


------------
### Compile Variables

Переменные компиляции доступны путем импорта пакета `"builtin"`, который компилятор делает доступным для каждого
исходного файла Zig. Он содержит константы времени компиляции, такие как текущая цель, порядок байтов и режим сборки.

```zig
const builtin = @import("builtin");
const separator = if (builtin.os.tag == .windows) '\\' else '/';
```

Пример того, что импортируется с помощью `@import("builtin")`:

```zig
const std = @import("std");
/// Версия Zig. При написании кода поддерживающего несколько версий Zig, предпочтительнее использовать
/// обнаружение функций (например, с помощью "@hasDecl" или "@hasField"), а не проверку версий.
pub const zig_version = std.SemanticVersion.parse(zig_version_string) catch unreachable;
pub const zig_version_string = "0.13.0";
pub const zig_backend = std.builtin.CompilerBackend.stage2_llvm;

pub const output_mode = std.builtin.OutputMode.Exe;
pub const link_mode = std.builtin.LinkMode.static;
pub const is_test = false;
pub const single_threaded = false;
pub const abi = std.Target.Abi.gnu;
pub const cpu: std.Target.Cpu = .{
    .arch = .x86_64,
    .model = &std.Target.x86.cpu.znver4,
    .features = std.Target.x86.featureSet(&[_]std.Target.x86.Feature{
        .@"64bit",
        .adx,
        .aes,
        .allow_light_256_bit,
        .avx,
        .avx2,
        .avx512bf16,
        .avx512bitalg,
        .avx512bw,
        .avx512cd,
        .avx512dq,
        .avx512f,
        .avx512ifma,
        .avx512vbmi,
        .avx512vbmi2,
        .avx512vl,
        .avx512vnni,
        .avx512vpopcntdq,
        .bmi,
        .bmi2,
        .branchfusion,
        .clflushopt,
        .clwb,
        .clzero,
        .cmov,
        .crc32,
        .cx16,
        .cx8,
        .evex512,
        .f16c,
        .fast_15bytenop,
        .fast_bextr,
        .fast_lzcnt,
        .fast_movbe,
        .fast_scalar_fsqrt,
        .fast_scalar_shift_masks,
        .fast_variable_perlane_shuffle,
        .fast_vector_fsqrt,
        .fma,
        .fsgsbase,
        .fsrm,
        .fxsr,
        .gfni,
        .invpcid,
        .lzcnt,
        .macrofusion,
        .mmx,
        .movbe,
        .mwaitx,
        .nopl,
        .pclmul,
        .pku,
        .popcnt,
        .prfchw,
        .rdpid,
        .rdpru,
        .rdrnd,
        .rdseed,
        .sahf,
        .sbb_dep_breaking,
        .sha,
        .shstk,
        .slow_shld,
        .sse,
        .sse2,
        .sse3,
        .sse4_1,
        .sse4_2,
        .sse4a,
        .ssse3,
        .vaes,
        .vpclmulqdq,
        .vzeroupper,
        .wbnoinvd,
        .x87,
        .xsave,
        .xsavec,
        .xsaveopt,
        .xsaves,
    }),
};
pub const os = std.Target.Os{
    .tag = .linux,
    .version_range = .{ .linux = .{
        .range = .{
            .min = .{
                .major = 6,
                .minor = 9,
                .patch = 2,
            },
            .max = .{
                .major = 6,
                .minor = 9,
                .patch = 2,
            },
        },
        .glibc = .{
            .major = 2,
            .minor = 39,
            .patch = 0,
        },
    }},
};
pub const target: std.Target = .{
    .cpu = cpu,
    .os = os,
    .abi = abi,
    .ofmt = object_format,
    .dynamic_linker = std.Target.DynamicLinker.init("/nix/store/k7zgvzp2r31zkg9xqgjim7mbknryv6bs-glibc-2.39-52/lib/ld-linux-x86-64.so.2"),
};
pub const object_format = std.Target.ObjectFormat.elf;
pub const mode = std.builtin.OptimizeMode.Debug;
pub const link_libc = false;
pub const link_libcpp = false;
pub const have_error_return_tracing = true;
pub const valgrind_support = true;
pub const sanitize_thread = false;
pub const position_independent_code = false;
pub const position_independent_executable = false;
pub const strip_debug_info = false;
pub const code_model = std.builtin.CodeModel.default;
pub const omit_frame_pointer = false;
```

Смотрите также:
- [Build Mode](#build-mode)


------------
### Root Source File

TODO: explain how root source file finds other files

TODO: pub fn main

TODO: pub fn panic

TODO: if linking with libc you can use export fn main

TODO: order independent top level declarations

TODO: lazy analysis

TODO: using comptime { _ = @import() }


------------
### Zig Build System

Система сборки Zig предоставляет кросс-платформенный, не зависящий от пользователя способ описания логики, необходимой
для создания проекта. В этой системе логика построения проекта записывается в файле `build.zig`, используя системный API
Zig Build для объявления и настройки артефактов сборки и других задач.

Вот несколько примеров задач, с которыми может помочь система сборки:
- Параллельное выполнение задач и кэширование результатов.
- Зависимость от других проектов.
- Предоставление пакета от которого будут зависеть другие проекты.
- Создание артефактов сборки с помощью компилятора Zig. Это включает в себя создание исходного Zig-кода, а также
исходного кода C и C++.
- Захват настроенных пользователем параметров и использование этих параметров для настройки сборки.
- Представление конфигурации сборки в виде значений [comptime](#comptime) путем предоставления файла, который может быть
импортирован с помощью Zig-кода.
- Кэширование артефактов сборки, чтобы избежать ненужного повторения шагов.
- Выполнение артефактов сборки или установленных в системе инструментов.
- Выполнение тестов и проверка соответствия результатов выполнения артефакта сборки ожидаемым значениям.
- Запуск `zig fmt` в кодовой базе или ее подмножестве.
- Пользовательские задачи.

Чтобы использовать систему сборки, запустите `zig build --help` чтобы открыть меню справки по использованию в командной
строке. Оно будет включать параметры относящиеся к конкретному проекту, которые были объявлены в сценарии `build.zig`.

На данный момент документация по системе сборки размещена на внешнем сервере: [Документация по системе
сборки](https://ziglang.org/learn/build-system/)


------------
### C

Хотя Zig не зависит от C и в отличие от большинства других языков, не зависит от `libc`, Zig признает важность
взаимодействия с существующим кодом на C.

Существует несколько способов с помощью которых Zig облегчает взаимодействие с C.


#### C Type Primitives

Они гарантированно совместимы с C ABI и могут использоваться как любой другой тип.

- `c_char`
- `c_short`
- `c_ushort`
- `c_int`
- `c_uint`
- `c_long`
- `c_ulong`
- `c_longlong`
- `c_ulonglong`
- `c_longdouble`

Чтобы взаимодействовать с типом C `void`, используйте `anyopaque`.

Смотрите также:
- [Primitive Types](#primitive-types)


#### Import from C Header File

Встроенную функцию `@cImport` можно использовать для прямого импорта символов из `.h`-файлов:

```zig
const c = @cImport({
    // See https://github.com/ziglang/zig/issues/515
    @cDefine("_NO_CRT_STDIO_INLINE", "1");
    @cInclude("stdio.h");
});
pub fn main() void {
    _ = c.printf("hello\n");
}
```
```zig
$ zig build-exe cImport_builtin.zig -lc
$ ./cImport_builtin
hello
```

Функция `@cImport` принимает выражение в качестве параметра. Это выражение вычисляется во время компиляции и
используется для управления директивами препроцессора и включения нескольких `.h`-файлов:

```zig
const builtin = @import("builtin");

const c = @cImport({
    @cDefine("NDEBUG", builtin.mode == .ReleaseFast);
    if (something) {
        @cDefine("_GNU_SOURCE", {});
    }
    @cInclude("stdlib.h");
    if (something) {
        @cUndef("_GNU_SOURCE");
    }
    @cInclude("soundio.h");
});
```

Смотрите также:
- [@cImport](#cimport)
- [@cInclude](#cInclude)
- [@cDefine](#cdefine)
- [@cUndef](#cundef)
- [@import](#import)


#### C Translation CLI

Функция перевода C в Zig доступна в качестве инструмента интерфейса командной строки через `zig translate-c`. Для этого
требуется одно имя файла в качестве аргумента. Также может использоваться набор необязательных флагов которые
передаются в `clang`. Переведенный файл записывается в стандартный вывод.


##### Command line flags

- `-I`: Укажите каталог для поиска включаемых файлов. Может использоваться несколько раз. Аналогично флагу [clang
-I](https://releases.llvm.org/12.0.0/tools/clang/docs/ClangCommandLineReference.html#cmdoption-clang-i-dir).
Текущий каталог по умолчанию не включен; используйте -I чтобы включить его.
- `-D`: Определяет макрос препроцессора. Эквивалентно флагу [clang
-D](https://releases.llvm.org/12.0.0/tools/clang/docs/ClangCommandLineReference.html#cmdoption-clang-d-macro).
- `-cflags [flags] --`: Передайте [clang произвольные дополнительные флаги командной
строки](https://releases.llvm.org/12.0.0/tools/clang/docs/ClangCommandLineReference.html). Примечание: список флагов
должен заканчиваться символом --
- `-target` [Целевая тройка](#targets) для транслированного Zig-кода. Если цель не указана, будет использоваться текущая
цель хоста.


##### Using -target and -cflags

**Важно!** При переводе кода на C с помощью `zig translate-c` вы **должны** использовать ту же тройку целей которую вы
будете использовать при компиляции транслированного кода. Кроме того, вы должны убедиться, что используемые `-cflags`
если таковые имеются, соответствуют `cflags` используемым кодом в целевой системе. Использование неправильных флагов
`-target` или `-cflags` может привести к сбоям синтаксического анализа `clang` или Zig, а также к незначительной
несовместимости ABI при компоновке с кодом на C.

```c
long FOO = __LONG_MAX__;
```
```bash
$ zig translate-c -target thumb-freestanding-gnueabihf varytarget.h|grep FOO
pub export var FOO: c_long = 2147483647;
$ zig translate-c -target x86_64-macos-gnu varytarget.h|grep FOO
pub export var FOO: c_long = 9223372036854775807;
```
```c
enum FOO { BAR };
int do_something(enum FOO foo);
```
```bash
$ zig translate-c varycflags.h|grep -B1 do_something
pub const enum_FOO = c_uint;
pub extern fn do_something(foo: enum_FOO) c_int;
$ zig translate-c -cflags -fshort-enums -- varycflags.h|grep -B1 do_something
pub const enum_FOO = u8;
pub extern fn do_something(foo: enum_FOO) c_int;
```


##### @cImport vs translate-c

`@cImport` и `zig translate-c` используют одни и те же базовые функции перевода с языка C, поэтому на техническом уровне
они эквивалентны. На практике `@cImport` полезен как способ быстрого и простого доступа к числовым константам,
определениям типов и типам записей без необходимости какой-либо дополнительной настройки. Если вам нужно передать
[cflags](#using--target-and--cflags) в `clang` или вы хотите отредактировать транслированный код, рекомендуется
использовать `zig translate-c` и сохранить результаты в файл. Распространенные причины для редактирования
сгенерированного кода включают в себя: изменение параметров `anytype` в функционально-подобных макросах на более
конкретные типы; изменение указателей `[*c]T` на `[*]T` или `*T` для повышения безопасности типов; а также [включение
или отключение безопасности во время выполнения](#setruntimesafety) в определенных функциях.

Смотрите также:
- [Targets](#targets)
- [C Type Primitives](#c-type-primitives)
- [Pointers](#pointers)
- [C Pointers](#c-pointers)
- [Import from C Header File](#import-from-c-header-file)
- [@cInclude](#cinclude)
- [@cImport](#cimport)
- [@setRuntimeSafety](#setruntimesafety)


#### C Translation Caching

Функция трансляции C (независимо от того, используется ли она через `zig translate-c` или `@cImport`) интегрируется с
системой кэширования Zig. При последующих запусках с тем же исходным файлом, целевым объектом и `cflags` будет
использоваться кэш вместо повторного перевода одного и того же кода.

Чтобы увидеть, где хранятся кэшированные файлы при компиляции кода, использующего `@cImport`, используйте флаг
`--verbose-cimport`:

```zig
const c = @cImport({
    @cDefine("_NO_CRT_STDIO_INLINE", "1");
    @cInclude("stdio.h");
});
pub fn main() void {
    _ = c;
}
```
```bash
$ zig build-exe verbose_cimport_flag.zig -lc --verbose-cimport
info(compilation): C import source: /home/andy/src/zig/.zig-cache/o/f4e9c68cba40c97888f064d67b031021/cimport.h
info(compilation): C import .d file: /home/andy/src/zig/.zig-cache/o/f4e9c68cba40c97888f064d67b031021/cimport.h.d
info(compilation): C import output: /home/andy/src/zig/.zig-cache/o/1b63455e1d0d323f51bdc4909717e28b/cimport.zig
$ ./verbose_cimport_flag
```

`cimport.h` содержит файл для трансляции (созданный на основе вызовов `@cInclude`, `@cDefine` и `@cUndef`), `cimport.h.d`
- это список зависимостей файлов, а `cimport.zig` содержит транслированные выходные данные.

Смотрите также:
- [Import from C Header File](#import-from-c-header-file)
- [C Translation CLI](#c-translation-cli)
- [@cInclude](#cinclude)
- [@cImport](#cimport)


#### Translation failures

Некоторые конструкции языка С не могут быть переведены в Zig - например, _goto_, структуры с битовыми полями и макросы
для вставки токенов. В Zig используется понижение, позволяющее продолжить трансляцию при наличии нетранслируемых
объектов.

Разновидностей понижения три - [opaque](#opaque), _extern_, и `@compileError`. Структуры и объединения языка С которые
не могут быть транслированы правильно, будут транслированы как `opaque{}`. Функции содержащие непрозрачные типы или
конструкции кода которые невозможно транслировать будут понижены до `extern` объявления. Таким образом, нетранслируемые
типы все еще могут использоваться в качестве указателей, а нетранслируемые функции могут вызываться до тех пор, пока
компоновщик знает о скомпилированной функции.

`@compileError` используется, когда определения верхнего уровня (глобальные переменные, прототипы функций, макросы) не
могут быть транслированы или понижены. Поскольку Zig использует отложенный анализ для объявлений верхнего
уровня, нетранслируемые объекты не вызовут ошибки компиляции в вашем коде, если вы на самом деле их не используете.

Смотрите также:
- [opaque](#opaque)
- [extern](#extern)
- [@compileError](#compileerror)


#### C Macros

При переводе с языка C делается все возможное чтобы перевести функционально-подобные макросы в эквивалентные
Zig-функции. Поскольку макросы на языке C работают на уровне лексических единиц, не все макросы на языке C могут быть
переведены на Zig. Макросы которые не могут быть переведены будут переведены в `@compileError`. Обратите внимание, что
код на C который использует макросы будет транслирован без каких-либо дополнительных проблем (поскольку Zig работает с
предварительно обработанным исходным кодом с расширенными макросами). Это просто сами макросы которые могут быть
недоступны для трансляции в Zig.

Рассмотрим следующий пример:

```c
#define MAKELOCAL(NAME, INIT) int NAME = INIT
int foo(void) {
   MAKELOCAL(a, 1);
   MAKELOCAL(b, 2);
   return a + b;
}
```
```bash
$ zig translate-c macro.c > macro.zig
```
```zig
pub export fn foo() c_int {
    var a: c_int = 1;
    _ = &a;
    var b: c_int = 2;
    _ = &b;
    return a + b;
}
pub const MAKELOCAL = @compileError("unable to translate C expr: unexpected token .Equal"); // macro.c:1:9
```

Обратите внимание, что `foo` был транслирован правильно, несмотря на использование непереводимого макроса. Значение
`MAKELOCAL` было изменено на `@compileError`, поскольку оно не может быть выражено как функция Zig; это просто означает,
что вы не можете напрямую использовать `MAKELOCAL` из Zig.

Смотрите также:
- [@compileError](#compileerror)


#### C Pointers

Этого типа следует избегать когда это возможно. Единственная веская причина для использования указателя на C
заключается в автоматически сгенерированном коде при трансляции кода на C.

При импорте файлов заголовков на языке С неясно, следует ли переводить указатели как одноэлементные (`*T`) или как
многоэлементные (`[*]T`). Указатели на языке C являются компромиссным решением, позволяющим Zig-коду напрямую
использовать транслированные файлы заголовков.

`[*c]T` - Указатель C.

- Поддерживает весь синтаксис двух других типов указателей (`*T`) и (`[*]T`).
- Приводит к другим типам указателей, а также к [опциональным указателям](#optional-pointers). Когда указатель С
преобразуется в опциональный указатель происходит [неопределенное поведение](#undefined-behavior) с проверкой
безопасности, если адрес равен 0.
- Разрешает обращение к адресу 0. На несвободных объектах разыменование адреса 0 является [неопределенным
поведением](#undefined-behavior) с проверкой безопасности. Опциональные указатели C вводят еще один бит для отслеживания
нуля, как и `?usize`. Обратите внимание, что создание опционального указателя C не требуется, так как можно использовать
обычные [опциональные указатели](#optional-pointers).
- Поддерживает приведение типов к целым числам и из них.
- Поддерживает сравнение с целыми числами.
- Не поддерживает атрибуты указателей только для Zig, такие как выравнивание. Используйте обычные указатели, пожалуйста!

Когда указатель C указывает на отдельную структуру (не массив), разыменуйте указатель C, чтобы получить доступ к полям
структуры или данным-элементам. Этот синтаксис выглядит следующим образом:

`ptr_to_struct.*.struct_member`

Это сравнимо с выполнением `->` в C.

Когда указатель C указывает на массив структур, синтаксис возвращается к следующему:

`ptr_to_struct_array[index].struct_member`


#### C Variadic Functions

Zig поддерживает внешние переменные функции.

```zig
const std = @import("std");
const testing = std.testing;

pub extern "c" fn printf(format: [*:0]const u8, ...) c_int;

test "variadic function" {
    try testing.expect(printf("Hello, world!\n") == 14);
    try testing.expect(@typeInfo(@TypeOf(printf)).Fn.is_var_args);
}
```
```bash
$ zig test test_variadic_function.zig -lc
1/1 test_variadic_function.test.variadic function...OK
All 1 tests passed.
Hello, world!
```

Функции с переменным количеством аргументов могут быть реализованы с помощью [@cVaStart](#cvastart), [@cVaEnd](#cvaend),
[@cVaArg](#cvaarg) и [@cVaCopy](#cvacopy).


```zig
const std = @import("std");
const testing = std.testing;
const builtin = @import("builtin");

fn add(count: c_int, ...) callconv(.C) c_int {
    var ap = @cVaStart();
    defer @cVaEnd(&ap);
    var i: usize = 0;
    var sum: c_int = 0;
    while (i < count) : (i += 1) {
        sum += @cVaArg(&ap, c_int);
    }
    return sum;
}

test "defining a variadic function" {
    if (builtin.cpu.arch == .aarch64 and builtin.os.tag != .macos) {
        // https://github.com/ziglang/zig/issues/14096
        return error.SkipZigTest;
    }
    if (builtin.cpu.arch == .x86_64 and builtin.os.tag == .windows) {
        // https://github.com/ziglang/zig/issues/16961
        return error.SkipZigTest;
    }

    try std.testing.expectEqual(@as(c_int, 0), add(0));
    try std.testing.expectEqual(@as(c_int, 1), add(1, @as(c_int, 1)));
    try std.testing.expectEqual(@as(c_int, 3), add(2, @as(c_int, 1), @as(c_int, 2)));
}
```
```bash
$ zig test test_defining_variadic_function.zig
1/1 test_defining_variadic_function.test.defining a variadic function...OK
All 1 tests passed.
```


#### Exporting a C Library

Одним из основных вариантов использования Zig является экспорт библиотеки с помощью C ABI для использования в других
языках программирования. Ключевое слово `export` перед функциями, переменными и типами делает их частью библиотечного
API:

```zig
export fn add(a: i32, b: i32) i32 {
    return a + b;
}
```

Чтобы создать статическую библиотеку:

```bash
$ zig build-lib mathtest.zig
```

Чтобы создать общую библиотеку:

```bash
$ zig build-lib mathtest.zig -dynamic
```

Вот пример с [системой сборки Zig](#zig-build-system):

```c
// Этот заголовок сгенерирован zig из файла mathtest.zig
#include "mathtest.h"
#include <stdio.h>

int main(int argc, char **argv) {
    int32_t result = add(42, 1337);
    printf("%d\n", result);
    return 0;
}
```
```zig
const std = @import("std");

pub fn build(b: *std.Build) void {
    const lib = b.addSharedLibrary(.{
        .name = "mathtest",
        .root_source_file = b.path("mathtest.zig"),
        .version = .{ .major = 1, .minor = 0, .patch = 0 },
    });
    const exe = b.addExecutable(.{
        .name = "test",
    });
    exe.addCSourceFile(.{ .file = b.path("test.c"), .flags = &.{"-std=c99"} });
    exe.linkLibrary(lib);
    exe.linkSystemLibrary("c");

    b.default_step.dependOn(&exe.step);

    const run_cmd = exe.run();

    const test_step = b.step("test", "Test the program");
    test_step.dependOn(&run_cmd.step);
}
```
```bash
$ zig build test
1379
```

Смотрите также:
- [export](#export)


#### Mixing Object Files

Вы можете смешивать объектные файлы Zig с любыми другими объектными файлами которые соответствуют C ABI. Пример:

```zig
const base64 = @import("std").base64;

export fn decode_base_64(
    dest_ptr: [*]u8,
    dest_len: usize,
    source_ptr: [*]const u8,
    source_len: usize,
) usize {
    const src = source_ptr[0..source_len];
    const dest = dest_ptr[0..dest_len];
    const base64_decoder = base64.standard.Decoder;
    const decoded_size = base64_decoder.calcSizeForSlice(src) catch unreachable;
    base64_decoder.decode(dest[0..decoded_size], src) catch unreachable;
    return decoded_size;
}
```
```c
// Этот заголовок сгенерирован zig из base64.zig
#include "base64.h"

#include <string.h>
#include <stdio.h>

int main(int argc, char **argv) {
    const char *encoded = "YWxsIHlvdXIgYmFzZSBhcmUgYmVsb25nIHRvIHVz";
    char buf[200];

    size_t len = decode_base_64(buf, 200, encoded, strlen(encoded));
    buf[len] = 0;
    puts(buf);

    return 0;
}
```
```zig
const std = @import("std");

pub fn build(b: *std.Build) void {
    const obj = b.addObject(.{
        .name = "base64",
        .root_source_file = b.path("base64.zig"),
    });

    const exe = b.addExecutable(.{
        .name = "test",
    });
    exe.addCSourceFile(.{ .file = b.path("test.c"), .flags = &.{"-std=c99"} });
    exe.addObject(obj);
    exe.linkSystemLibrary("c");
    b.installArtifact(exe);
}
```
```bash
$ zig build
$ ./zig-out/bin/test
all your base are belong to us
```


------------
### WebAssembly

Zig поддерживает сборку для WebAssembly "из коробки".


#### Freestanding

Для таких хост-сред, как веб-браузер и nodejs, сборка выполняется в виде исполняемого файла с использованием автономной
целевой операционной системы. Вот пример запуска Zig-кода, скомпилированного в WebAssembly с помощью nodejs.

```zig
extern fn print(i32) void;

export fn add(a: i32, b: i32) void {
    print(a + b);
}
```
```bash
$ zig build-exe math.zig -target wasm32-freestanding -fno-entry --export=add
```
```js
const fs = require('fs');
const source = fs.readFileSync("./math.wasm");
const typedArray = new Uint8Array(source);

WebAssembly.instantiate(typedArray, {
  env: {
    print: (result) => { console.log(`The result is ${result}`); }
  }}).then(result => {
  const add = result.instance.exports.add;
  add(1, 2);
});
```
```bash
$ node test.js
The result is 3
```


#### WASI

Поддержка Zig системного интерфейса WebAssembly (WASI) находится в стадии активной разработки. Пример использования
стандартной библиотеки и чтения аргументов командной строки:

```zig
const std = @import("std");

pub fn main() !void {
    var general_purpose_allocator = std.heap.GeneralPurposeAllocator(.{}){};
    const gpa = general_purpose_allocator.allocator();
    const args = try std.process.argsAlloc(gpa);
    defer std.process.argsFree(gpa, args);

    for (args, 0..) |arg, i| {
        std.debug.print("{}: {s}\n", .{ i, arg });
    }
}
```
```bash
$ zig build-exe wasi_args.zig -target wasm32-wasi
```
```bash
$ wasmtime wasi_args.wasm 123 hello
0: wasi_args.wasm
1: 123
2: hello
```

Более интересным примером может быть извлечение списка предварительных открытий из среды выполнения. Теперь это
поддерживается в стандартной библиотеке с помощью `std.fs.wasi.Preopens`:

```zig
const std = @import("std");
const fs = std.fs;

pub fn main() !void {
    var general_purpose_allocator = std.heap.GeneralPurposeAllocator(.{}){};
    const gpa = general_purpose_allocator.allocator();

    var arena_instance = std.heap.ArenaAllocator.init(gpa);
    defer arena_instance.deinit();
    const arena = arena_instance.allocator();

    const preopens = try fs.wasi.preopensAlloc(arena);

    for (preopens.names, 0..) |preopen, i| {
        std.debug.print("{}: {s}\n", .{ i, preopen });
    }
}
```
```bash
$ zig build-exe wasi_preopens.zig -target wasm32-wasi
```
```bash
$ wasmtime --dir=. wasi_preopens.wasm
0: stdin
1: stdout
2: stderr
3: .
```


------------
### Targets

Целевой - это компьютер, который будет использоваться для запуска исполняемого файла. Он состоит из архитектуры
процессора, набора включенных функций процессора, операционной системы, минимальной и максимальной версий операционной
системы, ABI и версии ABI-интерфейса.

Zig - это язык программирования общего назначения, что означает, что он предназначен для генерации оптимального кода для
большого набора целей. Команда `zig targets` предоставляет информацию обо всех целях, известных компилятору.

Если компилятору не задан параметр target, то по умолчанию выбирается *целевой компьютер*, а это означает, что
результирующий исполняемый файл будет _непригоден для копирования на другой компьютер_. Чтобы скопировать исполняемый
файл на другой компьютер, компилятору необходимо узнать о целевых требованиях с помощью параметра `-target`.

Стандартная библиотека Zig (`@import("std")`) имеет кросс-платформенные абстракции, что делает один и тот же исходный
код пригодным для многих целей. Некоторые коды более переносимы чем другие. В целом, Zig-код чрезвычайно переносим по
сравнению с другими языками программирования.

Для обеспечения работы кросс-платформенных абстракций Zig на каждой платформе требуются свои собственные реализации. Эти
реализации находятся на разной степени завершенности. К каждому помеченному выпуску компилятора прилагаются примечания к
выпуску, которые содержат полную таблицу поддержки для каждой целевой версии.


------------
### Style Guide

Компилятор не применяет эти соглашения по кодированию, но они включены в эту документацию вместе с компилятором чтобы
обеспечить ориентир на случай, если кто-то захочет указать авторитетному специалисту на согласованный стиль кодирования
Zig.


#### Avoid Redundancy in Names

Избегайте этих слов в названиях типов:
- Value
- Data
- Context
- Manager
- utils, misc, или чьи-то инициалы

Все является значением, все типы являются данными, все является контекстом, вся логика управляет состоянием. Ничто не
передается с помощью слова которое применимо ко всем типам.

Искушение использовать "utilities", "miscellaneous", или чьи-то инициалы - это ошибка в категоризации или, что более
распространено, чрезмерная категоризация. Такие объявления могут находиться в корне модуля которому они нужны, без
необходимости в пространстве имен.


#### Avoid Redundant Names in Fully-Qualified Namespaces

Компилятор присваивает каждому объявлению полное пространство имен, создавая древовидную структуру. Выбирайте имена
основанные на полном пространстве имен и избегайте избыточных сегментов имен.

```zig
const std = @import("std");

pub const json = struct {
    pub const JsonValue = union(enum) {
        number: f64,
        boolean: bool,
        // ...
    };
};

pub fn main() void {
    std.debug.print("{s}\n", .{@typeName(json.JsonValue)});
}
```
```bash
$ zig build-exe redundant_fqn.zig
$ ./redundant_fqn
redundant_fqn.json.JsonValue
```

В этом примере "json" повторяется в полном пространстве имен. Решение состоит в том чтобы удалить `Json` из `JsonValue`.
В этом примере у нас есть пустая структура с именем `json`, но помните, что файлы также являются частью пространства
имен с полным описанием.

Этот пример является исключением из правила, указанного в разделе [избегайте избыточности в
именах](#avoid-redundancy-in-names). Значение типа было сведено к его сути: это значение в формате `json`. Название не
может быть более конкретным не будучи неправильным.


#### Whitespace

- Отступ в 4 пробела
- Расставьте фигурные скобки на одной строке, если только вам не нужно их переносить.
- Если список элементов длиннее 2, поместите каждый элемент в отдельную строку и попробуйте поставить дополнительную
запятую в конце.
- Длина строки: стремитесь к 100; руководствуйтесь здравым смыслом.


#### Names

Грубо говоря: `camelCaseFunctionName`, `TitleCaseTypeName`, `snake_case_variable_name`. Более точно:

- Если `x` это тип, тогда `x` должен быть `TitleCase`, если только он не является структурой которая содержит 0 полей и
никогда не будет инстанцирован, то в этом случае он считается "пространством имен" и использует `snake_case`.
- Если `x` является вызываемым, а тип возврата `x` - это тип, то `x` должен быть `TitleCase`.
- Если `x` может быть вызван иным способом, то `x` должен быть в `camelCase`.
- В противном случае `x` должно быть `snake_case`.

Аббревиатуры, инициализмы, имена собственные или любые другие слова для которых в письменном английском языке
существуют правила написания заглавными буквами подчиняются соглашениям о наименовании, как и любые другие слова. Даже
акронимы, состоящие всего из 1 букв подчиняются этим соглашениям.

Имена файлов делятся на две категории: типы и пространства имен. Если файл (неявно являющийся структурой) имеет поля
верхнего уровня, он должен быть назван как любая другая структура с полями с использованием `TitleCase`. В противном
случае следует использовать `snake_case`. Имена каталогов должны быть в `snake_case`.

Это общие правила; если есть смысл сделать что-то другое, делайте то, что имеет смысл. Например, если существует
установленное соглашение, такое как `ENOENT`, то следуйте ему.


#### Examples

```zig
const namespace_name = @import("dir_name/file_name.zig");
const TypeName = @import("dir_name/TypeName.zig");
var global_var: i32 = undefined;
const const_name = 42;
const primitive_type_alias = f32;
const string_alias = []u8;

const StructName = struct {
    field: i32,
};
const StructAlias = StructName;

fn functionName(param_name: TypeName) void {
    var functionPointer = functionName;
    functionPointer();
    functionPointer = otherFunction;
    functionPointer();
}
const functionAlias = functionName;

fn ListTemplateFunction(comptime ChildType: type, comptime fixed_size: usize) type {
    return List(ChildType, fixed_size);
}

fn ShortList(comptime T: type, comptime n: usize) type {
    return struct {
        field_name: [n]T,
        fn methodName() void {}
    };
}

// Слово XML теряет регистр при использовании в идентификаторах Zig.
const xml_document =
    \\<?xml version="1.0" encoding="UTF-8"?>
    \\<document>
    \\</document>
;
const XmlParser = struct {
    field: i32,
};
// Инициалы BE (Big Endian) - это просто еще одно слово в именах идентификаторов Zig.
fn readU32Be() u32 {}
```

Дополнительные примеры приведены в [стандартной библиотеке
Zig](https://ziglang.org/documentation/0.13.0/#Zig-Standard-Library).


#### Doc Comment Guidance

- Опустите любую информацию которая является избыточной из-за названия документируемого объекта.
- Дублирование информации о нескольких схожих функциях приветствуется, поскольку это помогает IDE и другим инструментам
предоставлять более качественный текст справки.
- Используйте слово *assume* для обозначения инвариантов, нарушение которых приводит к неопределенному поведению.
- Используйте слово *assert* для обозначения инвариантов, нарушение которых приводит к неопределенному поведению с
проверкой безопасности.


------------
### Source Encoding

Исходный код Zig закодирован в UTF-8. Неверная последовательность байтов UTF-8 приводит к ошибке компиляции.

Во всем исходном коде Zig (в том числе и в комментариях) некоторые кодовые точки недопустимы:

- Управляющие символы Ascii, за исключением U+000a (LF), U+000d (CR) и U+0009 (HT): U+0000 - U+0008, U+000b - U+000c,
U+000e - U+0001f, U+007f.
- Окончания строк в Юникоде, отличных от Ascii: U+0085 (NEL), U+2028 (LS), U+2029 (PS).

LF (байтовое значение 0x0a, кодовая точка U+000a, `'\n'`) - это символ окончания строки в исходном коде Zig. Это
байтовое значение завершает каждую строку исходного Zig-кода, за исключением последней строки файла. Рекомендуется,
чтобы непустые исходные файлы заканчивались пустой строкой, что означает, что последний байт должен быть равен 0x0a
(LF).

Каждому LF может непосредственно предшествовать один CR (байтовое значение 0x0d, кодовая точка U+000d, `'\r'`), чтобы
сформировать окончание строки в стиле Windows, но это не рекомендуется. Обратите внимание, что в многострочных строках
последовательности CRLF будут кодироваться как LF при компиляции в Zig-программу. Использование CR в любом другом
контексте недопустимо.

Жесткие символы табуляции HT (байтовое значение 0x09, кодовая точка U+0009, `'\t'`) взаимозаменяемы с пробелами SP
(байтовое значение 0x20, кодовая точка U+0020, `' '`) в качестве разделителя символов, но использование жестких символов
табуляции не рекомендуется. Смотрите также [грамматику](#grammar).

Для обеспечения совместимости с другими инструментами компилятор игнорирует метку порядка байтов в кодировке UTF-8
(U+FEFF), если он является первой кодовой точкой Unicode в исходном тексте. Метка порядка байтов больше нигде в исходном
тексте не допускается.

Обратите внимание, что запуск `zig fmt` с исходным файлом позволит реализовать все рекомендации, упомянутые здесь.

Обратите внимание, что инструмент считывающий исходный код Zig может делать предположения, если исходный код считается
правильным кодом Zig. Например, при определении концов строк инструмент может использовать простой поиск, такой как
`/\n/`, или [расширенный](https://msdn.microsoft.com/en-us/library/dd409797.aspx) поиск, такой как
`/\r\n?|[\n\u0085\u2028\u2029]/`, и в том, и в другом случае окончания строк будут определены правильно. Для другого
примера, при определении пробела перед первым маркером в строке инструмент может либо использовать простой поиск, такой
как `/[\t]/`, или [расширенный](https://tc39.es/ecma262/#sec-characterclassescape) поиск, такой как `/\s/` и в любом
случае пробелы будут правильно идентифицированы.


------------
### Keyword Reference

|Keyword|Description|
|----|------------|
|`addrspace`|Ключевое слово `addrspace`. <br>TODO add documentation for addrspace|
|`align`|`align` может использоваться для указания выравнивания указателя. Его также можно использовать после объявления переменной или функции, чтобы указать выравнивание указателей на эту переменную или функцию. Смотрите также [Alignment](#alignment).|
|`allowzero`|Атрибут указателя `allowzero` позволяет указателю иметь нулевой адрес. Смотрите также [allowzero](#allowzero).|
|`and`|Логический оператор `and`. Смотрите также [Operators](#operators).|
|`anyframe`|`anyframe` может использоваться в качестве типа для переменных которые содержат указатели на функциональные фреймы. Смотрите также [Async Functions](#async-functions).|
|`anytype`|Параметры функции могут быть объявлены с использованием `anytype` вместо `type`. Тип будет определен при вызове функции. Смотрите также [Function Parameter Type Inference](#function-parameter-type-inference).|
|`asm`|`asm` запускает встроенное ассемблерное выражение. Это позволяет напрямую управлять машинным кодом, сгенерированным при компиляции. Смотрите также [Assembly](#assembly).|
|`await`|`await` может использоваться для приостановки текущей функции до тех пор, пока кадр предоставленный после `await` завершает работу. `await` копирует значение, возвращаемое из фрейма целевой функции вызывающему объекту. Смотрите также [Async Functions](#async-functions).|
|`break`|`break` может использоваться с меткой блока для возврата значения из блока. Его также можно использовать для выхода из цикла до естественного завершения итерации. Смотрите также [Blocks](#blocks), [while](#while), [for](#for).|
|`callconv`|`callconv` может использоваться для указания соглашения о вызове в типе функции. Смотрите также [Functions](#functions).|
|`catch`|`catch` может быть использован для оценки выражения, если выражение перед ним оценивается как ошибка. Выражение после `catch` может по желанию перехватить значение ошибки. Смотрите также [catch](#catch), [Operators](#Operators).|
|`comptime`|`comptime` перед объявлением может использоваться для обозначения переменных или параметров функции как известных во время компиляции. Он также может использоваться для гарантии выполнения выражения во время компиляции. Смотрите также [comptime](#comptime).|
|`const`|`const` объявляет переменную которая не может быть изменена. Используется в качестве атрибута указателя и означает, что значение на которое ссылается указатель не может быть изменено. Смотрите также [Variables](#variables).|
|`continue`|`continue` может использоваться в цикле для возврата к началу цикла. Смотрите также [while](#while), [for](#for).|
|`defer`|`defer` выполнит выражение когда поток управления покинет текущий блок. Смотрите также [defer](#defer).|
|`else`|`else` может использоваться для предоставления альтернативной ветви для `if`, `switch`, `while`, и `for` выражения. Если используется после выражения `if`, ветвь `else` будет выполнена, если тестовое значение возвращает значение `false`, `null` или `error`. Если используется в выражении `switch`, ветвь `else` будет выполнена, если тестовое значение не соответствует другим случаям. Если используется после выражения цикла, ветвь `else` будет выполнена, если цикл завершится без прерывания. Смотрите также [if](#if), [switch](#switch), [while](#while), [for](#for).|
|`enum`|`enum` определяет тип перечисления. Смотрите также [enum](#enum).|
|`errdefer`|`errdefer` выполнит выражение когда поток управления покинет текущий блок, если функция возвращает ошибку, выражение `errdefer` может захватить развернутое значение. Смотрите также [errdefer](#errdefer).|
|`error`|`error` определяет тип ошибки. Смотрите также [Errors](#errors).|
|`export`|`export` делает функцию или переменную внешне видимой в сгенерированном объектном файле. Экспортируемые функции по умолчанию соответствуют соглашению о вызове языка C. Смотрите также [Functions](#functions).|
|`extern`|`extern` можно использовать для объявления функции или переменной которая будет разрешена во время компоновки, если компоновка статическая или во время выполнения, если компоновка динамическая. Смотрите также [Functions](#functions).|
|`fn`|`fn` объявляет функцию. Смотрите также [Functions](#functions).|
|`for`|Выражение `for` может использоваться для перебора элементов среза, массива или кортежа. Смотрите также [for](#for).|
|`if`|Выражение `if` может проверять логические выражения, опциональные значения или объединения с ошибками. Для опциональных значений или объединения с ошибками выражение `if` может захватывать развернутое значение. Смотрите также [if](#if).|
|`inline`|`inline` может использоваться для обозначения выражения цикла таким образом, чтобы оно было развернуто во время компиляции. Его также можно использовать чтобы заставить функцию быть встроенной во всех местах вызова. Смотрите также [inline while](#inline-while), [inline for](#inline-for),[Functions](#functions).|
|`linksection`|Ключевое слово `linksection` можно использовать для указания того, в какой раздел будет помещена функция или глобальная переменная (например, `.text`).|
|`noalias`|Ключевое слово `noalias`.<br>TODO add documentation for noalias|
|`noinline`|`noinline` запрещает встраивать функцию во всех местах вызова. Смотрите также [Functions](#functions).|
|`nosuspend`|Ключевое слово `nosuspend` может использоваться перед блоком, оператором или выражением для обозначения области в которой не достигнуты точки приостановки. В частности, внутри области `nosuspend`: <br>- Использование ключевого слова `suspend` приводит к ошибке компиляции. <br>- Использование `await` для фрейма функции который еще не завершен приводит к [Undefined Behavior](#undefined-behavior) с проверкой безопасности. <br>- Вызов асинхронной функции может привести к [Undefined Behavior](#undefined-behavior) с проверкой безопасности, поскольку это эквивалентно функции `await async some_async_fn()`, которая содержит `await`. <br>- Код внутри области `nosuspend` не приводит к тому, что окружающая функция становится [async functions](#async-functions). <br>Смотрите также [Async Functions](#async-functions).|
|`opaque`|`opaque` определяет непрозрачный тип. Смотрите также [opaque](#opaque).|
|`or`|Логический оператор `or`. Смотрите также [Operators](#operators).|
|`orelse`|`orelse` может использоваться для оценки выражения, если предшествующее ему выражение принимает значение `null`. Смотрите также [Optionals](#optionals),[Operators](#operators).|
|`packed`|Ключевое слово `packed` перед определением структуры изменяет расположение структуры в памяти на гарантированное расположение в упакованном виде. Смотрите также [packed struct](#packed-struct).|
|`pub`|`pub` перед объявлением верхнего уровня делает объявление доступным для ссылки из файла, отличного от того в котором оно объявлено. Смотрите также [import](#import).|
|`resume`|`resume` продолжит выполнение фрейма функции после того как функция была приостановлена.|
|`return`|`return` выходит из функции со значением. Смотрите также [Functions](#functions).|
|`struct`|`struct` определяет структуру. Смотрите также [struct](#struct).|
|`suspend`|`suspend` приведет к возврату потока управления к месту вызова или возобновлению работы функции. `suspend` может также использоваться перед блокированием внутри функции, чтобы разрешить функции доступ к своему фрейму до того как поток управления вернется к месту вызова.|
|`switch`|Выражение `switch` может быть использовано для проверки значений общего типа. Варианты `switch` могут содержать значения полей [Tagged union](#tagged-union). Смотрите также [switch](#switch).|
|`test`|Ключевое слово `test` может использоваться для обозначения блока кода верхнего уровня, который используется для проверки того, что поведение соответствует ожиданиям. Смотрите также [Zig Test](#zig-test).|
|`threadlocal`|`threadlocal` можно использовать для указания переменной как локальной для потока. Смотрите также [Thread Local Variables](#thread-local-variables).|
|`try`|`try` оценивает выражение объединения с ошибкой. Если это ошибка, оно возвращается из текущей функции с той же ошибкой. В противном случае результатом выражения будет развернутое значение. Смотрите также [try](#try).|
|`union`|`union` определяет объединение. Смотрите также [union](#union).|
|`unreachable`|`unreachable` может использоваться для утверждения того, что поток управления никогда не достигнет определенного места. <br>В зависимости от режима сборки, `unreachable` может вызвать панику. <br>Вызывает панику в режимах `Debug` и `ReleaseSafe`, а также при использовании zig-теста. <br>Не вызывает паники в `ReleaseFast` и `ReleaseSmall` режиме. Смотрите также [unreachable](#unreachable).|
|`usingnamespace`|`usingnamespace` - это объявление верхнего уровня, оно импортирует все публичные объявления операнда которые должны быть структурой, объединением или перечислением, в текущую область видимости. Смотрите также [usingnamespace](#usingnamespace).|
|`var`|`var` объявляет переменную которая может быть изменена. Смотрите также [Variables](#variables).|
|`volatile`|`volatile` может использоваться для обозначения того, что загрузка или сохранение указателя имеет побочные эффекты. Он также может модифицировать встроенное ассеблерное выражение чтобы обозначить, что оно имеет побочные эффекты. Смотрите также [volatile](#volatile), [Assembly](#assembly).|
|`while`|Выражение `while` может использоваться для многократной проверки логического, опционального или ошибочного выражения объединения и остановки цикла, когда значение этого выражения оценивается как `false`, `null` или `error` соответственно. Смотрите также [while](#while).|


------------
### Appendix

#### Containers

_Контейнером_ в Zig называется любая синтаксическая конструкция которая действует как пространство имен для хранения
объявлений [переменных](#variables) и [функций](#functions). Контейнеры также являются определениями типов, которые
могут быть созданы в виде экземпляров. [Структуры](#struct), [перечисления](#enum), [объединения](#union), [типы с
неизвестным размером](#opaque) и даже исходные файлы Zig сами по себе являются контейнерами.

Хотя контейнеры (за исключением исходных файлов Zig) заключены в фигурные скобки, их не следует путать с блоками или
функциями. Контейнеры не содержат инструкций.


#### Grammar

```y
Root <- skip container_doc_comment? ContainerMembers eof

# *** Top level ***
ContainerMembers <- ContainerDeclaration* (ContainerField COMMA)* (ContainerField / ContainerDeclaration*)

ContainerDeclaration <- TestDecl / ComptimeDecl / doc_comment? KEYWORD_pub? Decl

TestDecl <- KEYWORD_test (STRINGLITERALSINGLE / IDENTIFIER)? Block

ComptimeDecl <- KEYWORD_comptime Block

Decl
    <- (KEYWORD_export / KEYWORD_extern STRINGLITERALSINGLE? / KEYWORD_inline / KEYWORD_noinline)? FnProto (SEMICOLON / Block)
     / (KEYWORD_export / KEYWORD_extern STRINGLITERALSINGLE?)? KEYWORD_threadlocal? GlobalVarDecl
     / KEYWORD_usingnamespace Expr SEMICOLON

FnProto <- KEYWORD_fn IDENTIFIER? LPAREN ParamDeclList RPAREN ByteAlign? AddrSpace? LinkSection? CallConv? EXCLAMATIONMARK? TypeExpr

VarDeclProto <- (KEYWORD_const / KEYWORD_var) IDENTIFIER (COLON TypeExpr)? ByteAlign? AddrSpace? LinkSection?

GlobalVarDecl <- VarDeclProto (EQUAL Expr)? SEMICOLON

ContainerField <- doc_comment? KEYWORD_comptime? !KEYWORD_fn (IDENTIFIER COLON)? TypeExpr ByteAlign? (EQUAL Expr)?

# *** Block Level ***
Statement
    <- KEYWORD_comptime ComptimeStatement
     / KEYWORD_nosuspend BlockExprStatement
     / KEYWORD_suspend BlockExprStatement
     / KEYWORD_defer BlockExprStatement
     / KEYWORD_errdefer Payload? BlockExprStatement
     / IfStatement
     / LabeledStatement
     / SwitchExpr
     / VarDeclExprStatement

ComptimeStatement
    <- BlockExpr
     / VarDeclExprStatement

IfStatement
    <- IfPrefix BlockExpr ( KEYWORD_else Payload? Statement )?
     / IfPrefix AssignExpr ( SEMICOLON / KEYWORD_else Payload? Statement )

LabeledStatement <- BlockLabel? (Block / LoopStatement)

LoopStatement <- KEYWORD_inline? (ForStatement / WhileStatement)

ForStatement
    <- ForPrefix BlockExpr ( KEYWORD_else Statement )?
     / ForPrefix AssignExpr ( SEMICOLON / KEYWORD_else Statement )

WhileStatement
    <- WhilePrefix BlockExpr ( KEYWORD_else Payload? Statement )?
     / WhilePrefix AssignExpr ( SEMICOLON / KEYWORD_else Payload? Statement )

BlockExprStatement
    <- BlockExpr
     / AssignExpr SEMICOLON

BlockExpr <- BlockLabel? Block

# An expression, assignment, or any destructure, as a statement.
VarDeclExprStatement
    <- VarDeclProto (COMMA (VarDeclProto / Expr))* EQUAL Expr SEMICOLON
     / Expr (AssignOp Expr / (COMMA (VarDeclProto / Expr))+ EQUAL Expr)? SEMICOLON

# *** Expression Level ***

# An assignment or a destructure whose LHS are all lvalue expressions.
AssignExpr <- Expr (AssignOp Expr / (COMMA Expr)+ EQUAL Expr)?

SingleAssignExpr <- Expr (AssignOp Expr)?

Expr <- BoolOrExpr

BoolOrExpr <- BoolAndExpr (KEYWORD_or BoolAndExpr)*

BoolAndExpr <- CompareExpr (KEYWORD_and CompareExpr)*

CompareExpr <- BitwiseExpr (CompareOp BitwiseExpr)?

BitwiseExpr <- BitShiftExpr (BitwiseOp BitShiftExpr)*

BitShiftExpr <- AdditionExpr (BitShiftOp AdditionExpr)*

AdditionExpr <- MultiplyExpr (AdditionOp MultiplyExpr)*

MultiplyExpr <- PrefixExpr (MultiplyOp PrefixExpr)*

PrefixExpr <- PrefixOp* PrimaryExpr

PrimaryExpr
    <- AsmExpr
     / IfExpr
     / KEYWORD_break BreakLabel? Expr?
     / KEYWORD_comptime Expr
     / KEYWORD_nosuspend Expr
     / KEYWORD_continue BreakLabel?
     / KEYWORD_resume Expr
     / KEYWORD_return Expr?
     / BlockLabel? LoopExpr
     / Block
     / CurlySuffixExpr

IfExpr <- IfPrefix Expr (KEYWORD_else Payload? Expr)?

Block <- LBRACE Statement* RBRACE

LoopExpr <- KEYWORD_inline? (ForExpr / WhileExpr)

ForExpr <- ForPrefix Expr (KEYWORD_else Expr)?

WhileExpr <- WhilePrefix Expr (KEYWORD_else Payload? Expr)?

CurlySuffixExpr <- TypeExpr InitList?

InitList
    <- LBRACE FieldInit (COMMA FieldInit)* COMMA? RBRACE
     / LBRACE Expr (COMMA Expr)* COMMA? RBRACE
     / LBRACE RBRACE

TypeExpr <- PrefixTypeOp* ErrorUnionExpr

ErrorUnionExpr <- SuffixExpr (EXCLAMATIONMARK TypeExpr)?

SuffixExpr
    <- KEYWORD_async PrimaryTypeExpr SuffixOp* FnCallArguments
     / PrimaryTypeExpr (SuffixOp / FnCallArguments)*

PrimaryTypeExpr
    <- BUILTINIDENTIFIER FnCallArguments
     / CHAR_LITERAL
     / ContainerDecl
     / DOT IDENTIFIER
     / DOT InitList
     / ErrorSetDecl
     / FLOAT
     / FnProto
     / GroupedExpr
     / LabeledTypeExpr
     / IDENTIFIER
     / IfTypeExpr
     / INTEGER
     / KEYWORD_comptime TypeExpr
     / KEYWORD_error DOT IDENTIFIER
     / KEYWORD_anyframe
     / KEYWORD_unreachable
     / STRINGLITERAL
     / SwitchExpr

ContainerDecl <- (KEYWORD_extern / KEYWORD_packed)? ContainerDeclAuto

ErrorSetDecl <- KEYWORD_error LBRACE IdentifierList RBRACE

GroupedExpr <- LPAREN Expr RPAREN

IfTypeExpr <- IfPrefix TypeExpr (KEYWORD_else Payload? TypeExpr)?

LabeledTypeExpr
    <- BlockLabel Block
     / BlockLabel? LoopTypeExpr

LoopTypeExpr <- KEYWORD_inline? (ForTypeExpr / WhileTypeExpr)

ForTypeExpr <- ForPrefix TypeExpr (KEYWORD_else TypeExpr)?

WhileTypeExpr <- WhilePrefix TypeExpr (KEYWORD_else Payload? TypeExpr)?

SwitchExpr <- KEYWORD_switch LPAREN Expr RPAREN LBRACE SwitchProngList RBRACE

# *** Assembly ***
AsmExpr <- KEYWORD_asm KEYWORD_volatile? LPAREN Expr AsmOutput? RPAREN

AsmOutput <- COLON AsmOutputList AsmInput?

AsmOutputItem <- LBRACKET IDENTIFIER RBRACKET STRINGLITERAL LPAREN (MINUSRARROW TypeExpr / IDENTIFIER) RPAREN

AsmInput <- COLON AsmInputList AsmClobbers?

AsmInputItem <- LBRACKET IDENTIFIER RBRACKET STRINGLITERAL LPAREN Expr RPAREN

AsmClobbers <- COLON StringList

# *** Helper grammar ***
BreakLabel <- COLON IDENTIFIER

BlockLabel <- IDENTIFIER COLON

FieldInit <- DOT IDENTIFIER EQUAL Expr

WhileContinueExpr <- COLON LPAREN AssignExpr RPAREN

LinkSection <- KEYWORD_linksection LPAREN Expr RPAREN

AddrSpace <- KEYWORD_addrspace LPAREN Expr RPAREN

# Fn specific
CallConv <- KEYWORD_callconv LPAREN Expr RPAREN

ParamDecl
    <- doc_comment? (KEYWORD_noalias / KEYWORD_comptime)? (IDENTIFIER COLON)? ParamType
     / DOT3

ParamType
    <- KEYWORD_anytype
     / TypeExpr

# Control flow prefixes
IfPrefix <- KEYWORD_if LPAREN Expr RPAREN PtrPayload?

WhilePrefix <- KEYWORD_while LPAREN Expr RPAREN PtrPayload? WhileContinueExpr?

ForPrefix <- KEYWORD_for LPAREN ForArgumentsList RPAREN PtrListPayload

# Payloads
Payload <- PIPE IDENTIFIER PIPE

PtrPayload <- PIPE ASTERISK? IDENTIFIER PIPE

PtrIndexPayload <- PIPE ASTERISK? IDENTIFIER (COMMA IDENTIFIER)? PIPE

PtrListPayload <- PIPE ASTERISK? IDENTIFIER (COMMA ASTERISK? IDENTIFIER)* COMMA? PIPE

# Switch specific
SwitchProng <- KEYWORD_inline? SwitchCase EQUALRARROW PtrIndexPayload? SingleAssignExpr

SwitchCase
    <- SwitchItem (COMMA SwitchItem)* COMMA?
     / KEYWORD_else

SwitchItem <- Expr (DOT3 Expr)?

# For specific
ForArgumentsList <- ForItem (COMMA ForItem)* COMMA?

ForItem <- Expr (DOT2 Expr?)?

# Operators
AssignOp
    <- ASTERISKEQUAL
     / ASTERISKPIPEEQUAL
     / SLASHEQUAL
     / PERCENTEQUAL
     / PLUSEQUAL
     / PLUSPIPEEQUAL
     / MINUSEQUAL
     / MINUSPIPEEQUAL
     / LARROW2EQUAL
     / LARROW2PIPEEQUAL
     / RARROW2EQUAL
     / AMPERSANDEQUAL
     / CARETEQUAL
     / PIPEEQUAL
     / ASTERISKPERCENTEQUAL
     / PLUSPERCENTEQUAL
     / MINUSPERCENTEQUAL
     / EQUAL

CompareOp
    <- EQUALEQUAL
     / EXCLAMATIONMARKEQUAL
     / LARROW
     / RARROW
     / LARROWEQUAL
     / RARROWEQUAL

BitwiseOp
    <- AMPERSAND
     / CARET
     / PIPE
     / KEYWORD_orelse
     / KEYWORD_catch Payload?

BitShiftOp
    <- LARROW2
     / RARROW2
     / LARROW2PIPE

AdditionOp
    <- PLUS
     / MINUS
     / PLUS2
     / PLUSPERCENT
     / MINUSPERCENT
     / PLUSPIPE
     / MINUSPIPE

MultiplyOp
    <- PIPE2
     / ASTERISK
     / SLASH
     / PERCENT
     / ASTERISK2
     / ASTERISKPERCENT
     / ASTERISKPIPE

PrefixOp
    <- EXCLAMATIONMARK
     / MINUS
     / TILDE
     / MINUSPERCENT
     / AMPERSAND
     / KEYWORD_try
     / KEYWORD_await

PrefixTypeOp
    <- QUESTIONMARK
     / KEYWORD_anyframe MINUSRARROW
     / SliceTypeStart (ByteAlign / AddrSpace / KEYWORD_const / KEYWORD_volatile / KEYWORD_allowzero)*
     / PtrTypeStart (AddrSpace / KEYWORD_align LPAREN Expr (COLON Expr COLON Expr)? RPAREN / KEYWORD_const / KEYWORD_volatile / KEYWORD_allowzero)*
     / ArrayTypeStart

SuffixOp
    <- LBRACKET Expr (DOT2 (Expr? (COLON Expr)?)?)? RBRACKET
     / DOT IDENTIFIER
     / DOTASTERISK
     / DOTQUESTIONMARK

FnCallArguments <- LPAREN ExprList RPAREN

# Ptr specific
SliceTypeStart <- LBRACKET (COLON Expr)? RBRACKET

PtrTypeStart
    <- ASTERISK
     / ASTERISK2
     / LBRACKET ASTERISK (LETTERC / COLON Expr)? RBRACKET

ArrayTypeStart <- LBRACKET Expr (COLON Expr)? RBRACKET

# ContainerDecl specific
ContainerDeclAuto <- ContainerDeclType LBRACE container_doc_comment? ContainerMembers RBRACE

ContainerDeclType
    <- KEYWORD_struct (LPAREN Expr RPAREN)?
     / KEYWORD_opaque
     / KEYWORD_enum (LPAREN Expr RPAREN)?
     / KEYWORD_union (LPAREN (KEYWORD_enum (LPAREN Expr RPAREN)? / Expr) RPAREN)?

# Alignment
ByteAlign <- KEYWORD_align LPAREN Expr RPAREN

# Lists
IdentifierList <- (doc_comment? IDENTIFIER COMMA)* (doc_comment? IDENTIFIER)?

SwitchProngList <- (SwitchProng COMMA)* SwitchProng?

AsmOutputList <- (AsmOutputItem COMMA)* AsmOutputItem?

AsmInputList <- (AsmInputItem COMMA)* AsmInputItem?

StringList <- (STRINGLITERAL COMMA)* STRINGLITERAL?

ParamDeclList <- (ParamDecl COMMA)* ParamDecl?

ExprList <- (Expr COMMA)* Expr?

# *** Tokens ***
eof <- !.
bin <- [01]
bin_ <- '_'? bin
oct <- [0-7]
oct_ <- '_'? oct
hex <- [0-9a-fA-F]
hex_ <- '_'? hex
dec <- [0-9]
dec_ <- '_'? dec

bin_int <- bin bin_*
oct_int <- oct oct_*
dec_int <- dec dec_*
hex_int <- hex hex_*

ox80_oxBF <- [\200-\277]
oxF4 <- '\364'
ox80_ox8F <- [\200-\217]
oxF1_oxF3 <- [\361-\363]
oxF0 <- '\360'
ox90_0xBF <- [\220-\277]
oxEE_oxEF <- [\356-\357]
oxED <- '\355'
ox80_ox9F <- [\200-\237]
oxE1_oxEC <- [\341-\354]
oxE0 <- '\340'
oxA0_oxBF <- [\240-\277]
oxC2_oxDF <- [\302-\337]

# From https://lemire.me/blog/2018/05/09/how-quickly-can-you-check-that-a-string-is-valid-unicode-utf-8/
# First Byte      Second Byte     Third Byte      Fourth Byte
# [0x00,0x7F]
# [0xC2,0xDF]     [0x80,0xBF]
#    0xE0         [0xA0,0xBF]     [0x80,0xBF]
# [0xE1,0xEC]     [0x80,0xBF]     [0x80,0xBF]
#    0xED         [0x80,0x9F]     [0x80,0xBF]
# [0xEE,0xEF]     [0x80,0xBF]     [0x80,0xBF]
#    0xF0         [0x90,0xBF]     [0x80,0xBF]     [0x80,0xBF]
# [0xF1,0xF3]     [0x80,0xBF]     [0x80,0xBF]     [0x80,0xBF]
#    0xF4         [0x80,0x8F]     [0x80,0xBF]     [0x80,0xBF]

mb_utf8_literal <-
       oxF4      ox80_ox8F ox80_oxBF ox80_oxBF
     / oxF1_oxF3 ox80_oxBF ox80_oxBF ox80_oxBF
     / oxF0      ox90_0xBF ox80_oxBF ox80_oxBF
     / oxEE_oxEF ox80_oxBF ox80_oxBF
     / oxED      ox80_ox9F ox80_oxBF
     / oxE1_oxEC ox80_oxBF ox80_oxBF
     / oxE0      oxA0_oxBF ox80_oxBF
     / oxC2_oxDF ox80_oxBF

ascii_char_not_nl_slash_squote <- [\000-\011\013-\046\050-\133\135-\177]

char_escape
    <- "\\x" hex hex
     / "\\u{" hex+ "}"
     / "\\" [nr\\t'"]
char_char
    <- mb_utf8_literal
     / char_escape
     / ascii_char_not_nl_slash_squote

string_char
    <- char_escape
     / [^\\"\n]

container_doc_comment <- ('//!' [^\n]* [ \n]* skip)+
doc_comment <- ('///' [^\n]* [ \n]* skip)+
line_comment <- '//' ![!/][^\n]* / '////' [^\n]*
line_string <- ("\\\\" [^\n]* [ \n]*)+
skip <- ([ \n] / line_comment)*

CHAR_LITERAL <- "'" char_char "'" skip
FLOAT
    <- "0x" hex_int "." hex_int ([pP] [-+]? dec_int)? skip
     /      dec_int "." dec_int ([eE] [-+]? dec_int)? skip
     / "0x" hex_int [pP] [-+]? dec_int skip
     /      dec_int [eE] [-+]? dec_int skip
INTEGER
    <- "0b" bin_int skip
     / "0o" oct_int skip
     / "0x" hex_int skip
     /      dec_int   skip
STRINGLITERALSINGLE <- "\"" string_char* "\"" skip
STRINGLITERAL
    <- STRINGLITERALSINGLE
     / (line_string                 skip)+
IDENTIFIER
    <- !keyword [A-Za-z_] [A-Za-z0-9_]* skip
     / "@" STRINGLITERALSINGLE
BUILTINIDENTIFIER <- "@"[A-Za-z_][A-Za-z0-9_]* skip


AMPERSAND            <- '&'      ![=]      skip
AMPERSANDEQUAL       <- '&='               skip
ASTERISK             <- '*'      ![*%=|]   skip
ASTERISK2            <- '**'               skip
ASTERISKEQUAL        <- '*='               skip
ASTERISKPERCENT      <- '*%'     ![=]      skip
ASTERISKPERCENTEQUAL <- '*%='              skip
ASTERISKPIPE         <- '*|'     ![=]      skip
ASTERISKPIPEEQUAL    <- '*|='              skip
CARET                <- '^'      ![=]      skip
CARETEQUAL           <- '^='               skip
COLON                <- ':'                skip
COMMA                <- ','                skip
DOT                  <- '.'      ![*.?]    skip
DOT2                 <- '..'     ![.]      skip
DOT3                 <- '...'              skip
DOTASTERISK          <- '.*'               skip
DOTQUESTIONMARK      <- '.?'               skip
EQUAL                <- '='      ![>=]     skip
EQUALEQUAL           <- '=='               skip
EQUALRARROW          <- '=>'               skip
EXCLAMATIONMARK      <- '!'      ![=]      skip
EXCLAMATIONMARKEQUAL <- '!='               skip
LARROW               <- '<'      ![<=]     skip
LARROW2              <- '<<'     ![=|]     skip
LARROW2EQUAL         <- '<<='              skip
LARROW2PIPE          <- '<<|'    ![=]      skip
LARROW2PIPEEQUAL     <- '<<|='             skip
LARROWEQUAL          <- '<='               skip
LBRACE               <- '{'                skip
LBRACKET             <- '['                skip
LPAREN               <- '('                skip
MINUS                <- '-'      ![%=>|]   skip
MINUSEQUAL           <- '-='               skip
MINUSPERCENT         <- '-%'     ![=]      skip
MINUSPERCENTEQUAL    <- '-%='              skip
MINUSPIPE            <- '-|'     ![=]      skip
MINUSPIPEEQUAL       <- '-|='              skip
MINUSRARROW          <- '->'               skip
PERCENT              <- '%'      ![=]      skip
PERCENTEQUAL         <- '%='               skip
PIPE                 <- '|'      ![|=]     skip
PIPE2                <- '||'               skip
PIPEEQUAL            <- '|='               skip
PLUS                 <- '+'      ![%+=|]   skip
PLUS2                <- '++'               skip
PLUSEQUAL            <- '+='               skip
PLUSPERCENT          <- '+%'     ![=]      skip
PLUSPERCENTEQUAL     <- '+%='              skip
PLUSPIPE             <- '+|'     ![=]      skip
PLUSPIPEEQUAL        <- '+|='              skip
LETTERC              <- 'c'                skip
QUESTIONMARK         <- '?'                skip
RARROW               <- '>'      ![>=]     skip
RARROW2              <- '>>'     ![=]      skip
RARROW2EQUAL         <- '>>='              skip
RARROWEQUAL          <- '>='               skip
RBRACE               <- '}'                skip
RBRACKET             <- ']'                skip
RPAREN               <- ')'                skip
SEMICOLON            <- ';'                skip
SLASH                <- '/'      ![=]      skip
SLASHEQUAL           <- '/='               skip
TILDE                <- '~'                skip

end_of_word <- ![a-zA-Z0-9_] skip
KEYWORD_addrspace   <- 'addrspace'   end_of_word
KEYWORD_align       <- 'align'       end_of_word
KEYWORD_allowzero   <- 'allowzero'   end_of_word
KEYWORD_and         <- 'and'         end_of_word
KEYWORD_anyframe    <- 'anyframe'    end_of_word
KEYWORD_anytype     <- 'anytype'     end_of_word
KEYWORD_asm         <- 'asm'         end_of_word
KEYWORD_async       <- 'async'       end_of_word
KEYWORD_await       <- 'await'       end_of_word
KEYWORD_break       <- 'break'       end_of_word
KEYWORD_callconv    <- 'callconv'    end_of_word
KEYWORD_catch       <- 'catch'       end_of_word
KEYWORD_comptime    <- 'comptime'    end_of_word
KEYWORD_const       <- 'const'       end_of_word
KEYWORD_continue    <- 'continue'    end_of_word
KEYWORD_defer       <- 'defer'       end_of_word
KEYWORD_else        <- 'else'        end_of_word
KEYWORD_enum        <- 'enum'        end_of_word
KEYWORD_errdefer    <- 'errdefer'    end_of_word
KEYWORD_error       <- 'error'       end_of_word
KEYWORD_export      <- 'export'      end_of_word
KEYWORD_extern      <- 'extern'      end_of_word
KEYWORD_fn          <- 'fn'          end_of_word
KEYWORD_for         <- 'for'         end_of_word
KEYWORD_if          <- 'if'          end_of_word
KEYWORD_inline      <- 'inline'      end_of_word
KEYWORD_noalias     <- 'noalias'     end_of_word
KEYWORD_nosuspend   <- 'nosuspend'   end_of_word
KEYWORD_noinline    <- 'noinline'    end_of_word
KEYWORD_opaque      <- 'opaque'      end_of_word
KEYWORD_or          <- 'or'          end_of_word
KEYWORD_orelse      <- 'orelse'      end_of_word
KEYWORD_packed      <- 'packed'      end_of_word
KEYWORD_pub         <- 'pub'         end_of_word
KEYWORD_resume      <- 'resume'      end_of_word
KEYWORD_return      <- 'return'      end_of_word
KEYWORD_linksection <- 'linksection' end_of_word
KEYWORD_struct      <- 'struct'      end_of_word
KEYWORD_suspend     <- 'suspend'     end_of_word
KEYWORD_switch      <- 'switch'      end_of_word
KEYWORD_test        <- 'test'        end_of_word
KEYWORD_threadlocal <- 'threadlocal' end_of_word
KEYWORD_try         <- 'try'         end_of_word
KEYWORD_union       <- 'union'       end_of_word
KEYWORD_unreachable <- 'unreachable' end_of_word
KEYWORD_usingnamespace <- 'usingnamespace' end_of_word
KEYWORD_var         <- 'var'         end_of_word
KEYWORD_volatile    <- 'volatile'    end_of_word
KEYWORD_while       <- 'while'       end_of_word

keyword <- KEYWORD_addrspace / KEYWORD_align / KEYWORD_allowzero / KEYWORD_and
         / KEYWORD_anyframe / KEYWORD_anytype / KEYWORD_asm / KEYWORD_async
         / KEYWORD_await / KEYWORD_break / KEYWORD_callconv / KEYWORD_catch
         / KEYWORD_comptime / KEYWORD_const / KEYWORD_continue / KEYWORD_defer
         / KEYWORD_else / KEYWORD_enum / KEYWORD_errdefer / KEYWORD_error / KEYWORD_export
         / KEYWORD_extern / KEYWORD_fn / KEYWORD_for / KEYWORD_if
         / KEYWORD_inline / KEYWORD_noalias / KEYWORD_nosuspend / KEYWORD_noinline
         / KEYWORD_opaque / KEYWORD_or / KEYWORD_orelse / KEYWORD_packed
         / KEYWORD_pub / KEYWORD_resume / KEYWORD_return / KEYWORD_linksection
         / KEYWORD_struct / KEYWORD_suspend / KEYWORD_switch / KEYWORD_test
         / KEYWORD_threadlocal / KEYWORD_try / KEYWORD_union / KEYWORD_unreachable
         / KEYWORD_usingnamespace / KEYWORD_var / KEYWORD_volatile / KEYWORD_while
```


#### Zen

- Точно передавайте намерения.
- Крайние случаи имеют значение.
- Отдавайте предпочтение чтению кода, а не его написанию.
- Есть только один очевидный способ сделать что-то.
- Сбои во время выполнения лучше, чем ошибки.
- Ошибки компиляции лучше, чем сбои во время выполнения.
- Инкрементальные улучшения.
- Избегайте локальных максимумов.
- Сократите объем который необходимо запомнить.
- Сосредоточьтесь на коде, а не на стиле.
- При выделении ресурсов может возникнуть сбой; освобождение ресурсов должно быть успешным.
- Память - это ресурс.
- Вместе мы служим пользователям.
