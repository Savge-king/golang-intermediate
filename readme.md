## **1. What is a Closure in Go?**

In Go, a **closure** is a function value that **captures variables from the scope in which it was defined**, even after that scope has finished executing.

That means:

* The **inner function** remembers the environment (variables) from where it was created.
* These captured variables **live on in memory** as long as the returned function value exists.
* Each returned function has its **own copy** of those variables.

---

### **Analogy**

Think of closures like:

> A backpack 🎒 that a function carries with it wherever it goes, containing the variables from the place where it was born.

---

## **2. The `adder()` example**

```go
func adder() func() int {
    i := 0
    fmt.Println("Previous value of i:", i)

    return func() int {
        i++
        fmt.Println("Adding 1 to i")
        return i
    }
}
```

### Step-by-Step:

1. When `adder()` is called, it creates a **local variable** `i := 0`.
2. It returns an **anonymous function** (`func() int { ... }`) that:

   * Accesses and modifies `i` even after `adder()` has finished running.
3. This works because the **closure** keeps a reference to `i` in memory (on the heap, not the stack).

---

### How `main()` uses it:

```go
sequence := adder()

fmt.Println(sequence())
fmt.Println(sequence())
fmt.Println(sequence())
```

**Execution flow:**

* `sequence := adder()`

  * Calls `adder()` → sets `i = 0` → returns a closure that can increment `i`.
  * `sequence` now holds that closure **and the "backpack" with i**.
* First `sequence()` → increments `i` to 1 → prints and returns 1.
* Second `sequence()` → increments `i` to 2 → prints and returns 2.
* Third `sequence()` → increments `i` to 3 → prints and returns 3.
* Fourth → increments to 4.

---

### Important: `sequence2 := adder()`

* This **calls `adder()` again**, creating a **new and independent `i`** starting from 0.
* That’s why `sequence2()` starts counting from 1 again.
* Each closure has **its own copy** of `i`.

---

## **3. The `multiplier` example inside `main`**

```go
multiplier := func() func(int) int {
    product := 2
    return func(x int) int {
        product *= x
        return product
    }
}()
```

### What’s happening:

1. We define an **anonymous function** that:

   * Declares `product := 2`
   * Returns a closure `func(x int) int` that multiplies `product` by `x`.
2. The `()` at the end **calls it immediately**, so `multiplier` becomes:

   * The closure `func(int) int` with `product` in its backpack.

**Usage:**

```go
fmt.Println(multiplier(5)) // product = 2 * 5 = 10
fmt.Println(multiplier(3)) // product = 10 * 3 = 30
```

* Notice that `product` **persists between calls**, because it’s stored in the closure’s environment.

---

## **4. Memory Perspective (Heap vs Stack)**

* Normally, a local variable like `i` or `product` would be on the **stack**, and vanish after the function returns.
* But closures need those variables **after the function exits**.
* Go’s compiler detects this and **allocates them on the heap** instead.
* The closure stores a reference to them → as long as the closure is alive, the variables stay alive in memory.

---

## **5. Why Closures are Useful**

* **Stateful functions** without global variables.
* **Factory functions** (like `adder` and `multiplier`) that produce customized behavior.
* **Callbacks** that need to remember context.
* **Encapsulation**: we hide variables from the outside world but keep them accessible to the inner function.

---

## **1. What is Recursion?**

Recursion is when a function **calls itself**, either **directly** or **indirectly**, to solve a problem.

A recursive function:

1. Has a **base case** — a condition where it stops calling itself.
2. Has a **recursive case** — where it calls itself with smaller/simpler input.

---

### **General Syntax in Go**

```go
func functionName(args) returnType {
    if baseCondition {
        return baseValue
    }
    // recursive case
    return functionName(smallerArgs)
}
```

---

## **2. Example — Factorial**

Mathematically:

```
n! = n × (n-1) × (n-2) × ... × 1
0! = 1 (base case)
```

**In Go:**

