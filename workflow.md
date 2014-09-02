## An easy developer workflow on Ubuntu

Some suggested ways to manage development on Ubuntu.

### Managing language versions

Don't mix apt's or your system's versions of Python, Ruby, or Node with the versions you wish to use in your development.

Instead, a modern best practice is to use lightweight tools that download and install versions of these languages to your home directory. You can have multiple versions installed, and switch between them very easily. These tools will manage the switch via symlinks. You will never need to run a command with `sudo`.

Especially nicely, you can set a version per-directory, so that when you `cd` into a project, the right version of the right language is turned on by default. It's very nice.

These don't compete with project dependency managers: you should still use `bundler` with Ruby, `virtualenv` with Python, and Node's built-in `node_modules`.

#### Ruby with rbenv

Use [rbenv](https://github.com/sstephenson/rbenv). It's extremely popular, and much lighter and simpler than rvm.

You should refer to [the installation instructions](https://github.com/sstephenson/rbenv#basic-github-checkout) for the most up-to-date details, but the setup workflow is this:

* Install to your home directory with `git clone https://github.com/sstephenson/rbenv.git ~/.rbenv`.
* Add the following to your `.bashrc`:

```
export PATH=$HOME/.rbenv/bin:$PATH
eval "$(rbenv init -)"
```

* Add an `rbenv install` command by installing [ruby-build](https://github.com/sstephenson/ruby-build) as a plugin with `git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build`.
* Start a new console session, then test it out by running `rbenv`.

Recommended versions of Ruby to install: the latest 1.9.3, and the latest 2.X (recommended default).

For example, install 2.1.2 with `rbenv install 2.1.2`. Use `rbenv global 2.1.2` to set your global default version of Ruby. Use `rbenv local 2.1.2` to drop a `.ruby-version` file in the current directory, that instructs rbenv to switch to that version upon entering the directory.

To update rbenv and ruby-build (for example, when a new version of Ruby is released), just `cd` into each of their directories and do a `git pull`.

##### Using bundler

You can use bundler just fine, but you will need to run `gem install bundler` once for each new Ruby version. Each Ruby version has its own gemset.

You may need to run `rbenv rehash` after installing bundler for the `bundle` command to appear on the path.

#### Python with pyenv

Use [pyenv](https://github.com/yyuu/pyenv).

pyenv is less popular in Python-land because of the community's strong attachment to virtualenv, because Python ships by default on Ubuntu, and because the Python 2 to 3 migration process has been so messy.

But in my experience, pyenv is a much saner way to manage Python versions and to separate them from your system's dependencies. It also integrates nicely with virtualenv and virtualenvwrapper, and makes it easy to switch between Python 2 and 3.

(If you don't know how virtualenv and virtualenvwrapper work, you really should [learn](http://virtualenvwrapper.readthedocs.org/en/latest/).)

You should refer to the [installation instructions](https://github.com/yyuu/pyenv#basic-github-checkout) for the most up-to-date instructions, but the setup workflow is this:

* Install to your home directory with `git clone https://github.com/yyuu/pyenv.git ~/.pyenv `.
* Install [pyenv-virtualenvwrapper](https://github.com/yyuu/pyenv-virtualenvwrapper) as a plugin with `git clone https://github.com/yyuu/pyenv-virtualenvwrapper.git ~/.pyenv/plugins/pyenv-virtualenvwrapper`.
* Add the following to your `~/.bashrc`:

```
export PYENV_ROOT=$HOME/.pyenv
export PATH=$PYENV_ROOT/bin:$PATH
eval "$(pyenv init -)"
pyenv virtualenvwrapper
```

* Your system Python will need `virtualenvwrapper` to be installed for first use of `pyenv virtualenvwrapper` to work.
* Start a new console session, then test it out by running `pyenv`.

Recommended versions of Python to install: the latest 2.7, and the latest 3.X.

For example, install 3.4.0 with `pyenv install 3.4.0`. Use `pyenv global 3.4.0` to set your global default version of Python. Use `pyenv local 3.4.0` to drop a `.python-version` file in the current directory, that instructs pyenv to switch to that version upon entering the directory.

To update pyenv and pyenv-virtualenvwrapper (for example, when a new version of Python is released), just `cd` into each of their directories and do a `git pull`.

##### Using virtualenv

Install virtualenvwrapper for a Python version with `pyenv virtualenvwrapper` (your `~/.bashrc` will automatically do this if it runs it before you do). Once you've done that, `mkvirtualenv`, `rmvirtualenv`, and `workon` all work as you expect them to.

**Note**: Making a virtualenv will freeze the version of Python that was active when you ran `mkvirtualenv`. Activating that virtualenv will activate the frozen Python version, even if pyenv was using a different one when you ran `workon`.

### Node with nvm

Use [nvm](https://github.com/creationix/nvm).

Honestly, this is less necessary in Node-land than it is in Python or Ruby. It's rare to need to switch between multiple versions of Node. It basically just lets you manage upgrades via simple nvm commands, insteading of downloading tarballs and running `make` commands.

Refer to the installation process for the most up-to-date instructions, but the setup workflow is this:

* Run their install command: `curl https://raw.githubusercontent.com/creationix/nvm/v0.7.0/install.sh | sh`
* Remove the line that was added to the bottom of your `~/.profile` about loading nvm.
* Add the following to your `~/.bashrc`:

```
export NVM_DIR=$HOME/.nvm
source $HOME/.nvm/nvm.sh
```

* Open a new console session, and run `nvm` to test it out.

Then set up nvm and forget about it:

* Install the latest version of Node (0.10 as of this writing) with `nvm install 0.10`.
* Set it to be your default with `nvm alias default 0.10`.

All should be well.
