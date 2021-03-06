## Рабочие пространства Cargo

В главе 12 мы создали пакет, включающий бинарный крейт и крейт библиотеки. По мере развития вашего проекта вы можете обнаружить, что крейт библиотеки продолжает увеличиваться и вы хотите разделить свой пакет ещё на несколько библиотечных крейтов. В этой ситуации Cargo предлагает функцию, называемую рабочими пространствами *workspace*, которая может помочь в управлении несколькими связанными пакетами, разработанными в тандеме.

### Создание рабочего пространства

*Рабочее пространство* является набором пакетов, которые совместно используют один и тот же файл *Cargo.lock* и папку для хранения конечных программных продуктов (будь то бинарные файлы или библиотеки). Давайте создадим проект, используя рабочее пространство, мы будем использовать простой код, чтобы сосредоточиться на структуре рабочего пространства. Есть несколько способов структурировать рабочее пространство; мы собираемся показать общий способ. У нас будет рабочее пространство, содержащее исполняемый файл и две библиотеки. Исполняемый файл, обеспечивающий основную функциональность, будет зависеть от двух библиотек. Одна библиотека будет предоставлять функцию `add_one`, а вторая функцию `add_two`. Эти три крейта будут частью одного рабочего пространства. Начнём с создания нового каталога для рабочей области:

```text
$ mkdir add
$ cd add
```

Затем в каталоге *add* мы создаём файл *Cargo.toml*, который настроит всю рабочую область. В этом файле не будет раздела `[package]` или метаданных, которые мы видели в других файлах *Cargo.toml*. Вместо этого он начнётся с раздела `[workspace]`, который позволит нам добавлять участников в рабочую область, указывая путь к нашему двоичному крейту; в этом случае этот путь *adder*:

<span class="filename">Файл: Cargo.toml</span>

```toml
[workspace]

members = [
    "adder",
]
```

Затем мы создадим исполняемый крейт `adder`, запустив команду `cargo new` в каталоге *add*:

```text
$ cargo new adder
     Created binary (application) `adder` project
```

На этом этапе мы можем создать рабочее пространство, запустив  команду `cargo build`. Файлы в каталоге *add* должны выглядеть следующим образом:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Рабочее пространство имеет одну целевую директорию *target* на верхнем уровне для размещения в ней скомпилированных артефактов; крейт `adder` не имеет своей собственной директории *target*. Даже если бы мы запускали `cargo build` из каталога *adder*, скомпилированные артефакты все равно оказались бы в *add/target*, а не в *add/adder/target*. Cargo структурирует каталог *target* в рабочем пространстве таким образом, потому что крейты в рабочем пространстве должны зависеть друг от друга. Если бы у каждого крейта был свой собственный каталог *target*, каждому крейту пришлось бы пере компилировать все другие крейты в рабочем пространстве, чтобы артефакты находились в собственных каталогах *target*. Используя один каталог *target*, крейты могут избежать ненужной сборки из исходных кодов.

### Добавление второго крейта в рабочее пространство

Теперь давайте создадим ещё один крейт в рабочем пространстве и назовём его `add-one`. Измените файл верхнего уровня *Cargo.toml*, чтобы указать путь к *add-one* в списке `members`:

<span class="filename">Файл: Cargo.toml</span>

```toml
[workspace]

members = [
    "adder",
    "add-one",
]
```

Затем сгенерируйте новый библиотечный крейт с именем `add-one` :

```text
$ cargo new add-one --lib
     Created library `add-one` project
```

Ваш каталог *add* должен теперь иметь следующие каталоги и файлы:

```text
├── Cargo.lock
├── Cargo.toml
├── add-one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Давайте в код библиотеки *add-one/src/lib.rs* добавим  функцию `add_one`:

<span class="filename">Файл: add-one/src/lib.rs</span>

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

Теперь, когда у нас есть крейт библиотеки в рабочей области, мы можем иметь исполняемый крейт `adder` зависящий от библиотечного крейта `add-one`. Сначала нам нужно добавить путь зависимости от `add-one` к *adder/Cargo.toml*.

<span class="filename">Файл: adder/Cargo.toml</span>

```toml
[dependencies]

add-one = { path = "../add-one" }
```

Cargo не предполагает, что крейты в рабочей области будут зависеть друг от друга, поэтому нам нужно чётко указать отношения зависимостей между крейтами.

Далее, давайте использовать функцию `add_one` из крейта `add-one` в крейте `adder`. Откройте файл *adder/src/main.rs* и добавьте строку `use` в верхней части, чтобы подключить новый библиотечный крейт `add-one` в область видимости. Затем измените `main` функцию для вызова функции `add_one` как в листинге 14-7.

<span class="filename">Файл: adder/src/main.rs</span>

```rust,ignore
use add_one;

