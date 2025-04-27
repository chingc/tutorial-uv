# uv

Managing Python environments doesn't have to be hard. A lot of the manual and error prone processes can be handled automatically with the right tool.

This tutorial introduces uv. An extremely fast and popular Python package and project manager. A single tool to replace `pip`, `pip-tools`, `pipx`, `poetry`, `pyenv`, `twine`, `virtualenv`, and more. It can even install and manage Python versions.

## Install

With homebrew:

```bash
$ brew install uv
```

Click [here](https://docs.astral.sh/uv/getting-started/installation/) for other installation methods.

To verify the installation, run `uv` and you should see a help menu listing available commands.

> [!NOTE]
> Upgrade or uninstall uv with homebrew.

## Managing Python Versions

Python does not need to be explicitly installed to use uv. By default, uv will automatically download Python versions when they are required. Even if a specific Python version is not requested, uv will download the latest version on demand.

For example, if there are no Python versions on your system, the following will install Python before creating a new virtual environment:

```bash
$ uv venv
```

If Python is already installed on your system, uv will detect and use it without configuration. You can also install and manage specific Python versions.

```bash
# Install the latest version
$ uv python install

# Install a specific version
$ uv python install 3.12

# Install multiple versions
$ uv python install 3.11 3.12

# Install an alternative implementation
$ uv python install pypy@3.11
```

> [!NOTE]
> Python does not publish official distributable binaries. As such, uv uses distributions from the Astral [python-build-standalone](https://github.com/astral-sh/python-build-standalone) project. See the [Python distributions](https://docs.astral.sh/uv/concepts/python-versions/#managed-python-distributions) documentation for more details.

To reinstall uv-managed Python versions, use `--reinstall`:

```bash
$ uv python install --reinstall
```

This will reinstall all previously installed Python versions. Improvements are constantly being added to the Python distributions, so reinstalling may resolve bugs even if the Python version does not change.

To uninstall uv-managed Python versions:

```bash
$ uv python uninstall 3.11
```

To view available and installed Python versions:

```bash
$ uv python list
```

A specific Python version can be requested with the `--python` flag in most uv commands. For example, when creating a virtual environment:

```bash
$ uv venv --python 3.13.3
```

If you have a specific Python version you'd like to use as the default, the `.python-version` file is helpful. It can be created with the `uv python pin` command:

```bash
# Created in the user configuration directory
$ uv python pin --global 3.13.3

# Created in the current working directory
$ uv python pin pypy@3.11
```

## Running Scripts

Before we talk about Python projects, let's quickly go over scripts. A Python script is a file intended for standalone execution (e.g. `python <script>.py`).

If your script has no dependencies, or depends on modules in the standard library, you can execute it with `uv run` with nothing more to think about:

```python
# example.py
import os
print(os.path.expanduser("~"))
```

```bash
$ uv run example.py
/Users/python-enjoyer
```

Note that if you use `uv run` in a project (a directory with a `pyproject.toml`), it will install the current project before running the script. If your script does not depend on the project, use the `--no-project` flag to skip this:

```bash
$ uv run --no-project example.py
```

> [!NOTE]
> The `--no-project` flag is part of uv and not the script, so it must be specified before the script name.

If your script has dependencies it's recommended to create a project or use Python's new format for inline metadata. Inline metadata allows dependencies for a script to be declared in the script itself.

```bash
# Initialize a script with inline metadata
$ uv init --script example.py --python 3.12

# Add dependencies to a script
$ uv add --script example.py 'requests<3' 'rich'
```

```python
# example.py

# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "requests<3",
#   "rich",
# ]
# ///

import requests
from rich.pretty import pprint

resp = requests.get("https://peps.python.org/api/peps.json")
data = resp.json()
pprint([(k, v["title"]) for k, v in data.items()][:10])
```

> [!IMPORTANT]
> Scripts that declare inline metadata are automatically executed in environments isolated from the project. Only the dependencies listed in the script are available. The `--no-project` flag is not required.

A shebang can be added to make a script executable without using `uv run`:

```bash
#!/usr/bin/env -S uv run --script
print("Hello, world!")
```

Ensure the script is executable (`chmod +x greet`) then run the script:

```bash
$ ./greet
Hello, world!
```

## Project Initialization

You can initialize a brand new project:

```bash
$ uv init example
$ cd example
```

Or, initialize an existing project:

```bash
$ cd example
$ uv init
```

This will create the following files:

```
.
├── .python-version
├── README.md
├── main.py
└── pyproject.toml
```

The `main.py` file contains a simple "Hello world" program. Try it with `uv run`:

```bash
$ uv run main.py
Hello from example!
```

You'll notice the project structure will now look like this:

```
.
├── .python-version
├── .venv
├── README.md
├── main.py
├── pyproject.toml
└── uv.lock
```

Except for `.venv`, all of this should be checked into version control.

### .python-version

The project's default Python version. This file tells uv which Python version to use when creating the project's virtual environment.

### .venv

The .venv folder contains your project's virtual environment, a Python environment that is isolated from the rest of your system. This is where uv will install your project's dependencies.

Notice how uv created this for you automatically. Very nice!

### README.md

The project's README file.

### main.py

Contains a simple "Hello world" program.

### pyproject.toml

Contains metadata about your project. Use this file to specify dependencies, as well as details about the project such as its description or license. You can edit this file manually, or use commands like `uv add` and `uv remove`.

You'll also use this file to specify uv configuration options in a `[tool.uv]` section.

### uv.lock

A cross-platform lockfile that contains exact information about your project's dependencies. It's a human-readable TOML file, but it's managed by uv and shouldn't be edited manually.

## Project Dependencies

Add or remove dependencies with `uv add` and `uv remove`. This will update `pyproject.toml`, `uv.lock`, and the project environment.

```bash
# Add a dependency
$ uv add requests

# Specify a version constraint
$ uv add 'requests==2.30.0'

# To update a version constraint
$ uv add 'requests==2.31.0'

# Add a git dependency (--tag, --branch, or --rev can be used for version constraint)
$ uv add git+https://github.com/psf/requests

# Migrating from requirements.txt
$ uv add -r requirements.txt -c constraints.txt

# Remove a dependency
$ uv remove requests
```

To see the dependency tree for your project:

```bash
$ uv tree
```

Use `uv lock` to upgrade a package:

```bash
# Upgrade to the latest version
$ uv lock --upgrade-package requests

# Upgrade to a specific version
$ uv lock --upgrade-package 'requests==2.32.3'

# Upgrade all packages:
$ uv lock --upgrade
```

Version constraints are respected when upgrading.

> [!TIP]
> Instead of upgrading a package, update its version constraint.

### "It works on my machine."

> [!CAUTION]
> It's important to understand how we end up here.

When considering if the lockfile is up-to-date, uv will check if it matches the project metadata. For example, if you add a dependency to your `pyproject.toml`, the lockfile will be considered outdated. Similarly, if you change the version constraints for a dependency such that the locked version is excluded by the constraint, the lockfile will be considered outdated. However, if you change the version constraints such that the existing locked version is still within the constraint, the lockfile will still be considered up-to-date.

The last case is important.

This means, if you have `requests>=2.32.3` as a version constraint, uv will not consider the lockfile outdated when a new version is released. The lockfile needs to be explicitly updated if you want to upgrade the dependency.

If you have lots of developers and systems, you could easily be in a situation where some machines are running `2.32.3` and some machines are running a newer version.

> [!TIP]
> "It works on my machine" can be very difficult to troubleshoot. Use the `==` version constraint to avoid this problem and maintain consistency across all systems.

## Project Commands

Use `uv run` to run arbitrary scripts or commands in your project environment.

Prior to every `uv run` invocation, uv will verify that the lockfile is up-to-date and the environment is up-to-date with the lockfile, keeping your project synced without the need for manual intervention. `uv run` guarantees your command is run in a consistent and locked environment.

For example, to use flask:

```bash
$ uv add flask
$ uv run -- flask run -p 3000
```

> [!NOTE]
> The `--` is shell syntax that separates the main command (`uv`) from the subcommand (`flask`). This is helpful if the subcommand has its own flags and arguments.

Or, to run a script:

```bash
$ uv run example.py
```

To run commands without `uv run` you need to manually sync and activate the environment:

```bash
$ uv sync
$ source .venv/bin/activate
$ flask run -p 3000
$ python example.py
```

This involves more steps, and activation can differ per shell and platform. It is error prone and should be avoided.

## Using Tools

Many Python packages provide applications that can be used as tools. You can use `uvx` to easily run tools without the need to install them:

```bash
$ uvx pycowsay hello from uv

  -------------
< hello from uv >
  -------------
   \   ^__^
    \  (oo)\_______
       (__)\       )\/\
           ||----w |
           ||     ||
```

Tools are installed into temporary isolated environments when using `uvx`.

> [!NOTE]
> `uvx` is an alias for convenience. The above is equivalent to `uv tool run pycowsay hello from uv`.

If a tool is used often, it's useful to install it to a persistent environment and add it to the `PATH` instead of invoking `uvx` repeatedly.

```bash
# Install a tool
$ uv tool install ruff

# It should now be available
$ ruff --version

# Upgrade a tool
$ uv tool upgrade ruff

# Upgrade all tools
$ uv tool upgrade --all

# List tools
$ uv tool list

# Uninstall a tool
$ uv tool uninstall ruff
```

## Caching

uv uses aggressive caching to avoid re-downloading (and re-building) dependencies that have already been accessed in prior runs.

If you're running into issues you think is related to the cache, uv includes a few escape hatches:

- To force uv to revalidate cached data for all dependencies, pass `--refresh` to any command (e.g. `uv sync --refresh`).
- To force uv to revalidate cached data for a specific dependency pass `--refresh-package` to any command (e.g. `uv sync --refresh-package flask`).
- To force uv to ignore existing installed versions, pass `--reinstall` to any installation command (e.g. `uv sync --reinstall`).

You can also remove entries from the cache:

- `uv cache clean` removes all cache entries from the cache directory, clearing it out entirely.
- `uv cache clean ruff` removes all cache entries for the `ruff` package, useful for invalidating the cache for one or more set of packages.
- `uv cache prune` removes all unused cache entries. For example, the cache directory may contain entries created in previous uv versions that are no longer necessary and can be safely removed.

## Locking and Syncing

Locking and syncing are important concepts that deserve clarification and reiteration.

Locking is the process of resolving your project's dependencies (specified in `pyproject.toml`) into a lockfile. Syncing is the process of installing packages from the lockfile into the project environment.

Both are automatic in uv. For example, when `uv run` is used, the project is locked and synced before invoking the requested command. This ensures the project environment is always up-to-date.

The lockfile can be manually created or updated with `uv lock`. Likewise, the environment can be manually synced with `uv sync`.

## GitHub Actions

For GitHub Actions, use the official [setup-uv](https://github.com/astral-sh/setup-uv) action. It installs uv, adds it to `PATH`, (optionally) persists the cache, and more, with support for all uv-supported platforms.

```
- uses: astral-sh/setup-uv@v6
  with:
    enable-cache: true
```

You can now use uv on GitHub Actions as you would locally.

> [!NOTE]
> The action will warn if the workdir is empty, because this is usually the case when `actions/checkout` is configured to run after `setup-uv`.

Checkout first, or you can ignore this by setting the `ignore-empty-workdir` input to `true`.

```
- uses: astral-sh/setup-uv@v6
  with:
    ignore-empty-workdir: true
```

> [!TIP]
> You want to make each step of your GitHub Actions workflow as simple as possible to easily identify points of failure. This includes installation of dependencies.

Use `uv sync` to install dependencies and `--locked` to ensure the lockfile is up-to-date. If the lockfile is missing or outdated, uv will exit with an error.

```
- name: Install dependencies
  run: uv sync --locked
```

Check out this sample workflow: [basic_uv.yml](https://github.com/chingc/tutorial-github-actions/blob/main/.github/workflows/basic_uv.yml)

## References

- [uv](https://docs.astral.sh/uv/)
- [uv: Guides](https://docs.astral.sh/uv/guides/)
- [uv: Python versions](https://docs.astral.sh/uv/concepts/python-versions/)
- [uv: Managing dependencies](https://docs.astral.sh/uv/concepts/projects/dependencies/)
- [uv: Running commands in projects](https://docs.astral.sh/uv/concepts/projects/run/)
- [uv: Tools](https://docs.astral.sh/uv/concepts/tools/)
- [uv: Caching](https://docs.astral.sh/uv/concepts/cache/)
- [uv: Locking and syncing](https://docs.astral.sh/uv/concepts/projects/sync/)
