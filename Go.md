# Go 

[TOC]

## Setting Up Go Project

A go project is organized into packages, which are a collection of source files within the same folder. These files are compiled together and the functions, variables and constants defined in the same package are visible to all other source files.

A collection of packages is called a module. The module is declared in a go.mod file in that directory and all the files and sub-folders belong to this module, until a folder with its own go.mod file.

Note that the module structure does not need to follow the directory structure but it is ideal if it does.

```wiki
Each module’s path not only serves as an import path prefix for its packages, but also indicates where the go command should look to download it. For example, in order to download the module golang.org/x/tools, the go command would consult the repository indicated by https://golang.org/x/tools (described more here). 

An import path is a string used to import a package. A package’s import path is its module path joined with its subdirectory within the module. For example, the module github.com/google/go-cmp contains a package in the directory cmp/. That package’s import path is github.com/google/go-cmp/cmp. Packages in the standard library do not have a module path prefix. 
```



```bash
$ mkdir hello # Alternatively, clone it if it already exists in version control.
$ cd hello
$ go mod init example.com/user/hello
go: creating new go.mod: module example.com/user/hello
$ cat go.mod
module example.com/user/hello

go 1.14
$
```



```go
go install .
```



## Basics

### Main

Every program starts running in the main package.

```go
package main
import "fmt"
func main(){
  fmt.Println("Hello World!")
}
```

### Exported Names

Exported names in a package start with a capital letter.

### Declaration

```go
x int
p *int
a [3]int

func add(x int, y int) int
func add(x, y int) int

var a, b c bool
```

### Initializers

```go
var i, j int = 1, 2

//type can be omitted if initializers are present
var a, b, c = 'a', 'b', 'c'
```

### Short declarations

```go
func add(x, y int) int {
  // var and type can be omitted within a function when := is used.
  a, b := 1, 2
}
```

### Basic types

```go
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128

// variables that are declared but not initialized are given their zero value
0 for numeric types,
false for the boolean type, and
"" (the empty string) for strings.
```

### Type Conversion

In go the variable is parenthesized when type-casting it. 

```go
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
```

> Assignment between different types requires explicit conversion
>
> ```
> var x int = 3
> var flt float64 = x
> ```
>
> This will cause the error -  `cannot use x (type int) as type float64 in assignment`



###Type Inference

When declaring a variable without specifying an explicit type (either by using the `:=` syntax or `var =` expression syntax), the variable's type is inferred from the value on the right hand side.  

### Constants

Declared using the `const` keyword.

```go
const PI = 3.14
```

> Note 1: `const` cannot be initialized using `:=`
>
> Note 2: `const` has to be initialized immediately after it has been declared

### Numeric Constants

High Precision values.

```go
package main

import "fmt"
// Note the grouping of multiple constant values
const (
	// Create a huge number by shifting a 1 bit left 100 places.
	// In other words, the binary number that is 1 followed by 100 zeroes.
	Big = 1 << 100
	// Shift it right again 99 places, so we end up with 1<<1, or 2.
	Small = Big >> 99
)

func needInt(x int) int { return x*1 + 1 }
func needFloat(x float64) float64 {
	return x / Big
}

func main() {
	fmt.Println(Small) // prints 2
	fmt.Println(needInt(Small)) // prints 2
	fmt.Println(needFloat(Small)) // prints 1.5777218104420236e-30
	fmt.Println(needFloat(Big)) // prints 1
}

```



### Generating Random Numbers

```go
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	// Seeding with the same value results in the same random sequence each run.
	// For different numbers, seed with a different value, such as
	// time.Now().UnixNano(), which yields a constantly-changing number.
	rand.Seed(86)
	fmt.Println(rand.Intn(100))
	fmt.Println(rand.Intn(100))
	fmt.Println(rand.Intn(100))

}
```





## Control Flow Statements

### For/While

>  `while` is spelled as `for` in Go ;-)

```go
package main

import "fmt"

func main() {
	sum := 0
  	/*
  	* init statment: executed before the first iteration (optional)
  	* condition statement: evaluated before every iteration
  	* post statement: executed at the end of every iteration (optional)
  	*
  	* Note the lack of () in the for loop syntax
  	*/
	for i := 0; i < 10; i++ {
		sum += i
	}
	fmt.Println(sum)
}

func infinteLoop() {
  for {
  }
}
```



### If

