# Programación de un juego de adivinanzas

¡Saltemos a Rust trabajando juntos en un proyecto práctico! Este capítulo le presenta algunos conceptos comunes de Rust mostrándole cómo usarlos en un programa real. ¡Aprenderá sobre `let` , `match` , métodos, funciones asociadas, uso de cajas externas y más! Los siguientes capítulos explorarán estas ideas con más detalle. En este capítulo, practicarás los fundamentos.

Implementaremos un problema clásico de programación para principiantes: un juego de adivinanzas. Así es como funciona: el programa generará un número entero aleatorio entre 1 y 100. Luego le pedirá al jugador que ingrese una suposición. Después de ingresar una conjetura, el programa indicará si la conjetura es demasiado baja o demasiado alta. Si la suposición es correcta, el juego imprimirá un mensaje de felicitación y saldrá.

## Configurar un nuevo proyecto

Para configurar un nuevo proyecto, vaya al directorio de *proyectos* que creó en el Capítulo 1 y cree un nuevo proyecto usando Cargo, así:

```text
$ cargo new guessing_game
$ cd guessing_game
```

El primer comando, `cargo new` , toma el nombre del proyecto (juego de `guessing_game` ) como primer argumento. El segundo comando cambia al directorio del nuevo proyecto.

Mire el archivo *Cargo.toml* generado:

<span class="filename">Nombre de archivo: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
edition = "2018"

[dependencies]
```

Si la información del autor que Cargo obtuvo de su entorno no es correcta, corríjala en el archivo y guárdelo nuevamente.

Como viste en el Capítulo 1, la `cargo new` genera un "¡Hola, mundo!" programa para usted. Consulte el archivo *src / main.rs* :

<span class="filename">Nombre de archivo: src / main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Ahora compilemos este "¡Hola, mundo!" programe y ejecútelo en el mismo paso usando el comando `cargo run` :

```text
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 1.50 secs
     Running `target/debug/guessing_game`
