# winpgbuild
A repo containing Github actions for building PostgreSQL and it's dependencies 
on Windows. A number of build tools are also included to enable reproducable
builds, though these are generally downloaded as pre-built utilities.

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

### Dependencies

* gettext: 0.22.5
* icu: 76.1
* iconv: 1.17
* krb5: 1.21.3. Note that this isn't used in the PostgreSQL build as it will cause compile errors.
* libxml2: 2.13.5
* libxslt: 1.1.42
* lz4: 1.10.0
* openssl: 3.0.15
* ossp-uuid: 1.6.2
* zlib: 1.3.1
* zstd: 1.5.6

### Tools

* diffutils: 2.8.7-1
* meson: 1.6.0
* ninja: 1.12.1
* pkgconf: 2.3.0
* winflexbison: 2.5.25

### PostgreSQL

The PostgreSQL versions are being built are:

* 17.1
* 16.5
* 15.9
* 14.14
* 13.17
* 12.21

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