```go
if x < 0 {
    
}

// if statements can have a short initializer at the beginning.
// the scope of the initialized variable is limited to the if block

func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	}
  
  	// v cannot be returned here.
	return lim
}

// if and else syntax 
func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	} else {
		fmt.Printf("%g >= %g\n", v, lim)
	}
	// can't use v here, though
	return lim
}
```

### Switch

> Evaluation occurs from top to bottom and stops at the first case statement that evaluates to true, unless an explicit `fallthrough` is specified

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	fmt.Print("Go runs on ")
	switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
      	// explicit fallthrough instead of explicit break statments
      	// a case statment in Go breaks by default!
		fallthrough
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.", os)
	}
}
```

### Defer

> https://blog.golang.org/defer-panic-and-recover

A `defer` statement defers the execution of a function until the surrounding function returns.  

The deferred call's arguments are evaluated immediately, but the function call is not executed until the surrounding function returns.

```go
package main

import "fmt"

func main() {
	defer fmt.Println("world")

	fmt.Println("hello")
}
```

> Deferred function calls are pushed onto a stack. When a function returns, its deferred calls are executed in last-in-first-out order.

```go
package main

import "fmt"

func main() {
	fmt.Println("counting")

	for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}

	fmt.Println("done")
}
```

The output of the above snipped is - 

```go
counting
done
9
8
7
6
5
4
3
2
1
0
```

## Functions

A function can take zero or more arguments.   

In this example, `add` takes two parameters of type `int`.   

Notice that the type comes *after* the variable name.   

```go
package main

import "fmt"

func add(x int, y int) int {
	return x + y
}

func main() {
	fmt.Println(add(42, 13))
}

```

When two or more consecutive named function parameters share a type, you can omit the type from all but the last.

```go
package main

import "fmt"

func add(x, y int) int {
	return x + y
}

func main() {
	fmt.Println(add(42, 13))
}

```

A function can return any number of results.   

The `swap` function returns two strings.   

```go
package main

import "fmt"

func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}

```





## Types

### String

The standard library’s `strings` package provides many useful string-related functions. Here are some examples to give you a sense of the package.

```go
// The standard library's `strings` package provides many
// useful string-related functions. Here are some examples
// to give you a sense of the package.

package main

import s "strings"
import "fmt"

// We alias `fmt.Println` to a shorter name as we'll use
// it a lot below.
var p = fmt.Println

func main() {

    // Here's a sample of the functions available in
    // `strings`. Since these are functions from the
    // package, not methods on the string object itself,
    // we need pass the string in question as the first
    // argument to the function. You can find more
    // functions in the [`strings`](http://golang.org/pkg/strings/)
    // package docs.
    p("Contains:  ", s.Contains("test", "es"))
    p("Count:     ", s.Count("test", "t"))
    p("HasPrefix: ", s.HasPrefix("test", "te"))
    p("HasSuffix: ", s.HasSuffix("test", "st"))
    p("Index:     ", s.Index("test", "e"))
    p("Join:      ", s.Join([]string{"a", "b"}, "-"))
    p("Repeat:    ", s.Repeat("a", 5))
    p("Replace:   ", s.Replace("foo", "o", "0", -1))
    p("Replace:   ", s.Replace("foo", "o", "0", 1))
    p("Split:     ", s.Split("a-b-c-d-e", "-"))
    p("ToLower:   ", s.ToLower("TEST"))
    p("ToUpper:   ", s.ToUpper("test"))
    p()

    // Not part of `strings`, but worth mentioning here, are
    // the mechanisms for getting the length of a string in
    // bytes and getting a byte by index.
    p("Len: ", len("hello"))
    p("Char:", "hello"[1])
}

// Note that `len` and indexing above work at the byte level.
// Go uses UTF-8 encoded strings, so this is often useful
// as-is. If you're working with potentially multi-byte
// characters you'll want to use encoding-aware operations.
// See [strings, bytes, runes and characters in Go](https://blog.golang.org/strings)
// for more information.
```

Output - 

```go
Contains:   true
Count:      2
HasPrefix:  true
HasSuffix:  true
Index:      1
Join:       a-b
Repeat:     aaaaa
Replace:    f00
Replace:    f0o
Split:      [a b c d e]
ToLower:    test
ToUpper:    TEST

	

Len:  5
Char: 101

