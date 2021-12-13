# Wordweaver

This repository contains documentation for Wordweaver, a secure communications project. Much is yet undecided, but the project has the following goals:

1. Create a reasonably secure protocol for carrying data.
2. Provide tools for sharing text, files, and (potentially) real-time communications.
3. Wrap them up in interfaces simple enough for inexperienced computer users to use and understand their basic functions.

Additionally, Wordweaver is tentatively targeting the following platforms:

* Desktop platforms (Working title "Loom")
  (Likely to use a hybrid-web framework for portability)
  * Linux (Flatpak)
  * MacOS/Darwin (contingent on gaining access to test hardware)
  * Windows
* Mobile platforms (Working title "Crochet")
  * Android
  * IOS (contingent on gaining access to test hardware)
  * Mobile Linux (Flatpak)
* Low-level interfaces (Working title "Needle")
  * `tty` frontend
  * Rust Crate (`no_std` is unlikely but possible)
  * JavaScript bindings (via WASM)
  * C(++) bindings