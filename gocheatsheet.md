# Go (Golang) Cheatsheet 🚀

Go is an open-source programming language designed for building simple, reliable, and efficient software.

---

### 1. Basics

* **Hello World:**
    ```go
    package main

    import "fmt"

    func main() {
        fmt.Println("Hello, Go!")
    }
    ```
* **Running Code:**
    ```bash
    go run main.go
    ```
* **Building Executable:**
    ```bash
    go build main.go
    ./main
    ```
* **Variables:**
    * **Declaration with type:**
        ```go
        var name string = "Gopher"
        var age int = 30
        ```
    * **Type inference (short declaration operator `:=`):**
        ```go
        message := "Hello" // string
        count := 10        // int
        isTrue := true     // bool
        ```
    * **Multiple declarations:**
        ```go
        var a, b int = 1, 2
        c, d := "hello", "world"
        ```
* **Constants:**
    ```go
    const Pi = 3.14159
    const (
        StatusOK = 200
        StatusNotFound = 404
    )
    ```
* **Basic Data Types:**
    * `bool` (true/false)
    * Numeric types: `int`, `int8`, `int16`, `int32`, `int64`, `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr`
    * Floating-point types: `float32`, `float64`
    * Complex types: `complex64`, `complex128`
    * `string`
    * `byte` (alias for `uint8`)
    * `rune` (alias for `int32`, represents a Unicode code point)

---

### 2. Control Flow

* **If/Else If/Else:**
    ```go
    x := 10
    if x > 10 {
        fmt.Println("x is greater than 10")
    } else if x == 10 {
        fmt.Println("x is 10")
    } else {
        fmt.Println("x is less than 10")
    }
    ```
