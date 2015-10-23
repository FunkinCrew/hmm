# Haxe Module Manager (`hmm`)

`hmm` is a small helper for `haxelib` that allows you to specify, install,
and update project dependencies using lib.haxe.org libraries, git, or
mercurial repositories.

This exists because `haxelib` does not yet support specifying git, mercurial,
or other non `haxelib`-based project dependencies in the `haxelib.json`,
`.hxml`, etc. files.  Once `haxelib` adds full git and hg support, this
project can probably go away.

`hmm` relies on the new `haxelib` local repo support, for installing project-local
Haxe libs in a `.haxelib` directory.  See `haxelib newrepo` for more
details on this.  It also uses `haxelib` itself for actually installing
libraries (both from lib.haxe.org and via git/hg/etc.).

# Installing hmm

```
> haxelib --global install hmm
> haxelib --global run hmm setup
```

- The `--global` flag is needed to make sure the tool is installed globally.
- The `setup` command creates a link to the `hmm` tool at `/usr/local/bin/hmm` for ease of use.
- If you do not run `setup`, you can use the tool by running:

`haxelib --global run hmm [command] [options]`

rather than:

`hmm [command] [options]`

# Example workflows

### Installation

```sh
# Make sure hmm is installed (only needed once)
> haxelib --global install hmm
> haxelib --global run hmm setup
```

### Self-update (update the global hmm install)
```sh
> hmm hmm-update
```

### Self-removal (uninstall the global hmm install)
```sh
> hmm hmm-remove
```

### New project setup and dependency installation

```
# Create your project directory
> mkdir my-project
> cd my-project

# initialize hmm and local .haxelib (create hmm.json and empty .haxelib/ directory in my-project/
> hmm init

# install some libraries from lib.haxe.org
> hmm haxelib utest
> hmm haxelib chrome-extension 45.0.1

# install some libraries via git repos
> hmm git thx.core git@github.com:fponticelli/thx.core master src
> hmm git mithril https://github.com/ciscoheat/mithril-hx master mithril

# install some libraries via mercurial repos
> hmm hg orm https://bitbucket.org/yar3333/haxe-orm default library

# view the hmm.json
> cat hmm.json
{
  "dependencies": [
    {
      "name": "chrome-extension",
      "type": "haxelib",
      "version": "45.0.1"
    },
    {
      "name": "mithril",
      "type": "git",
      "dir": "mithril",
      "ref": "master",
      "url": "https://github.com/ciscoheat/mithril-hx"
    },
    {
      "name": "orm",
      "type": "hg",
      "dir": "library",
      "ref": "default",
      "url": "https://bitbucket.org/yar3333/haxe-orm"
    },
    {
      "name": "thx.core",
      "type": "git",
      "dir": "src",
      "ref": "master",
      "url": "git@github.com:fponticelli/thx.core"
    },
    {
      "name": "utest",
      "type": "haxelib",
      "version": ""
    }
  ]
}

# view the local haxelib installs
> haxelib list
chrome-extension: [45.0.1]
mithril: git [dev:/Users/awhite/temp/haxelib-test-3/.haxelib/mithril/git/mithril]
orm: hg [dev:/Users/awhite/temp/my-project/.haxelib/orm/hg/library]
thx.core: git [dev:/Users/awhite/temp/haxelib-test-3/.haxelib/thx,core/git/src]
utest: [1.3.10]
```

### Existing project setup if the project has an hmm.json file
```sh
# Clone a project that is using hmm
> git clone git@github.com:someuser/someproject
> cd someproject

# Install all libs from hmm.json
> hmm install
```

# Project config file (hmm.json)

