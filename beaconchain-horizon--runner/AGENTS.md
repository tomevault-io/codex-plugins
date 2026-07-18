## Making changes

### Tests

Whenever possible, changes should be accompanied by non-trivial tests that meaningfully exercise the core functionality of the new code being introduced.

All tests are in the `Test/` directory at the repo root. Fast unit tests are in the `Test/L0` directory and by convention have the suffix `L0.cs`. For example: unit tests for a hypothetical `src/Runner.Worker/Foo.cs` would go in `src/Test/L0/Worker/FooL0.cs`.

Run tests using this command:

```sh
cd src && ./dev.sh test
```

### Formatting

After editing .cs files, always format the code using this command:

```sh
cd src && ./dev.sh format
```

### Feature Flags

Wherever possible, all changes should be safeguarded by a feature flag; `Features` are declared in [Constants.cs](src/Runner.Common/Constants.cs).

---
> Source: [beaconchain-horizon/runner](https://github.com/beaconchain-horizon/runner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-18 -->