```go
package main

import "fmt"

func factorial(n int) int {
    if n == 0 {
        return 1 // base case
    }
    return n * factorial(n-1) // recursive call
}

func main() {
    fmt.Println(factorial(5)) // Output: 120
}
```

**How it works:**

```
factorial(5) → 5 * factorial(4)
factorial(4) → 4 * factorial(3)
factorial(3) → 3 * factorial(2)
factorial(2) → 2 * factorial(1)
factorial(1) → 1 * factorial(0)
factorial(0) → 1  (base case reached)

Now it returns:
5 * 4 * 3 * 2 * 1 = 120
```

---

## **3. Example — Fibonacci**

Fibonacci sequence:

```
0, 1, 1, 2, 3, 5, 8, ...
fib(n) = fib(n-1) + fib(n-2)
```

**In Go:**

```go
func fibonacci(n int) int {
    if n <= 1 {
        return n // base cases: fib(0)=0, fib(1)=1
    }
    return fibonacci(n-1) + fibonacci(n-2) // recursive case
}
```

---

## **4. Important Points in Go Recursion**

* **Base case is mandatory** — without it, recursion runs forever and crashes with a stack overflow.
* Go optimizes tail recursion *sometimes*, but not as aggressively as some languages (like Scheme).
* Recursion is more readable for problems like:

  * Tree traversal
  * Divide and conquer algorithms
  * Mathematical definitions
* But **iteration is often faster** in Go due to lower function call overhead.

---

## **5. Stack vs Heap in Recursion**

* Every recursive call adds a **stack frame** (variables, return address).
* Deep recursion can cause **stack overflow** if too many calls happen before hitting the base case.
* That’s why we must ensure **small inputs progress toward the base case**.

---

# What is a pointer?

A **pointer** is a value that holds the **memory address** of another value. In Go a pointer type is written `*T` — “pointer to `T`”.

```go
var a int = 42
var p *int = &a   // p is a pointer to a
fmt.Println(*p)   // 42   (dereference)
```

* `&` — **address-of** operator: `&a` returns `*int`.
* `*` — **dereference** operator: `*p` gives the value stored at that address.

# Why use pointers?

* **Modify caller’s value**: functions receive copies by default; pass a pointer to let the callee change the original.
* **Avoid large copies**: pass pointer to big structs instead of copying them.
* **Shared mutable state** when multiple parts of code should see updates.

# Basic examples

**Modify an int via pointer**

```go
func inc(x *int) {
    *x++         // change the caller's variable
}
n := 5
inc(&n)
fmt.Println(n) // 6
```

**Swap two values**

```go
func swap(a, b *int) {
    *a, *b = *b, *a
}
```

# nil pointers and panics

* A pointer’s zero value is `nil`.
* **Dereferencing `nil`** causes a runtime panic: `panic: runtime error: invalid memory address or nil pointer dereference`.

```go
var p *int
fmt.Println(p == nil) // true
// fmt.Println(*p) // panic
```

Always check for `nil` before dereferencing if it might be nil.

# Pointers & composite types

* **Pointer to struct**

  ```go
  type User struct { Name string }
  u := &User{Name: "Skyy"} // shorthand, allocates and returns *User
  ```

* **Pointer to array**

  ```go
  arr := [3]int{1,2,3}
  var pa *[3]int = &arr
  fmt.Println(pa[1]) // 2  (pointer-to-array supports indexing)
  ```

* **Slices, maps, channels** are *reference types* already (they contain internal pointers). You rarely need `*[]T` or `*map[K]V`. Use pointer to slice only if you must replace the slice header itself (length/capacity) in the caller.

# `new` vs `&T{}` vs `make`

* `new(T)` returns `*T` with zeroed T (allocates memory, may be on heap).

  ```go
  p := new(int) // p *int, *p == 0
  ```
* `&T{}` or `&MyStruct{Field:...}` is common and idiomatic — often the compiler allocates on the heap if needed (escape analysis).
* `make` is for **slices, maps, channels** (it returns an initialized value, not a pointer).