`hmm` requires a `hmm.json` file in the root of your project, which
specifies the project dependencies (similar to npm's package.json).

Example `hmm.json`:

```json
{
  "dependencies": [
    {
      "name": "thx.core",
      "type": "haxelib",
      "version": "0.34.0"
    },
    {
      "name": "thx.promise",
      "type": "git",
      "url": "git@github.com:fponticelli/thx.promise",
      "ref": "master",
      "dir": "src"
    },
    {
      "name": "svg2ppt",
      "type": "git",
      "url": "git@github-pellucid:pellucidanalytics/svg2ppt",
      "ref": "master",
      "dir": "src"
    },
    {
      "name": "mithril",
      "type": "haxelib"
    }
  ]
}
```

Each dependency is an object with the following keys:

- `name` - the name of the Haxe library
- `type` - one of `haxelib`, `git`, or `hg`
- `version` - the haxelib library version (for `haxelib` libs)
- `url` - the git/hg URL (for `git` or `hg` libs)
- `ref` - the git/hg ref (branch, commit, tag, etc.) (for `git` and `hg`
  installs)
- `dir` - the root classpath directory (for `git` and `hg` installs)

# Commands

- Almost all `hmm` commands should be run from your project root
  directory (where the `hmm.json` file should be located).
- See `hmm help` for information about specific commands.

```sh
> hmm help
Usage: hmm <command> [options]

commands:

    help - shows a usage message

    version - print hmm version

    setup - creates a symbolic link to hmm at /usr/local/bin/hmm

    init - initializes the current working directory for hmm usage

    from-hxml - reads -lib lines specified in one or more .hxml files and adds them to hmm.json

        usage: hmm from-hxml <hxml-path> [hxml-path2 hxml-path3 ...]

        example:
        hmm from-hxml build.hxml
        - adds dependencies in hmm.json for each -lib line in build.hxml

    to-hxml - dumps the libraries in hmm.json in a `-lib name[:version]` format for use in an .hxml file

    install - installs libraries listed in hmm.json

    haxelib - adds a lib.haxe.org-based dependency to hmm.json, and installs the dependency using `haxelib install`

        usage: hmm haxelib <name> [version]

        arguments:
        - name - the name of the library (required)
        - version - the version to install (default: "")

        example:

        hmm haxelib thx.core
        - adds and installs the current version of thx.core (no version specified)

        hmm haxelib thx.core 1.2.3
        - adds and installs thx.core version 1.2.3

    git - adds a git-based dependency to hmm.json, and installs the dependency using `haxelib git`

        usage: hmm git <name> <url> [ref] [dir]

        arguments:
        - name - the name of the library (required)
        - url - the clone url or path to the git repo (required)
        - ref - the branch name/tag name/committish to use when installing/updating the library (default: "master")
        - dir - the sub-directory in the git repo where the code is located (default: "")

        ref and sub-directory are optional, however, to specify sub-directory, you must also specify the ref.

        example:

        hmm git thx.core git@github.com:fponticelli/thx.core
        - assumes ref is "master" and sub-directory is the root of the repo

        hmm git thx.core git@github.com:fponticelli/thx.core some-tag src
        - assumes ref is "some-tag" and sub-directory is "src"


    hg - adds a hg-based dependency to hmm.json, and installs the dependency using `haxelib hg`

        usage: hmm hg <name> <url> [ref] [dir]

        arguments:
        - name - the name of the library (required)
        - url - the clone url or path to the hg repo (required)
        - ref - the branch name/tag name/committish to use when installing/updating the library (default: "default")
        - dir - the sub-directory in the hg repo where the code is located (default: "")

        ref and sub-directory are optional, however, to specify sub-directory, you must also specify the ref.

        example:

        hmm hg orm https://bitbucket.org/yar3333/haxe-orm default library
        - install "orm" from bitbucket at branch "default" with sub-directory "library"


    update - updates libraries listed in hmm.json

    remove - removes one or more library dependencies from hmm.json, and removes them via `haxelib remove`

        usage: hmm remove <name> [name2 name3 ...]

        arguments:
        - name - the name of the library to remove (required)
        - name2 name3 ... - additional library names to remove

        example:

        hmm remove thx.core
        hmm remove thx.core mithril


    check - checks if the current .haxelib installations match the hmm.json-specified versions/refs/etc.

    clean - removes local .haxelib directory

    hmm-update - updates the hmm tool

    hmm-remove - removes the hmm tool
```