```

#### String Comparison 

String comparison can be done using either

1.  the strings package - 

```
func Compare(a, b string) int
```

Compare returns an integer comparing two strings lexicographically.The result will be 0 if a==b, -1 if a < b, and +1 if a > b.

Compare is included only for symmetry with package bytes.It is usually clearer and always faster to use the built-instring comparison operators ==, <, >, and so on.

2. using the `==` operator.


#### String Hash

```go
package main

import (
        "fmt"
        "hash/fnv"
)

func hash(s string) uint32 {
        h := fnv.New32a()
        h.Write([]byte(s))
        return h.Sum32()
}

func main() {
        fmt.Println(hash("HelloWorld"))
        fmt.Println(hash("HelloWorld."))
}
```

#### String Conversion

Converting int to string - 

```go
package main

import (
    "strconv"
    "fmt"
)

func main() {
    t := strconv.Itoa(123)
    fmt.Println(t)
}
```

Converting string to int - 

```go
package main

import (
    "strconv"
    "fmt"
)

func main() {
    t := strconv.Atoi("123")
    fmt.Println(t)
}
```




### Pointers

A pointer holds the memory address of a value.

The type `*T` is a pointer to a `T` value. Its zero value is `nil`.  

```go
var p *int
```

The `&` operator generates a pointer to its operand.  

```go
i := 42
p = &i
```

The `*` operator denotes the pointer's underlying value.  

```go
fmt.Println(*p) // read i through the pointer p
*p = 21         // set i through the pointer p
```

This is known as "dereferencing" or "indirecting".  

**Unlike C, Go has no pointer arithmetic.**

### Struct

```go
package main
import "fmt"

type Vertex struct {
	X int
	Y int
}

// Struct Literals
// Note the curly braces for initialization
var (
	v1 = Vertex{1, 2}  // has type Vertex
	v2 = Vertex{Y: 3, X: 1}  // Y:0 is implicit
	v3 = Vertex{}      // X:0 and Y:0
	p  = &Vertex{1, 2} // has type *Vertex
)

func main() {
	var v Vertex = Vertex{1, 2}
  	// struct fields are accessed using dot
	v.X = 4
	fmt.Println(v.X)
  	p := &v
  	/* To access the field X of a struct when we have the struct pointer p we could write 		(*p).X. However, that notation is cumbersome, so the language permits us instead to 	  write just p.X, without the explicit dereference.
  	*/
  	p.X = 44
}
```

### Arrays

The type `[n]T` is an array of `n` values of type `T`.  

The expression  

```
var a [10]int
```

declares a variable `a` as an array of ten integers.

> Note that n has to be a constant value.
>
> ```go
> length := 5
> a := [length]int{1,2,3,4,5}
> ```
>
> gives the error - `non-constant array bound length` . To create dynamically sized arrays see the section `using make to create slices`

```go
package main

import "fmt"

func main() {
	var a [2]string
	a[0] = "Hello"
	a[1] = "World"
	fmt.Println(a[0], a[1])
	fmt.Println(a)

	primes := [6]int{2, 3, 5, 7, 11, 13}
	subjects := [2]string{"OS", "DBMS"}
	fmt.Println(primes)
	fmt.Println(names)
}
```

#### Sorting

```go
// Go's `sort` package implements sorting for builtins
// and user-defined types. We'll look at sorting for
// builtins first.

package main

import "fmt"
import "sort"

func main() {

    // Sort methods are specific to the builtin type;
    // here's an example for strings. Note that sorting is
    // in-place, so it changes the given slice and doesn't
    // return a new one.
    strs := []string{"c", "a", "b"}
    sort.Strings(strs)
    fmt.Println("Strings:", strs)

    // An example of sorting `int`s.
    ints := []int{7, 2, 4}
    sort.Ints(ints)
    fmt.Println("Ints:   ", ints)

    // We can also use `sort` to check if a slice is
    // already in sorted order.
    s := sort.IntsAreSorted(ints)
    fmt.Println("Sorted: ", s)
}
```

#### Sorting User-Defined structs 

```go
package main

import (
	"fmt"
	"sort"
)

type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("%s: %d", p.Name, p.Age)
}

// ByAge implements sort.Interface for []Person based on
// the Age field.
type ByAge []Person

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }

