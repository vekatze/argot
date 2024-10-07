# argot

`argot` is a monadic option parser for the [Neut](https://vekatze.github.io/neut/) programming language.

## Installation

```sh
neut get argot https://github.com/vekatze/argot/raw/main/archive/0-1-9.tar.zst
```

## Summary

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

// run a parser
define run<a>(mx: parser(a)): either(error, a)

// decode a parse error into a text
define report(e: error): text

// preset: parse a flag (`--enable-color`)
define flag(key: &text): parser(bool)

// preset: parse an integer (`--my-key 1234`)
define int-required(key: &text): parser(int)

// preset: parse a float (`--my-key 23.45`)
define float-required(key: &text): parser(float)

// preset: parse text (`--clang-option "-fsanitize=address"`)
define text-required(key: &text): parser(text)

// preset: parse a positional argument (the `foo` in `my-command foo`)
define text-argument(name: &text): parser(text)

// combinator: make given parser optional
define optional<a>(p: parser(a)): parser(?a)

// combinator: run given parser repeatedly until it fails
inline many<a>(p: parser(a)): parser(list(a))

// combinator: try given parsers one by one
define choice<a>(ps: list(parser(a)), fallback: parser(a)): parser(a)
```

## Example

```neut
// see source/test.nt

data config {
| Config(v1: bool, v2: bool, v3: int, v4: text, v5: list(text))
}

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

The above can read arguments like:

```
-b --integer 123 aoe -i test -i hoge
```
