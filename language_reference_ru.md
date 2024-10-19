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

Часто самый эффективный способ узнать что-то новое - это посмотреть примеры, поэтому в этой документации показано, как
использовать каждую из функций **Zig**. Все это находится на одной странице, так что вы можете выполнять поиск с помощью
инструмента поиска вашего браузера.

Примеры кода в этом документе скомпилированы и протестированы в рамках основного набора тестов **Zig**.

Этот HTML-документ не зависит от внешних файлов, поэтому вы можете использовать его в автономном режиме.

### Стандартная библиотека **Zig**

Стандартная библиотека **Zig** имеет собственную документацию.

Стандартная библиотека **Zig** содержит часто используемые алгоритмы, структуры данных и определения, которые помогут вам
создавать программы или библиотеки. В этой документации вы увидите множество примеров стандартной библиотеки **Zig**,
используемых в этой документации. Чтобы узнать больше о стандартной библиотеке **Zig**, перейдите по ссылке выше.

### Hello World

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


В большинстве случаев более целесообразно выполнять запись в stderr, а не в стандартный вывод, и не имеет значения,
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


### Комментарии

**Zig** поддерживает 3 типа комментариев. Обычные комментарии игнорируются, но компилятор использует комментарии doc и
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

В **Zig** нет многострочных комментариев (например, таких как /* */ comments в C). Это позволяет Zig обладать свойством,
позволяющим выделять каждую строку кода из контекста.


#### Комментарии к документу