func main() {
	people := []Person{
		{"Bob", 31},
		{"John", 42},
		{"Michael", 17},
		{"Jenny", 26},
	}

	fmt.Println(people)
	// There are two ways to sort a slice. First, one can define
	// a set of methods for the slice type, as with ByAge, and
	// call sort.Sort. In this first example we use that technique.
	sort.Sort(ByAge(people))
	fmt.Println(people)

	// The other way is to use sort.Slice with a custom Less
	// function, which can be provided as a closure. In this
	// case no methods are needed. (And if they exist, they
	// are ignored.) Here we re-sort in reverse order: compare
	// the closure with ByAge.Less.
	sort.Slice(people, func(i, j int) bool {
		return people[i].Age > people[j].Age
	})
	fmt.Println(people)

}

```



### Slices

An array has a fixed size. A slice, on the other hand, is a dynamically-sized, flexible view into the elements of an array. In practice, slices are much more common than arrays.  

 The type `[]T` is a slice with elements of type `T`.  

A slice is formed by specifying two indices, a low and high bound, separated by a colon:

```go
a[low : high]
```

This selects a half-open range which includes the first    element, but excludes the last one.  

The following expression creates a slice which includes    elements 1 through 3 of `a`:  

```go
a[1:4]
```

```go
package main

import "fmt"

func main() {
	var primes [6]int

	var s []int = primes[1:4]
    // Valid
	s = primes[0:5]
  	// Invalid: cannot use [10]int literal (type [10]int) as type [6]int in assignment
	primes = [10]int{1,2,3,4,5,6,7,8,9,0}
	fmt.Println(s)
}
```

Note: Slices are like references to the underlying array. They do not store any data

```go
package main

import "fmt"

func main() {
	names := [4]string{
		"John",
		"Paul",
		"George",
		"Ringo",
	}
	fmt.Println(names)

	a := names[0:2]
	b := names[1:3]
	fmt.Println(a, b)

	b[0] = "XXX"
	fmt.Println(a, b)
	fmt.Println(names)
}

```

The output of the above snippet is - 

```go
[John Paul George Ringo]
[John Paul] [Paul George]
[John XXX] [XXX George]
[John XXX George Ringo]
```

#### Slice Literals

A slice literal is like an array literal without the length.  

This is an array literal:  

```go
[3]bool{true, true, false}
```

**And this creates the same array as above, then builds a slice that references it:  **

```go
[]bool{true, true, false}
```

```go
package main

import "fmt"

type x struct {
	a string
	b int
}

func main() {
	q := []int{2, 3, 5, 7, 11, 13}
	fmt.Println(q)

	r := []bool{true, false, true, true, false, true}
	fmt.Println(r)

	s := []struct {
		i int
		b bool
	}{
		{2, true},
		{3, false},
		{5, true},
		{7, true},
		{11, false},
		{13, true},
	}
	fmt.Println(s)

	v := x{"String", 23}
	fmt.Println(v)

}

```

Note: Slice Defaults

```go
// Equivalent statements
a[0:10]
a[:10]
a[0:]
a[:]
```

#### Slice Length and Capacity

```go
package main

import "fmt"

func main() {
	s := []int{2, 3, 5, 7, 11, 13}
	printSlice(s)

	// Slice the slice to give it zero length.
	s = s[:0]
	printSlice(s)

	// Extend its length.
	s = s[:4]
	printSlice(s)

	// Drop its first two values.
	s = s[2:]
	printSlice(s)
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

```

The output of the above code  is - 

```go
len=6 cap=6 [2 3 5 7 11 13]
len=0 cap=6 []
len=4 cap=6 [2 3 5 7]
len=4 cap=6 [5 7]
```

#### Nil Slice

```go
package main

import "fmt"

func main() {
	var s []int
	fmt.Println(s, len(s), cap(s))
	if s == nil {
		fmt.Println("nil!")
	}
}

// Output
[] 0 0
nil!
```

#### Using make to create slices

Slices can be created with the built-in `make` function;    this is how you create dynamically-sized arrays.  

The `make` function allocates a zeroed array and returns a slice that refers to that array:  

```
a := make([]int, 5)  // len(a)=5
```

To specify a capacity, pass a third argument to `make`:  

```
b := make([]int, 0, 5) // len(b)=0, cap(b)=5

b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]      // len(b)=4, cap(b)=4
```

http://127.0.0.1:3999/moretypes/14

#### Slices of Slices

A slice can contain slices.

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	// Create a tic-tac-toe board.
	board := [][]string{
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
	}

	// The players take turns.
	board[0][0] = "X"
	board[2][2] = "O"
	board[1][2] = "X"
	board[1][0] = "O"
	board[0][2] = "X"

	for i := 0; i < len(board); i++ {
        //strings.Join - joins array using the delimiter in second arg
		fmt.Printf("%s\n", strings.Join(board[i], " "))
	}
}
```

