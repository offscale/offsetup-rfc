%%%
title = "Offsetup"
abbrev = "OSETUP"
area = "Internet"
workgroup = "Offscale.io"
keyword = ["configuration", "cross-platform"]
#date = 2019-12-07T00:00:00Z

[seriesInfo]
name = "RFC"
value = "OSETUPv0.0.1"
status = "informational"

[[author]]
surname="Marks"
fullname="Samuel Marks"
organization = "Sydney Scientific"
  [author.address]
  email = "samuel@offscale.io"
%%%

{mainmatter}

# Overview
In application development, projects tend to be multi-tier (e.g.: database, backend, and frontend).

Software-engineers develop in Docker, Windows, Linux, Mac & others. Generally the README.md or INSTALL.md specify how to install on each platform. This includes dependencies like CMake, Node.js and Postgres.

This document proposes a new [dotfile](https://wiki.archlinux.org/index.php/Dotfiles)—to be placed in the root of a repository—that explicitly outlines dependencies for each OS and platform.

This acts in a similar way to PaaS config files, with the exception that this isn't limited to one operating system (e.g.: a specific Linux distribution), or one platform (e.g.: Heroku, CloudFoundry).

## Side point
To prevent potential issues, and improve performance, the tools which work on this dotfile must be statically built, e.g.: in C, C++, D, Go, Rust, as anything which has external dependencies could have their dependencies replaced by this same tool.

This RFC explains how this system could be built, and its limitations.

# Terminology

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**",
"**SHOULD NOT**", "**RECOMMENDED**", "**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this
document are to be interpreted as described in BCP 14 [@!RFC2119] [@!RFC8174] when, and only when,
they appear in all capitals, as shown here.

# Command-line interface
## `--init`, `init`, `--new`, `new`
Create a new configuration file. Will infer simple dependencies from existence Node.Js, Go or Python, but not more complicated dependencies like Postgres and Redis.

## `{configuration}` (positional), `--configuration`, `--config`, `-c`
Specify configuration file, if not specified will look for `.offsetup.yml` or `offsetup.yml`.

## `install`, `--install`, `-i`
Install the project, and all its dependencies.

## `uninstall`, `remove`, `--uninstall`, `--remove`, `--rm`
Remove the project. Use `--remove-libs` to also remove the OS dependencies.

## `start`, `run`, `up`
Runs the project. Will inform user to run `install` [manually] if any of the dependencies aren't met.

## `stop`, `down`
Stops the project. Will have a nonzero exit code and a warning message if it's not started.

## `--install-priority`
Comma separated list of priorities, will take precedence over whatever is in the the config file.

# Configuration file(s)
Filename begins with `offsetup`, `.offsetup`, with optional suffixes of `-` followed by alphanumeric sequences, and a `.yml` extension (referring to YAML).

## Full example
```yaml
name: random python project name
version: '0.1.4'
dependencies:
  platforms:
    ubuntu:
      versions:
      - '14.04'
      - '>16.04'
      apt:
      - build-essential
      - cmake
      - python2
      - curl
    mac:
      versions:
      - '>=10.14'
      brew:
      - cmake
      - python@2
      - curl
    windows:
      version:
      - >=7600
      arch: x86_64
      download_directory: 'C:\opt\Downloads'
      download:
      - uri: https://www.7-zip.org/a/7z1900-x64.msi
        sha512: ''
      - uri: https://github.com/mridgers/clink/releases/download/1.0.0a1/clink-1.0.0a1.823d84.zip
        sha512: ''
      - uri: https://github.com/git-for-windows/git/releases/download/v2.21.0.windows.1/Git-2.21.0-64-bit.exe
        sha512: ''
      - uri: https://download.microsoft.com/download/7/9/6/796EF2E4-801B-4FC4-AB28-B59FBF6D907B/VCForPython27.msi
        sha512: ''
      - uri: https://github.com/Kitware/CMake/releases/download/v3.14.2/cmake-3.14.2-win64-x64.msi
        sha512: ''
      - uri: https://www.python.org/ftp/python/2.7.16/python-2.7.16.amd64.msi
        sha512: ''
      - uri: https://curl.haxx.se/windows/dl-7.64.1/curl-7.64.1-win64-mingw.zip
        sha512: ''
      install_prefix: 'C:\opt\bin'
      install_all: true
  applications:
    postgres:
      pkg: https://github.com/offscale/offpostgres
      version:
      - '>9.6.4'
      env: RDBMS_URI
      features:
      - postgis
      skip_install: false
      install_priority:
      - docker
      - native
      users:
      - name: awesome_user
        password: '$env_var'
      databases:
      - name: awesome_db
        owner: awesome_user
    redis:
      version: '>5'
      env: REDIS_URL
      skip_install: true
      fail_silently: true
      install_priority:
      - docker
      - native
exposes:
  ports:
    - 80
    - 443
```

## Tiny example
To enable small YAML files—similar to the ones you get from various dotfile CI/CD tools—one can define a base YAML (like above), and reference it like here:

### Referencing configuration
```yaml
name: project name
version: '0.1.4'
inherits: 'file:///opt/config/go.yml'
```

## Fields

### `name`
String containing name of project.

### `version`
String containing version of project.

### `inherits` (optional)
String in [@!RFC3986] (for `file:` [@!RFC8089]) format, linking a YAML file to inherit from.

### `dependencies` (optional, if `inherits` is provided)
MAY contain `os` key AND MAY contain `systems`

### `dependencies.platforms.<platform>`
Describes the requirements for this platform.

#### `.<platform specific dependency manager>`
The `brew` and `apt` lines referenced in [the full example](#name-full-example) above are not given the names `os_specific_package_manager` because there are multiple available on each platform, so it's best to leave that open (not to mention it's more explicit).

Additionally, by avoiding `sudo apt-get install -y <package name>` lines, we can guard the installs with a:
```sh
dpkg-query --showformat='${Version}' --show 'postgresql-9.6'
```

Which is much speedier than running `apt-get install -y`.

#### `.version`
Platform version contained in a string. Prefer to use a build number like `>=7600` rather than `>=Windows 7`, however both SHOULD be supported.

#### `.arch` (optional)
Architecture of system. Can be a string or a list of strings (if you support multiple architectures).

#### `.download_directory` (optional)
String pointing to download directory. Directory is created if it does not exist AND the `.download` key is non empty.

#### `.install_prefix` (optional)
Location to install archives & installers from the `.download` key.

#### `install_all` (optional)
Boolean when, if set to true, will sequentially extract/install all that was downloaded in the `download` key.


#### `download` (optional)
List of Strings, which refer to paths to download from. Strings are in [@!RFC3986] (for `file:` [@!RFC8089]) format. Expectation is this can be processed in parrallel.

### Applications
This will do one of two things:

  0. Confirm connection to chosen applications—if already available, accessible through `env` var—and confirm version is in correct range
  1. Install the chosen systems

#### Authoring a new application
[Full example](#name-full-example) is the syntax for authoring new applications, note that its `dependencies.applications` field is optional.

#### Installing application(s)
Application(s) will be installed **once**, if `skip_install` is not `false`, the system is not able to be connected to through `env` var, and if the service is unable to be started through regular means (e.g.: `sudo systemctl start <service_name>`).

Attempts to install the system will be made in the order of `install_priority`.

##### `pkg`
If there is a package for the chosen system in the `pkg` key [or in our registry (i.e.: `offscale/off<system>`)], then use that version. Formalising the format of the packages in the registry may be done in another RFC. Until then, keep in mind that it should be idempotent and accept arbitrary k/v pairs.

###### `native`
Install locally. TBD how to handle if they prefer system wide or extracting—and possibly building—from archives.

###### `docker`
If on FreeBSD, and Docker is required, it'll be ignored and the next item in the `install_priority` list will be chosen.

If Docker, and `docker` is not found on the system, it will be installed.

`service_name::docker` || `service_name.docker` namespace will contain the Docker installer. It will do all the `docker build`, `docker pull`, `docker run` (and related inverted functions to make this idempotent).

###### `cloud`
Deploy to any remote or local provider. Details TBD.

{backmatter}
