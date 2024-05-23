# winpgbuild
A repo containing Github actions for building PostgreSQL and it's dependencies on Windows.

## Respository variables

Github Action Repository Variables are used to specify the version of components to build.
These are listed below in square brackets.

### Dependencies

* libxml2 [LIBXML2_VERSION]
* libxslt [LIBXSLT_VERSION]
* MIT Kerberos (krb5) [KRB5_VERSION]. Note that this isn't used in the PostgreSQL build as it will cause compile errors.
* OpenSSL [OPENSSL_VERSION]
* zlib [ZLIB_VERSION]

### PostgreSQL

The PostgreSQL version is specified in the POSTGRESQL_VERSION repository variable.
Currently supported versions of PostgreSQL <= 16 should work. All dependencies 
should automatically be included in the build, *except* for MIT Kerberos (the gssapi
configuration option).