# Returning pointer to local variable — safe

Go performs **escape analysis**. If a local variable is returned via pointer, the compiler will move it to the heap — that’s safe.

```go
func newCounter() *int {
    i := 0
    return &i   // safe: i escapes to heap
}
```

# Pointer receivers vs value receivers (methods)

* **Pointer receiver** `func (p *T) M()`:

  * Method can modify the receiver.
  * Avoids copying large structs.
* **Value receiver** `func (t T) M()`:

  * Method gets a copy; cannot change original.
  * Good for small, immutable receivers.

**Note:** you can call pointer-receiver methods on addressable values — the compiler will take the address automatically (`v.M()` becomes `(&v).M()` if `v` is addressable). But an interface holding a `T` value does not automatically become `*T`.

```go
type Big struct{ buf [1024]byte }

func (b *Big) Zero() { ... }  // pointer receiver to avoid copying
```

# Pointer equality and comparison

* You can compare pointers: `p == q`, `p != nil`.
* Two pointers are equal when they point to the same memory location.

# Pointer to pointer

```go
var x int = 1
var p *int = &x
var pp **int = &p
fmt.Println(**pp) // 1
```

# No pointer arithmetic

Go **does not** allow pointer arithmetic like C. That makes programs safer and more portable.

# Concurrency considerations

Pointers are frequently shared across goroutines. **Sharing mutable state requires synchronization** (e.g., `sync.Mutex`, `sync/atomic`) to avoid race conditions.

```go
// Danger: race if two goroutines modify shared *int without sync
```

Use the Go race detector (`go run -race`) in development.

# Performance trade-offs

* Passing a pointer avoids copying big values — saves CPU and memory.
* But pointers may force heap allocation (GC overhead) and hurt cache locality. For small structs, prefer value receivers and passing by value.
* Measure (benchmarks) rather than guessing.

# Unsafe pointers

* `unsafe.Pointer` exists and can do conversions or mimic pointer arithmetic — **dangerous** and platform-dependent. Use only when necessary and with full understanding.

# Practical idioms & patterns

* **Factory returning pointer**

  ```go
  func NewUser(name string) *User { return &User{Name: name} }
  ```

* **Mutable method via pointer receiver**

  ```go
  func (u *User) SetName(n string) { u.Name = n }
  ```

* **Avoid pointer-to-slice/map unless needed** — slices/maps/channels already behave like references.

* **Use pointer for large structs** or when mutation is required.

# Example: using pointer receiver vs value receiver

```go
type Counter struct { n int }

func (c *Counter) Inc() { c.n++ }      // modifies receiver
func (c Counter) Value() int { return c.n } // read-only

c := Counter{}
c.Inc()             // works; compiler takes &c
fmt.Println(c.Value())
```

# Debugging tips

* `fmt.Printf("%p\n", p)` prints pointer address.
* `go vet`, `go test -race` find misuse in concurrent code.
* Be mindful of `nil` checks before deref.

# Quick checklist for using pointers

* Want to **modify** the argument? → pass pointer.
* Passing **large struct**? → consider pointer to avoid copying.
* Returning early-created value? → `&value` is fine (escape analysis handles heap).
* Using slices/maps/channels? → usually **no pointer needed**.
* Sharing across goroutines? → **sync** or use channels.

---

## Short summary

* Pointers (`*T`) hold addresses; `&` & `*` are address-of and dereference.
* They let you mutate caller’s data and avoid copying large values.
* Nil pointer dereferences panic — always guard if needed.
* Go forbids pointer arithmetic; `unsafe` exists for low-level cases.
* Use pointers for mutable/large data, but measure performance and be mindful of GC and concurrency.

Here’s the benchmark result for simulating a large struct in Python to compare **pass-by-value** vs **pass-by-reference**:

