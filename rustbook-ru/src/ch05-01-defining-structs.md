## Определение и инициализация структур

Внешне структуры похожи на кортежи рассмотренные в главе 3. Также как кортежи, структуры могут содержать разные типы данных. Но в отличии от кортежей, вы именуете части данных так, чтобы было ясно что эти имена означают. Поэтому структуры более удобны для создания новых типов данных, так как нет необходимости запоминать порядковый номер какого-либо значения внутри экземпляра структуры.

Для определения структуры, указывается ключевое слово `struct` и её название. Название должно описывать значение частей данных сгруппированных месте. Далее, в фигурных скобках через запятую определяются имена и типы частей данных. Каждый элемент называется *поле*. Листинг 5-1, описывает структуру для хранения информации о учётной записи пользователя:

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

<span class="caption">Листинг 5-1: определение структуры <code>User</code></span>

После определения структуры можно создавать её *экземпляр*, назначая определённое значение каждому полю с соответствующим типом данных. Экземпляр создаётся, указывая имя структуры, затем добавляем фигурные скобки включающие пары ключ/значение (`key: value`), где ключами являются имена полей и значениями являются данные, которые мы хотим сохранить в поля. Нет необходимости чётко следовать порядку объявления полей в описании структуры (но всё-таки желательно, для удобства чтения). Другими словами, объявление структуры - это вроде общего шаблона для нашего типа, а экземпляр структуры заполняет данный шаблон определёнными данными для создания значений нашего типа. Например, можно объявить пользователя как в листинге 5-2:

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
```

<span class="caption">Листинг 5-2: создание экземпляра структуры <code>User</code></span>

Чтобы получить определённое значение поля структуры, мы можем использовать точечную нотацию (как в кортеже). Если нужен только электронный адрес, можно использовать `user1.email` везде где нужно его значение. Если экземпляр структуры изменяемый, то для изменения значения данных одного поля структуры, мы присваиваем ему новое значение используя точечную нотацию. Листинг 5-3 показывает как изменить значение в поле `email` изменяемого экземпляра `User`:

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

user1.email = String::from("anotheremail@example.com");
```

<span class="caption">Листинг 5-3: изменение значения поля <code>email</code> экземпляра структуры <code>User</code></span>

Заметим, что весь экземпляр структуры должен быть изменяемым; Rust не позволяет помечать изменяемыми отдельные поля. Как в любом выражением, можно создавать новый экземпляр структуры в качестве последнего выражения тела функции для явного возврата нового экземпляра.

На листинге 5-4 функция `build_user` возвращает экземпляр `User` с указанным адресом и именем. Поле `active` получает значение `true`, а поле `sign_in_count` получает значение `1`.

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}
```

<span class="caption">Листинг 5-4: функция <code>build_user</code> принимает электронный адрес, имя и возвращает экземпляр <code>User</code></span>

Имеет смысл называть имена параметров функции также как названия полей структуры, но повторение полей `email` и `username` переменных не удобно. Если структура имеет много полей, то повторение станет более раздражающим. К счастью есть удобное сокращение!

### Использование сокращения инициализации полей, когда переменные и имена  совпадают

Так как имена параметров и полей структуры являются полностью идентичными в листинге 5-4, можно использовать синтаксис *сокращения инициализации поля*, чтобы переписать `build_user` так, чтобы он работал точно также, но не содержал повторений для `email` и `username`, как в листинге 5-5.

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

<span class="caption">Листинг 5-5: функция <code>build_user</code> использует сокращение инициализации полей, потому что параметры <code>email</code> и <code>username</code> имеют имена как поля структуры</span>

Здесь происходит создание нового экземпляра структуры  `User`, которая имеет поле с именем `email`. Мы хотим установить  поле `email` в значение параметра `email` функции `build_user`. Так как поле `email` и параметр `email` имеют одинаковое название, нужно только писать `email` вместо кода `email: email`.

### Создание экземпляра структуры из экземпляра другой структуры с помощью синтаксиса обновления структуры

Часто является полезным создание нового экземпляра структуры, который использует значения старого экземпляра, но что-то меняет в нем. Это делается с помощью *синтаксиса обновления структур*.

Сначала листинг 5-6 показывает как создать новый экземпляр  `User` для переменной `user2` без синтаксиса обновления. Устанавливаются значения `email` и `username`, но используются те же значения из переменной `user1`, как сделано в листинге 5-2.

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
# let user1 = User {
#     email: String::from("someone@example.com"),
#     username: String::from("someusername123"),
#     active: true,
#     sign_in_count: 1,
# };
#
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    active: user1.active,
    sign_in_count: user1.sign_in_count,
};
```

