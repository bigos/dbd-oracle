# dbd-oracle: Oracle database driver for CL-DBI

[![Build Status](https://travis-ci.org/sergadin/dbd-oracle.svg?branch=master)](https://travis-ci.org/sergadin/dbd-oracle)
[![Coverage Status](https://coveralls.io/repos/github/sergadin/dbd-oracle/badge.svg?branch=master)](https://coveralls.io/github/sergadin/dbd-oracle?branch=master)
[![Quicklisp](http://quickdocs.org/badge/dbd-oracle.svg)](http://quickdocs.org/dbd-oracle/)

This driver is based on OCI bindings developed for CLSQL.

## Usage

### Connecting to ORACLE database

Connection can be estabslished by providing appropriate database name
to `dbi:connect` method. The value of `database-name` key argument is an
ORACLE connect string that might include hostname, listener port
number, and oracle SID or service name. For example,

```common-lisp
(defvar *connection*
  (dbi:connect :oracle
               :database-name "127.0.0.1:1521/orcl"
               :username "nobody"
               :password "1234"
               :encoding :utf-8))
```

Encoding parameter is used for conversion of character strings between
OCI library and Lisp. :utf-8 is the default encoding. Possible
values for encoding parameter is the same as for CFFI string
conversion functions.


### Loading OCI library

The application makes an attempt to load Oracle OCI library during the
first call to `dbi:connect`. By default, the OCI library is searched in
ORACLE_HOME, ORACLE_HOME/lib, and ORACLE_HOME/bin directories, as well
as in the current directory. If ORACLE_HOME environment variable is
not set, then search paths can be provided by setting
`dbd.oracle:*foreign-library-search-paths*` variable. The value
assigned to this variable should be a list of pathnames. For example,

```common-lisp
  (let* ((dbd.oracle:*foreign-library-search-paths* '(#p"/opt/oracle/"))
         (connection (dbi:connect :oracle
                                  :database-name "localhost/orcl"
                                  :username "nobody"
                                  :password "1234")))
     (dbi:disconect connection))
```

You may need to create a symbolic link for `libclntsh.so` pointing to
an appropriate library, e.g. `libclntsh.so.12.1`.

### Running queries

Queries are evaluated using CL-DBI interface. Parameters bindings is
not guaranteed to work properly due to incompatible syntax for
placeholders definitions used in CL-DBI and Oracle queries. Oracle
requires parameter name to start with a colon, while CL-DBI uses the
question mark `?` to define placeholders. Simple substitution of *all*
occurrences of " ?" (a space followed by a question mark) by
Oracle-friendly column expressions is performed for the entire SQL
expression.

```common-lisp
(defvar *connection*
  (dbi:connect :mysql
               :database-name "test"
               :username "nobody"
               :password "1234"))

(let* ((query (dbi:prepare *connection*
                           "SELECT * FROM somewhere WHERE flag = ? OR updated_at > ?"))
       (result (dbi:execute query 0 "2011-11-01")))
  (loop for row = (dbi:fetch result)
     while row
     ;; process "row".
       ))
```

The effectve query would be
`"SELECT * FROM somewhere WHERE flag = :1 OR updated_at > :2"`.
If a question mark appears inside a quoted string of the SQL query,
then the effective query could not be identical to the original
one. The rule of thumb is to avoid using French punctuation style in
literal constants inside the main body of SQL queries (bound strings
are processed as is), and to put space before every placeholder. Automatic
substitution of question marks may be disabled by setting
`format-placeholders` key parameter of `dbi:connect` to NIL.

## Testing

This code was tested using Oracle 11g server under following client configurations:

* 64-bit Clozure CL 1.9 under Windows 7 using 64-bit version of Oracle instant client 12.1.0.1
* 64-bit SBCL 1.3.2 under Ubuntu 14.04 64-bit using Oracle instant client 12.1.0.1
* 32-bit LispWorks Personal Edition on Mac OS X 10.10 using 32-bit version of Oracle instant client 11.2.0.4

## License

Lisp Lesser GNU Public License
