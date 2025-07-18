---
layout: post
title:  "Use Pipx to Install Aider and Other Non-System-Managed Python Packages on Linux"
date:   2025-07-17
#tags: [python, aider, linux, debian]
---

## Bottom Line Up Front

Aider is a popular AI-powered coding assistant for the command line. Here are the commands to install Aider using pipx:

```bash
sudo apt update
sudo apt install -y pipx

# Adds .local/bin to PATH in .bashrc
pipx ensurepath

pipx install aider-chat

# Required for aider to use playwright for web scraping: Installs playwright in the same virtual environment as aider
# --include-apps installs the executable in a PATH-accessible location, e.g., `.local/bin`
pipx inject --include-apps aider-chat playwright

# Aider appears to require pandoc in addition to playwright when scraping
sudo apt install -y pandoc

# Download chromium for playwright (note: requires sudo to install additional dependencies from apt) 
playwright install --with-deps chromium

# Required if you're using AWS Bedrock for your LLM
pipx inject aider-chat boto3
```

## The Problem with System-Managed Python

Trying to install Aider with regular pip on Debian 12 results in this error:

```bash
$ pip install -U aider-install
error: externally-managed-environment

× This environment is externally managed
╰─> To install Python packages system-wide, try apt install
    python3-xyz, where xyz is the package you are trying to
    install.
    
    If you wish to install a non-Debian-packaged Python package,
    create a virtual environment using python3 -m venv path/to/venv.
    Then use path/to/venv/bin/python and path/to/venv/bin/pip. Make
    sure you have python3-full installed.
    
    If you wish to install a non-Debian-packaged Python application,
    it may be easiest to use pipx install xyz, which will manage a
    virtual environment for you. Make sure you have pipx installed.
    
    See /usr/share/doc/python3.11/README.venv for more information.

note: If you believe this is a mistake, please contact your Python installation or OS distribution provider. You can override this, at the risk of breaking your Python installation or OS, by passing --break-system-packages.
hint: See PEP 668 for the detailed specification.
```

## Using Pipx to Install Non-System-Managed Python Packages

By default, Python packages are managed by `apt` on Debian, just like any other package, and they can be installed like this: `sudo apt install python3-<package>`.
Debian's official repositories provide a curated selection of Python packages that have been packaged and reviewed by the Debian community, and can be installed with `apt`. 
The advantages of using official Debian-provided Python packages from `apt` are stability and security.
However, the downsides, when compared to installing packages from the Python Package Index (PyPI), which is where packages are pulled from when you `pip install <some package>`, are:
1. Fewer packages available
2. The most recent version is likely to be unavailable
3. Must be installed system-wide with sudo privileges

## Installing Packages with Pipx

Following the first recommendation from the error message, you could create a virtual environment using `python3 -m venv .venv`. However, this requires manual activation every time you want to use the tool, which can become cumbersome, especially with multiple virtual environments for different tools.

Instead, you can follow the second recommendation and use pipx, which manages virtual environments for you and is a more straightforward solution.

`pipx` creates a virtual environment per CLI tool, so you don't have to manage multiple virtual environments.
Install pipx from the official Debian repository with `sudo apt install -y pipx`.
After this, to install a package, run `pipx install <desired package>`. This creates a new virtual environment just for the package you wanted to install.

If the package is a CLI tool, the executable is placed in a bin directory for your user (`~/.local/bin/` on Debian) and is given a shebang pointing to the Python executable located in `~/.local/pipx/venvs/<package>/bin/`.
Next, use `pipx ensurepath` to add the local bin directory to your PATH.

If you need to install another package as a dependency for a specific tool, like `playwright` for `aider` to use for web scraping, you can use `pipx inject <name of the first package (e.g., aider)> <dependency package (e.g., playwright)>`. Add the `--include-apps` flag to add the installed tool to your PATH.
This is necessary to run `playwright install --with-deps chromium` to install headless chromium for scraping.

## Troubleshooting

One thing to note: I had to install pandoc on the system to avoid this error when scraping with Aider:

```bash
Unable to install pandoc: [Errno 2] No such file or directory: 'ar'
```
