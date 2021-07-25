---
title: Gotcha on golang
tags:
  - Programming
  - Go
  - Golang
date: 2021-07-25 23:00:00 +0700
---

Encountered some wierd bug about 3 weeks ago, this is simplified version.

We have `func testNil(input interface{}) bool { return input == nil }`

But when we declare `var ptr *int` and called `testNil(ptr)` it return false.

After doing some research, I simplified the answer to myself that Golang's comparison with _nil_ is to test whether that variable has value stored in it.

In above code, _input_ has value stored in it, hence it return false.

But if we cast the interface to its underlying type, then the nil comparison is tested on the underlying value as well.

`func testNil(input interface{}) bool { return input.(*int) == nil }` will return true if we declare `var ptr *int` and called `testNil(ptr)`.

Footnote: You can use `switch input.(type)` to check underlying type of interface and cast it accordingly.

Footnote2: If we define `func testNil(foo, bar interface{}) bool { return foo == bar }` this function called
$$
  `var foo interface{}`
  `var bar *int`
  `testNil(foo, bar)`
$$
will return false as they are not the same type.