* **Pass-by-value simulation** (copy-like behavior): **\~0.02948 seconds**
* **Pass-by-reference simulation** (pointer-like behavior): **\~0.000096 seconds**

📌 **Interpretation in the context of Go:**

* In Go, when we pass large structs **by value**, it copies all the fields to a new memory location (on the stack or heap), which is **slower** and uses **more memory**.
* When we pass them **by pointer** (e.g., `*MyStruct`), we’re only passing an **8-byte memory address**, so it’s much **faster** and avoids extra memory allocations.
* This is exactly why in real-world Go projects, for large structs, functions often take `*Struct` instead of `Struct`.
---

## **1️⃣ What is a String in Go?**

In Go:

```go
str := "Hello"
```

* A **string** is **immutable** — we cannot change its contents once created.
* Internally, a `string` is just a **read-only slice of bytes**:

  * **Pointer to the underlying byte array**
  * **Length of the string**

We can visualize:

```
"Hello"
[Pointer] ───▶ [72 101 108 108 111]  // ASCII byte values
Length = 5
```

📌 **Key points:**

* Strings are stored as **UTF-8 encoded bytes**.
* Since UTF-8 can take 1–4 bytes per character, indexing a string directly (`str[i]`) gives **bytes**, **not characters**.

---

## **2️⃣ UTF-8 and Why `str[i]` Might Not Work for All Characters**

Example:

```go
str := "Hé"
fmt.Println(len(str))  // 3
fmt.Println(str[0])    // 72 ('H')
fmt.Println(str[1])    // 195 (part of 'é')
fmt.Println(str[2])    // 169 (rest of 'é')
```

Here, `"é"` takes **2 bytes** (`0xC3 0xA9`), so:

* Index `1` gives the **first byte** of `é`, not the whole character.

---

## **3️⃣ What is a Rune in Go?**

* A **rune** is just an alias for `int32` in Go.
* It represents a **Unicode code point** — the numeric value of a character.
* Unlike a byte, a rune **can store the entire character**, even if it’s multi-byte in UTF-8.

Example:

```go
var r rune = 'é'
fmt.Println(r)         // 233 (Unicode code point)
fmt.Printf("%c\n", r)  // é
```

---

## **4️⃣ Converting Strings to Runes**

If we want to process actual **characters** instead of raw bytes:

```go
str := "Hé😊"
runes := []rune(str)
fmt.Println(len(runes)) // 3 (characters)
fmt.Println(runes)      // [72 233 128522]
```

Here:

* `"H"` → `72`
* `"é"` → `233`
* `"😊"` → `128522` (U+1F60A)

---

## **5️⃣ Iterating Over a String Correctly**

🔴 **Wrong way** (byte by byte):

```go
for i := 0; i < len(str); i++ {
    fmt.Printf("%c ", str[i]) // May print broken chars for multi-byte
}
```

✅ **Correct way** (range over runes):

```go
for i, r := range str {
    fmt.Printf("Index %d: %c\n", i, r)
}
```

* `range` automatically decodes UTF-8 into runes.
* `i` will be the **byte index**, not rune index.

---

## **6️⃣ Memory & Performance Notes**

| Type     | Size     | Meaning                            | Use Case                    |
| -------- | -------- | ---------------------------------- | --------------------------- |
| `byte`   | 1 byte   | ASCII value / raw byte             | Low-level byte handling     |
| `rune`   | 4 bytes  | Unicode code point (UTF-8 decoded) | Proper character handling   |
| `string` | 16 bytes | Read-only byte slice (pointer+len) | Storing immutable text data |

* Strings are **immutable**, so modifying them means creating a **new string**.
* Runes are stored in UTF-8 form inside strings, so multi-byte characters are **not** directly indexable.

---

## **7️⃣ Real-world Example**

```go
package main

import "fmt"

func main() {
    str := "Go💙"

    fmt.Println("Length in bytes:", len(str))       // 6
    fmt.Println("Length in runes:", len([]rune(str))) // 3

    fmt.Println("\nIterating over runes:")
    for i, r := range str {
        fmt.Printf("Byte index: %d, Rune: %c, CodePoint: %U\n", i, r, r)
    }
}
```

