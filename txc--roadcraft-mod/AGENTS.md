This is a Go based repository with a Ruby client for certain API endpoints. It is primarily responsible for ingesting metered usage for GitHub and recording that usage. Please follow these guidelines when contributing:

This is a Go application that modifies different vehicles in the game RoadCraft from Sabre Interactive.

## Code Standards

### Required Before Each Commit
- Run `make fmt` before committing any changes to ensure proper code formatting
- This will run gofmt on all Go files to maintain consistent style

### Development Flow
- Build: `make build`
- Test: `make test`
- Full CI check: `make all` (includes build, fmt, lint, test)

## Repository Structure
- `cmd/`: Main service entry points and executables
- `internal/`: Logic related to interactions with other GitHub services
- `docs/`: Documentation
- `testing/`: Test helpers and fixtures

## Key Guidelines
- When creating new files, include a package-level comment describing the purpose of the package.
- Verify that all new and existing code passes `go vet` checks.
- Follow Go best practices and idiomatic patterns
- Use existing libraries and frameworks where possible
- Maintain existing code structure and organization
- Use dependency injection patterns where appropriate
- Write unit tests for new functionality. Use table-driven unit tests when possible.
- Document public APIs and complex logic. Suggest changes to the `docs/` folder when appropriate
- Tests should be placed in the same package as the code they are testing, using `_test.go` suffix for test files.
- Ensure all tests pass before submitting a pull request.
- Try to achieve high test coverage for new code. (Goal is 80%+)
- Test must cover (identified) edge cases and error conditions in addition to happy paths.
- Use descriptive commit messages that explain the purpose of the changes.
- Review existing code for style and consistency before adding new code.

---

## Information about RoadCraft modding

There are two files that are vital for this application:
- `default_other.pak`: A compressed file that contains various CLS files for different vehicles in the game.
- `default_other.pak.cache`: A cache file that stores metadata about the compressed file to speed up access.

It reads a compressed file, named `default_other.pak` that is located at `root/paks/client/default` in the game directory inside the Steam Library.
The file is a actually a ZIP-file, compressed with "Store".

For example, on "Zikz 605E - Mobile Scalper", we need to update the file `ssl/autogen_designer_wizard/trucks/auto_zikz_605e_mobile_scalper_res/auto_zikz_605e_mobile_scalper_res.cls` inside `default_other.pak`.
- To allow scraping everywhere, we change the `properties.prop_truck_mobile_sand_screen.allowedPercent` to `0`.
- To increase the quarry radius:
  - `properties.prop_truck_mobile_sand_screen.sandDistance` from `150` to `600`.
  - `properties.prop_usable.smartsEntryPoints.SandStorage.checkers.UsableCheckerDistance.distance` from `150` to `600`.
  - `properties.prop_usable.smartsEntryPoints.SandStorage.focusDistance` from `150` to `600`.
After modifying the CLS file, we need to repackage it back into `default_other.pak` using the same compression method.
And delete the cache file `default_other.pak.cache` to ensure the game regenerates it with the updated content.

---

### Information about CLS files

The CLS format for RoadCraft is not officially documented.
Information about it has been gathered from community sources and reverse engineering efforts.
When making changes to CLS files, ensure that you validate the changes in-game to confirm they work as intended.

For the "Zikz 605E - Mobile Scalper", the original CLS file looks like this:
```
properties   =   {
   geom   =   {
      nameTpl   =   "zikz_605e_mobile_scalper_res"
   }
   prop_truck_view_controller   =   {
      engineFiringDelay   =   1000
   }
   ...
   prop_truck_mobile_sand_screen   =   {
      chainName   =   "Ramp"
      allowedMaterials   =   [
         {
            material   =   "sand"
         },
         {
            material   =   "mud_sand"
         },
         {
            material   =   "sand_dry"
         },
         {
            material   =   "sand_mud"
         }
      ]
      locator   =   "SandReciever"
      sandOnBelt   =   True
      materialCheckRadius   =   6
      allowedPercent   =   0.4
      belt1SpeedCoef   =   0.3
      belt2SpeedCoef   =   0.8
      sandDistance   =   150
   }
   ...
   prop_usable   =   {
      smartsEntryPoints   =   {
         MalfuctionSmartPoint   =   {
            __type   =   "SmartEntryPointMalfunction"
         }
         SandStorage   =   {
            checkers   =   {
               UsableCheckerDistance   =   {
                  distance   =   150
                  tracker   =   {
                     locator   =   "SandReciever"
                     __type   =   "Tracker"
                  }
               }
            }
            focusDistance   =   150
            __type   =   "SmartEntryPointMobileSandScreen"
         }
      }
   }
   ...
}
```

---
> Source: [TXC/roadcraft-mod](https://github.com/TXC/roadcraft-mod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-06 -->
