---
title: Gotcha on golang
tags:
  - Programming
  - Go
  - Golang
date: 2021-07-25 23:00:00 +0700
---

Encountered some wierd bug about 3 weeks ago, this is simplified version.

We have 

``` go
func testNil(input interface{}) bool { 
  return input == nil 
}

func main() {
  var ptr *int
  log.Printf("%v", testNil(ptr)) // this return false, but I expect it to be true
}
```

After doing some research, I simplified the answer to myself that Golang's comparison with _nil_ is to test whether that variable has value stored in it.

In above code, _input_ has value stored in it, hence it return false.

But if we cast the interface to its underlying type, then the nil comparison is tested on the underlying value as well.

``` go
func testNil(input interface{}) bool { 
  return input.(*int) == nil 
} 

func main() {
  var ptr *int
  log.Printf("%v", testNil(ptr)) // now it's true
}
```

Footnote: You can use `switch input.(type)` to check underlying type of interface and cast it accordingly.

Footnote2:

``` go
func testEqual(foo, bar interface{}) bool { 
  return foo == bar 
}

func main() {
  var foo interface{}
  var bar *int
  log.Printf("%v", testEqual(foo, bar)) // this is false, because it is not the same type
}
```