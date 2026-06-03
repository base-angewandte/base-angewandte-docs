# Local Dev System Setup

## Linux

This is the basic setup guide for a fresh Linux client system, to
have everything ready in order to work on our different base applications.
This should work on most Debian-based distributions. It was specifically tested on:

- Debian 13 (Trixie)

Here are the steps to install everything you will need to work with the _base_ codebase:

1. Install some basic packages needed in our setup:
   - ```
     sudo apt install curl git build-essential libssl-dev \
       libffi-dev zlib1g-dev libsqlite3-dev liblzma-dev libbz2-dev \
       libncurses5-dev libreadline-dev tk8.6-dev
     ```
   - ```{note}
     Disclaimer: if you really solely work on frontend code or only on backend code, you could either
     leave out the following step 2 (uv) or 3 (nvm), to not clutter your system. But we suggest
     installing both, so you are ready also to run frontend or backend code on your machine any time -
     which will be necessary for testing at some point, when you do not want or cannot rely on the
     dev containers.
     ```
2. Install [uv](https://docs.astral.sh/uv/) based on the
   [uv install instructions](https://docs.astral.sh/uv/getting-started/installation/).
   We suggest to use the standalone installer if you don't have a specific setup that requires another method.
   - Check if everything is working with `uv` and `uv --version`.
   - For a quick ref on how to use `uv` check the [](./tools.md) section.
3. Install [nvm](https://github.com/nvm-sh/nvm) and the latest LTS version of Node.js:
   - ```bash
     curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
     source ~/.bashrc
     nvm install --lts
     ```
   - Check if everything is working with `nvm -v` and `node -v`.
4. Install docker
   - You can either install the whole [Docker Desktop](https://docs.docker.com/desktop/) or just install the
     [Docker Engine](https://docs.docker.com/engine/) (and make sure the `docker compose` plugin works)
     - some of our projects are still tailored towards `docker-compose` instead of `docker compose`. While this
       should be phased out soon, in the meantime you can emulate `docker-compose` by doing the following:
       ```bash
       echo 'docker compose "$@"' | sudo tee /usr/local/bin/docker-compose
       sudo chmod +x /usr/local/bin/docker-compose
       ```
   - A note on permissions: by default you will need root privileges to user docker. If you do
     not want to always use `sudo` in front of e.g. `docker ps`, you can add your system user
     to the group `docker`, e.g. with `sudo usermod -a -G docker myusername`. Start a new
     terminal session afterward, so that the group info is updated in the environment.
5. Set up git
   - Follow the instructions in [the setup section of the git guide](./using_git.md#setup) to set up `git`.

Now you have a basic setup, except for an IDE you might want to use to work on code.
We mostly use PyCharm for backend and WebStorm for frontend stuff, but any other preferred IDE can be used,
as long as our coding guidelines are followed.

Depending on which projects you work on, and whether you do frontend or backend stuff,
there will be some additional requirements specific to the project.
This is described in the project's documentation.
The general procedure looks like this:

1. Clone the required repositories
2. Create how many environments you need (recommended: one environment per project) with `uv` for the backend stuff
   and `nvm` to activate the proper node environment for frontend stuff.
   - See the [](./tools.md) section for a quick reference on how to use pyenv.
3. Follow the steps on in either the README.md in the repo root folder or the requirements.md and install.md files
   in the docs/source/ (or in older projects only /docs) folder.
