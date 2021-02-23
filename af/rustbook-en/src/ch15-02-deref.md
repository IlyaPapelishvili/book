## Treating Smart Pointers Like Regular References with the `Deref` Trait

Implementing the `Deref` trait allows you to customize the behavior of the *dereference operator*, `*` (as opposed to the multiplication or glob operator). By implementing `Deref` in such a way that a smart pointer can be treated like a regular reference, you can write code that operates on references and use that code with smart pointers too.

Kom ons kyk eers hoe die operasie met verskillende verwysings met gereelde verwysings werk. Dan sal ons probeer om 'n aangepaste tipe te definieer wat soos `Box<T>` , en kyk waarom die operasie vir verskille nie soos 'n verwysing op ons pas gedefinieerde tipe werk nie. Ons ondersoek hoe die implementering van die `Deref` eienskap dit vir slim aanwysers moontlik maak om op maniere soortgelyk aan verwysings te werk. Dan gaan ons kyk na Rust se *dwangfunksie* en hoe dit ons kan gebruik met verwysings of slim aanwysings.

> Note: there’s one big difference between the `MyBox<T>` type we’re about to build and the real `Box<T>`: our version will not store its data on the heap. We are focusing this example on `Deref`, so where the data is actually stored is less important than the pointer-like behavior.

### Na aanleiding van die aanwyser na die waarde met die storingoperateur

'N Gereelde verwysing is 'n tipe aanwyser, en een manier om aan 'n wyser te dink, is as 'n pyl na 'n waarde wat êrens anders gestoor word. In lys 15-6 skep ons 'n verwysing na 'n `i32` waarde en gebruik dan die operasie om die verwysing na die data te volg:

<span class="filename">Lêernaam: src / main.rs</span>

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

<span class="caption">Listing 15-6: Gebruik die operator van verskillende verwysings om 'n verwysing na 'n <code>i32</code> waarde te volg</span>

Die veranderlike `x` het 'n `i32` waarde, `5` . Ons stel `y` gelyk aan 'n verwysing na `x` . Ons kan beweer dat `x` gelyk is aan `5` . As ons egter 'n bewering wil maak oor die waarde in `y` , moet ons `*y` gebruik om die verwysing na die waarde waarna dit wys te volg (dus *verskil* ). As ons eenmaal `y` , het ons toegang tot die heelgetal waarop `y` kan wys dat ons met `5` kan vergelyk.

If we tried to write `assert_eq!(5, y);` instead, we would get this compilation error:

```text
error[E0277]: can't compare `{integer}` with `&{integer}`
 --> src/main.rs:6:5
  |
6 |     assert_eq!(5, y);
  |     ^^^^^^^^^^^^^^^^^ no implementation for `{integer} == &{integer}`
  |
  = help: the trait `std::cmp::PartialEq<&{integer}>` is not implemented for
  `{integer}`
```

Dit is nie toegelaat om 'n getal en 'n verwysing na 'n getal te vergelyk nie omdat dit verskillende soorte is. Ons moet die afleidingsoperateur gebruik om die verwysing na die waarde waarna dit verwys, te volg.

### Gebruik `Box<T>` Soos 'n verwysing

Ons kan die kode in lys 15-6 herskryf om 'n `Box<T>` in plaas van 'n verwysing te gebruik; die afleidingsoperateur sal werk soos aangedui in Listing 15-7:

<span class="filename">Lêernaam: src / main.rs</span>

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

<span class="caption">Lys 15-7: gebruik die afleidingsoperateur op 'n <code>Box&lt;i32&gt;</code></span>

Die enigste verskil tussen Listing 15-7 en Listing 15-6 is dat ons hier `y` instel as 'n voorbeeld van 'n blokkie wat na die waarde in `x` eerder as 'n verwysing wat na die waarde van `x` . In die laaste bewering, kan ons die dereference operateur gebruik om wyser die boks se volg in dieselfde manier as wat ons gedoen het toe `y` was 'n verwysing. Vervolgens ondersoek ons wat spesiaal is aan `Box<T>` wat ons in staat stel om die afleidingsoperateur te gebruik deur ons eie vaksoort te definieer.