* **For Loop (Go's only loop construct):**
    * **Classic for loop:**
        ```go
        for i := 0; i < 5; i++ {
            fmt.Println(i)
        }
        ```
    * **While loop equivalent:**
        ```go
        sum := 1
        for sum < 100 {
            sum += sum
        }
        fmt.Println(sum)
        ```
    * **Infinite loop:**
        ```go
        for {
            // ...
        }
        ```
    * **For-range (iterate over arrays, slices, strings, maps, channels):**
        ```go
        numbers := []int{1, 2, 3}
        for index, value := range numbers {
            fmt.Printf("Index: %d, Value: %d\n", index, value)
        }

        // To only get values:
        for _, value := range numbers {
            fmt.Println(value)
        }
        ```
* **Switch Statement:**
    ```go
    day := "Monday"
    switch day {
    case "Monday":
        fmt.Println("It's Monday!")
    case "Tuesday", "Wednesday": // Multiple cases
        fmt.Println("Midweek")
    default:
        fmt.Println("Weekend")
    }

    // Switch with no condition (like if/else if/else)
    temp := 25
    switch {
    case temp < 0:
        fmt.Println("Freezing")
    case temp >= 0 && temp < 20:
        fmt.Println("Cool")
    default:
        fmt.Println("Warm")
    }
    ```

---

### 3. Functions

* **Defining a Function:**
    ```go
    func greet(name string) {
        fmt.Printf("Hello, %s!\n", name)
    }

    func add(a int, b int) int { // or func add(a, b int) int
        return a + b
    }
    ```
* **Calling a Function:**
    ```go
    greet("Alice")
    result := add(5, 3) // result is 8
    ```
* **Multiple Return Values:**
    ```go
    func swap(x, y string) (string, string) {
        return y, x
    }
    firstName, lastName := swap("John", "Doe")
    ```
* **Named Return Values (often used for clarity/error handling):**
    ```go
    func divide(numerator, denominator float64) (result float64, err error) {
        if denominator == 0 {
            err = fmt.Errorf("cannot divide by zero")
            return // returns 0.0 and the error
        }
        result = numerator / denominator
        return // returns result and nil error
    }
    ```
* **Variadic Functions (accepts a variable number of arguments):**
    ```go
    func sumAll(nums ...int) int {
        total := 0
        for _, num := range nums {
            total += num
        }
        return total
    }
    fmt.Println(sumAll(1, 2, 3, 4)) // 10
    ```

---

### 4. Data Structures

* **Arrays:** Fixed-size sequence of elements of the same type.
    ```go
    var a [5]int // declares an array of 5 integers, initialized to zeros
    a[2] = 10    // assign value
    fmt.Println(a[2])
    b := [3]int{1, 2, 3} // declare and initialize
    ```
* **Slices:** Dynamic-sized, flexible view into elements of an array. More common than arrays.
    ```go
    s := []int{1, 2, 3, 4, 5} // creates a slice with 5 elements
    fmt.Println(s[1:4])       // slice from index 1 (inclusive) to 4 (exclusive) -> [2 3 4]
    s = append(s, 6)          // append element
    fmt.Println(len(s))       // length
    fmt.Println(cap(s))       // capacity

    // Creating slices with make
    t := make([]int, 5)        // slice of 5 ints, all zeroed
    u := make([]int, 0, 5)     // slice of 0 length, capacity 5
    u = append(u, 1, 2)
    ```
* **Maps (Hash Maps/Dictionaries):** Unordered collection of key-value pairs.
    ```go
    m := make(map[string]int) // map with string keys and int values
    m["apple"] = 1
    m["banana"] = 2
    fmt.Println(m["apple"])

    delete(m, "apple") // delete a key-value pair

    value, ok := m["banana"] // check if key exists
    if ok {
        fmt.Println("Banana exists with value:", value)
    }

    // Declare and initialize
    ages := map[string]int{"Alice": 30, "Bob": 25}
    ```
* **Structs:** Typed collection of fields. Similar to classes in other languages, but without methods directly on the struct (methods are defined separately).
    ```go
    type Person struct {
        Name string
        Age  int
    }

    p := Person{Name: "Charlie", Age: 40}
    fmt.Println(p.Name)

    // Anonymous struct
    anon := struct {
        X int
        Y string
    }{X: 10, Y: "hello"}
    ```

---

### 5. Pointers

* Go is "pass by value". Pointers allow functions to modify the original value.
    ```go
    func increment(n *int) { // n is a pointer to an int
        *n++ // dereference n and increment its value
    }

    value := 5
    increment(&value) // pass the address of value
    fmt.Println(value) // 6
    ```
* `&` (address-of operator): Gives the memory address of a variable.
* `*` (dereference operator): Accesses the value stored at a pointer's address.

---

### 6. Methods and Interfaces

* **Methods:** Functions associated with a specific type (receiver).
    ```go
    type Circle struct {
        Radius float64
    }

    // Method with a value receiver
    func (c Circle) Area() float64 {
        return 3.14 * c.Radius * c.Radius
    }

    // Method with a pointer receiver (can modify the struct)
    func (c *Circle) Scale(factor float64) {
        c.Radius *= factor
    }

    myCircle := Circle{Radius: 5}
    fmt.Println(myCircle.Area()) // Call value receiver method

    myCircle.Scale(2) // Call pointer receiver method
    fmt.Println(myCircle.Radius) // Radius is now 10
    ```
* **Interfaces:** Define a set of method signatures. A type implements an interface if it implements all the methods declared by the interface.
    ```go
    type Shape interface {
        Area() float64
    }

    // Circle implicitly implements Shape because it has an Area() method
    // type Circle struct { Radius float64 }
    // func (c Circle) Area() float64 { ... }

    func printArea(s Shape) {
        fmt.Printf("Area: %.2f\n", s.Area())
    }

    c := Circle{Radius: 7}
    printArea(c) // Pass Circle to a function expecting a Shape
    ```

---

### 7. Concurrency (Goroutines and Channels)

* **Goroutines:** Lightweight threads managed by the Go runtime.
    * Start with `go` keyword.
    ```go
    func say(s string) {
        for i := 0; i < 3; i++ {
            fmt.Println(s)
        }
    }

    func main() {
        go say("hello") // Starts a new goroutine
        say("world")    // Runs in the main goroutine
    }
    ```
* **Channels:** Typed conduits through which you can send and receive values with a channel operator (`<-`). Used for communication between goroutines.
    ```go
    func sum(s []int, c chan int) {
        sum := 0
        for _, v := range s {
            sum += v
        }
        c <- sum // Send sum to channel c
    }

    func main() {
        a := []int{7, 2, 8, -9, 4, 0}
        c := make(chan int) // Create a channel of ints

        go sum(a[:len(a)/2], c) // Sum first half
        go sum(a[len(a)/2:], c) // Sum second half

        x, y := <-c, <-c // Receive from channel c (blocking operation)
        fmt.Println(x, y, x+y) // Prints sums and total
    }
    ```
* **Buffered Channels:** Channels with a buffer. Sends are non-blocking as long as the buffer is not full.
    ```go
    ch := make(chan int, 2) // Buffered channel with capacity 2
    ch <- 1
    ch <- 2
    // ch <- 3 // This would block because buffer is full
    fmt.Println(<-ch) // 1
    ```
* **`select` Statement:** Used to wait on multiple channel operations.
    ```go
    c1 := make(chan string)
    c2 := make(chan string)

    go func() {
        time.Sleep(1 * time.Second)
        c1 <- "one"
    }()
    go func() {
        time.Sleep(2 * time.Second)
        c2 <- "two"
    }()

    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-c1:
            fmt.Println("received", msg1)
        case msg2 := <-c2:
            fmt.Println("received", msg2)
        }
    }
    ```
* **`sync` Package:** Provides primitives for low-level synchronization (e.g., `Mutex`, `WaitGroup`).
    ```go
    import (
        "fmt"
        "sync"
    )

    func main() {
        var wg sync.WaitGroup
        for i := 0; i < 5; i++ {
            wg.Add(1) // Increment counter for each goroutine
            go func(i int) {
                defer wg.Done() // Decrement counter when goroutine finishes
                fmt.Printf("Worker %d done\n", i)
            }(i)
        }
        wg.Wait() // Block until counter is zero
        fmt.Println("All workers finished")
    }
    ```

---

### 8. Error Handling

* Go functions typically return `error` as the last return value.
    ```go
    import (
        "errors"
        "fmt"
    )

    func divide(a, b float64) (float64, error) {
        if b == 0 {
            return 0, errors.New("cannot divide by zero")
        }
        return a / b, nil
    }

    func main() {
        res, err := divide(10, 2)
        if err != nil {
            fmt.Println("Error:", err)
        } else {
            fmt.Println("Result:", res)
        }

        res, err = divide(10, 0)
        if err != nil {
            fmt.Println("Error:", err) // Error: cannot divide by zero
        }
    }
    ```
* **Custom Error Types:** Implement the `error` interface (`Error() string` method).
    ```go
    type DivideByZeroError struct {
        Message string
    }

    func (e *DivideByZeroError) Error() string {
        return e.Message
    }

    func safeDivide(a, b float64) (float64, error) {
        if b == 0 {
            return 0, &DivideByZeroError{Message: "division by zero attempted"}
        }
        return a / b, nil
    }

    func main() {
        _, err := safeDivide(10, 0)
        if err != nil {
            if _, ok := err.(*DivideByZeroError); ok {
                fmt.Println("Caught custom divide by zero error:", err)
            } else {
                fmt.Println("Generic error:", err)
            }
        }
    }
    ```
* **`defer` Statement:** Schedules a function call to be run after the surrounding function returns. Often used for cleanup.
    ```go
    func main() {
        defer fmt.Println("world") // This will print after "hello"
        fmt.Println("hello")
    }
    ```

---

### 9. Packages and Modules

* **Packages:** A way to organize Go code. Every Go program is made up of packages.
    * `main` package: The executable program's entry point.
    * `fmt`: For formatted I/O.
    * `os`: Operating system interaction.
    * `io`: Basic I/O primitives.
    * `net/http`: HTTP client and server implementations.
* **Importing Packages:**
    ```go
    import "fmt"
    import (
        "net/http"
        "time"
    )
    ```
* **Modules (Go 1.11+):** Manage dependencies.
    * **Initialize a new module:**
        ```bash
        go mod init <module-path> # e.g., go mod init [example.com/mymodule](https://example.com/mymodule)
        ```
    * **Download dependencies:**
        ```bash
        go mod tidy
        ```
    * **Vendoring dependencies:**
        ```bash
        go mod vendor
        ```

---

### 10. File I/O

* **Reading a file:**
    ```go
    package main

    import (
        "fmt"
        "io/ioutil"
        "os"
    )

    func main() {
        data, err := ioutil.ReadFile("example.txt") // Deprecated in Go 1.16+, use os.ReadFile
        if err != nil {
            fmt.Println("Error reading file:", err)
            return
        }
        fmt.Println(string(data))

        // Go 1.16+
        dataNew, err := os.ReadFile("example.txt")
        if err != nil {
            fmt.Println("Error reading file (new):", err)
            return
        }
        fmt.Println(string(dataNew))
    }
    ```
* **Writing to a file:**
    ```go
    package main

    import (
        "fmt"
        "io/ioutil" // Deprecated in Go 1.16+, use os.WriteFile
        "os"
    )

    func main() {
        content := []byte("Hello, Go file!\n")
        err := ioutil.WriteFile("output.txt", content, 0644) // 0644 is file permission
        if err != nil {
            fmt.Println("Error writing file:", err)
            return
        }
        fmt.Println("File written successfully (old).")

        // Go 1.16+
        err = os.WriteFile("output_new.txt", content, 0644)
        if err != nil {
            fmt.Println("Error writing file (new):", err)
            return
        }
        fmt.Println("File written successfully (new).")
    }
    ```

---

### 11. Testing

* Go has a built-in testing framework. Test files end with `_test.go`.
* Test functions start with `Test` and take `*testing.T` as an argument.

    ```go
    // mymath_test.go
    package mymath

    import "testing"

    func Add(a, b int) int {
        return a + b
    }

    func TestAdd(t *testing.T) {
        result := Add(1, 2)
        expected := 3
        if result != expected {
            t.Errorf("Add(1, 2) = %d; expected %d", result, expected)
        }
    }
    ```
* **Running tests:**
    ```bash
    go test
    ```
* **Running tests with verbose output:**
    ```bash
    go test -v
    ```
* **Running specific test:**
    ```bash
    go test -run TestAdd
    ```