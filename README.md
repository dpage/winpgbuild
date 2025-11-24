# winpgbuild

A repo containing Github actions for building PostgreSQL and it's dependencies
on Windows. A number of build tools are also included to enable reproducable
builds, though these are generally downloaded as pre-built utilities.

## Build Status

| Category | Workflow | Status |
|----------|----------|--------|
| **Core Libraries** | Build gettext | ![Build gettext](https://github.com/dpage/winpgbuild/actions/workflows/gettext.yml/badge.svg) |
| | Build ICU | ![Build ICU](https://github.com/dpage/winpgbuild/actions/workflows/icu.yml/badge.svg) |
| | Build KRB5 | ![Build KRB5](https://github.com/dpage/winpgbuild/actions/workflows/krb5.yml/badge.svg) |
| | Build libiconv | ![Build libiconv](https://github.com/dpage/winpgbuild/actions/workflows/libiconv.yml/badge.svg) |
| | Build libxml2 | ![Build libxml2](https://github.com/dpage/winpgbuild/actions/workflows/libxml2.yml/badge.svg) |
| | Build libxslt | ![Build libxslt](https://github.com/dpage/winpgbuild/actions/workflows/libxslt.yml/badge.svg) |
| | Build OpenSSL | ![Build OpenSSL](https://github.com/dpage/winpgbuild/actions/workflows/openssl.yml/badge.svg) |
| | Build ossp-uuid | ![Build ossp-uuid](https://github.com/dpage/winpgbuild/actions/workflows/ossp-uuid.yml/badge.svg) |
| **Compression** | Build lz4 | ![Build lz4](https://github.com/dpage/winpgbuild/actions/workflows/lz4.yml/badge.svg) |
| | Build zlib | ![Build zlib](https://github.com/dpage/winpgbuild/actions/workflows/zlib.yml/badge.svg) |
| | Build zstd | ![Build zstd](https://github.com/dpage/winpgbuild/actions/workflows/zstd.yml/badge.svg) |
| **Build Tools** | Package diffutils | ![Package diffutils](https://github.com/dpage/winpgbuild/actions/workflows/diffutils.yml/badge.svg) |
| | Package Meson | ![Package Meson](https://github.com/dpage/winpgbuild/actions/workflows/meson.yml/badge.svg) |
| | Package Ninja | ![Package Ninja](https://github.com/dpage/winpgbuild/actions/workflows/ninja.yml/badge.svg) |
| | Package pkgconf | ![Package pkgconf](https://github.com/dpage/winpgbuild/actions/workflows/pkgconf.yml/badge.svg) |
| | Package winflexbison | ![Package winflexbison](https://github.com/dpage/winpgbuild/actions/workflows/winflexbison.yml/badge.svg) |
| **Bundling** | Bundle dependencies | ![Bundle dependencies](https://github.com/dpage/winpgbuild/actions/workflows/bundle-deps.yml/badge.svg) |
| **PostgreSQL** | Build PostgreSQL | ![Build PostgreSQL](https://github.com/dpage/winpgbuild/actions/workflows/postgresql.yml/badge.svg) |
| | Build PostgreSQL (Dev) | ![Build PostgreSQL (Dev)](https://github.com/dpage/winpgbuild/actions/workflows/postgresql-dev.yml/badge.svg) |

## Automation
We really need to automate the builds, as Github will only retain the artefacts
for 90 days so we need to ensure they're kept fresh. Yes, we could upload them
to S3 or something, but whatever...

Automating the build of all components is, tricky. Github has a limit of 20
workflows that can be called in a single workflow dispatch; and we use 2 per 
component, one to get the version, and one to run the build.

We could put everything into a single workflow (or maybe two or three), but
that would become extremely difficult to navigate and maintain.

We could also dispatch workflow runs using the Github API, however, that doesn't
give us an ID when we run a workflow so we can't poll for it's completion (and
even if we did, we'd likely hit the API call quota quickly).

So for now, we're just running all workflows daily, at times determined by
their "dependency group". We allow an hour for each group, and hope everything
builds in time. If not, we may end up using the previous build of a dependency.

## Version Information

Versions for each artifact are specified in [manifest.json](https://github.com/dpage/winpgbuild/blob/main/manifest.json)

Currently supported versions of PostgreSQL should work. All dependencies 
should automatically be included in the build, *except* for MIT Kerberos (the 
gssapi configuration option). See 
https://www.postgresql.org/message-id/CA%2BOCxoxwsgi8QdzN8A0OPGuGfu_1vEW3ufVBnbwd3gfawVpsXw%40mail.gmail.com
for more information.

## Adding a package.
* in [manifest.json](https://github.com/dpage/winpgbuild/blob/main/manifest.json) add an element to the array for the package. The elements are more or less in alphabetical order (libiconv being the exception) 

Currently we only use `name` and `version` in workflows. `source` and `license` are informational.

```
{
    "name": "diffutils",
    "version": "2.8.7-1",
    "source": "https://downloads.sourceforge.net/project/gnuwin32/diffutils/2.8.7-1/diffutils-2.8.7-1-bin.zip",
    "licence": "https://git.savannah.gnu.org/cgit/diffutils.git/tree/COPYING"
},
```      

 then in the workflow file manifest.yml add an output in the outputs section.

```
DIFFUTILS_VERSION:
    description: "diffutils version"
    value: ${{ jobs.set_versions.outputs.output18 }}
```

A new output will need to be defined for the jobs.

```
    output18: ${{ steps.step1.outputs.DIFFUTILS_VERSION }}
```

after which a github output variable will be created named `uppercase($name)_VERSION=$version` and can be used in subsequent workflows.

## Using this workflow from another workflow

All of the workflows can be called from another workflow, however you will have to provide your own version of manifest.json.

## TODO

The following dependencies are yet to be completed:

* Perl
* Python
* TCL

The following dependencies haven't been supported on Windows but potentially 
could be in the future under Meson.

* Bonjour
* LLVM
* Readline (or libedit, as it's not GPL)