#### Appending to Slice

```go
func append(s []T, vs ...T) []T
```

The first parameter `s` of `append` is a slice of type `T`, and the rest are    `T` values to append to the slice.  

The resulting value of `append` is a slice containing all the elements of the original slice plus the provided values.  

**Note**: If the backing array of `s` is too small to fit all the given values a bigger array will be allocated. The returned slice will point to the newly allocated array. 

```go
package main

import "fmt"

func main() {
	var s []int
	printSlice(s)

	// append works on nil slices.
	s = append(s, 0)
	printSlice(s)

	// The slice grows as needed.
	s = append(s, 1)
	printSlice(s)

	// We can add more than one element at a time.
	s = append(s, 2, 3, 4)
	printSlice(s)
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}
```

* Note: Appending Slice to slice - 

  ```go
  append([]int{1,2}, []int{3,4}...)
  ```

  Add dots after the second slice

#### Range of a slice

When ranging over a slice, two values are returned for each iteration. The first is the index, and the second is a copy of the element at that index.

> ​    You can skip the index or value by assigning to `_`

```go
package main

import "fmt"

var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

func main() {
	for i, v := range pow {
		fmt.Printf("2**%d = %d\n", i, v)
	}
}
```

The output is - 

```go
2**0 = 1
2**1 = 2
2**2 = 4
2**3 = 8
2**4 = 16
2**5 = 32
2**6 = 64
2**7 = 128
```

### Maps

**NOTE**: Use `sync.RWMutex` when concurrently accessing maps!!

```go
package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m map[string]Vertex

func main() {
	m = make(map[string]Vertex)
	m["Bell Labs"] = Vertex{
		40.68433, -74.39967,
	}
	fmt.Println(m["Bell Labs"])
}
```

#### Map Literals

Note: Map literals are like struct literals, but the keys are required.

```go
package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}

func main() {
	fmt.Println(m)
}

```

#### Mutating Maps

Insert or update an element in map `m`:  

```
m[key] = elem
```

​    Retrieve an element:  

```
elem = m[key]
```

​    Delete an element:  

```
delete(m, key)
```

​    Test that a key is present with a two-value assignment:  

```
elem, ok = m[key]
```

​    If `key` is in `m`, `ok` is `true`. If not, `ok` is `false`.  

​    If `key` is not in the map, then `elem` is the zero value for the map's element type.  

​    *Note*: if `elem` or `ok` have not yet been declared you could use a short declaration form:  

```
elem, ok := m[key]
```

```go
package main

import "fmt"

func main() {
	m := make(map[string]int)

	m["Answer"] = 42
	fmt.Println("The value:", m["Answer"])

	m["Answer"] = 48
	fmt.Println("The value:", m["Answer"])

	delete(m, "Answer")
	fmt.Println("The value:", m["Answer"])

	v, ok := m["Answer"]
	fmt.Println("The value:", v, "Present?", ok)
}

```

Output - 

```go
The value: 42
The value: 48
The value: 0
The value: 0 Present? false
```

#### Iterating Over Maps

```go
for k, v := range m { 
    fmt.Printf("key[%s] value[%s]\n", k, v)
}
```



### Function Values

Functions are values too. They can be passed around just like other values.  Function values may be used as function arguments and return values.

```go
package main

import (
	"fmt"
	"math"
)

func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))

	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}
```

Output - 

```
13
5
81
```

### Function Closures

Go functions may be closures. A closure is a function value that references variables from outside its body. The function may access and assign to the referenced variables; in this sense the function is "bound" to the variables.  

For example, the `adder` function returns a closure. Each closure is bound to its own `sum` variable.  

```go
package main

import "fmt"

func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}
```

## Restful APIs

### Defining json keys for structs

```go
package models

// You can use tilde's to define json keys that
// are different from variable names in a struct
type Flashcard struct {
	VisibleCharacter string   `json:"visible"`
	Answer           string   `json:"answer"`
	Options          []string `json:"options"`
}

```