**Output:**

```
Length in bytes: 6
Length in runes: 3

Iterating over runes:
Byte index: 0, Rune: G, CodePoint: U+0047
Byte index: 1, Rune: o, CodePoint: U+006F
Byte index: 2, Rune: 💙, CodePoint: U+1F499
```

---

✅ **In short:**

* **String** = Immutable UTF-8 bytes.
* **Rune** = Unicode code point (`int32`).
* Use `[]byte` for raw byte processing, `[]rune` for character-safe processing.
* Always remember UTF-8 characters can be multi-byte — `str[i]` is not always a full character.

---

## **1️⃣ What Are Formatting Verbs?**

In Go, **formatting verbs** are special placeholders we use inside `fmt` functions like `fmt.Printf`, `fmt.Sprintf`, etc., to control **how data is printed**.

Example:

```go
name := "Skyy"
age := 29
fmt.Printf("Name: %s, Age: %d\n", name, age)
```

Output:

```
Name: Skyy, Age: 29
```

Here:

* `%s` → prints a string
* `%d` → prints a decimal integer

---

## **2️⃣ Common Categories of Formatting Verbs**

Let’s break them down:

---

### **A. General Verbs**

| Verb  | Meaning                                     |
| ----- | ------------------------------------------- |
| `%v`  | Default format (auto-chooses based on type) |
| `%+v` | Like `%v`, but with struct field names      |
| `%#v` | Go syntax representation of the value       |
| `%T`  | Type of the value                           |
| `%%`  | Literal `%` character                       |

Example:

```go
type Person struct {
    Name string
    Age  int
}
p := Person{"Alice", 30}

fmt.Printf("%v\n", p)   // {Alice 30}
fmt.Printf("%+v\n", p)  // {Name:Alice Age:30}
fmt.Printf("%#v\n", p)  // main.Person{Name:"Alice", Age:30}
fmt.Printf("%T\n", p)   // main.Person
fmt.Printf("%%\n")      // %
```

---

### **B. Boolean**

| Verb | Meaning       |
| ---- | ------------- |
| `%t` | true or false |

```go
fmt.Printf("%t\n", true) // true
```

---

### **C. Integer**

| Verb | Meaning                             |
| ---- | ----------------------------------- |
| `%d` | Decimal (base 10)                   |
| `%b` | Binary                              |
| `%o` | Octal                               |
| `%x` | Hexadecimal (lowercase)             |
| `%X` | Hexadecimal (uppercase)             |
| `%c` | Character (from Unicode code point) |
| `%q` | Quoted character literal            |

Example:

```go
n := 65
fmt.Printf("%d\n", n)  // 65
fmt.Printf("%b\n", n)  // 1000001
fmt.Printf("%o\n", n)  // 101
fmt.Printf("%x\n", n)  // 41
fmt.Printf("%X\n", n)  // 41
fmt.Printf("%c\n", n)  // A
fmt.Printf("%q\n", n)  // 'A'
```

---

### **D. Floating Point & Complex Numbers**

| Verb | Meaning                                 |
| ---- | --------------------------------------- |
| `%f` | Decimal point, no exponent              |
| `%F` | Same as `%f`                            |
| `%e` | Scientific notation (e.g., `1.23e+03`)  |
| `%E` | Scientific notation (uppercase)         |
| `%g` | `%e` or `%f`, whichever is more compact |
| `%G` | `%E` or `%F`, whichever is more compact |
| `%x` | Hexadecimal fraction (lowercase)        |
| `%X` | Hexadecimal fraction (uppercase)        |

Example:

```go
pi := 3.1415926535
fmt.Printf("%f\n", pi)  // 3.141593
fmt.Printf("%.2f\n", pi) // 3.14
fmt.Printf("%e\n", pi)  // 3.141593e+00
fmt.Printf("%g\n", pi)  // 3.14159
```

