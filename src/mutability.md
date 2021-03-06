% Изменяемость (mutability)

Изменяемость, то есть возможность изменить что-то, работает в Rust несколько
иначе, чем в других языках. Во-первых, по умолчанию связанные имена не
изменяемы:

```rust,ignore
let x = 5;
x = 6; // ошибка!
```

Изменяемость можно добавить с помощью ключевого слова `mut`:

```rust
let mut x = 5;

x = 6; // нет проблем!
```

Это изменяемое [связанное имя][vb]. Когда связанное имя изменяемо, это означает,
что мы можем поменять связанное с ним значение. В примере выше не то, чтобы само
значение `x` менялось, просто имя `x` связывается с другим значением типа `i32`.

[vb]: variable-bindings.html

Если же вы хотите изменить само связанное значение, вам понадобится
[изменяемая ссылка][mr]:

```rust
let mut x = 5;
let y = &mut x;
```

[mr]: references-and-borrowing.html

`y` — это неизменяемое имя для изменяемой ссылки. Это значит, что `y` нельзя
связать ещё с чем-то (`y = &mut z`), но можно изменить то, на что указывает
связанная ссылка (`*y = 5`). Тонкая разница.

Конечно, вы можете объявить и изменяемое имя для изменяемой ссылки:

```rust
let mut x = 5;
let mut y = &mut x;
```

Теперь `y` можно связать с другим значением, и само это значение тоже можно
менять.

Стоит отметить, что `mut` — это часть [шаблона][pattern], поэтому можно делать
такие вещи:

```rust
let (mut x, y) = (5, 6);

fn foo(mut x: i32) {
# }
```

[pattern]: patterns.html

# Внутренняя (interior) и внешняя (exterior) изменяемость

Однако, когда мы говорим, что что-либо «неизменяемо» в Rust, это не означает,
что оно совсем не может измениться. Мы говорим о «внешней изменяемости». Для
примера рассмотрим [`Arc<T>`][arc]:

```rust
use std::sync::Arc;

let x = Arc::new(5);
let y = x.clone();
```

[arc]: http://doc.rust-lang.org/std/sync/struct.Arc.html

Когда мы вызываем метод `clone()`, `Arc<T>` должна обновить счётчик ссылок. Мы
не использовали модификатор `mut`, а значит `x` — неизменяемое имя. Мы не можем
получить ссылку (`&mut 5`) или сделать что-то подобное. И что же?

Для того чтобы понять это, мы должны вернуться назад к основам философии Rust,
к сохранности памяти и механизму, гарантирующему это, к системе
[владения][ownership], и, в частности, к [заимствованию][borrowing]:

> Одновременно у вас может быть только один из двух перечисленных ниже видов
> заимствования, но не оба сразу:
>
> * одна или более неизменяемых ссылок (`&T`) на ресурс,
> * ровно одна изменяемая ссылка (`&mut T`) на ресурс.

[ownership]: ownership.html
[borrowing]: references-and-borrowing.html#borrowing

Итак, что же здесь на самом деле является «неизменяемым»? Безопасно ли иметь два
указателя на один объект? В случае с `Arc<T>`, да: изменяемый объект полностью
находится внутри самой структуры. По этой причине, метод `clone()` возвращает
неизменяемую ссылку (`&T`). Если бы он возвращал изменяемую ссылку (`&mut T`),
то у нас были бы проблемы. Таким образом, `let mut z = Arc::new(5);` объявляет
атомарный счётчик ссылок с внешней изменяемостью.

Другие типы, например те, что определены в модуле [`std::cell`][stdcell],
напротив, имеют «внутреннюю изменяемость». Например:

```rust
use std::cell::RefCell;

let x = RefCell::new(42);

let y = x.borrow_mut();
```

[stdcell]: http://doc.rust-lang.org/std/cell/index.html

RefCell возвращает изменяемую ссылку `&mut` при помощи метода `borrow_mut()`. А
не опасно ли это? Что, если мы сделаем так:

```rust,ignore
use std::cell::RefCell;

let x = RefCell::new(42);

let y = x.borrow_mut();
let z = x.borrow_mut();
# (y, z);
```

Это приведёт к панике во время исполнения. Вот что делает `RefCell`: он
принудительно выполняет проверку правил заимствования во время исполнения и
вызывает `panic!`, если они были нарушены.

Стоит отметить, что тип изменяемости — внутренняя или внешняя — определяется
самим типом. Нет способа волшебно превратить значение с внутренней изменяемостью
в значение со внешней, и наоборот.

Всё это подводит нас к другим аспектам правил изменяемости Rust. Давайте
поговорим о них.

## Изменяемость на уровне полей

Изменяемость — это свойство либо ссылки (`&mut`), либо имени (`let mut`). Это
значит, что, например, у вас не может быть [структуры][struct], часть полей
которой изменяется, а другая часть — нет:

```rust,ignore
struct Point {
    x: i32,
    mut y: i32, // нельзя
}
```

Изменяемость структуры определяется при её связывании:

```rust,ignore
struct Point {
    x: i32,
    y: i32,
}

let mut a = Point { x: 5, y: 6 };

a.x = 10;

let b = Point { x: 5, y: 6};

b.x = 10; // error: cannot assign to immutable field `b.x`
```

[struct]: structs.html

Однако, используя [`Cell<T>`][cell], вы можете эмулировать изменяемость на
уровне полей:

```rust
use std::cell::Cell;

struct Point {
    x: i32,
    y: Cell<i32>,
}

let point = Point { x: 5, y: Cell::new(6) };

point.y.set(7);

println!("y: {:?}", point.y);
```

[cell]: http://doc.rust-lang.org/std/cell/struct.Cell.html

Это выведет на экран `y: Cell { value: 7 }`. Мы успешно изменили значение `y`.