### Definieer ons eie slimwyser

Kom ons bou 'n slim aanwyser soortgelyk aan die tipe `Box<T>` wat deur die standaardbiblioteek aangebied word, om te sien hoe slim wysers anders optree as verwysings. Dan sal ons kyk hoe ons die moontlikheid kan toevoeg om die verskille-operateur te gebruik.

Die `Box<T>` -tipe word uiteindelik gedefinieer as 'n tupelstructuur met een element, dus in Listing 15-8 word 'n `MyBox<T>` op dieselfde manier gedefinieër. Ons definieer ook 'n `new` funksie wat ooreenstem met die `new` funksie wat in `Box<T>` gedefinieër is.

<span class="filename">Lêernaam: src / main.rs</span>

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
```

<span class="caption">Lys 15-8: Definiëring van 'n <code>MyBox&lt;T&gt;</code> tipe</span>

We define a struct named `MyBox` and declare a generic parameter `T`, because we want our type to hold values of any type. The `MyBox` type is a tuple struct with one element of type `T`. The `MyBox::new` function takes one parameter of type `T` and returns a `MyBox` instance that holds the value passed in.

Kom ons probeer die toevoeging van die `main` funksie in Listing 15-7 te Listing 15-8 en om dit te verander om te gebruik die `MyBox<T>` tik ons het gedefinieer in plaas van `Box<T>` . Die kode in aanbieding 15-9 kan nie saamgestel word nie, want Rust weet nie hoe om MyBox te `MyBox` .

<span class="filename">Lêernaam: src / main.rs</span>

```rust,ignore,does_not_compile
fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

<span class="caption">Listing 15-9: Probeer <code>MyBox&lt;T&gt;</code> op dieselfde manier as wat ons verwysings gebruik het en <code>Box&lt;T&gt;</code></span>

Hier is die gevolglike samestellingsfout:

```text
error[E0614]: type `MyBox<{integer}>` cannot be dereferenced
  --> src/main.rs:14:19
   |
14 |     assert_eq!(5, *y);
   |                   ^^
```

Ons `MyBox<T>` -tipe kan nie herverwys word nie, omdat ons nie die vermoë op ons tipe geïmplementeer het nie. Om die verwysing met die `*` operateur moontlik te maak, implementeer ons die `Deref` eienskap.

### Die behandeling van 'n tipe soos 'n verwysing deur die uitvoering van die `Deref` eienskap

Soos in hoofstuk 10 bespreek, moet ons, om 'n eienskap te implementeer, implementerings verskaf vir die vereiste metodes van die eienskap. Die `Deref` eienskap, wat deur die standaardbiblioteek voorsien word, vereis dat ons een metode genaamd `deref` wat `self` leen en 'n verwysing na die innerlike gegewens gee. Lys 15-10 bevat 'n implementering van `Deref` om by die definisie van `MyBox` te voeg:

<span class="filename">Lêernaam: src / main.rs</span>

```rust
use std::ops::Deref;

# struct MyBox<T>(T);
impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}
```

<span class="caption">Lys 15-10: Implementering van <code>Deref</code> op <code>MyBox&lt;T&gt;</code></span>

Die `type Target = T;` sintaksis definieer 'n verwante tipe vir die `Deref` eienskap om te gebruik. Geassosieerde tipes is 'n effens ander manier om 'n generiese parameter te verklaar, maar u hoef u vir eers nie daaroor te bekommer nie; ons sal dit in hoofstuk 19 in meer besonderhede bespreek.

We fill in the body of the `deref` method with `&self.0` so `deref` returns a reference to the value we want to access with the `*` operator. The `main` function in Listing 15-9 that calls `*` on the `MyBox<T>` value now compiles, and the assertions pass!

