# argot

`argot` is a monadic option parser for the [Neut](https://vekatze.github.io/neut/) programming language.

## Installation

```sh
neut get argot https://github.com/vekatze/argot/raw/main/archive/0-1-12.tar.zst
```

## Types

### Basics

```neut
data parser(a) {
| Parser(
    reader: (list(text)) -> pair(either(error, a), list(text)),
  )
}

// monadic return
define return<a>(x: a): parser(a)

// monadic bind
define argument-parser<a, b>(mx: parser(a), k: (a) -> parser(b)): parser(b)

// runs a parser
define run<a>(mx: parser(a)): either(error, a)

// converts a parse error into a human-readable text
define report(e: error): text
```

### Combinators

```neut
// makes given parser optional
define optional<a>(p: parser(a)): parser(?a)

// runs given parser repeatedly until it fails
inline many<a>(p: parser(a)): parser(list(a))

// tries given parsers one by one
define choice<a>(ps: list(parser(a)), fallback: parser(a)): parser(a)
```

### Presets

```neut
// parses a flag (`--enable-color`)
define flag(key: &text): parser(bool)

// parses an integer (`--my-key 1234`)
define int-required(key: &text): parser(int)

// parses a float (`--my-key 23.45`)
define float-required(key: &text): parser(float)

// parses a text (`--clang-option "-fsanitize=address"`)
define text-required(key: &text): parser(text)

// parses a positional argument (the `foo` in `my-command foo`)
define text-argument(name: &text): parser(text)
```

## Example

```neut
// defines a type that stores the result of argument parsing
data config {
| Config(v1: bool, v2: bool, v3: int, v4: text, v5: list(text))
}

// defines an argument parser
define my-argument-parser(): parser(config) {
  with argument-parser {
    bind v1 = flag("-b") in
    bind v2 = flag("-c") in
    bind mv3 = optional(int-required("--integer")) in
    bind mv4 = optional(text-required("--clang-option")) in
    bind v5 = many(text-required("-i")) in
    return(
      Config of {
        v1,
        v2,
        v3 = from-option(mv3, 123),
        v4 = from-option(mv4, *""),
        v5,
      },
    )
  }
}

// tries the argument parser
define zen(): unit {
  let result = run(my-argument-parser()) in
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
