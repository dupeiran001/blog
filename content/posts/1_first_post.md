+++
title = 'First Post'
date = 2023-12-21T17:35:08+08:00
draft = false
+++

## Introduction

This is **bold** text, and this is *emphasized* text.

Visit the [Hugo](https://gohugo.io) website!


```rust
fn main(){
  eprintln!("{}", "hello world");

  let a = 1;

  println!("{}", b(a));


}

fn b(a: i32) -> i32{
  let (tx, rx) = std::sync::mpsc::channel();
  std::thread::spawn(move || {
    let b = a + 1;
    tx.send(b).unwrap();
  });

  return rx.receive().unwrap();
}
```

- 1
- 2
- 3

> cite 
> cite

### title