fn main() {
    let num = 10;
    println!("Hello, world! {} plus one is {}!", num, add_one::add_one(num));
}
```

<span class="caption">Листинг 14-12: Использование функционала библиотечного крейта <code>add-one</code> в крейте <code>adder</code></span>

Давайте соберём рабочее пространство, запустив `cargo build` в верхнего уровня *add*!

```text
$ cargo build
   Compiling add-one v0.1.0 (file:///projects/add/add-one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.68 secs
```

Чтобы запустить бинарный крейт из каталога *add*, нам нужно указать какой пакет из рабочей области мы хотим использовать с помощью аргумента `-p` и названия пакета в команде `cargo run` :

```text
$ cargo run -p adder
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

Эта команда запускает код в *adder/src/main.rs*, который зависит от крейта `add-one`.

#### Зависимость от внешних крейтов в рабочем пространстве

Обратите внимание, что в рабочем пространстве на верхнем уровне есть только один файл *Cargo.lock*, нет наличия *Cargo.lock* в каталоге каждого крейта. Это гарантирует, что все крейты используют одну и ту же версию всех зависимостей. Если мы добавим крейт `rand` в *adder/Cargo.toml* и *add-one/Cargo.toml* файлы, Cargo разрешит оба из них в одну версию `rand` и запишет её в один *Cargo.lock*. Использованием всеми крейтами одинаковых зависимостей в рабочем пространстве означает, что крейты в рабочем пространстве всегда будут совместимы с друг с другом. Давайте добавим крейт `rand` в раздел `[dependencies]` в файле *add-one/Cargo.toml*, чтобы можно было использовать крейт `rand` в крейте `add-one`:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class="filename">Файл: add-one/Cargo.toml</span>

```toml
[dependencies]
rand = "0.5.5"
```

Теперь мы можем добавить `use rand;` в файл  *add-one/src/lib.rs* и сделать сборку рабочего пространства, запустив `cargo build` в каталоге *add*, что загрузит и скомпилирует `rand` крейт:

```text
$ cargo build
    Updating crates.io index
  Downloaded rand v0.5.5
   --snip--
   Compiling rand v0.5.5
   Compiling add-one v0.1.0 (file:///projects/add/add-one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 10.18 secs
```

Файл *Cargo.lock* верхнего уровня теперь содержит информацию о зависимости `add-one` к крейту `rand`. Тем не менее, не смотря на то что `rand` использован где-то в рабочем пространстве, мы не можем использовать его в других крейтах рабочего пространства, пока не добавим крейт `rand` в отдельные *Cargo.toml* файлы. Например, если мы добавим `use rand;` в файл *adder/src/main.rs* крейта  `adder`, то получим ошибку:

```text
$ cargo build
   Compiling adder v0.1.0 (file:///projects/add/adder)
error: use of unstable library feature 'rand': use `rand` from crates.io (see
issue #27703)
 --> adder/src/main.rs:1:1
  |
1 | use rand;
```

Чтобы исправить ошибку, отредактируйте файл *Cargo.toml* крейта `adder` и укажите, что `rand` является зависимостью и для этого крейта. Сборка крейта `adder` добавит крейт `rand` в список зависимостей крейта `adder` в его *Cargo.lock*, но нет будет загружать дополнительные копии `rand`. Cargo гарантирует, что каждый крейт рабочего пространства, использующий крейт `rand`, будет использовать одну и ту же версию. Использование одной и той же версии `rand` в рабочем пространстве экономит место, потому что мы не будем иметь несколько копий и есть гарантия, что крейты в рабочем пространстве будут совместимы друг с другом.

#### Добавление теста в рабочее пространство

Для дальнейшего улучшения проекта, добавим тест функцию `add_one::add_one` внутри крейта `add_one`:

<span class="filename">Файл: add-one/src/lib.rs</span>

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(3, add_one(2));
    }
}
```

Выполните команду `cargo test` в каталоге верхнего уровня *adder*:

```text
$ cargo test
   Compiling add-one v0.1.0 (file:///projects/add/add-one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27 secs
     Running target/debug/deps/add_one-f0253159197f7841

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/adder-f88af9d2cc175a5e

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests add-one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Первый раздел вывода показывает, что тест `it_works` в крейте `add-one` прошёл. В следующем разделе показано, что в крейте `adder` было найдено ноль тестов, а затем в последнем разделе показано, что в документации крейта `add-one` также было найдено ноль тестов. Выполнение команды `cargo test` в рабочем пространстве структурированном таким образом, будет запускать тесты для всех крейтов в рабочего пространства.

Мы также можем запустить тесты для одного конкретного крейта в рабочем пространстве из каталог верхнего уровня с помощью флага `-p` и указанием имени крейта для которого мы хотим запустить тесты:

```text
$ cargo test -p add-one
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/add_one-b3235fea9a156f74

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests add-one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Эти выходные данные показывают, что выполнение `cargo test` запускает только тесты для крейта `add-one` и не запускает тесты крейта `adder`.

Если вы публикуете крейты из такого рабочего пространства на [crates.io](https://crates.io/), то каждый крейт должен быть опубликован отдельно. Команда `cargo publish` не имеет флага `--all` или `-p`, поэтому вы должны перейти в каталог каждого крейта и запустить `cargo publish` в каждом крейте рабочего пространства для его публикации.

Для дополнительной практики добавьте крейт `add-two` в данное рабочее пространство аналогичным способом, как делали с крейт `add-one` !

По мере роста вашего проекта рассмотрите возможность использования рабочего пространства, так как легче понимать более маленькие, отдельные компоненты, чем один большой кусок кода. Кроме того, сохраняя крейты в рабочем пространстве можно облегчить координацию между ними, если они часто меняются одновременно.
