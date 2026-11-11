# Dependency Management Workflow

This is a dependency management workflow guide for ROVOTICS software development. ROVOTICS develops and maintains a number of custom ROS packages that depend on Python libraries, system packages, and other ROS packages. The goal is to streamline dependency installation, reliably manage potential package conflicts, and facilitate reproducible development environments.

## What Dependencies Exist

ROVOTICS software may depend on several types of packages:

* ROS packages like **rclpy** and **std_msgs**
* System packages like **python3-gi** and **gstreamer1.0-plugins-base**
* Third-party Python packages hosted on PyPI, like **linuxpy**

## Dependency Management Challenges

By default, Python package managers are prevented from modifying the system Python environment according to PEP 668. This protection can be explicitly overridden, but modifying the system Python environment can cause conflicts with system-installed packages and dependencies required by ROS.

To avoid modifying the system Python environment, third-party Python dependencies are installed in a **virtual environment**. A virtual environment provides an isolated Python environment with its own package installation directory.

By default, a virtual environment cannot access packages installed in the system environment. Since ROS depends on Python packages installed as part of the system and ROS installation, ROVOTICS virtual environments are configured to allow access to system site-packages.

This creates an important consideration: if a package is installed both in the virtual environment and in the system environment, Python's module search path determines which installation is imported. A virtual environment can therefore introduce version conflicts if it overrides a package required by ROS.

## Dependency Management Tools

ROVOTICS uses different tools for different types of dependencies:

| Dependency | Examples | Management |
|---|---|---|
| ROS packages | **rclpy**, **std_msgs** | **rosdep** |
| System packages | **python3-gi**, **gstreamer1.0-plugins-base** | **rosdep / apt** |
| PyPI packages | **linuxpy**, **numpy**, **scipy** | **uv** |

**rosdep** is responsible for installing ROS and system dependencies declared by ROS packages.

**uv** is responsible for managing third-party Python dependencies hosted on PyPI.

This separation allows system-level dependencies to remain managed by the system package manager while application-level Python dependencies remain isolated and reproducible.

---

# rosdep Workflow

## Installation

To manage ROS and system dependencies, ROVOTICS uses **rosdep**, which is available through the **ros-dev-tools** package. See the [ROS 2 Jazzy installation documentation](https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html) for installation instructions.

## Declaring Dependencies

ROS and system dependencies should be declared using their corresponding **rosdep keys** in the package's `package.xml` file.

For example:

```xml
<depend>rclpy</depend>
<depend>std_msgs</depend>
<depend>python3-gi</depend>
```

The dependency should be declared in `package.xml` rather than relying on a developer to manually install the package.

When installing dependencies, rosdep resolves each key to the appropriate package-manager dependency for the target system.

## Common rosdep Commands

Initialize rosdep once per machine:

```bash
sudo rosdep init
```

Update rosdep's dependency database:

```bash
rosdep update
```

Install all dependencies declared by packages in a workspace:

```bash
rosdep install --from-paths src --ignore-src -r -y
```

Check whether all workspace dependencies are satisfied:

```bash
rosdep check --from-paths src --ignore-src
```

See what a rosdep key resolves to:

```bash
rosdep resolve <rosdep-key>
```

Search for a rosdep key:

```bash
rosdep search <term>
```

## Adding a ROS or System Dependency

When a package gains a new ROS or system dependency:

1. Add the appropriate rosdep key to `package.xml`.
2. Run rosdep to install the dependency:

```bash
rosdep install --from-paths src --ignore-src -r -y
```

3. Build the workspace:

```bash
colcon build
```

4. Commit the updated `package.xml`.

Do not rely on manually installed system packages without declaring them in `package.xml`.

---

# uv Workflow

## Installation

ROVOTICS uses **uv** to manage Python packages hosted on PyPI.

To install uv:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

After installation, restart the shell or follow the instructions printed by the installer so that `uv` is available on `PATH`.

## Initializing the Environment

The Python environment is managed at the workspace level.

From the directory containing the `pyproject.toml`:

```bash
uv init --bare
```

Pin the Python version used by the project:

```bash
uv python pin 3.12
```

Create the virtual environment:

```bash
uv venv --system-site-packages
```

The resulting project should contain:

```text
pyproject.toml
uv.lock
.python-version
.venv/
```

The `.venv` directory is local to each machine and should not be committed to Git.

## Adding Python Dependencies

To add a runtime dependency:

```bash
uv add <package>
```

For example:

```bash
uv add linuxpy
```

To add a development-only dependency:

```bash
uv add --dev pytest
```

`uv add` updates both `pyproject.toml` and `uv.lock`.

The dependency should be added with uv rather than manually installing it with `pip`.

## Removing Python Dependencies

Remove a dependency with:

```bash
uv remove <package>
```

For example:

```bash
uv remove linuxpy
```

This updates the project's dependency configuration and lockfile.

## Synchronizing the Environment

To create or synchronize the virtual environment from the project's dependency configuration:

```bash
uv sync
```

This should be run after cloning the repository or whenever the project's dependencies change.

To check whether the environment is synchronized without modifying it:

```bash
uv sync --check
```

The expected workflow is:

```text
uv sync --check
        ↓
Is the environment synchronized?
        ↓
      No
        ↓
    uv sync
```

## Activating the Environment

The virtual environment can be activated with:

```bash
source .venv/bin/activate
```

Verify that the expected Python interpreter is active:

```bash
which python
```

It should point to the project's virtual environment:

```text
/path/to/project/.venv/bin/python
```

When finished:

```bash
deactivate
```

## Inspecting Installed Packages

To list packages currently installed in the environment:

```bash
uv pip list
```

To view the project's dependency tree:

```bash
uv tree
```

To inspect the dependency tree for a particular package:

```bash
uv tree --package <package>
```

To determine which dependencies require a particular package:

```bash
uv tree --invert <package>
```

For example:

```bash
uv tree --invert numpy
```

This is useful for identifying why a package is present in the environment.

---

# Dependency Ownership



As a general rule, **do not manually install a dependency through a different package manager when it is already managed by ROVOTICS' dependency workflow**.

In particular:

* Do not use `pip install` to add a project dependency managed by uv.
* Do not manually install an APT or ROS dependency without declaring it in `package.xml`.
* Do not commit `.venv`.
* Do commit `pyproject.toml`, `uv.lock`, and `.python-version`.

## Python Package Conflicts

Because the ROVOTICS virtual environment has access to system site-packages, a Python package may be available from both the virtual environment and the system/ROS environment.

Before adding a PyPI dependency, consider whether it overlaps with a package required by ROS or Ubuntu.

If a package is already provided by ROS or the system, prefer using the existing installation rather than installing a separate version into the ROVOTICS virtual environment.

If a different version is required, verify that it is compatible with the ROS packages using it before allowing the virtual environment to override the system version.

The location of an imported package can be inspected with:

```bash
python3 -c "import <package>; print(<package>.__file__)"
```

The Python module search path can be inspected with:

```bash
python3 -c "import sys; print('\n'.join(sys.path))"
```

These commands are useful when diagnosing unexpected package versions or import errors.
