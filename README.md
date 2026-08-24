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

lean-compose describes Jetpack Compose layouts in Lean, following what
[lean-html](https://github.com/paulbutcher/lean-html) does for HTML content models.
The view type is indexed by where a view is allowed to appear, so invalid nesting is
a type error rather than a layout bug found at runtime.

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

Passing a button where the top bar belongs does not compile:

```
error: Application type mismatch: The argument
  button "no" "x"
has type
  View Category.content
```

Compose makes no such distinction of its own: any `@Composable` nests inside any
other, and a slot expecting a bar will accept a button and lay it out incorrectly.

- [x] Content-model categories: `.content`, `.topBar`, `.navItem`
- [x] Renders to JSON for a host to interpret
- [x] Nothing `partial`, nothing that can panic, no `sorry`, builds with `warningAsError`

## Getting Started

The only dependency is Lean 4.

```bash
lake build
lake test
```

To use it in your own package, add the following to `lakefile.toml`:

```toml
[[require]]
name = "lean-compose"
git = "https://github.com/saviorand/lean-compose"
```

## Rendering

`View.toJson` emits a tree for a host to interpret rather than generating Kotlin, so
a UI can change without recompiling the application around it.
[lean-android-compose](https://github.com/saviorand/lean-android-compose) provides a
`LeanView.kt` that turns this into real Composables.

`toJson` and `childrenJson` are mutually recursive rather than using a nested
`List.map`, since a closure hides the recursive call from Lean's structural
termination checker.

## Tests

Structural facts are theorems. The string-level facts are runtime checks instead:
`escape` and `toJson` are built from `String.foldl` and string append, neither of
which reduces definitionally, `simp` on a whole document exhausts the heartbeat
limit, and `native_decide` would close them only by adding a trusted axiom for what
is really an output-format check.

## Roadmap

- [ ] More of the Compose surface: lazy lists, text fields, images
- [ ] Ordering constraints within a parent, currently unmodelled
- [ ] Render on-device rather than at build time, so layouts can depend on runtime
      state. The export is in place, but the host-side path
      [still crashes](https://github.com/saviorand/lean-android-compose)

## Contributing

Contributions are welcome, particularly new node kinds and the theorems that should
accompany them.

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
