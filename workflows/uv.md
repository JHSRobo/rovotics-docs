# Python Dependencies with uv

Use `uv` for third-party Python packages published on PyPI. `pyproject.toml` records the dependencies the project asks for, while `uv.lock` records the exact resolved versions so developers can recreate the environment.

ROVOTICS uses a virtual environment that can also see system site-packages. This lets Python code use ROS and operating-system Python packages while keeping added PyPI packages out of the protected system Python environment.

## Install uv

Install `uv` with the upstream installer, then follow its printed instructions if the command is not yet on your `PATH`:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## Create the project environment

From the directory that should contain the Python project files, initialize the project and environment:

```bash
uv init --bare
uv python pin 3.12
uv venv --system-site-packages
uv sync
```

`uv python pin` records the Python version in `.python-version`. `uv venv --system-site-packages` creates `.venv/` with access to system-installed Python packages. `uv sync` installs the project's declared packages and creates or updates `uv.lock` as needed.

Commit `pyproject.toml`, `uv.lock`, and `.python-version`. Do not commit `.venv/`; it is created separately on each computer.

## Add or remove packages

Add a runtime package, a development-only package, or remove a package with:

```bash
uv add linuxpy
uv add --dev pytest
uv remove linuxpy
```

These commands update the project metadata and lockfile. Use `uv add` to record project dependencies instead of installing them manually with `pip`.

## Keep the environment in sync

After cloning the project or changing its dependencies, install the versions recorded by the project files:

```bash
uv sync
```

To check whether the environment matches the project files without changing it:

```bash
uv sync --check
```

## Activate and inspect the environment

Activation makes the environment's `python` and other tools the defaults in the current shell:

```bash
source .venv/bin/activate
```

Check which Python executable the shell will run. It should be inside this project's `.venv/` directory:

```bash
which python
```

Leave the environment when finished:

```bash
deactivate
```

Use these commands to understand what is installed and why:

```bash
uv pip list
uv tree
uv tree --package <package>
uv tree --invert <package>
```

`uv pip list` shows installed packages. `uv tree` shows the project's dependency tree. `--package` focuses the tree on one package; `--invert` shows which project dependencies bring a package in (for example, `uv tree --invert numpy`).

## Avoid ROS and Python version conflicts

Because `.venv/` can see system packages, a package installed into the virtual environment may take precedence over a system or ROS version with the same name. Before adding a PyPI package, check whether ROS or the operating system already provides it. Prefer the existing version when it meets the need; if you require another version, confirm it is compatible with the ROS packages that use it.

To see where an import comes from, replace `<module>` with its Python module name:

```bash
python -c "import <module>; print(<module>.__file__)"
```

To see the directories Python searches for modules:

```bash
python -c "import sys; print('\\n'.join(sys.path))"
```

For ROS and system packages, see [rosdep](rosdep.md). For source packages such as `rov_interfaces`, see [ROS workspace management](workspace-management.md). The overview of all three dependency layers is in [README](README.md).
