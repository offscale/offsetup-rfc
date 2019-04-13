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

This RFC explains how this system could be built, and its limitations.

# Terminology

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**",
"**SHOULD NOT**", "**RECOMMENDED**", "**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this
document are to be interpreted as described in BCP 14 [@!RFC2119] [@!RFC8174] when, and only when,
they appear in all capitals, as shown here.

# Command-line interface
## `--init`, `init`, `--new`, `new`
Create a new configuration file. Will infer simple dependencies from existence Node.Js, Go or Python, but not more complicated dependenceis like Postgres and Redis.

## `{configuration}` (positional), `--configuration`, `--config`, `-c`
Specify configuration file, if not specified will look for `.offsetup.yml` or `offsetup.yml`.

# Configuration file(s)
Filename begins with `offsetup`, `.offsetup`, with optional suffixes of `-` followed by alphanumeric sequences, and a `.yml` extension (referring to YAML).
```yaml
name: project name
version: '0.1.4'
dependencies:
  os:
    linux:
      ubuntu:
        versions:
        - '14.04'
        - '>16.04'
      run:
        with_prefix: sudo apt install -y
        - build-essential
        - cmake
  systems:
    postgres:
      version: '>9.6.4'
      env: RDBMS_URI
      features:
      - postgis
    redis:
      version: '>5'
      env: REDIS_URL
```

{backmatter}
