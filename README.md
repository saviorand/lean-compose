<a name="readme-top"></a>

<div align="center">
  <h3 align="center">lean-compose</h3>

  <p align="center">
    A typed DSL for authoring Compose UI in Lean 4
    <br/>

   ![Written in Lean][language-shield]
   [![Apache 2.0 License][license-shield]][license-url]
   [![Contributors Welcome][contributors-shield]][contributors-url]

  </p>
</div>

## Overview

`lean-compose` describes Jetpack Compose layouts in Lean, following what
[lean-html](https://github.com/paulbutcher/lean-html) does for HTML content models:
the view type is indexed by where a view is allowed to appear, so invalid nesting is
a type error instead of a layout bug you notice on a screen.

```lean
def demoScreen (count : Nat) : View .content :=
  scaffold
    (bar := some (topAppBar "Lean-authored UI"))
    (column (mod := { padding := 16 }) [
      text "This layout was authored in Lean." "titleMedium",
      spacer 12,
      text s!"count = {count}",
      row [ button "increment" "inc", spacer 8, button "reset" "reset" ]
    ])
```

put a button where the top bar goes and it doesn't compile:

```
error: Application type mismatch: The argument
  button "no" "x"
has type
  View Category.content
```

Compose itself makes no such distinction. any `@Composable` nests inside any other,
and a slot expecting a bar will take a button and lay it out wrong.

- [x] content-model categories: `.content`, `.topBar`, `.navItem`
- [x] renders to JSON for a host to interpret
- [x] nothing `partial`, nothing that can panic, no `sorry`, builds with `warningAsError`

## Getting Started

the only dependency is Lean 4.

```bash
lake build
lake test
```

to use it in your own package, add to `lakefile.toml`:

```toml
[[require]]
name = "lean-compose"
git = "https://github.com/saviorand/lean-compose"
```

## How it renders

`View.toJson` emits a tree for a host to interpret rather than generating Kotlin, so
a UI can change without recompiling the app around it.
[lean-android-compose](https://github.com/saviorand/lean-android-compose) has a
`LeanView.kt` that turns this into real Composables.

`toJson` and `childrenJson` are mutually recursive instead of using a nested
`List.map`, because a closure hides the recursive call from Lean's structural
termination checker.

## Tests

structural facts are theorems. the string-level ones are runtime checks instead:
`escape` and `toJson` are built from `String.foldl` and append, neither of which
reduces definitionally, `simp` on a whole document blows the heartbeat limit, and
`native_decide` would only close them by adding a trusted axiom for what's really an
output-format check.

## Roadmap

- [ ] more of the Compose surface: lazy lists, text fields, images
- [ ] ordering constraints within a parent, currently unmodelled
- [ ] render on-device instead of at build time so layouts can depend on runtime
      state. the export is there but the host side
      [still crashes](https://github.com/saviorand/lean-android-compose)

## Contributing

contributions welcome, especially new node kinds and the theorems that should come
with them.

## License

Distributed under the Apache 2.0 License. See [LICENSE](LICENSE) for more information.

## Acknowledgments

* [lean-html](https://github.com/paulbutcher/lean-html) and
  [lean-htmx](https://github.com/paulbutcher/lean-htmx), whose approach this follows

<!-- MARKDOWN LINKS & IMAGES -->
[language-shield]: https://img.shields.io/badge/language-lean4-blueviolet
[license-shield]: https://img.shields.io/github/license/saviorand/lean-compose?logo=github
[license-url]: https://github.com/saviorand/lean-compose/blob/main/LICENSE
[contributors-shield]: https://img.shields.io/badge/contributors-welcome!-blue
[contributors-url]: https://github.com/saviorand/lean-compose#contributing
