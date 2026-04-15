# AI agent guide for philosophers_42

This repo contains a 42-school Dining Philosophers implementation using POSIX threads. Core code lives under `philosophers/`; `practice/` and `thinking_sleeping_p/` hold experiments and are not part of the main build.

## Build and run
- Build the main binary from `philosophers/`:
  - `make` (see `philosophers/Makefile`, uses `-Wall -Wextra -Werror -pthread`)
  - Output: `philosophers/philo`
- Run (expected 42-style arguments):
  - `./philo <num_philos> <time_to_die> <time_to_eat> <time_to_sleep> [num_meals]`
  - All times are milliseconds. Arguments are positive integers (parsing via `ft_atoi`, validation via `ft_isdigit`).
- Root `Makefile` is a scaffold; use the `philosophers/` one for the actual build. If you add new `.c` files, update `SRCS` in `philosophers/Makefile`.

## Architecture at a glance
- Types (see `philosophers/includes/philo.h`):
  - `t_data`: global simulation state
    - `num_philos`, `time_to_die`, `time_to_eat`, `time_to_sleep`, optional `num_meals`
    - `start_time` in ms
    - `forks`: array of `pthread_mutex_t` (size = `num_philos`)
    - `print_mutex`: serialize logging
    - `philos`: array of `t_philo`
  - `t_philo`: per-philosopher state
    - `id` and fork indices: `left_fork = id`, `right_fork = (id + 1) % num_philos`
    - `last_meal` (ms), `meals_eaten`
    - `thread` handle, back-pointer `data`
- Key functions:
  - `init_data(t_data*, int argc, char** argv)`: parse/validate args; allocate/init forks and philos; set `start_time`.
  - `philo_routine(void*)`: philosopher loop (take forks, eat, sleep, think) — to be implemented against the patterns below.
  - Utils in `utils1.c` (+ `utils2.c` placeholder): `ft_atoi`, `ft_isdigit`, `ft_putstr_fd` etc.
  - Time helpers in header: `get_time_in_ms`, `ft_usleep` (ms-based sleep with proper yielding).

## Concurrency patterns you must follow
- Forks are mutexes:
  - Store forks as `pthread_mutex_t forks[num_philos]` in `t_data`.
  - In `t_philo`, store only the fork indices as `int`. Never copy a mutex inside a struct.
  - Lock by index: `pthread_mutex_lock(&data->forks[idx])`.
- Deadlock avoidance: use a total order or parity scheme.
  - Preferred: lock lower-indexed fork first, then higher (min/max of left/right).
  - Alternate: even philosophers grab right then left; odd grab left then right.
- Serialized output: guard prints with `print_mutex` via `print_action(t_philo*, const char*)`.
- Time/memory:
  - All times in ms (`long long`). Update `philo->last_meal` immediately after eating.
  - If you add a monitor thread, protect shared fields (`last_meal`, `all_ate`) with a dedicated mutex or consistent access discipline.

## Control flow (intended)
1. `main.c`: call `init_data(&data, argc, argv)`; create `num_philos` threads on `philo_routine`; `pthread_join` them; cleanup via `destroy_data`.
2. `init.c`: allocate `data->forks` and `data->philos`; `pthread_mutex_init` each fork; init `print_mutex`; set ids/indices/timestamps.
3. `philo_routine`: loop until stop condition (death detected or `num_meals` reached):
   - take forks (ordered lock) → print "has taken a fork" twice
   - print "is eating" → sleep `time_to_eat` → `meals_eaten++`, update `last_meal`
   - release forks → print "is sleeping" → sleep `time_to_sleep` → print "is thinking"

## Conventions and gotchas
- Strict flags: code must compile with `-Wall -Wextra -Werror`.
- Do not busy-wait; use `ft_usleep(ms)` for timing and small stagger for even/odd start if needed.
- Always `pthread_mutex_destroy` all forks and `print_mutex` in `destroy_data`.
- If you add files, include headers via `#include "./includes/philo.h"` as in current sources and keep functions declared in `philo.h`.

## Pointers to examples in this repo
- Data types: `philosophers/includes/philo.h`
- Build: `philosophers/Makefile`
- Utils and argument parsing: `philosophers/utils1.c`
- Entry points to extend: `philosophers/main.c`, `philosophers/init.c`
- Experiments (not built): `practice/`, `thinking_sleeping_p/`

If anything above is unclear (e.g., chosen deadlock strategy, monitor thread expectations), ask for confirmation before implementing, and keep instructions updated once patterns stabilize.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thedreamJK7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