Sonder die `Deref` eienskap kan die samesteller slegs verwysings `&` verwysings gebruik. Die `deref` metode gee die samesteller die vermoë om 'n waarde van enige soort wat implemente neem `Deref` en noem die `deref` metode om 'n kry `&` verwysing dat hy weet hoe om dereference.

Toe ons `*y` in lys 15-9 binnegegaan het, het Rust agter die skerms hierdie kode gebruik:

```rust,ignore
*(y.deref())
```

Roest vervang die `*` operator met 'n oproep na die `deref` metode en dan 'n duidelike verskil, sodat ons nie hoef te dink of ons die `deref` metode moet noem nie. Met hierdie Rust-funksie kan ons kode skryf wat identies funksioneer, of ons 'n gewone verwysing het of 'n tipe wat `Deref` implementeer.

Die rede waarom die `deref` metode 'n verwysing na 'n waarde gee, en dat die duidelike verskille buite die hakies in `*(y.deref())` steeds nodig is, is die eienaarskapstelsel. As die `deref` metode die waarde direk terugstuur in plaas van 'n verwysing na die waarde, sal die waarde uit die `self` geskuif word. Ons wil nie die innerlike waarde binne `MyBox<T>` in hierdie geval of in die meeste gevalle waar ons die afleidingsoperateur gebruik, eienaarskap neem nie.

Note that the `*` operator is replaced with a call to the `deref` method and then a call to the `*` operator just once, each time we use a `*` in our code. Because the substitution of the `*` operator does not recurse infinitely, we end up with data of type `i32`, which matches the `5` in `assert_eq!` in Listing 15-9.

### Implisiete Deref dwang met funksies en metodes

*Deref coercion* is a convenience that Rust performs on arguments to functions and methods. Deref coercion converts a reference to a type that implements `Deref` into a reference to a type that `Deref` can convert the original type into. Deref coercion happens automatically when we pass a reference to a particular type’s value as an argument to a function or method that doesn’t match the parameter type in the function or method definition. A sequence of calls to the `deref` method converts the type we provided into the type the parameter needs.

Deref dwang is by Rust gevoeg sodat programmeerders funksie- en metodeoproepe hoef nie soveel eksplisiete verwysings en ander verwysings met `&` en `*` hoef by te voeg nie. Met die deref-dwangfunksie kan ons ook meer kode skryf wat kan werk vir verwysings of slim aanwysers.

Om die dwang in werking te sien, gebruik ons die `MyBox<T>` -tipe wat ons in Listing 15-8 gedefinieer het, asook die implementering van `Deref` wat ons in Listing 15-10 bygevoeg het. In lys 15-11 word die definisie van 'n funksie met 'n string-snyparameter aangedui:

<span class="filename">Lêernaam: src / main.rs</span>

```rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}
```

<span class="caption">Lys 15-11: A <code>hello</code> funksie wat die parameter het <code>name</code> van tipe <code>&amp;str</code></span>

Ons kan die `hello` funksie met 'n snytjie as argument noem, soos `hello("Rust");` byvoorbeeld. Deref dwang maak dit moontlik om `hello` te noem met verwysing na die waarde van die tipe `MyBox<String>` , soos in Listing 15-12 getoon:

<span class="filename">Lêernaam: src / main.rs</span>

```rust
# use std::ops::Deref;
#
# struct MyBox<T>(T);
#
# impl<T> MyBox<T> {
#     fn new(x: T) -> MyBox<T> {
#         MyBox(x)
#     }
# }
#
# impl<T> Deref for MyBox<T> {
#     type Target = T;
#
#     fn deref(&self) -> &T {
#         &self.0
#     }
# }
#
# fn hello(name: &str) {
#     println!("Hello, {}!", name);
# }
#
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
```

<span class="caption">Lys 15-12: <code>hello</code> roep met 'n verwysing na 'n <code>MyBox&lt;String&gt;</code> -waarde, wat werk weens die noodsaaklike dwang</span>