Комментарий к документу doc начинается ровно с трех косых черт (т.е. ///, но не с ////); несколько комментариев к
документу doc подряд объединяются в многострочный комментарий к документу doc. Комментарий к документу документирует
все, что следует непосредственно за ним.

```zig
/// A structure for storing a timestamp, with nanosecond precision (this is a
/// multiline doc comment).
const Timestamp = struct {
    /// The number of seconds since the epoch (this is also a doc comment).
    seconds: i64, // signed so we can represent pre-1970 (not a doc comment)
    /// The number of nanoseconds past the second (doc comment again).
    nanos: u32,

    /// Returns a `Timestamp` struct representing the Unix epoch; that is, the
    /// moment of 1970 Jan 1 00:00:00 UTC (this is a doc comment too).
    pub fn unixEpoch() Timestamp {
        return Timestamp{
            .seconds = 0,
            .nanos = 0,
        };
    }
};
```

Комментарии Doc разрешены только в определенных местах; наличие комментария doc в неожиданном месте, например, в
середине выражения или непосредственно перед комментарием, не относящимся к doc, является ошибкой компиляции.

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

Комментарии в формате Doc могут чередоваться с обычными комментариями. В настоящее время при создании документации по
вашему вниманию общие рекомендации, которые объединяются с рекомендациями в документе doc.

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

### Значения

```zig
// Top-level declarations are order-independent:
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

**<p style="text-align:center">Primitive Types</p>**
|Type|C Equivalent|Description
|---|---|---|
| <span style="color:blue">`i8`</span>| `int8_t` | signed 8-bit integer
| <span style="color:blue">`u8`</span>| `uint8_t` | unsigned 8-bit integer
| <span style="color:blue">`i16`</span>| `int16_t` | signed 16-bit integer |
| <span style="color:blue">`u16`</span>| `uint16_t` | unsigned 16-bit integer
| <span style="color:blue">`i32`</span>| `int32_t` | signed 32-bit integer
| <span style="color:blue">`u32`</span>| `uint32_t` | unsigned 32-bit integer
| <span style="color:blue">`i64`</span>| `int64_t` | signed 64-bit integer
| <span style="color:blue">`u64`</span>| `uint64_t` | unsigned 64-bit integer
| <span style="color:blue">`i128`</span>| `__int128` | signed 128-bit integer
| <span style="color:blue">`u128`</span>| `unsigned __int128` | unsigned 128-bit integer
| <span style="color:blue">`isize`</span>| `intptr_t` | signed pointer sized integer
| <span style="color:blue">`usize`</span>| `uintptr_t`, `size_t` | unsigned pointer sized integer. Also see #5185
| <span style="color:blue">`c_char`</span>| `char` | for ABI compatibility with C
| <span style="color:blue">`c_short`</span>| `short` | for ABI compatibility with C
| <span style="color:blue">`c_ushort`</span>| `unsigned short` | for ABI compatibility with C
| <span style="color:blue">`c_int`</span> | `int` | for ABI compatibility with C
| <span style="color:blue">`c_uint`</span>| `unsigned int` | for ABI compatibility with C
| <span style="color:blue">`c_long`</span> | `long` | for ABI compatibility with C
| <span style="color:blue">`c_ulong`</span>| `unsigned long` | for ABI compatibility with C
| <span style="color:blue">`c_longlong`</span>| `long long` | for ABI compatibility with C
| <span style="color:blue">`c_ulonglong`</span>| `unsigned long long` | for ABI compatibility with C
| <span style="color:blue">`c_longdouble`</span>| `long double` | for ABI compatibility with C
| <span style="color:blue">`f16`</span>| `_Float16` | 16-bit floating point (10-bit mantissa) IEEE-754-2008 binary16
| <span style="color:blue">`f32`</span>| `float` | 32-bit floating point (23-bit mantissa) IEEE-754-2008 binary32
| <span style="color:blue">`f64`</span>| `double` | 64-bit floating point (52-bit mantissa) IEEE-754-2008 binary64
| <span style="color:blue">`f80`</span>| `double` | 80-bit floating point (64-bit mantissa) IEEE-754-2008 80-bit extended precision
| <span style="color:blue">`f128`</span>| `_Float128` | 128-bit floating point (112-bit mantissa) IEEE-754-2008 binary128
| <span style="color:blue">`bool`</span>| `bool` | true or false
| <span style="color:blue">`anytype`</span>| `void` | Used for type-erased pointers.
| <span style="color:blue">`void`</span>| `(none)` |  Always the value void{}
| <span style="color:blue">`noreturn`</span>| `(none)` | the type of break, continue, return, unreachable, and while (true) {}
| <span style="color:blue">`type`</span>| `(none)` | the type of types
| <span style="color:blue">`anyerror`</span>| `(none)` | an error code
| <span style="color:blue">`comptime_int`</span>| `(none)` | Only allowed for comptime-known values. The type of integer literals.
| <span style="color:blue">`comptime_float`</span>| `(none)` | Only allowed for comptime-known values. The type of float literals.

В дополнение к целочисленным типам, указанным выше, можно использовать ссылки на целые числа произвольной разрядности,
используя идентификатор i или u, за которым следуют цифры. Например, идентификатор i7 относится к 7-разрядному целому
числу со знаком. Максимально допустимая разрядность для целочисленного типа равна 65535.

**<p style="text-align:center">Primitive Values</p>**
|Name|Description|
|---|---|
| <span style="color:red">`true`</span> and <span style="color:red">`false`</span> | <span style="color:blue">`bool`</span> values |
| <span style="color:red">`null`</span> | used to set an optional type to <span style="color:red">`null`</span> |
| <span style="color:red">`undefined`</span> | used to leave a value unspecified |


#### Строковые литералы и литералы с кодовой точкой в Юникоде

Строковые литералы - это константные одноэлементные указатели на массивы байтов, заканчивающиеся нулем. Тип строковых
литералов кодирует как длину, так и тот факт, что они заканчиваются нулем, и, таким образом, они могут быть
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

**<p style="text-align:center">Escape Sequences</p>**
|Escape Sequence|Name|
|---|---|
|`\n` |Newline
|`\r` | Carriage Return
|`\t` | Tab
|`\\` | Backslash
|`\'` | Single Quote
|`\"` | Double Quote
|`\xNN` | hexadecimal 7-bit byte value (2 digits)
|`\u{NNNNNN}` | hexadecimal Unicode code point UTF-8 encoded (1 or more digits)

Note that the maximum valid Unicode point is <span style="color:red">`0x10ffff`</span>.

#### Многострочные строковые литералы

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
    // It works at file scope as well as inside functions.
    const y = 5678;

    // Once assigned, an identifier cannot be changed.
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
значение `const` применяется ко всем байтам к которым непосредственно обращается идентификатор. Указатели имеют свою
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

Используйте <span style="color:red">`undefined`</span> чтобы оставить переменные неинициализированными:
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

Значение <span style="color:red">`undefined`</span> может быть приведено к любому типу. Как только это произойдет, уже
невозможно будет определить, что значение не определено. Значение <span style="color:red">`undefined`</span> может быть
любым, даже бессмысленным в соответствии с типом. В переводе на английский <span style="color:red">`undefined`</span>
означает "Не имеющее смысла значение". Использование этого значения может привести к ошибке. Значение будет
неиспользуемым или перезаписанным перед использованием.

В режиме отладки **Zig** записывает <span style="color:red">`0xaa`</span> байт в неопределенную память. Это делается для
раннего выявления ошибок и помогает обнаружить использование неопределенной памяти в отладчике. Однако такое поведение
является только особенностью реализации, а не семантикой языка, поэтому не гарантируется, что оно будет наблюдаться в
коде.
