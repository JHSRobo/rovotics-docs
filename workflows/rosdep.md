# ROS and System Dependencies with rosdep

ROS packages list the ROS and system packages they need in `package.xml`. `rosdep` reads those declarations and installs the matching packages for the computer's operating system.

An internal package such as `rov_interfaces` is different: `rosdep` cannot fetch its source. Import it from Git using the workspace's `.repos` manifest and build it with `colcon` as described in [ROS workspace management](workspace-management.md). `rosdep` can then install the external dependencies declared by `rov_interfaces` itself.

## Declare dependencies

Add recognized rosdep keys to the package's `package.xml`. For example:

```xml
<depend>rclpy</depend>
<depend>std_msgs</depend>
<depend>python3-gi</depend>
```

This makes the requirement part of the package, so other developers can install it as part of workspace setup rather than relying on undocumented manual installation.

## Install and initialize rosdep

`rosdep` is included in the ROS development tools. Follow the [ROS 2 Jazzy Ubuntu installation instructions](https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html) to set up the ROS apt repository and install `ros-dev-tools`.

Initialize rosdep once on each computer, then update its key database when setting up or refreshing the machine:

```bash
sudo rosdep init
rosdep update
```

## Install dependencies for a workspace

From the workspace root, install dependencies declared by packages under `src/`:

```bash
rosdep install --from-paths src --ignore-src -r -y
```

`--ignore-src` tells rosdep to skip dependencies whose packages are already in the source workspace. This matters for source-built packages such as `rov_interfaces`: the workspace will build that package with `colcon` instead of asking rosdep to find and install it as an external package. `-r` continues checking after an individual failure, and `-y` accepts installation prompts.

After installing dependencies, build the workspace:

```bash
colcon build
```

## Other useful commands

Run these from the workspace root:

```bash
# Check whether declared dependencies are installed
rosdep check --from-paths src --ignore-src

# Find which platform package a rosdep key maps to
rosdep resolve <rosdep-key>

# Search for rosdep keys matching a term
rosdep search <term>
```

When a package gains a dependency, add it to its `package.xml`, then repeat the workspace install and build steps above. Commit the updated manifest. For internal source dependencies, see [workspace management](workspace-management.md); for Python packages from PyPI, see [uv](uv.md). The overall ownership model is in the [dependency overview](README.md).
