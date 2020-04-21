
# How to setup a macOS development machine

By default macOS environments are configured to execute lots of different types of applications, but executing things and compiling things are two very different activities that require totally different libraries and system configurations. The following steps will change those default behaviors to make setting up and maintaining development projects a whole lot easier. IOW, this is how to setup a macOS development environment from scratch.

> Every project is different and every Developer understands that in order to be effective in today's modern world of programming tools you have to be able, and willing, to wear a lot of hats. This phenomena is often referred to as "polyglot programming". But before you can do that effectively you need to understand your machine and how to turn it into a swiss army knife.
> 
> &mdash; Tobius

We will be setting up seven specific technology capabilities (in this order):

1. [Xcode](#xcode) _to support iOS development and to re-add the missing Apple development libraries_
- [Homebrew](#homebrew) _to support nearly every open source macOS software package that exists_
- [Ruby](#ruby) _to support all Ruby programming language versions and Gem dependencies_
- [Python](#python) _to support all Python programming language versions and Pod dependencies_
- [Node](#node) _to support all Node.js programming language versions and Node package dependencies (no cute name on this one)_
- [Java](#java) _to support all Java programming language versions_
- [Android](#android) _to support Android development_

We will be achieving three main goals:

- Add native support for every major (hence popular) programming language
- Eliminate root permission errors (e.g. never use `sudo` to install packages)
- Eliminate version lock-in issues (e.g. support custom dependency versions per project)

## <a id="xcode"></a>Step 1: Xcode

Xcode has everything you need to build iOS applications and quite a few things that you don't. In short Xcode is, in some ways, the difference between having "user" libraries and "development" libraries installed on your macOS environment. Lots of development tools are built on top of those development libraries so we need to add them first.

1. Update macOS to the latest available distro through the App store and install the latest security updates
- Install Xcode via the [App store](https://apps.apple.com/us/app/xcode/id497799835)
- Launch Xcode and accept the license agreements to be able to install other components
- Select the command line tools in Xcode (Preferences -> Locations -> Command Line Tools)
- Verify the installation with `xcodebuild -version`

For more info:

- Xcode Overview via [developer.apple.com/documentation/xcode](https://developer.apple.com/documentation/xcode/)

## <a id="homebrew"></a>Step 2: Homebrew

Homebrew is "the missing package manager for macOS" that we will be using to install, manage, and update most system level package dependencies.

- Install Homebrew via `curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh | bash`
- Update Homebrew via `brew update`

For more info:

- Homebrew website via [brew.sh](https://brew.sh/)

## <a id="ruby"></a>Step 3: Ruby

Ruby is pre-installed on macOS, but it is an older version that requires constant toggling between user and root permission levels and has no concept of switching between multiple versions (a constant need for Developers). To solve for this we will be using `rbenv` to gain full control over Ruby versions and Gem packages.

But before that we want to replace the built-in versions of Readline and OpenSSL with the Homebrew versions to preempt a few known incompatiblity issues that occur with the built-in macOS versions.

- Install readline and openssl via `brew install readline openssl`
- Append rules to `.zshrc`:

```zsh
# Configure compiler paths
local READLINE_PATH=$(brew --prefix readline)
local OPENSSL_PATH=$(brew --prefix openssl)
export LDFLAGS="-L$READLINE_PATH/lib -L$OPENSSL_PATH/lib"
export CPPFLAGS="-I$READLINE_PATH/include -I$OPENSSL_PATH/include"
export PKG_CONFIG_PATH="$READLINE_PATH/lib/pkgconfig:$OPENSSL_PATH/lib/pkgconfig"

# Use Homebrew version before system version
export PATH=$OPENSSL_PATH/bin:$PATH
```

Now we're ready to install rbenv.

- Install rbenv via `brew install rbenv`
- Append rules to `.zshrc`:

```zsh
# Use the OpenSSL from Homebrew
# Note: ruby-build has an internal version of openssl but it doesn't get updated
export RUBY_CONFIGURE_OPTS="--with-openssl-dir=$OPENSSL_PATH"

# Extract the latest semantic version of Ruby from rbenv (for convenience)
export LATEST_RUBY_VERSION=$(rbenv install -l | grep -Eo '\d+\.\d+\.\d+' | tail -1)

# Load rbenv
eval "$(rbenv init -)"
```

- Restart the Terminal
- Install the latest stable version of Ruby via `rbenv install $LATEST_RUBY_VERSION`
- Set the latest stable version of Ruby as the default via `rbenv global $LATEST_RUBY_VERSION`
- Restart the Terminal

Now you can switch between ruby versions manually with `rbenv`, automatically by adding `.ruby-version` files to your project folders, and install gem dependencies (per Ruby version) without `sudo` workarounds and permission errors.

_Note: There are a lot of tutorials and instructions out there that will tell you to use `sudo gem install` commands. Please know that unless you are buiding a production system by hand they are all wrong. Never do that on a development machine, it will come back to haunt you._

_Note: The other very popular Ruby version management solution is RVM but I refuse to use it because it does incredibly invasive things to your system like overriding the `cd` command. No thank you, `rbenv` provides all of the same functionalities but in a far less invasive way._

For more info:

- Ruby programming language via [ruby-lang.org](https://www.ruby-lang.org/en/)
- rbenv project via [github.com/rbenv/rbenv](https://github.com/rbenv/rbenv)

## <a id="python"></a>Step 4: Python

An older (2.x) version of Python is pre-installed on macOS and, besides being incompatible with most modern Python scripts (3.x), it suffers all the same issues that the system installed version of Ruby suffers from; root permission errors, global version lock-ins, etc. To solve for this we will be using `pyenv` which will feel very familiar to `rbenv` with nearly identical CLI syntaxes since it was originally forked from the `rbenv` project.

- Install pyenv via `brew install pyenv`
- Append rules to `.zshrc`:

```zsh
# Load pyenv
eval "$(pyenv init -)"

# Extract the latest semantic version of Python from pyenv (for convenience)
export LATEST_PYTHON_VERSION=$(pyenv install -l | grep -Eo '\d+\.\d+\.\d+' | tail -1)
```

- Restart the Terminal
- Install the latest stable version of Python via `pyenv install $LATEST_PYTHON_VERSION`
- Set the latest stable version of Python as the default via `pyenv global $LATEST_PYTHON_VERSION`
- Restart the Terminal

For more info:

- Python programming language via [python.org](https://www.python.org/)
- pyenv project via [github.com/pyenv/pyenv](https://github.com/pyenv/pyenv)

## <a id="node"></a>Step 5: Node

Node is not pre-installed on macOS so there's nothing to fix or replace, we just need to add it.

- Install nvm via `brew install nvm`
- Restart the Terminal
- Install the latest stable version of Node via `nvm install $LATEST_NODE_VERSION`
- Set the latest stable version of Node as the default via `nvm alias default $LATEST_NODE_VERSION`
- Restart the Terminal

For more info:

- Node.js programming language via [nodejs.org](https://nodejs.org/en/)
- nvm project via [github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

## <a id="java"></a>Step 6: Java

Java is no longer pre-installed on macOS so there's nothing to fix or replace, we just need to add support for the versions that we want to use.

- Install `jabba` via `curl -sL https://github.com/shyiko/jabba/raw/master/install.sh | bash -s -- --skip-rc`
- Append rules to `.zshrc`:

```zsh
# Load jabba
[ -s "$HOME/.jabba/jabba.sh" ] && source "$HOME/.jabba/jabba.sh"

# Extract the latest semantic version of Java from jabba
export LATEST_JAVA_VERSION=$(jabba ls-remote --os darwin | grep -Eo 'zulu@\d+\.\d+\.\d+.+' | sort -V | tail -1)
```

- Restart the Terminal
- List the available Java versions via `jabba ls-remote`
- Install the latest stable version of Java via `jabba install $LATEST_JAVA_VERSION`
- Restart the Terminal

For more info:

- Java programming language via [docs.oracle.com/javase/specs](https://docs.oracle.com/javase/specs/)
- jabba project via [github.com/shyiko/jabba](https://github.com/shyiko/jabba)
- OpenJDK builds by Zulu via [azul.com/products/zulu-community](https://www.azul.com/products/zulu-community/)

## <a id="android"></a>Step 7: Android

Android Studio has everything you need to build Android applications, especially now that we are no longer reliant on a version locked in Java SDK solution.

- Download and Install Android Studio via [developer.android.com/studio](https://developer.android.com/studio)
- Choose "Custom Install"
- Under "SDK Components Setup" enable everything (AVD is not checked by default)
- Append rules to `.zshrc`

```zsh
# Add Android tools to the $PATH
export ANDROID_HOME="$HOME/Library/Android/sdk"
export ANDROID_SDK_ROOT="$HOME/Library/Android/sdk"
export PATH="$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools"
```

- Restart the Terminal

For more info:

- Android Studio Overview via [developer.android.com/studio/intro](https://developer.android.com/studio/intro)

