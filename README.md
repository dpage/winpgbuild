# winpgbuild
A repo containing Github actions for building PostgreSQL and it's dependencies 
on Windows.

## Respository variables

Github Action Repository Variables are used to specify the version of 
components to build. These are listed below in square brackets.

### Dependencies

* gettext [GETTEXT_VERSION]
* icu [ICU_VERSION]
* iconv [ICONV_VERSION]
* krb5 [KRB5_VERSION]. Note that this isn't used in the PostgreSQL build as it will cause compile errors.
* libxml2 [LIBXML2_VERSION]
* libxslt [LIBXSLT_VERSION]
* lz4 [LZ4_VERSION]
* meson [MESON_VERSION]
* ninja [NINJA_VERSION]
* openssl [OPENSSL_VERSION]
* ossp-uuid [OSSP_UUID_VERSION].
* zlib [ZLIB_VERSION]
* zstd [ZSTD_VERSION]

### PostgreSQL

The PostgreSQL versions are specified in the POSTGRESQL_VERSIONS repository 
variable, as a JSON style array, e.g.

```[16.3, 15.7, 14.12, 13.15, 12.19]```

Currently supported versions of PostgreSQL <= 16 should work. All dependencies 
should automatically be included in the build, *except* for MIT Kerberos (the 
gssapi configuration option). See 
https://www.postgresql.org/message-id/CA%2BOCxoxwsgi8QdzN8A0OPGuGfu_1vEW3ufVBnbwd3gfawVpsXw%40mail.gmail.com
for more information.

## TODO

The following dependencies are yet to be completed:

* Readline
* Perl
* Python
* TCL

The following dependencies haven't been supported on Windows but potentially 
could be in the future under Meson.

* Bonjour
* LLVM

