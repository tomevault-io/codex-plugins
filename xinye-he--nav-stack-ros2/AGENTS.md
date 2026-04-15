## Quick orientation — what this workspace is

This is a ROS2 multi-package workspace focused on GPS-only navigation and IMU integration.
Major runtime components (in `gps_imu_ws/src`) include:
- `gps_path_server`: reads CSV waypoints and publishes `/global_path` (nav_msgs/Path) and `/global_path_geo` (gps_msgs/GlobalPath).
- `gps_local_follower`: a pure-pursuit style follower that subscribes `/global_path` and `/odometry/filtered_map` and publishes `/drive_cmd` (gps_msgs/AngleSpeed).
- `gps_nav_bringup`: launch files and parameters which wire `robot_localization`, `navsat_transform_node`, the path server and follower together for sim/real runs.

Key message definitions live in `gps_msgs/msg/` (AngleSpeed.msg, GlobalPath.msg, Waypoint.msg). These are ament_cmake-built and consumed by ament_python nodes.

## High-level data flow
- CSV -> `gps_path_server` -> `/global_path` (Path) -> `gps_local_follower` -> `/drive_cmd` (AngleSpeed)
- `robot_localization` + `navsat_transform_node` produce `/odometry/filtered_map` that the follower uses for geometry.
- `gps_path_server` also publishes `/global_path_geo` using `gps_msgs/GlobalPath` for higher-level tooling.

## How to build & run (most common workflows)
- Build the workspace with colcon from the workspace root (e.g. `gps_imu_ws` or this repository root if you use multiple ws):

```bash
cd /path/to/gps_imu_ws
colcon build --symlink-install
source install/setup.bash
```

- Launch the sim bringup (examples located at `gps_nav_bringup/launch`):

```bash
source install/setup.bash
ros2 launch gps_nav_bringup gps_nav_sim.launch.py
```

- The launch uses `GPS_PATH_CSV` env var to pick a CSV; you can override it:

```bash
export GPS_PATH_CSV=/path/to/route.csv
ros2 launch gps_nav_bringup gps_nav_sim.launch.py
```

- Bag replay: there are recorded bags in `gps_imu_ws/bag/`—use `ros2 bag play` to replay sensor and nav data for offline testing.

## Important project-specific conventions & gotchas
- Mixed build types: message package `gps_msgs` uses `ament_cmake` (ROS IDL generation). Most nodes (`gps_path_server`, `gps_local_follower`, `gps_nav_bringup`) are `ament_python`. Build the whole workspace with colcon to ensure messages are generated before python nodes import them.
- Node/topic names are global (leading `/`). Examples:
  - `/global_path` (nav_msgs/Path) published by `gps_path_server` — this node locks the first published path (it ignores subsequent periodic publishes to avoid progress jumps).
  - `/global_path_geo` (gps_msgs/GlobalPath) contains geo waypoints (see `gps_msgs/msg/Waypoint.msg`).
  - `/odometry/filtered_map` (nav_msgs/Odometry) is produced by `robot_localization` and expected by the follower.
  - `/drive_cmd` publishes `gps_msgs/AngleSpeed` (heading_deg, speed_mps).
- CSV formatting for `gps_path_server`: expects columns including `index`, `lat`, `lon`, `heading_deg`, `point_type`, `next_seg_type`. Latitude/longitude are handled/rounded to 7 decimals by the loader — see `gps_path_server/path_server.py` for exact parsing and validation.

## CSV example
`gps_path_server` expects a CSV with at least the following header columns:

```
index,lat,lon,heading_deg,point_type,next_seg_type
```

Example row (one waypoint):

```
1,37.1234567,-122.1234567,90.0,0,0
```

- `lat` and `lon` are rounded to 7 decimal places by the loader; if the CSV values are not 7-decimal strings the node will warn and round.
- `heading_deg` is the desired path tangent at that waypoint in degrees (used to construct segment end tangents).
- `next_seg_type`: controls interpolation between this waypoint and the next. Discovered behavior in `gps_path_server/path_server.py`:
  - `1` => use arc interpolation (circular arc between tangents)
  - any other value => straight-line sampling between points
- `point_type`: the server preserves this field and publishes it inside `gps_msgs/Waypoint` via `/global_path_geo`. It is not used by `gps_path_server` for geometry — treat it as an opaque application tag for higher-level tools.
- Indices should be monotonic; the loader treats the route as cyclic (it samples pairs (i, i+1) with `(i+1)%N`) so a closed loop is supported.
- Frames & conventions: ENU (x East, y North). Several utility functions perform yaw<->bearing conversions; be careful of bearing vs yaw when integrating other modules.
- Container/RViz notes: launch files set environment vars for running RViz inside containers (`LIBGL_ALWAYS_SOFTWARE`, `QT_X11_NO_MITSHM`). See `docker/` for container-related setup if you run inside Docker.

## Files to inspect first (quick map for an AI agent)
- `gps_path_server/gps_path_server/path_server.py` — CSV parser, sampling (line/arc), publishes `/global_path` and `/global_path_geo`.
- `gps_local_follower/gps_local_follower/local_follower.py` — main follower logic, many in-code tuning parameters, path locking, corner-stop logic and published topics.
- `gps_msgs/msg/*.msg` — types: AngleSpeed, GlobalPath, Waypoint.
- `gps_nav_bringup/launch/*.launch.py` — example runtime wiring (sim/real). Useful for discovering parameter names and remappings.
- `docker/` — scripts and Dockerfile that show how the team runs the stack in containers and why certain env vars are set for RViz.

## Typical edit/test iteration (recommended)
1. Modify a node (Python). Unit-test locally where possible.
2. Rebuild the workspace (ensure `gps_msgs` is built if you changed messages): `colcon build`.
3. Source `install/setup.bash` and run the appropriate launch (see `gps_nav_bringup/launch`).

## Small examples you can use when authoring code or tests
- To publish a step trigger (single-step follower) use the `/sim/step` Empty topic (see `local_follower.py` subscription).
- To reload CSV without restarting the node, call the service `/path_server/reload` (std_srvs/Trigger).

---
If you'd like, I can refine this (shorten, add short code snippets, or include a quick-run checklist tuned to your preferred workspace root). What should I expand or clarify?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Xinye-He) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