### Using Mux to Route HTTP requests

Package `gorilla/mux` implements a request router and dispatcher for matching incoming requests to
their respective handler.

https://github.com/gorilla/mux

```go
import (
	"encoding/json"
	"log"
	"net/http"

	"github.com/gorilla/mux"
)

var esClient *elasticsearch.Client
var esError error

func main() {
	r := mux.NewRouter()
	r.HandleFunc("/flashcards", RequestHandler).Methods("GET")
	log.Fatal(http.ListenAndServe(":8000", r))
}

func FlashcardsHandler(w http.ResponseWriter, r *http.Request) {
    response = // Response for the HTTP request
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)
	json.NewEncoder(w).Encode(response)
}

```



## Type assertions

 For an expression `x` of [interface type](https://golang.org/ref/spec#Interface_types) and a type `T`, the primary expression 

```go
x.(T)
```

asserts that `x` is not `nil` and that the value stored in `x` is of type `T`. The notation `x.(T)` is called a *type assertion*. 

More precisely, if `T` is not an interface type, `x.(T)` asserts that the dynamic type of `x` is [identical](https://golang.org/ref/spec#Type_identity) to the type `T`. In this case, `T` must [implement](https://golang.org/ref/spec#Method_sets) the (interface) type of `x`; otherwise the type assertion is invalid since it is not possible for `x` to store a value of type `T`. If `T` is an interface type, `x.(T)` asserts that the dynamic type of `x` implements the interface `T`. 

If the type assertion holds, the value of the expression is the value stored in `x` and its type is `T`. If the type assertion is false, a [run-time panic](https://golang.org/ref/spec#Run_time_panics) occurs. In other words, even though the dynamic type of `x` is known only at run time, the type of `x.(T)` is known to be `T` in a correct program. 

```go
var x interface{} = 7          // x has dynamic type int and value 7
i := x.(int)                   // i has type int and value 7

type I interface { m() }

func f(y I) {
	s := y.(string)        // illegal: string does not implement I (missing method m)
	r := y.(io.Reader)     // r has type io.Reader and the dynamic type of y must implement both I and io.Reader
	…
}
```

 A type assertion used in an [assignment](https://golang.org/ref/spec#Assignments) or initialization of the special form 

```
v, ok = x.(T)
v, ok := x.(T)
var v, ok = x.(T)
var v, ok T1 = x.(T)
```

 yields an additional untyped boolean value. The value of `ok` is `true` if the assertion holds. Otherwise it is `false` and the value of `v` is the [zero value](https://golang.org/ref/spec#The_zero_value) for type `T`. No [run-time panic](https://golang.org/ref/spec#Run_time_panics) occurs in this case. 



## Testing in Golang

 Package testing provides support for automated testing of Go packages. It is intended to be used in concert with the “go test” command, which automates execution of any function of the form 

```
func TestXxx(*testing.T)
```

 where Xxx does not start with a lowercase letter. The function name serves to identify the test routine. 

 Within these functions, use the Error, Fail or related methods to signal failure. 

 To write a new test suite, create a file whose name ends _test.go that contains the TestXxx functions as described here. Put the file in the same package as the one being tested. The file will be excluded from regular package builds but will be included when the “go test” command is run. For more detail, run “go help test” and “go help testflag”. 

 A simple test function looks like this: 

```go
func TestAbs(t *testing.T) {
    got := Abs(-1)
    if got != 1 {
        t.Errorf("Abs(-1) = %d; want 1", got)
    }
}
```



## Methods

Go does not have classes. However, you can define methods on types.  A method is a function with a special *receiver* argument.  The receiver appears in its own argument list between the `func` keyword and    the method name.  In this example, the `Abs` method has a receiver of type `Vertex` named `v`.  

```go
package main

import (
    "fmt"
    "strings"
)

type Student struct {
    name, id string
    age int8
}

func (s Student) FirstName() string {
    nameSplit := strings.Fields(s.name)
    return nameSplit[0]
}

func main() {
    var s Student = Student{"Kunal Kashilkar", "A46", 24}
    fmt.Println(s.FirstName())
}
```

> You can only declare a method with a receiver whose type is defined in the same package as the method. You cannot declare a method with a receiver whose type is defined in another package (which includes the built-in types such as `int`).

#### Pointer Receivers

Pointer Receivers can be used to define methods that modify the variable. These are generally more common than value receivers.

```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10)
	fmt.Println(v.Abs())
}
```

#### Method and Pointer Indirection

```go
package main

import "fmt"

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func ScaleFunc(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(2) // This method has a pointer receiver but we can call it on a value.
    		   // This is a convenience provided by Go, which interprets this as
    		   // as (&v).Scale(5) since the Scale method has a pointer receiver. 
	ScaleFunc(&v, 10)

	p := &Vertex{4, 3}
	p.Scale(3)
	ScaleFunc(p, 8)

	fmt.Println(v, p)
}
```

The equivalent thing happens in the reverse direction.  

Functions that take a value argument must take a value of that specific type:  

```go
var v Vertex
fmt.Println(AbsFunc(v))  // OK
fmt.Println(AbsFunc(&v)) // Compile error!
```

while methods with value receivers take either a value or a pointer as the  	receiver when they are called:  

```go
var v Vertex
fmt.Println(v.Abs()) // OK
p := &v
fmt.Println(p.Abs()) // OK
```

In this case, the method call `p.Abs()` is interpreted as `(*p).Abs()`.  

#### Choosing a value or pointer receiver

There are two reasons to use a pointer receiver.  

The first is so that the method can modify the value that its receiver points to.  

The second is to avoid copying the value on each method call.    This can be more efficient if the receiver is a large struct, for example.  

In this example, both `Scale` and `Abs` are with receiver type `*Vertex`,    even though the `Abs` method needn't modify its receiver.  

In general, all methods on a given type should have either value or pointer   receivers, but not a mixture of both.   

### Interfaces

An *interface type* is defined as a set of method signatures.

```go
package main

import (
	"fmt"
	"math"
)

type Abser interface {
	Abs() float64
}

func main() {
	var a Abser
	f := MyFloat(-math.Sqrt2)
	v := Vertex{3, 4}

	a = f  // a MyFloat implements Abser
	a = &v // a *Vertex implements Abser

	// In the following line, v is a Vertex (not *Vertex)
	// and does NOT implement Abser.
	a = v // This will give an error.

	fmt.Println(a.Abs())
}

type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

```

#### Interface Values

Under the covers, interface values can be thought of as a tuple of a value and a    concrete type:  

```
(value, type)
```

An interface value holds a value of a specific underlying concrete type.  

Calling a method on an interface value executes the method of the same name on its underlying type. 

```go
package main

import (
	"fmt"
	"math"
)

type I interface {
	M()
}

type T struct {
	S string
}

func (t *T) M() {
	fmt.Println(t.S)
}

type F float64

func (f F) M() {
	fmt.Println(f)
}

func main() {
	var i I

	i = &T{"Hello"}
	describe(i)
	i.M()

	i = F(math.Pi)
	describe(i)
	i.M()
}

func describe(i I) {
	fmt.Printf("(%v, %T)\n", i, i)
}

```

```go
(&{Hello}, *main.T)
Hello
(3.141592653589793, main.F)
3.141592653589793
```

#### Interface Nil values

```go
package main
 
 import (
     "fmt"
 )
 
 type I interface {
     M() 
 }
 
 type F float64
 
 func (f F) M() {
     fmt.Println(f);
 }
 
 type Vertex struct {
     Lat, Long float64
 }
 
 func (v *Vertex) M() {
     if v == nil {
         fmt.Println("nil")
         return
     }   
 
     fmt.Printf("Lat = %f, Long = %f \n", v.Lat, v.Long)
 }
 
 func main() {
     var i I 
 
     vertex := Vertex{34.34, 35.23}
 
     i = &vertex
     i.M();
 
     var ver *Vertex
     i = ver 
     describe(i)
     i.M()
 
     ver = &Vertex{54.23,64.34}
     i = ver 
     describe(i)
     i.M()
 }
 
 func describe(i I) {
     fmt.Printf("(%v, %T)\n", i, i)
 }
```

```go
Lat = 34.340000, Long = 35.230000 
(<nil>, *main.Vertex)
nil
(&{54.23 64.34}, *main.Vertex)
Lat = 54.230000, Long = 64.340000 
```

#### Nil Interface Value

A nil interface value holds neither value nor concrete type.  

Calling a method on a nil interface is a run-time error because there is no type inside the interface tuple to indicate which *concrete* method to call. 

```
(<nil>, <nil>)
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0x1 addr=0x0 pc=0x4838ab]
```

#### The empty interface

The interface type that specifies zero methods is known as the *empty interface*:  

```
interface{}
```

An empty interface may hold values of any type.    (Every type implements at least zero methods.)  

Empty interfaces are used by code that handles values of unknown type.    For example, `fmt.Print` takes any number of arguments of type `interface{}`.  

#### Stringer Interface

One of the most ubiquitous interfaces is [`Stringer`](http://golang.org/pkg/fmt/#Stringer) defined by the [`fmt`](http://golang.org/pkg/fmt/) package.  

```
type Stringer interface {
    String() string
}
```

A `Stringer` is a type that can describe itself as a string. The `fmt` package    (and many others) look for this interface to print values.  

```go
package main

import "fmt"

type IPAddr [4]byte

// TODO: Add a "String() string" method to IPAddr.
func (ip IPAddr) String() string {
	return fmt.Sprintf("%v.%v.%v.%v", ip[0],ip[1],ip[2],ip[3])
}

func main() {
	hosts := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for name, ip := range hosts {
		fmt.Printf("%v: %v\n", name, ip)
	}
}

```

### Errors

Go programs express error state with `error` values.  

The `error` type is a built-in interface similar to `fmt.Stringer`:  

```
type error interface {
    Error() string
}
```

(As with `fmt.Stringer`, the `fmt` package looks for the `error` interface when printing values.)  

Functions often return an `error` value, and calling code should handle errors by testing whether the error equals `nil`.  

```
i, err := strconv.Atoi("42")
if err != nil {
    fmt.Printf("couldn't convert number: %v\n", err)
    return
}
fmt.Println("Converted integer:", i)
```

A nil `error` denotes success; a non-nil `error` denotes failure.  

```go
package main

import (
	"fmt"
	"time"
)

type MyError struct {
	When time.Time
	What string
}

func (e *MyError) Error() string {
	return fmt.Sprintf("at %v, %s",
		e.When, e.What)
}

func run() error {
	return &MyError{
		time.Now(),
		"it didn't work",
	}
}

func main() {
	if err := run(); err != nil {
		fmt.Println(err)
	}
}
```

You can construct one of these values with the `errors.New` function. It takes a string that it converts to an `errors.errorString` and returns as an `error` value.  

```go
// New returns an error that formats as the given text.
func New(text string) error {
    return &errorString{text}
}
```

​    Here's how you might use `errors.New`:  

```
func Sqrt(f float64) (float64, error) {
    if f < 0 {
        return 0, errors.New("math: square root of negative number")
    }
    // implementation
}
```

### Goroutines

 A *goroutine* is a lightweight thread managed by the Go runtime.  

```
go f(x, y, z)
```

starts a new goroutine running  

```
f(x, y, z)
```

The evaluation of `f`, `x`, `y`, and `z` happens in the current goroutine and the execution of `f` happens in the new goroutine.  

Goroutines run in the same address space, so access to shared memory must be synchronized. The [`sync`](https://golang.org/pkg/sync/) package provides useful primitives, although you won't need them much in Go as there are other primitives.



### Channels

Channels are a typed conduit through which you can send and receive values with the channel operator, `<-`.  

```
ch <- v    // Send v to channel ch.
v := <-ch  // Receive from ch, and
           // assign value to v.
```

(The data flows in the direction of the arrow.)  

Like maps and slices, channels must be created before use:  

```
ch := make(chan int)
```

By default, sends and receives block until the other side is ready. This allows goroutines to synchronize without explicit locks or condition variables.  

The example code sums the numbers in a slice, distributing the work between two goroutines.    Once both goroutines have completed their computation, it calculates the final result.

```go
package main

import "fmt"

func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // send sum to c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // receive from c

	fmt.Println(x, y, x+y)
}
```

Output - 

```go
-5 17 12
```

#### Buffered Channels

Channels can be *buffered*.  Provide the buffer length as the second argument to `make` to initialize a buffered channel:  

```
ch := make(chan int, 100)
```

Sends to a buffered channel block only when the buffer is full. Receives block when the buffer is empty.



**Note** : Using buffered channels could lead to deadlock - 

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 2)
	ch <- 1
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}

```

Output - 

```go
1
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:
main.main()
	/home/kashilkarkunal/go/src/channelgo/main.go:9 +0xf1
```

