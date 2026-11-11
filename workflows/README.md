# Dependency Management

ROVOTICS workspaces combine source repositories with ROS, system, and Python dependencies. Each layer has its own declaration and tool:

| Layer | What it supplies | Declaration | Tool |
| --- | --- | --- | --- |
| ROS workspace sources | ROVOTICS packages such as `rov_interfaces` | `topside.repos` or `bottomside.repos` | `vcs` |
| ROS and system packages | Packages such as `rclpy`, `std_msgs`, `python3-gi`, and GStreamer plugins | Each ROS package's `package.xml` | `rosdep` |
| PyPI packages | Third-party Python libraries such as `linuxpy` | `pyproject.toml` and `uv.lock` | `uv` |

The workspace manifest determines which source repositories are present. For example, `rov_interfaces` must be fetched from its Git repository and built from source with `colcon`; it is not installed by `rosdep` or `uv`. Its own external ROS/system dependencies are still declared in `package.xml` and installed with `rosdep`.

## Typical workspace setup

1. Install `vcs` and import the appropriate source manifest (see [ROS workspace management](workspace-management.md)).
2. Install external ROS and system dependencies with `rosdep` (see [rosdep](rosdep.md)).
3. Create or synchronize the Python environment with `uv` (see [uv](uv.md)).
4. Build the workspace with `colcon`.

## Files to commit

Commit the workspace `.repos` manifest, ROS package `package.xml` files, Python `pyproject.toml`, `uv.lock`, and `.python-version` when used. Keep the machine-local `.venv/` directory out of Git.

Detailed guides: [ROS workspace management](workspace-management.md), [ROS and system dependencies](rosdep.md), and [Python dependencies](uv.md).
