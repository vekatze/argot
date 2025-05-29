# argot

`argot` is an option parser for the [Neut](https://vekatze.github.io/neut/) programming language.

## Installation

```sh
neut get argot https://github.com/vekatze/argot/raw/main/archive/0-1-26.tar.zst
```

## Types

### Basics

```neut
// an opaque type that holds the internal state of the parser
data argot-kit

// creates an `argot-kit`
define make-argot-kit(args: list(text)): argot-kit

// creates an `argot-kit` from argv
define make-argot-kit-from-argv(): argot-kit

// the type of parsers
inline argot(a) {
  (k: &argot-kit) -> either(error, a)
}

// convert an error to human-readable text
define report(e: error): text
```

### Parsers

```neut
// makes given parser optional
define optional<a>(p: argot(a)): argot(?a)

// runs given parser repeatedly until it fails
inline many<a>(p: argot(a)): argot(list(a))

// tries given parsers one by one
define choice<a>(ps: list(argot(a)), fallback: argot(a)): argot(a)

// succeeds only if there are no remaining arguments
define end-of-input: argot(unit)
```

### Presets

```neut
// parses a flag (`--enable-color`)
define flag(key: &text): argot(bool)

// parses an integer (`--my-key 1234`)
define int-required(key: &text): argot(int)

// parses a float (`--my-key 23.45`)
define float-required(key: &text): argot(float)

// parses a text (`--clang-option "-fsanitize=address"`)
define text-required(key: &text): argot(text)

// parses a positional argument (the `foo` in `my-command foo`)
define text-argument(name: &text): argot(text)
```

### Utilities

```neut
define get-num-of-items(k: &argot-kit): int

define export-state(k: &argot-kit): list(text)

define import-state(k: &argot-kit, xs: list(text)): unit

// state == ["foo", "bar", "buz", "qux"]
// ↓ pop-key(k, "bar")
// state == ["foo", "buz", "qux"], output == True
define pop-key(k: &argot-kit, key: &text): bool

// state == ["foo", "bar", "buz", "qux"]
// ↓ pop-key-value(k, "bar")
// state == ["foo", "qux"], output == Right("buz")
define pop-key-value(k: &argot-kit, key: &text): either(error, text)

// state == ["foo", "bar", "buz", "qux"]
// ↓ pop-head(k)
// state == ["bar", "buz", "qux"], output == Right("bar")
define pop-head(k: &argot-kit): either(error, text)
```

## Example

```neut
// defines a type that stores the result of argument parsing
data config {
| Config(v1: bool, v2: bool, v3: int, v4: text, v5: list(text))
}

// defines an argument parser
define my-argument-parser: argot(config) {
  function (k) {
    try v1 = flag("-b")(k) in
    try v2 = flag("-c")(k) in
    try mv3 = optional(int-required("--integer"))(k) in
    try mv4 = optional(text-required("--clang-option"))(k) in
    try v5 = many(text-required("-i"))(k) in
    try _ = end-of-input(k) in
    Right(
      Config of {
        v1,
        v2,
        v3 = from-right(mv3, 123),
        v4 = from-right(mv4, *""),
        v5,
      },
    )
  }
}

// tries the argument parser
define zen(): unit {
  pin k = make-argot-kit-from-argv() in
  let result = my-argument-parser(k) in
  match result {
  | Left(e) =>
    printf("{}\n", [report(e)])
  | Right(v) =>
    let Config of {v1, v2, v3, v4, v5} = v in
    (..)
  }
}
```

The parser `my-argument-parser()` can handle arguments like the following:

```
-b --integer 123 -i test -i hoge
```
