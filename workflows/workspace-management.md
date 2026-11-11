# ROS Workspace Management

A ROS workspace is a directory containing ROS packages, usually under `src/`, that are built together. ROVOTICS records the source repositories for each workspace in a `.repos` manifest and uses `vcs` to fetch them.

## Install vcs

`vcs` is the command-line tool provided by the ROS `ros-dev-tools` package. On a system where the ROS apt repositories are configured, install the development tools bundle with:

```bash
sudo apt update
sudo apt install ros-dev-tools
```

If you only need the repository import tool, its Debian package is `python3-vcstool`:

```bash
sudo apt install python3-vcstool
```

Use the ROS distribution's installation guide to configure its apt repository and install any other prerequisites. Check the tool is available with `vcs --help`.

## Import a workspace

The `topside.repos` and `bottomside.repos` manifests list the source repositories for their respective workspaces. Each entry records a repository URL and version. From the workspace root, import the matching manifest into `src/`:

```bash
vcs import src < topside.repos
```

The same command pattern works with `bottomside.repos` for the bottomside workspace. Keep the manifests in version control so the source set can be recreated. To update repositories already imported, run `vcs pull src` from the workspace root.

The `.repos` manifest fetches source; it does not install external dependencies declared by that source. Install those ROS and system packages with [rosdep](rosdep.md), and manage PyPI packages with [uv](uv.md). See the [dependency overview](README.md) for the division of responsibilities.