---

### **E. String & Slice of Bytes**

| Verb | Meaning                          |
| ---- | -------------------------------- |
| `%s` | Plain string                     |
| `%q` | Double-quoted string (Go syntax) |
| `%x` | Hexadecimal (lowercase)          |
| `%X` | Hexadecimal (uppercase)          |

Example:

```go
s := "GoLang"
fmt.Printf("%s\n", s)  // GoLang
fmt.Printf("%q\n", s)  // "GoLang"
fmt.Printf("%x\n", s)  // 476f4c616e67
fmt.Printf("%X\n", s)  // 476F4C616E67
```

---

### **F. Pointer**

| Verb | Meaning        |
| ---- | -------------- |
| `%p` | Pointer in hex |

Example:

```go
x := 42
fmt.Printf("%p\n", &x) // e.g., 0xc000014090
```

---

## **3️⃣ Width, Precision, and Flags**

We can fine-tune formatting:

* **Width**: Minimum number of characters (`%6d` → pad with spaces if less than 6 digits)
* **Precision**: Digits after decimal for floats (`%.2f` → 2 decimal places)
* **Flags**:

  * `-` → Left-justify
  * `+` → Always show sign
  * `0` → Pad with zeros

Example:

```go
n := 42
pi := 3.14159

fmt.Printf("|%6d|\n", n)    // |    42|
fmt.Printf("|%-6d|\n", n)   // |42    |
fmt.Printf("|%06d|\n", n)   // |000042|
fmt.Printf("%.2f\n", pi)    // 3.14
fmt.Printf("%+f\n", pi)     // +3.141590
```

---

## **4️⃣ Summary Table of Most Common Verbs**

| Verb        | Usage                   |
| ----------- | ----------------------- |
| `%v`        | Default value           |
| `%+v`       | Struct with field names |
| `%#v`       | Go syntax               |
| `%T`        | Type                    |
| `%t`        | Boolean                 |
| `%d`        | Decimal integer         |
| `%b`        | Binary                  |
| `%o`        | Octal                   |
| `%x` / `%X` | Hexadecimal             |
| `%c`        | Unicode character       |
| `%q`        | Quoted string/char      |
| `%f`        | Decimal float           |
| `%e` / `%E` | Scientific notation     |
| `%g` / `%G` | Compact float           |
| `%p`        | Pointer address         |

---

### **String Comparison in GoLang** (Lexicographical Order)

In GoLang, when we compare two strings using relational operators (`<`, `>`, `==`, etc.), the comparison is done **lexicographically** — meaning it follows the order of Unicode code points, very similar to how words are sorted in a dictionary.

#### **How It Works**

* Go compares strings **byte by byte**, starting from the first character of both strings.
* It uses the **Unicode code point** values (UTF-8 encoded) for comparison.
* The comparison stops as soon as it finds the first differing byte between the two strings.
* If all compared characters are equal and the strings are of different lengths, the shorter string is considered "less" than the longer one.

---

#### **Example**

```go
str1 := "Apple 🍎"
str2 := "Banana 🍌"
str3 := "App 📱"
str4 := "ab 🔤"

fmt.Println(str1 < str2) // true
fmt.Println(str3 < str1) // true
fmt.Println(str4 < str1) // false
```

**Step-by-step reasoning:**

1. `"Apple 🍎" < "Banana 🍌"` → We compare `'A'` (65) with `'B'` (66). Since `65 < 66`, result is `true`.
2. `"App 📱" < "Apple 🍎"` → First three characters `"App"` match. Next, space `' '` (32) is less than `'l'` (108), so result is `true`.
3. `"ab 🔤" < "Apple 🍎"` → First characters: `'a'` (97) vs `'A'` (65). Since `97 > 65`, result is `false`.

---

💡 **Important Notes:**