<span class="caption">Листинг 5-6: создание экземпляра <code>User</code> присвоением  полям значений из <code>user1</code></span>

Используя синтаксис обновления структуры, можно получить тот же эффект, используя меньше кода как показано в листинге 5-7. Синтаксис `..` указывает, что оставшиеся поля устанавливаются не явно и должны иметь значения из указанного экземпляра.

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
# let user1 = User {
#     email: String::from("someone@example.com"),
#     username: String::from("someusername123"),
#     active: true,
#     sign_in_count: 1,
# };
#
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1
};
```

<span class="caption">Листинг 5-7: использование синтаксиса обновления структур для установки значений <code>email</code> и <code>username</code> экземпляра <code>User</code>, но использование остальных значений из полей экземпляра переменной <code>user1</code></span>

Код в листинге Listing 5-7 также создаёт экземпляр переменной  `user2`, который имеет другое значение поля `email` и `username`, но те же значение для полей `active` и `sign_in_count` что у переменной `user1`.

### Использование кортеж структур без именованных полей для создания разных типов

Можно определять структуры с помощью сокращённой записи, очень напоминающей кортежи (такое определение называют *кортежными структурами*).  Кортежная структура имеет дополнительное значение из-за имени в объявлении, но не имеет имён ассоциированных с её полями, у них есть только типы. Кортежные  структуры удобны, когда нужно дать всему кортежу имя и сделать его отличным от других кортежей, при этом именование полей как у обычной структуры будет подробным или избыточным.

Определение кортежной структуры начинается ключевым словом `struct`, названием структуры за которым следуют типы в кортеже. Например, вот определение и использование двух кортежных структур с именами `Color` и `Point`:

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

Обратите внимание, что переменные `black` и `origin` разного типа, потому что они являются экземплярами разных кортежных структур.  Каждая определяемая структура является собственным типом, не смотря на то, что поля внутри структуры имеют одинаковые типы. Например, функция принимающая параметром тип `Color` не может принять аргумент типа `Point`, не смотря на то, что оба типа состоят из трёх значений `i32`. Тем не менее, экземпляры кортежных структур ведут себя как кортежи: их можно разделять на отдельные части, использовать `.` за которой идёт индекс для доступа к отдельному значению и т.д.

### Единичные структуры без полей

Можно также определять структуры без полей! Они называются  *unit-like, единично-подобные структуры* потому что ведут себя подобно единичному типу `()`. Единично-подобные структуры могут быть полезны в ситуации, в которой нужно реализовать типаж некоторого типа, но нет никаких данных для сохранения в самом типе. Мы обсудим типажи в главе 10.

> ### Владение данными структуры
> При определении структуры `User` листинга 5-1 мы использовали тип `String` владеющий данными вместо `&str`. Это было осознанное решение, т.к. мы хотели, чтобы экземпляры структур владели всеми своими данными и чтобы данные были действительными во время всего существования структуры.
> Возможно сделать так, чтобы структуры сохраняли ссылки на данные которыми владеет кто-то другой, но это требует использования *времён жизни*, функциональности Rust о которой мы поговорим в главе 10. Времена жизни гарантируют, что данные на которые ссылается структура, действительны столько же времени, сколько действительна сама структура. Допустим, вы пробуете сохранить ссылку в структуре без указания времени жизни, вот так, но это не будет работать:
> <span class="filename">Файл: src/main.rs</span>
> ```rust,ignore,does_not_compile
> struct User {
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
>     active: bool,
> }
>
> fn main() {
>     let user1 = User {
>         email: "someone@example.com",
>         username: "someusername123",
>         active: true,
>         sign_in_count: 1,
>     };
> }
> ```
> Компилятор будет жаловаться на необходимость определения времени жизни:
> ```text
> error[E0106]: missing lifetime specifier
>  -->
>   |
> 2 |     username: &str,
>   |               ^ expected lifetime parameter
>
> error[E0106]: missing lifetime specifier
>  -->
>   |
> 3 |     email: &str,
>   |            ^ expected lifetime parameter
> ```
> В главе 10 мы обсудим, как исправить такие ошибки сохранения ссылок в структурах, но сейчас мы исправим подобные ошибки, используя типы со владение данными, такими как `String` вместо типа ссылок, таких как `&str`.
