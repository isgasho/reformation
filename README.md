[![Build Status](https://travis-ci.org/hukumka/reformation.svg?branch=master)](https://travis-ci.org/hukumka/reformation)

# reformation

Parsing via regular expressions using format syntax

Deriving trait `Reformation` will also implement
trait `FromStr`, with `Err=Box<Error>`

Derive will require attribute reformation to specify format string,
which will be treated as format string -> regular expression string

Types implementing `Reformation` by default:

+ signed integers: `i8` `i16` `i32` `i64` `i128` `isize`
+ unsigned integers: `u8` `u16` `u32` `u64` `u128` `usize`
+ floats: `f32` `f64`
+ `String`

```rust
use reformation::Reformation;

#[derive(Reformation)]
#[reformation(r"{year}-{month}-{day} {hour}:{minute}")]
struct Date{
    year: u16,
    month: u8,
    day: u8,
    hour: u8,
    minute: u8,
}

fn main(){
    let date: Date = "2018-12-22 20:23".parse().unwrap();

    assert_eq!(date.year, 2018);
    assert_eq!(date.month, 12);
    assert_eq!(date.day, 22);
    assert_eq!(date.hour, 20);
    assert_eq!(date.minute, 23);
}
```

Format string behaves as regular expression, so special symbols needs to be escaped.
Also they can be used for more flexible format strings.
AVOID capture groups, since they would mess up with indexing of capture group
generated by macro. use non-capturing groups `r"(?:)"` instead.

```rust
use reformation::Reformation;

// '{' is special symbol in both format and regex syntax, so it must be escaped twice.
// Say hello to good old escape hell.
#[derive(Reformation)]
#[reformation(r"Vec\{{{x},\s*{y},\s*{z}\}}")]
struct Vec{
    x: f64,
    y: f64,
    z: f64,
}

fn main(){
    // spaces between coordinates does not matter, since any amount of spaces
    // matches to r"\s*"
    let v: Vec = "Vec{-0.4,1e-3,   2e-3}".parse().unwrap();

    assert_eq!(v.x, -0.4);
    assert_eq!(v.y, 0.001);
    assert_eq!(v.z, 0.002);
}
```
