[![Releases](https://img.shields.io/badge/Releases-v1.0-blue?style=for-the-badge)](https://github.com/Savge-king/golang-intermediate/releases)

# Intermediate Golang Concepts: CLI, Interfaces, Pointers, JSON

<img src="https://blog.golang.org/gopher/gopher.png" alt="Go Gopher" width="140" style="float:right; margin: 0 0 12px 12px;" />

A practical guide to core intermediate Go topics. This repo collects focused examples and patterns you will use when building real Go programs. It covers CLI tools, closures, interface design, pointers, methods, JSON handling, regex, runes, recursion, time formatting, and other items you will meet after the basics.

- Topics: cli, closures, epoch, fmt, golang-interface, golang-package, json, methods, pointers-go, recursion, regex, runes, structs-go, time-formatting, verbs
- Releases: Download the executable from the Releases page and run it: https://github.com/Savge-king/golang-intermediate/releases

Table of contents
- What this repo contains
- Quick start (download and run release)
- Guided examples by topic
  - CLI and flags
  - Closures and function values
  - Pointers and methods
  - Interfaces and composition
  - Structs, JSON, and encoding
  - Runes and regex
  - Recursion and epochs
  - Time formatting and verbs
- Layout and how to run code locally
- Contributing, testing, and coding style
- License and links

What this repo contains
- Short, focused programs you can run.
- Small libraries that show idiomatic patterns.
- Tests that show intended behavior.
- Example data files and sample JSON.
- A CLI tool that bundles several demos (see Releases).

Quick start (download and run release)
- Visit the Releases page and download the asset that matches your OS and architecture: https://github.com/Savge-king/golang-intermediate/releases
- The release will include a platform binary or a compressed archive. Download the binary file named with the pattern golang-intermediate_<os>_<arch> or the archive that contains it.
- Make the file executable and run it on your machine:
  - Linux / macOS (example): chmod +x golang-intermediate_linux_amd64 && ./golang-intermediate_linux_amd64
  - Windows (example): double-click golang-intermediate_windows_amd64.exe or run it from PowerShell.
- The binary exposes small subcommands that map to the topics in this repo. Run the CLI with --help to see options.

Guided examples by topic

CLI and flags
- Use the standard flag package for small CLIs and cobra or urfave/cli for larger tools.
- Keep the main function slim. Move logic to packages.
- Example pattern:
  - package cmd for command wiring
  - package internal for core logic
- Example snippet:
```go
func run(args []string) error {
    fs := flag.NewFlagSet("demo", flag.ExitOnError)
    name := fs.String("name", "world", "name to greet")
    fs.Parse(args)
    fmt.Printf("Hello, %s\n", *name)
    return nil
}
```

Closures and function values
- Use closures to capture local state for callbacks and iterators.
- Return function types when you want a factory pattern.
- Example pattern: a counter closure.
```go
func counter(start int) func() int {
    i := start
    return func() int {
        i++
        return i
    }
}
```
- Closures make tests easier when you replace state with function instances.

Pointers and methods
- Use pointers for methods that mutate state or for large structs.
- Use value receivers for small immutable types.
- Example:
```go
type Buffer struct {
    data []byte
}

func (b *Buffer) Append(p []byte) {
    b.data = append(b.data, p...)
}

func (b Buffer) Len() int {
    return len(b.data)
}
```
- Prefer pointer receivers when any method needs to mutate the receiver.

Interfaces and composition
- Design interfaces small and focused. One-method interfaces work well in many cases.
- Use interface satisfaction by value or pointer depending on the receiver.
- Example:
```go
type Writer interface {
    Write(p []byte) (int, error)
}

func Save(w Writer, data []byte) error {
    _, err := w.Write(data)
    return err
}
```
- Compose behaviors by embedding interfaces in structs.

Structs, JSON, and encoding
- Use struct tags to map JSON keys and to control omitempty.
- Keep exported fields capitalized for encoding.
- Example:
```go
type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email,omitempty"`
}
```
- Decode to concrete types or to map[string]interface{} for flexible input.
- Example of encoding:
```go
u := User{ID: 1, Name: "Sam"}
b, _ := json.Marshal(u)
fmt.Println(string(b))
```

Runes and regex
- Treat runes when you need to handle Unicode code points rather than bytes.
- Use []rune(s) to safely index characters.
- Use regexp for matching. Precompile with regexp.MustCompile when you reuse the pattern.
- Example:
```go
r := []rune("Gophér")
fmt.Println(r[1]) // code point for 'o' or accent
pattern := regexp.MustCompile(`^[a-zA-Z0-9_-]+$`)
ok := pattern.MatchString("valid_name")
```

Recursion and epochs
- Use recursion for tree traversal, DFS, and divide-and-conquer.
- Keep recursion depth in mind and prefer iteration when the stack may grow large.
- Epoch and timestamp handling: prefer time.Time for conversions and use Unix/UnixNano for epoch values.
- Example:
```go
func sumTree(n *Node) int {
    if n == nil { return 0 }
    return n.Value + sumTree(n.Left) + sumTree(n.Right)
}
```

Time formatting and verbs
- Use time.Format with layout values built from a reference time Mon Jan 2 15:04:05 -0700 MST 2006.
- Use layout constants for ISO and RFC formats.
- Example:
```go
t := time.Now()
fmt.Println(t.Format(time.RFC3339))
fmt.Println(t.Format("2006-01-02 15:04:05"))
```
- For parsing epoch seconds: time.Unix(epoch, 0).UTC().

Practical patterns and idioms
- Error handling: return errors and wrap context with fmt.Errorf("action: %w", err).
- Testing: keep tests fast and deterministic. Use table-driven tests.
- Formatting: use gofmt or gofmt -w to format code.
- Documentation: write short doc comments for exported types and functions.

Layout and how to run code locally
- Clone the repo:
  - git clone https://github.com/Savge-king/golang-intermediate.git
- Build the CLI:
  - go build ./cmd/golang-intermediate
- Run examples from the examples or cmd folder. Use go test ./... to run tests.

Example project structure
- cmd/
  - golang-intermediate/  # CLI entry
- pkg/
  - cli/                  # small CLI helpers
  - jsonx/                # JSON helpers and types
  - grep/                 # regex examples
- examples/
  - closures/
  - pointers/
  - recursion/
- docs/
  - cheatsheets.md
- scripts/
  - build-release.sh

Contributing, testing, and coding style
- Open an issue for a bug or improvement.
- Fork and create a PR for a change.
- Run go test ./... and make sure tests pass.
- Keep functions short. Prefer clear names over clever code.
- Use plain comments to explain intent, not behavior.

Common commands
- Build: go build ./...
- Test: go test ./...
- Format: gofmt -w .
- Vet: go vet ./...

Helpful links and images
- Official Go site: https://golang.org
- Go docs and formats: https://pkg.go.dev
- Go Gopher artwork: https://blog.golang.org/gopher
- Releases page (download and run the provided binary): https://github.com/Savge-king/golang-intermediate/releases

Examples and snippets in this repo aim to be small, focused, and ready for copy-paste. The CLI binary available in Releases bundles demo commands that run these snippets from the command line. Download the right asset for your OS from the Releases page and execute the binary to try the demos locally: https://github.com/Savge-king/golang-intermediate/releases

License
- MIT License. Check the LICENSE file for details.