Hello, world!
```

El comando de `run` es útil cuando necesita iterar rápidamente en un proyecto, como lo haremos en este juego, probando rápidamente cada iteración antes de pasar a la siguiente.

Reopen the *src/main.rs* file. You’ll be writing all the code in this file.

## Procesando una suposición

La primera parte del programa del juego de adivinanzas solicitará la entrada del usuario, procesará esa entrada y comprobará que la entrada esté en la forma esperada. Para comenzar, permitiremos que el jugador ingrese una suposición. Ingrese el código del Listado 2-1 en *src / main.rs.*

<span class="filename">Nombre de archivo: src / main.rs</span>

```rust,ignore
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```

<span class="caption">Listado 2-1: Código que obtiene una suposición del usuario y lo imprime</span>

Este código contiene mucha información, así que vamos a repasarlo línea por línea. Para obtener la entrada del usuario y luego imprimir el resultado como salida, necesitamos traer la biblioteca `io` (entrada / salida) al alcance. La biblioteca `io` proviene de la biblioteca estándar (que se conoce como `std` ):

```rust,ignore
use std::io;
```

De forma predeterminada, Rust incluye solo unos pocos tipos en el alcance de cada programa en [el *preludio*](../std/prelude/index.html)<!-- ignorar --> . Si un tipo que desea usar no está en el preludio, debe traer ese tipo al alcance explícitamente con una declaración de `use` . El uso de la biblioteca `std::io` proporciona una serie de funciones útiles, incluida la capacidad de aceptar la entrada del usuario.

Como vio en el Capítulo 1, la función `main` es el punto de entrada al programa:

```rust,ignore
fn main() {
```

La sintaxis `fn` declara una nueva función, los paréntesis, `()` , indican que no hay parámetros, y el corchete, `{` , inicia el cuerpo de la función.

Como también aprendió en el Capítulo 1, `println!` es una macro que imprime una cadena en la pantalla:

```rust,ignore
println!("Guess the number!");

println!("Please input your guess.");
```

Este código imprime un mensaje que indica qué es el juego y solicita información del usuario.

### Almacenamiento de valores con variables

A continuación, crearemos un lugar para almacenar la entrada del usuario, como este:

```rust,ignore
let mut guess = String::new();
```

¡Ahora el programa se está poniendo interesante! Están sucediendo muchas cosas en esta pequeña línea. Tenga en cuenta que esta es una declaración `let` , que se usa para crear una *variable* . Aquí hay otro ejemplo:

```rust,ignore
let foo = bar;
```

Esta línea crea una nueva variable llamada `foo` y la une al valor de la variable `bar` . En Rust, las variables son inmutables por defecto. Discutiremos este concepto en detalle en la [sección "Variables y mutabilidad".](ch03-01-variables-and-mutability.html#variables-and-mutability)<!-- ignorar --> en el Capítulo 3. El siguiente ejemplo muestra cómo usar `mut` antes del nombre de la variable para hacer una variable mutable:

```rust,ignore
let foo = 5; // immutable
let mut bar = 5; // mutable
```

> Nota: La sintaxis `//` inicia un comentario que continúa hasta el final de la línea. Rust ignora todo en los comentarios, que se analizan con más detalle en el Capítulo 3.

Let’s return to the guessing game program. You now know that `let mut guess` will introduce a mutable variable named `guess`. On the other side of the equal sign (`=`) is the value that `guess` is bound to, which is the result of calling `String::new`, a function that returns a new instance of a `String`. <a href="../std/string/struct.String.html" data-md-type="link">`String`</a><!-- ignore --> is a string type provided by the standard library that is a growable, UTF-8 encoded bit of text.

La `::` sintaxis en la `::new` línea indica que `new` es una *función asociada* del tipo `String` . Una función asociada se implementa en un tipo, en este caso `String` , en lugar de en una instancia particular de `String` . Algunos lenguajes llaman a esto un *método estático* .

Esta `new` función crea una nueva cadena vacía. Encontrará una `new` función en muchos tipos, porque es un nombre común para una función que genera un nuevo valor de algún tipo.

Para resumir, `let mut guess = String::new();` line ha creado una variable mutable que actualmente está vinculada a una nueva instancia vacía de `String` . ¡Uf!

Recuerde que incluimos la funcionalidad de entrada / salida de la biblioteca estándar con `use std::io;` en la primera línea del programa. Ahora llamaremos a la función `stdin` desde el módulo `io` :

```rust,ignore
io::stdin().read_line(&mut guess)
    .expect("Failed to read line");
```

Si no hubiéramos puesto la línea `use std::io` al principio del programa, podríamos haber escrito esta llamada de función como `std::io::stdin` . La función `stdin` devuelve una instancia de [`std::io::Stdin`](../std/io/struct.Stdin.html)<!-- ignorar --> , que es un tipo que representa un identificador de la entrada estándar para su terminal.

La siguiente parte del código, `.read_line(&mut guess)` , llama a [`read_line`](../std/io/struct.Stdin.html#method.read_line)<!-- ignorar --> en el identificador de entrada estándar para obtener la entrada del usuario. También estamos pasando un argumento a `read_line` : `&mut guess` .

The job of `read_line` is to take whatever the user types into standard input and place that into a string, so it takes that string as an argument. The string argument needs to be mutable so the method can change the string’s content by adding the user input.

El `&` indica que este argumento es una *referencia* , lo que le brinda una forma de permitir que varias partes de su código accedan a un dato sin necesidad de copiar esos datos en la memoria varias veces. Las referencias son una característica compleja y una de las principales ventajas de Rust es lo seguro y fácil que es utilizar las referencias. No necesita conocer muchos de esos detalles para terminar este programa. Por ahora, todo lo que necesita saber es que, al igual que las variables, las referencias son inmutables de forma predeterminada. Por lo tanto, debe escribir `&mut guess` lugar de `&guess` para que sea mutable. (El Capítulo 4 explicará las referencias con más detalle).

### Manejo de fallas potenciales con el tipo de `Result`

Aún no hemos terminado con esta línea de código. Aunque lo que hemos discutido hasta ahora es una sola línea de texto, es solo la primera parte de la única línea lógica de código. La segunda parte es este método:

```rust,ignore
.expect("Failed to read line");
```

Cuando llama a un método con la sintaxis `.foo()` , a menudo es aconsejable introducir una nueva línea y otros espacios en blanco para ayudar a dividir las líneas largas. Podríamos haber escrito este código como:

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

Sin embargo, una línea larga es difícil de leer, por lo que es mejor dividirla: dos líneas para dos llamadas a métodos. Ahora analicemos lo que hace esta línea.

Como se mencionó anteriormente, `read_line` pone lo que el usuario escribe en la cadena que le estamos pasando, pero también devuelve un valor, en este caso, un [`io::Result`](../std/io/type.Result.html)<!-- ignorar --> . Rust tiene varios tipos denominados `Result` en su biblioteca estándar: un <a href="../std/result/enum.Result.html" data-md-type="link">`Result`</a> genérico<!-- ignorar --> así como versiones específicas para submódulos, como `io::Result` .

Los tipos de `Result` son [*enumeraciones*](ch06-00-enums.html)<!-- ignorar --> , a menudo denominado *enumeraciones* . Una enumeración es un tipo que puede tener un conjunto fijo de valores, y esos valores se denominan *variantes de* la enumeración. El capítulo 6 cubrirá las enumeraciones con más detalle.

Para `Result` , las variantes son `Ok` o `Err` . La variante `Ok` indica que la operación fue exitosa, y dentro de `Ok` está el valor generado exitosamente. La variante `Err` significa que la operación falló y `Err` contiene información sobre cómo o por qué falló la operación.

El propósito de estos tipos de `Result` es codificar la información de manejo de errores. Los valores del tipo de `Result` , como los valores de cualquier tipo, tienen métodos definidos. Una instancia de `io::Result` tiene un [método de `expect`](../std/result/enum.Result.html#method.expect)<!-- ignorar --> que puedes llamar. Si esta instancia de `io::Result` es un valor `Err` , `expect` hará que el programa se bloquee y muestre el mensaje que pasó como argumento `expect` . Si el método `read_line` devuelve `Err` , probablemente sea el resultado de un error proveniente del sistema operativo subyacente. Si esta instancia de `io::Result` es un valor `Ok` , `expect` tomará el valor de retorno que `Ok` tiene y le devolverá solo ese valor para que pueda usarlo. En este caso, ese valor es el número de bytes que el usuario ingresó en la entrada estándar.

Si no llama a `expect` , el programa se compilará, pero recibirá una advertencia:

```text
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
warning: unused `std::result::Result` which must be used
  --> src/main.rs:10:5
   |
10 |     io::stdin().read_line(&mut guess);
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
   = note: #[warn(unused_must_use)] on by default
```

Rust advierte que no ha utilizado el valor de `Result` devuelto por `read_line` , lo que indica que el programa no ha manejado un posible error.

La forma correcta de suprimir la advertencia es escribir realmente el manejo de errores, pero debido a que solo desea bloquear este programa cuando ocurre un problema, puede usar `expect` . Aprenderá a recuperarse de errores en el Capítulo 9.

### Impresión de valores con `println!` Marcadores de posición

Aparte de las llaves de cierre, solo hay una línea más para discutir en el código agregado hasta ahora, que es la siguiente:

```rust,ignore
println!("You guessed: {}", guess);
```

Esta línea imprime la cadena en la que guardamos la entrada del usuario. El conjunto de llaves, `{}` , es un marcador de posición: piense en `{}` como pequeñas pinzas de cangrejo que mantienen un valor en su lugar. Puede imprimir más de un valor utilizando corchetes: el primer conjunto de corchetes contiene el primer valor enumerado después de la cadena de formato, el segundo conjunto contiene el segundo valor, y así sucesivamente. Imprimir varios valores en una llamada a `println!` se vería así:

```rust
let x = 5;
let y = 10;

println!("x = {} and y = {}", x, y);
```

Este código imprimiría `x = 5 and y = 10` .

### Probando la primera parte

Probemos la primera parte del juego de adivinanzas. Ejecútelo usando `cargo run` :

```text
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53 secs
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

En este punto, la primera parte del juego está lista: recibimos información del teclado y luego la imprimimos.

## Generando un número secreto

A continuación, necesitamos generar un número secreto que el usuario intentará adivinar. El número secreto debe ser diferente cada vez para que el juego sea divertido de jugar más de una vez. Usemos un número aleatorio entre 1 y 100 para que el juego no sea demasiado difícil. Rust aún no incluye la funcionalidad de números aleatorios en su biblioteca estándar. Sin embargo, el equipo de Rust proporciona una [caja de `rand`](https://crates.io/crates/rand) .

### Usar una caja para obtener más funcionalidad

Recuerde que una caja es una colección de archivos de código fuente de Rust. El proyecto que hemos estado construyendo es una *caja binaria* , que es un ejecutable. La caja `rand` es una *caja de biblioteca* , que contiene código destinado a ser utilizado en otros programas.

El uso de cajas externas por parte de Cargo es donde realmente brilla. Antes de que podamos escribir código que use `rand` , necesitamos modificar el archivo *Cargo.toml* para incluir la caja `rand` como dependencia. Abra ese archivo ahora y agregue la siguiente línea en la parte inferior debajo del encabezado de la sección `[dependencies]` que Cargo creó para usted:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">Nombre de archivo: Cargo.toml</span>

```toml
[dependencies]
rand = "0.5.5"
```

En el archivo *Cargo.toml* , todo lo que sigue a un encabezado es parte de una sección que continúa hasta que comienza otra sección. La sección `[dependencies]` es donde le dice a Cargo de qué cajas externas depende su proyecto y qué versiones de esas cajas necesita. En este caso, especificaremos la caja `rand` con el especificador de versión semántica `0.5.5` . Cargo entiende el [control de versiones semántico](http://semver.org)<!-- ignorar --> (a veces llamado *SemVer* ), que es un estándar para escribir números de versión. El número `0.5.5` es en realidad una abreviatura de `^0.5.5` , que significa "cualquier versión que tenga una API pública compatible con la versión 0.5.5".

Ahora, sin cambiar nada del código, construyamos el proyecto, como se muestra en el Listado 2-2.

```text
$ cargo build
    Updating crates.io index
  Downloaded rand v0.5.5
  Downloaded libc v0.2.62
  Downloaded rand_core v0.2.2
  Downloaded rand_core v0.3.1
  Downloaded rand_core v0.4.2
   Compiling rand_core v0.4.2
   Compiling libc v0.2.62
   Compiling rand_core v0.3.1
   Compiling rand_core v0.2.2
   Compiling rand v0.5.5
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53 s
```

<span class="caption">Listado 2-2: El resultado de ejecutar <code>cargo build</code> después de agregar la caja rand como dependencia</span>

Es posible que vea diferentes números de versión (¡pero todos serán compatibles con el código, gracias a SemVer!), Y las líneas pueden estar en un orden diferente.

Ahora que tenemos una dependencia externa, Cargo obtiene las últimas versiones de todo del *registro* , que es una copia de los datos de [Crates.io](https://crates.io/) . Crates.io es donde las personas en el ecosistema de Rust publican sus proyectos de código abierto de Rust para que otros los usen.

Después de actualizar el registro, Cargo revisa la sección `[dependencies]` y descarga las cajas que aún no tiene. En este caso, aunque solo enumeramos `rand` como una dependencia, Cargo también tomó `libc` y `rand_core` , porque `rand` depende de que funcionen. Después de descargar las cajas, Rust las compila y luego compila el proyecto con las dependencias disponibles.

Si vuelve a ejecutar inmediatamente la `cargo build` sin realizar ningún cambio, no obtendrá ningún resultado aparte de la línea de `Finished` . Cargo sabe que ya ha descargado y compilado las dependencias, y no ha cambiado nada sobre ellas en su archivo *Cargo.toml* . Cargo también sabe que no ha cambiado nada sobre su código, por lo que tampoco lo vuelve a compilar. Sin nada que hacer, simplemente sale.

Si abre el archivo *src / main.rs* , realiza un cambio trivial y luego lo guarda y compila nuevamente, solo verá dos líneas de salida:

```text
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53s
```

Estas líneas muestran que Cargo solo actualiza la compilación con su pequeño cambio en el archivo *src / main.rs.* Sus dependencias no han cambiado, por lo que Cargo sabe que puede reutilizar lo que ya ha descargado y compilado para esas. Simplemente reconstruye tu parte del código.

#### Garantizar compilaciones reproducibles con el archivo *Cargo.lock*

Cargo tiene un mecanismo que garantiza que pueda reconstruir el mismo artefacto cada vez que usted o cualquier otra persona compile su código: Cargo usará solo las versiones de las dependencias que especificó hasta que indique lo contrario. Por ejemplo, ¿qué sucede si la próxima semana la versión 0.5.6 de la caja `rand` sale y contiene una corrección de error importante pero también contiene una regresión que romperá su código?

The answer to this problem is the *Cargo.lock* file, which was created the first time you ran `cargo build` and is now in your *guessing_game* directory. When you build a project for the first time, Cargo figures out all the versions of the dependencies that fit the criteria and then writes them to the *Cargo.lock* file. When you build your project in the future, Cargo will see that the *Cargo.lock* file exists and use the versions specified there rather than doing all the work of figuring out versions again. This lets you have a reproducible build automatically. In other words, your project will remain at `0.5.5` until you explicitly upgrade, thanks to the *Cargo.lock* file.

#### Actualizar una caja para obtener una nueva versión

Cuando *usted* desea actualizar una caja, Transporte proporciona otro comando, `update` , que ignorará el archivo *Cargo.lock* y averiguar todas las últimas versiones que se ajusten a sus especificaciones en *Cargo.toml.* Si eso funciona, Cargo escribirá esas versiones en el archivo *Cargo.lock* .

Pero de forma predeterminada, Cargo solo buscará versiones superiores a `0.5.5` y menores a `0.6.0` . Si la caja `rand` ha lanzado dos nuevas versiones, `0.5.6` y `0.6.0` , vería lo siguiente si ejecutara `cargo update` :

```text
$ cargo update
    Updating crates.io index
    Updating rand v0.5.5 -> v0.5.6
```

En este punto, también notarás un cambio en tu archivo *Cargo.lock y notarás* que la versión de la caja `rand` que estás usando ahora es `0.5.6` .

Si quisiera utilizar la versión `0.6.0` `rand` o cualquier versión de la serie `0.6.x` , tendría que actualizar el archivo *Cargo.toml* para que se vea así:

```toml
[dependencies]
rand = "0.6.0"
```

La próxima vez que ejecute `cargo build` , Cargo actualizará el registro de cajas disponibles y reevaluará sus requisitos de `rand` acuerdo con la nueva versión que haya especificado.

Hay mucho más que decir sobre [Cargo](http://doc.crates.io)<!-- ignorar --> y [su ecosistema](http://doc.crates.io/crates-io.html)<!-- ignorar --> que discutiremos en el Capítulo 14, pero por ahora, eso es todo lo que necesita saber. Cargo facilita la reutilización de bibliotecas, por lo que los rustáceos pueden escribir proyectos más pequeños que se ensamblan a partir de varios paquetes.

### Generando un número aleatorio

Ahora que ha agregado la caja `rand` a *Cargo.toml* , comencemos a usar `rand` . El siguiente paso es actualizar *src / main.rs* , como se muestra en el Listado 2-3.

<span class="filename">Nombre de archivo: src / main.rs</span>

```rust,ignore
use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```

<span class="caption">Listado 2-3: Agregar código para generar un número aleatorio</span>

En primer lugar, añadir una `use` línea: `use rand::Rng` . El rasgo `Rng` define los métodos que implementan los generadores de números aleatorios, y este rasgo debe estar dentro del alcance para que podamos usar esos métodos. El capítulo 10 cubrirá los rasgos en detalle.

A continuación, agregamos dos líneas en el medio. La función `rand::thread_rng` nos dará el generador de números aleatorios particular que vamos a usar: uno que sea local al hilo de ejecución actual y sembrado por el sistema operativo. Luego llamamos al método `gen_range` en el generador de números aleatorios. Este método está definido por el rasgo `Rng` que trajimos al alcance con la declaración `use rand::Rng` . El método `gen_range` toma dos números como argumentos y genera un número aleatorio entre ellos. Es inclusivo en el límite inferior pero exclusivo en el límite superior, por lo que debemos especificar `1` y `101` para solicitar un número entre 1 y 100.

> Note: You won’t just know which traits to use and which methods and functions to call from a crate. Instructions for using a crate are in each crate’s documentation. Another neat feature of Cargo is that you can run the `cargo doc --open` command, which will build documentation provided by all of your dependencies locally and open it in your browser. If you’re interested in other functionality in the `rand` crate, for example, run `cargo doc --open` and click `rand` in the sidebar on the left.

La segunda línea que agregamos a la mitad del código imprime el número secreto. Esto es útil mientras estamos desarrollando el programa para poder probarlo, pero lo eliminaremos de la versión final. ¡No es un gran juego si el programa imprime la respuesta tan pronto como comienza!

Intente ejecutar el programa varias veces:

```text
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53 secs
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4
$ cargo run
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

Debería obtener diferentes números aleatorios, y todos deberían ser números entre 1 y 100. ¡Excelente trabajo!

## Comparando la conjetura con el número secreto

Ahora que tenemos la entrada del usuario y un número aleatorio, podemos compararlos. Ese paso se muestra en el Listado 2-4. Tenga en cuenta que este código no se compilará todavía, como explicaremos.

<span class="filename">Nombre de archivo: src / main.rs</span>

```rust,ignore,does_not_compile
use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {

    // ---snip---

    println!("You guessed: {}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```

<span class="caption">Listado 2-4: Manejo de los posibles valores de retorno de comparar dos números</span>

El primer bit nuevo aquí es otra declaración de `use` , que trae un tipo llamado `std::cmp::Ordering` al alcance de la biblioteca estándar. Al igual que `Result` , `Ordering` es otra enumeración, pero las variantes de `Ordering` son `Less` , `Greater` e `Equal` . Estos son los tres resultados que son posibles cuando compara dos valores.

Luego agregamos cinco líneas nuevas en la parte inferior que usan el tipo de `Ordering` . El método `cmp` compara dos valores y se puede llamar en cualquier cosa que se pueda comparar. Toma una referencia a lo que quieras comparar: aquí se compara la `guess` con el `secret_number` . Luego, devuelve una variante de la enumeración `Ordering` incluimos en el alcance con la declaración de `use` . Usamos un [`match`](ch06-02-match.html)<!-- ignorar --> expresión para decidir qué hacer a continuación en función de qué variante de `Ordering` se devolvió desde la llamada a `cmp` con los valores en `guess` y `secret_number` .

Una expresión de `match` está formada por *brazos* . Un brazo consta de un *patrón* y el código que se debe ejecutar si el valor dado al comienzo de la expresión de `match` ajusta al patrón de ese brazo. Rust toma el valor dado para `match` y mira a través del patrón de cada brazo por turno. La construcción y los patrones de `match` son características poderosas en Rust que le permiten expresar una variedad de situaciones que su código puede encontrar y asegurarse de manejarlas todas. Estas características se cubrirán en detalle en el Capítulo 6 y el Capítulo 18, respectivamente.

Veamos un ejemplo de lo que sucedería con la expresión de `match` que se usa aquí. Digamos que el usuario ha adivinado 50 y que el número secreto generado aleatoriamente esta vez es 38. Cuando el código compara 50 con 38, el método `cmp` devolverá `Ordering::Greater` , porque 50 es mayor que 38. La expresión de `match` obtiene el `Ordering::Greater` valor y comienza a comprobar el patrón de cada brazo. Mira el patrón del primer brazo, `Ordering::Less` , y ve que el valor `Ordering::Greater` no coincide con `Ordering::Less` , por lo que ignora el código en ese brazo y pasa al siguiente brazo. El siguiente patrón de la mano, `Ordering::Greater` , *hace* partido `Ordering::Greater` ! El código asociado en ese brazo se ejecutará e imprimirá ¡ `Too big!` a la pantalla. La expresión de `match` finaliza porque no es necesario mirar el último brazo en este escenario.

Sin embargo, el código del Listado 2-4 aún no se compilará. Vamos a intentarlo:

```text
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
error[E0308]: mismatched types
  --> src/main.rs:23:21
   |
23 |     match guess.cmp(&secret_number) {
   |                     ^^^^^^^^^^^^^^ expected struct `std::string::String`, found integer
   |
   = note: expected type `&std::string::String`
   = note:    found type `&{integer}`

error: aborting due to previous error
Could not compile `guessing_game`.
```

The core of the error states that there are *mismatched types*. Rust has a strong, static type system. However, it also has type inference. When we wrote `let mut guess = String::new()`, Rust was able to infer that `guess` should be a `String` and didn’t make us write the type. The `secret_number`, on the other hand, is a number type. A few number types can have a value between 1 and 100: `i32`, a 32-bit number; `u32`, an unsigned 32-bit number; `i64`, a 64-bit number; as well as others. Rust defaults to an `i32`, which is the type of `secret_number` unless you add type information elsewhere that would cause Rust to infer a different numerical type. The reason for the error is that Rust cannot compare a string and a number type.

En última instancia, queremos convertir la `String` el programa lee como entrada en un tipo de número real para poder compararlo numéricamente con el número secreto. Podemos hacerlo agregando las siguientes dos líneas al cuerpo de la función `main` :

<span class="filename">Nombre de archivo: src / main.rs</span>

```rust,ignore
// --snip--

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    let guess: u32 = guess.trim().parse()
        .expect("Please type a number!");

    println!("You guessed: {}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```

Las dos nuevas líneas son:

```rust,ignore
let guess: u32 = guess.trim().parse()
    .expect("Please type a number!");
```

Creamos una variable llamada `guess` . Pero espera, ¿el programa no tiene ya una variable llamada `guess` ? Lo hace, pero Rust nos permite *sombrear* el valor anterior de `guess` con uno nuevo. Esta función se utiliza a menudo en situaciones en las que desea convertir un valor de un tipo a otro. El sombreado nos permite reutilizar el nombre de la variable de `guess` lugar de obligarnos a crear dos variables únicas, como `guess_str` y `guess` por ejemplo. (El Capítulo 3 cubre el sombreado con más detalle).

We bind `guess` to the expression `guess.trim().parse()`. The `guess` in the expression refers to the original `guess` that was a `String` with the input in it. The `trim` method on a `String` instance will eliminate any whitespace at the beginning and end. Although `u32` can contain only numerical characters, the user must press <span class="keystroke">enter</span> to satisfy `read_line`. When the user presses <span class="keystroke">enter</span>, a newline character is added to the string. For example, if the user types <span class="keystroke">5</span> and presses <span class="keystroke">enter</span>, `guess` looks like this: `5\n`. The `\n` represents “newline,” the result of pressing <span class="keystroke">enter</span>. The `trim` method eliminates `\n`, resulting in just `5`.

The [`parse` method on strings](../std/primitive.str.html#method.parse)<!-- ignore --> parses a string into some kind of number. Because this method can parse a variety of number types, we need to tell Rust the exact number type we want by using `let guess: u32`. The colon (`:`) after `guess` tells Rust we’ll annotate the variable’s type. Rust has a few built-in number types; the `u32` seen here is an unsigned, 32-bit integer. It’s a good default choice for a small positive number. You’ll learn about other number types in Chapter 3. Additionally, the `u32` annotation in this example program and the comparison with `secret_number` means that Rust will infer that `secret_number` should be a `u32` as well. So now the comparison will be between two values of the same type!

The call to `parse` could easily cause an error. If, for example, the string contained `A👍%`, there would be no way to convert that to a number. Because it might fail, the `parse` method returns a `Result` type, much as the `read_line` method does (discussed earlier in <a href="#handling-potential-failure-with-the-result-type" data-md-type="link">“Handling Potential Failure with the `Result` Type”</a><!-- ignore -->). We’ll treat this `Result` the same way by using the `expect` method again. If `parse` returns an `Err` `Result` variant because it couldn’t create a number from the string, the `expect` call will crash the game and print the message we give it. If `parse` can successfully convert the string to a number, it will return the `Ok` variant of `Result`, and `expect` will return the number that we want from the `Ok` value.

¡Ejecutemos el programa ahora!

```text
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43 secs
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

¡Agradable! Aunque se agregaron espacios antes de la conjetura, el programa aún se dio cuenta de que el usuario adivinó 76. Ejecute el programa varias veces para verificar el comportamiento diferente con diferentes tipos de entrada: adivine el número correctamente, adivine un número que sea demasiado alto, y adivina un número demasiado bajo.

Tenemos la mayor parte del juego funcionando ahora, pero el usuario solo puede hacer una suposición. ¡Cambiemos eso agregando un bucle!

## Permitir múltiples suposiciones con bucle

La palabra clave `loop` crea un ciclo infinito. Agregaremos eso ahora para brindar a los usuarios más oportunidades de adivinar el número:

<span class="filename">Nombre de archivo: src / main.rs</span>

```rust,ignore
// --snip--

    println!("The secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");

        // --snip--

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => println!("You win!"),
        }
    }
}
```

Como puede ver, hemos movido todo en un ciclo desde el indicador de entrada de adivinanzas en adelante. Asegúrese de sangrar las líneas dentro del bucle otros cuatro espacios cada una y ejecute el programa nuevamente. Observe que hay un nuevo problema porque el programa está haciendo exactamente lo que le dijimos que hiciera: ¡solicite otra conjetura para siempre! ¡No parece que el usuario pueda salir!

El usuario siempre puede interrumpir el programa usando el atajo de teclado <span class="keystroke">ctrl-c</span> . Pero hay otra forma de escapar de este monstruo insaciable, como se menciona en la discusión de `parse` en ["Comparación de la suposición con el número secreto"](#comparing-the-guess-to-the-secret-number)<!-- ignorar --> : si el usuario ingresa una respuesta no numérica, el programa se bloqueará. El usuario puede aprovechar eso para salir, como se muestra aquí:

```text
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 1.50 secs
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit
thread 'main' panicked at 'Please type a number!: ParseIntError { kind: InvalidDigit }', src/libcore/result.rs:785
note: Run with `RUST_BACKTRACE=1` for a backtrace.
error: Process didn't exit successfully: `target/debug/guess` (exit code: 101)
```

Escribir `quit` realidad cierra el juego, pero también lo hará cualquier otra entrada que no sea numérica. Sin embargo, esto es subóptimo por decir lo menos. Queremos que el juego se detenga automáticamente cuando se adivine el número correcto.

### Dejar de fumar después de una suposición correcta

Programemos el juego para que se cierre cuando el usuario gane agregando una declaración de `break` :

<span class="filename">Nombre de archivo: src / main.rs</span>

```rust,ignore
// --snip--

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

¡Añadiendo la línea de `break` después de `You win!` hace que el programa salga del ciclo cuando el usuario adivina el número secreto correctamente. Salir del bucle también significa salir del programa, porque el bucle es la última parte de `main` .

### Manejo de entrada no válida

Para refinar aún más el comportamiento del juego, en lugar de bloquear el programa cuando el usuario ingresa un no número, hagamos que el juego ignore un no número para que el usuario pueda seguir adivinando. Podemos hacer eso alterando la línea donde la `guess` se convierte de `String` a `u32` , como se muestra en el Listado 2-5.

<span class="filename">Nombre de archivo: src / main.rs</span>

```rust,ignore
// --snip--

io::stdin().read_line(&mut guess)
    .expect("Failed to read line");

let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};

println!("You guessed: {}", guess);

// --snip--
```

<span class="caption">Listado 2-5: Ignorar una suposición que no sea un número y pedir otra suposición en lugar de bloquear el programa</span>

Cambiar de una llamada de `expect` a una expresión de `match` es la forma en que generalmente se pasa de fallar en un error a manejarlo. Recuerde que `parse` devuelve un tipo de `Result` y el `Result` es una enumeración que tiene las variantes `Ok` o `Err` . Estamos usando una expresión de `match` aquí, como hicimos con el resultado de `Ordering` del método `cmp` .

Si el `parse` puede convertir correctamente la cadena en un número, devolverá un valor `Ok` que contiene el número resultante. Ese valor `Ok` coincidirá con el patrón del primer brazo, y la expresión de `match` solo devolverá el valor `num` que produjo el `parse` y lo colocará dentro del valor `Ok` . Ese número terminará justo donde lo queremos en la nueva variable de `guess` que estamos creando.

Si `parse` *no* puede convertir la cadena en un número, devolverá un valor `Err` que contiene más información sobre el error. El valor `Err` no coincide con el patrón `Ok(num)` en el primer brazo de `match` , pero sí coincide con el patrón `Err(_)` en el segundo brazo. El guión bajo, `_` , es un valor general; en este ejemplo, estamos diciendo que queremos hacer coincidir todos los valores `Err` , sin importar la información que tengan dentro. Entonces, el programa ejecutará el código del segundo brazo, `continue` , lo que le dice al programa que vaya a la siguiente iteración del `loop` y pregunte por otra conjetura. Entonces, efectivamente, ¡el programa ignora todos los errores que el `parse` puede encontrar!

Ahora todo en el programa debería funcionar como se esperaba. Vamos a intentarlo:

```text
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

Awesome! With one tiny final tweak, we will finish the guessing game. Recall that the program is still printing the secret number. That worked well for testing, but it ruins the game. Let’s delete the `println!` that outputs the secret number. Listing 2-6 shows the final code.

<span class="filename">Nombre de archivo: src / main.rs</span>

```rust,ignore
use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

<span class="caption">Listado 2-6: Código completo del juego de adivinanzas</span>

## Resumen

En este punto, ha construido con éxito el juego de adivinanzas. ¡Felicidades!

Este proyecto fue una forma práctica de presentarle muchos conceptos nuevos de Rust: `let` , `match` , métodos, funciones asociadas, el uso de cajas externas y más. En los próximos capítulos, aprenderá sobre estos conceptos con más detalle. El Capítulo 3 cubre conceptos que tienen la mayoría de los lenguajes de programación, como variables, tipos de datos y funciones, y muestra cómo usarlos en Rust. El Capítulo 4 explora la propiedad, una característica que hace que Rust sea diferente de otros idiomas. El Capítulo 5 analiza las estructuras y la sintaxis de los métodos, y el Capítulo 6 explica cómo funcionan las enumeraciones.
