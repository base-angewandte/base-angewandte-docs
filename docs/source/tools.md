# Tools and Environments

## Python

### uv

`uv` is our main tool to manage Python versions and virtual environments. Check out the
[uv docs](https://docs.astral.sh/uv/) and its Getting Started guide. Here is just a
short quick-ref on how to use it in our projects.

#### Installation

Download and run the installation script (Linux & macOS):
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Everything should be up and running. Test it with `uv --version`. For different
installation procedures check out the [installation section](https://docs.astral.sh/uv/getting-started/installation/)
in the official uv docs. If you don't have a specific requirement for a different
source/package, we suggest using the standalone installer.

#### Usage

Here are just a few commands we regularly use, for more detailed infos check out the
[official docs](https://docs.astral.sh/uv/), especially the _First Steps_ and _Features_
sections.

```bash
# check available python versions and which you have already installed
uv python list

# create a new Python 3.12 virtual environment in the .venv folder
uv venv -p 3.12 .venv

# activate the new environment
source .venv/bin/activate

# sync all packages in your venv with what is listed in requirements.txt
uv pip sync requirements.txt

# deactivate you virtual environment (to switch back to the systems main python environment)
deactivate

# install pre-commit
uv tool install pre-commit --with pre-commit-uv
```

### pyenv and pyenv-virtualenv

Here we outline some notes on installation and usage of pyenv and virtual Python environments
with pyenv-virtualenv.

```{warning}
In our newer setup we are using `uv`, but some older projects still might be documented
based on a pyenv setup. This section is kept here, if you still want to use `pyenv` for
some specific projects. But generally we do advise switching to `uv`.
```

#### Installation

See https://github.com/pyenv/pyenv#installation and https://github.com/pyenv/pyenv-virtualenv#installation for installation instructions.

```{note}
If you use the pyenv installer script, mentioned in the install docs, note that  pyenv-virtualenv
is also installed with it.
```

```{note}
If you are running into performance problems with your shell, remove the line `eval "$(pyenv virtualenv-init -)"` from your shell's `.rc` file.
```

```{important}
On Linux systems pyenv will install new Python versions withouth several module dependencies, if you don't install the
appropriate dev libraries before. This is specifically a problem for `pre-commit`, which depends on `libsqlite3-dev`.
So to be on the safe side install the following libraries (here shown for Ubuntu based distros):

> `sudo apt install libsqlite3-dev liblzma-dev libbz2-dev libncurses5-dev libreadline-dev tk8.6-dev`.

You might decide that you definitely will not need all of those, e.g. all the tk8.6 related packages. But be sure
to double-check for warnings when you do a `pyenv install`, in case you run into mysterious `ModuleNotFound`errors.
(Once you have installed the dev libs, you can also just reinstall a specific python version, without loosing your
existing virtualenvs.)
```

#### Usage

List all available Python versions:

```
pyenv install --list
```

Display latest Release:

```
pyenv latest <prefix>
pyenv latest 3.11
```

Install a Python version:

```
# install latest release
pyenv install <prefix>
pyenv install 3.11

# install a specific version
pyenv install <version>
pyenv install 3.11.1
```

List Python versions you have installed:

```
pyenv versions
```

Switch Python versions globally, locally or for current shell session:

```
# global [select globally for your user account]
pyenv global <version>
pyenv global 3.11.1

# local [automatically select whenever you are in the current directory (or its subdirectories)]
pyenv local <version>
pyenv local 3.11.1

# current shell session
pyenv shell <version>
pyenv shell 3.11.1
```

Uninstall Python version(s):

```
pyenv uninstall <versions>
pyenv uninstall 3.11.1
```

Create a virtualenv:

```
pyenv virtualenv <version> <name>
pyenv virtualenv 3.8.5 showroom
```

Activate/Deactivate a virtualenv:

```
# activate
pyenv activate <name>
pyenv activate showroom

# deactivate
pyenv deactivate
```

```{note}
You can also set a virtualenv locally with `pyenv local <name>`.
```

List existing virtualenvs:

```
pyenv virtualenvs
```

Delete existing virtualenv:

```
pyenv uninstall <name>
pyenv uninstall showroom
```

For more background on managing Python virtual environments see this article:

- [Pyenv: A Complete Guide to Managing Python Versions](https://ioflood.com/blog/pyenv/#Installing_and_Switching_Between_Python_Versions). Gabriel Ramuglia @ I/O Flood. 2023-09-05.

## Git

### BFG Repo-Cleaner

BFG can be used to remove sensitive data from a repository.

#### Usage

1. Install BFG, e.g. via homebrew for MacOS, or download the jar-File from https://rtyley.github.io/bfg-repo-cleaner/

2. BFG doesn't modify the latest commit (on `main` or `HEAD`), so ensure it is already clean.

3. Clone a fresh copy of your repo, using the `--mirror` flag:

   ```
   git clone --mirror ssh://git@github.com:base-angewandte/example.git
   ```

4. Clean Files and Strings.

   Examples for deleting files:

   ```
   bfg --delete-files id_{dsa,rsa}  example.git
   bfg --delete-files "file_name_*.py"  example.git
   ```

   Example for removing blob:

   ```
   bfg --strip-blobs-bigger-than 50M  example.git
   ```

   Example for replacing strings listed in a file (lines can be prefixed with `regex:` or `glob:` if required) with `***REMOVED***`:

   ```
   bfg --replace-text passwords.txt  example.git
   ```

   ```{note}
   If you are using the jar file, `bfg` is an alias for `java -jar bfg.jar` in all examples.
   ```

5. BFG will update your commits and all branches and tags so they are clean, but it doesn't physically delete the unwanted stuff. So you need to run the following to achieve that as well:

   ```
   cd example.git
   git reflog expire --expire=now --all && git gc --prune=now --aggressive
   ```

6. Push the cleaned repository:

   ```
   git push
   ```

   ```{note}
   This will update **all** refs on the remote server.
   ```
