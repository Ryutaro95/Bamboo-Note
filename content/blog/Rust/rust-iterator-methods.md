
## fold
https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.fold

イテレータの要素全てから１つの結果を得たい時に使える



```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    let sum = v.iter().fold(0, |sum, x| sum + x);
    println!("{}", sum); // 55
}
```