Here we’re calling the `hello` function with the argument `&m`, which is a reference to a `MyBox<String>` value. Because we implemented the `Deref` trait on `MyBox<T>` in Listing 15-10, Rust can turn `&MyBox<String>` into `&String` by calling `deref`. The standard library provides an implementation of `Deref` on `String` that returns a string slice, and this is in the API documentation for `Deref`. Rust calls `deref` again to turn the `&String` into `&str`, which matches the `hello` function’s definition.

As Rust nie die dwang dwing nie, sou ons die kode in Listing 15-13 moes skryf in plaas van die code in Listing 15-12 om `hello` te noem met die waarde van die tipe `&MyBox<String>` .

<span class="filename">Lêernaam: src / main.rs</span>

```rust
# use std::ops::Deref;
#
# struct MyBox<T>(T);
#
# impl<T> MyBox<T> {
#     fn new(x: T) -> MyBox<T> {
#         MyBox(x)
#     }
# }
#
# impl<T> Deref for MyBox<T> {
#     type Target = T;
#
#     fn deref(&self) -> &T {
#         &self.0
#     }
# }
#
# fn hello(name: &str) {
#     println!("Hello, {}!", name);
# }
#
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&(*m)[..]);
}
```

<span class="caption">Lys 15-13: Die kode wat ons sou moes skryf as Rust nie dwang dwing nie</span>

Die `(*m)` verwys die `MyBox<String>` in 'n `String` . Toe die `&` en `[..]` neem 'n string deel van die `String` wat gelyk is aan die hele string na die ondertekening van pas is `hello` . Die kode sonder enige dwang is moeiliker om te lees, skryf en verstaan met al hierdie simbole. Deur dwang dwang kan Rust hierdie omskakelings outomaties vir ons hanteer.

Wanneer die `Deref` eienskap vir die betrokke tipes gedefinieër word, sal Rust die tipes analiseer en `Deref::deref` soveel keer as nodig gebruik om 'n verwysing te kry wat ooreenstem met die tipe parameter. Die aantal kere wat `Deref::deref` ingevoeg moet word, word op kompileringstyd opgelos, dus is daar geen lopietydperk om voordeel te trek uit deref-dwang nie!

### Hoe Deref-dwang met wisselvalligheid in wisselwerking tree

Soortgelyk aan hoe u die `Deref` eienskap gebruik om die `*` -operateur op onveranderlike verwysings te ignoreer, kan u die `DerefMut` eienskap gebruik om die `*` -operator op veranderlike verwysings te ignoreer.

Roes dwing dwang as dit in drie gevalle soorte en eienskappe vind:

- Van `&T` tot `&U` wanneer `T: Deref<Target=U>`
- Van `&mut T` tot `&mut U` wanneer `T: DerefMut<Target=U>`
- Van `&mut T` tot `&U` wanneer `T: Deref<Target=U>`

Die eerste twee gevalle is dieselfde, behalwe vir veranderlikheid. Die eerste geval lui dat as u 'n `&T` , en `T` `Deref` op 'n tipe `U` implementeer, kan u 'n `&U` deursigtig kry. In die tweede geval word gesê dat dieselfde dwang vir veranderlike verwysings plaasvind.

Die derde geval is lastiger: Roes sal ook 'n veranderlike verwysing na 'n onveranderlike gedwing word. Maar die omgekeerde is *nie* moontlik nie: onveranderlike verwysings sal nooit na veranderlike verwysings gedwing word nie. As gevolg van die leenreëls, as u 'n veranderlike verwysing het, moet die veranderbare verwysing die enigste verwysing na daardie data wees (anders kan die program nie saamgestel word nie). Die omskakeling van een veranderlike verwysing na een onveranderlike verwysing sal nooit die leenreëls oortree nie. Die omskakeling van 'n onveranderlike verwysing na 'n veranderlike verwysing sou vereis dat daar slegs een onveranderlike verwysing na daardie gegewens is, en die leenreëls waarborg dit nie. Daarom kan Rust nie die aanname maak dat die omskakeling van 'n onveranderlike verwysing na 'n veranderlike verwysing moontlik is nie.
