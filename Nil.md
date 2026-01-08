# Nil

- Nil means nothing
- Null means not any
- None means not one

so nil is a zero. 

History of nil
- Tony hoare ( from his book go concurrency concepts are taken ) -> he also created nil in algo language
- He called it billion dollar mistake. It was the invention of null reference in 1965.


### What are zero values ??

bool -> false
numbers 0
string ""
pointers, slices, maps, channels, functions, interfaces -> nil

Nil is important because it is zero value of most of the types in the go.

Zero value of struct is a struct with the zero value of each it's field.

what is type of nil -> is it pointer, zero or something or map or what ?? 
- nil has no type ( according to doc )

0 has type int, false has type bbool

so a := "", a = 0 will compile but a := nil will not compile because nil is not of any type.

a := nil // use of untyped nil -- error

Go doc: nil is a predeclared identifier representing the zero value for a pointer, channel, func, interface, map or slice type.

Nil is not a keyword -- interesting :)

go has 25* keywords -- break, case, chan, struc,t, const, continue, default, defer, else, for, fallthrough, func, go, goto, if, import, interface, map, package, range, return, select, struct, switch, type, var

nil is something that you can redefine. 

var nil = errors.New("ha ha")

Pointers in go:
- they point to position in memory

Nil pointer --> something that doesn't point to anything.

Slices in go:
- when it is nil --> that doesn't have backing array. length and capacity are there.

maps, channels and functions are pretty much of same type. they points to something ( which we don't need to discuss ). 

Interfaces ( interesting )
Interface is not a poinnter ( different from above)
Two components -- (type, value)


var s fmt.Stringer // Stringer(nil,nil) // nil=nil true

var s fmt.Stringer = p // Stringer(*Person, nil)

fmt.Println(s == nil) // false

- So pointer to person nil is not a nil. 


When is nil not a nil ?
For eg.

func do() error { // when error was returned it got wrapped in interface as error(*doError, nil) // so it is not equal to nil
    var err *doError
    return err
}

func main(){
    err := do() // error(*doError, nil)
    fmt.Println(err == nil) // false
}

So avoid declaring concrete nil error vars



***Very Interesting Case***

func do() *doError { // return nil of type *doError // it will cause no issue if compared with = nil
    return nil
}

func wrapDo() error { // let's say above function gets called by any wrapper func, then it will have return type error(*doError, nil)
    return do()
}

func main() {
    err := wrapDo()
    fmt.Println(err == nil) // false due to interface of different type
}

Rob Pike said that --> make the zero value usefull

How are these usefull ?

Pointers

- Pointer receivers are usefull

for eg.
type person struct {}
func sayHi(p *person) { fmt.Println("hi") } -- we can't use this if let's say p is nil
func (p *person) sayHi() { fmt.Println("hi") } -- we can always use this even if p is nil

var p *person
p.sayHi() // hi

nil recievers are usefull

func (t *tree) Sum() int {
    if t == nil {
        return 0
    }

    return t.v + t.l.Sum() + t.r.Sum()
}

// improved code quality using nil recievers


Slice
s[i] -- panice index out of range

append on nil slices -- works 

Nil maps

var m map[t]u

len(m) // 0
for range m // iterates 0 times
v, ok := m[i] // zero(u), falsee
m[i] = v // panic, assignment to entry in nil map

// if using read only maps then nil just works..

Channels ( dave jenny )

var c chan t // nothing works with nil channels

<-c // blocks forever
c <- x // blocks forever
close(c) // panic: close of nil channel

func merge(out chan<-int, a, b <-chan int)

```go
// _Channels_ are the pipes that connect concurrent
// goroutines. You can send values into channels from one
// goroutine and receive those values into another
// goroutine.

package main

import "fmt"

func main() {
	var out chan int = make(chan int)
	var a chan int = make(chan int)
	var b chan int = make(chan int)

	go func(a chan int) {
		for i := range 10 {
			a <- i
		}
		close(a)
	}(a)

	go func(a chan int) {
		for i := 11; i < 15; i++ {
			a <- i
		}
		close(a)
	}(b)

	go merge(out, a, b)

	for val := range out {
		fmt.Println("Output value:", val)
	}

}

func merge(out chan int, a, b <-chan int) {
	var aDone bool
	var bDone bool

	for {
		select {
		case valA, more := <-a:
			{
				if !more {
					aDone = true
					a = nil
				}
				// fmt.Println("a value printed", valA)
				out <- valA
			}
		case valB, more := <-b:
			{
				if !more {
					bDone = true
					b = nil
				} // mark nil when you don't want to recieve anything from the channel, nil channel blocks, closed channel does not blocks
				// fmt.Println("b value printed", valB)
				out <- valB
			}
		}

		if aDone && bDone {
			close(out)
			return
		}
	}
}
```

nil is usefull in above case, we can switch off channel when we set it's value nil.

when you try to recieve from them it blocks forever.

Diactivated that channle, so that select case will never activate again ha ha.

- NOTE: so select nil chans to disable select case

nil funcs
- go has first class functions
- functions can be used as struct fields
- they need a zero value, logically it is nil


type Foo struct {
    f func() error
}

***nil funcs for default values***

lazy initialization of variables

nil can also imply default behaviour

```go
func NewServer(logger func(string, ..interface{})) {
    if logger == nil {
        logger = log.Printf
    }

    logger("initializing Service")
}
```


**Interfaces**

The nil interface is used as a signal.

why does nil *per 

nil values can satisfy an interface

***nil values and default values***

```go
func doSum(s Summer) int {
    if s == nil {
        return 0
    }

    return s.Sum()
}
```

so when we use nil --> implice go with the default thing

for eg. in http server

http.HandleFunc("locahost:8080", nil) // use default mux for the http package

NOTE: so use nil interfaces to signal default values
