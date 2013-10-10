PSR-4: Autoloadable Library
===========================

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).


1. Overview
-----------

This PSR specifies the rules for structuring the class files of a PHP library
such that any compliant autoloader can load the classes of that library.


2. Definitions
--------------

- **class**: A PHP class, interface or trait.

- **autoloadable class**: A class which is intended to be loaded by an
  autoloader.

- **namespace**: A PHP namespace.

- **autoloadable namespace**: A namespace which contains autoloadable classes.

- **root namespace**: The shortest common namespace of a set of autoloadable
  classes of a library.

- **fully qualified name**: The full class or namespace name, including the
  leading namespace separator. (This is per the
  [Name Resolution Rules](http://php.net/manual/en/language.namespaces.rules.php)
  from the PHP manual.)

- **unqualified name**: The part of a fully qualified name that succeeds the
  last namespace separator. Given a fully qualified name of `\Foo\Bar\Baz\Qux`,
  the unqualified name is `Qux`.

- **parent namespace**: The part of a fully qualified namespace name that
  precedes the last namespace separator. Given a namespace of
  `\Foo\Bar\Baz\Qux`, the parent namespace is `\Foo\Bar\Baz`. Namespaces with
  only one namespace separator don't have a parent namespace.


3. Specification
----------------

1. A library MUST have at least one root namespace.

   > Allow multiple root namespaces, for example:
   >
   > * Acme\Package -> src/
   > * Acme\Test\Package -> test/

2. Each autoloadable class MUST belong to one of the root namespaces.

3. Each root namespace MUST have exactly one corresponding directory in the
   library. This directory MAY be the library root itself.

4. A root namespace's corresponding directory MUST NOT be contained in another
   root namespace's corresponding directory.

5. Each autoloadable namespace below a root namespace MUST have exactly one
   corresponding directory in the library. That directory MUST be a subdirectory
   of the parent namespace's corresponding directory. The directory name MUST
   equal the namespace's unqualified name.

   > Inductive definition of the namespace->directory mapping.
   >
   > We focus on namespaces with autoloadable classes only. Frameworks can do
   > whatever they want with other namespaces.

6. Each autoloadable class MUST be contained in a file located in the
   corresponding directory of the class' namespace. The file name MUST equal the
   class' unqualified name suffixed with `.php`.

   > Again we focus on autoloadable classes. A library may contain other classes
   > that don't satisfy this rule.


4. Example Autoloaders
----------------------

For example implementations of a compliant autoloader, please see [the relevant
wiki page][]. Example implementations MUST NOT be regarded as part of the
specification. They are examples only, and MAY change at any time.

[the relevant wiki page]: https://github.com/php-fig/fig-standards/wiki/PSR-4-Example-Implementations