* Go is **case-sensitive** in string comparisons because Unicode code points for uppercase and lowercase letters are different.
* Emojis and non-ASCII characters also follow their Unicode code point order.
* If we want case-insensitive comparison, we can use functions from the `strings` package like `strings.EqualFold()`.

---

Here’s our **Go Formatting Verbs Cheat Sheet** so we can quickly reference them whenever we print or format values in Go.
We’ll group them by category for clarity.

---

## **📌 1. General Verbs**

| Verb  | Meaning                  | Example Output                  |
| ----- | ------------------------ | ------------------------------- |
| `%v`  | Default format           | `fmt.Printf("%v", 123)` → `123` |
| `%+v` | Struct with field names  | `"{Name:Skyy Age:29}"`          |
| `%#v` | Go-syntax representation | `"[]int{1, 2, 3}"`              |
| `%T`  | Type of the value        | `"int"`, `"string"`             |
| `%%`  | Literal percent sign     | `"50%"`                         |

---

## **📌 2. Boolean**

| Verb | Meaning           | Example  |
| ---- | ----------------- | -------- |
| `%t` | `true` or `false` | `"true"` |

---

## **📌 3. Integer**

| Verb | Meaning                   | Example      |
| ---- | ------------------------- | ------------ |
| `%b` | Base 2 (binary)           | `1010`       |
| `%c` | Character from code point | `'A'` for 65 |
| `%d` | Base 10 (decimal)         | `123`        |
| `%o` | Base 8 (octal)            | `173`        |
| `%q` | Quoted character          | `'A'`        |
| `%x` | Base 16 lowercase         | `7b`         |
| `%X` | Base 16 uppercase         | `7B`         |
| `%U` | Unicode format            | `U+0041`     |

---

## **📌 4. Floating-Point & Complex Numbers**

| Verb        | Meaning                                            | Example      |
| ----------- | -------------------------------------------------- | ------------ |
| `%b`        | Exponent as power of two                           | `132p-2`     |
| `%e`        | Scientific notation (lowercase)                    | `1.23e+03`   |
| `%E`        | Scientific notation (uppercase)                    | `1.23E+03`   |
| `%f` / `%F` | Decimal point, no exponent                         | `123.456000` |
| `%g`        | Compact representation (smallest between %e or %f) | `1.23`       |
| `%G`        | Compact representation (%E or %F)                  | `1.23`       |

---

## **📌 5. String & Slice of Bytes**

| Verb | Meaning                   | Example       |
| ---- | ------------------------- | ------------- |
| `%s` | Plain string              | `"Hello"`     |
| `%q` | Quoted string (Go syntax) | `"\"Hello\""` |
| `%x` | Hexadecimal (lowercase)   | `68656c6c6f`  |
| `%X` | Hexadecimal (uppercase)   | `68656C6C6F`  |

---

## **📌 6. Pointer**

| Verb | Meaning               | Example        |
| ---- | --------------------- | -------------- |
| `%p` | Pointer address (hex) | `0xc0000140a0` |

---

## **📌 7. Width & Precision**

* **Width**: Minimum number of characters to print (pads with spaces by default)

  ```go
  fmt.Printf("|%6d|", 45) // |    45|
  ```
* **Precision**: Digits after decimal for floats / max chars for strings

  ```go
  fmt.Printf("|%.2f|", 3.14159) // |3.14|
  fmt.Printf("|%.3s|", "Golang") // |Gol|
  ```
* **Width + Precision**:

  ```go
  fmt.Printf("|%6.2f|", 3.14159) // |  3.14|
  ```

---

## **📌 8. Flags**

| Flag | Meaning                                    | Example             |
| ---- | ------------------------------------------ | ------------------- |
| `-`  | Left align                                 | `%-6d` → `"45    "` |
| `+`  | Always show sign                           | `%+d` → `+45`       |
| `0`  | Pad with zeros                             | `%06d` → `000045`   |
| `#`  | Alternate form (e.g., `0x` prefix for hex) | `%#x` → `0x7b`      |

---