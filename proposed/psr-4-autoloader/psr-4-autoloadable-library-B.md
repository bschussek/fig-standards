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

- **Library**: A directory which contains PHP code.

- **Class**: A PHP class, interface or trait.

- **Autoloadable Class**: A class which is intended to be loaded by an
  autoloader.

- **Namespace**: A PHP namespace.

  > Note: `\` is *not* a namespace. It is the global space.
  >
  > A valid PHP namespace is anything that may follow the "namespace" keyword.

- **Autoloadable Namespace**: A namespace which contains autoloadable classes.

- **Fully Qualified Name**: The full class or namespace name, including the
  leading namespace separator. (This is per the
  [Name Resolution Rules](http://php.net/manual/en/language.namespaces.rules.php)
  from the PHP manual.)

- **Unqualified Name**: The part of a fully qualified name that succeeds the
  last namespace separator. Given a fully qualified name of `\Foo\Bar\Baz\Qux`,
  the unqualified name is `Qux`.

- **Parent Namespace**: The part of a fully qualified namespace name that
  precedes the last namespace separator. Given a namespace of
  `\Foo\Bar\Baz\Qux`, the parent namespace is `\Foo\Bar\Baz`. Namespaces with
  only one namespace separator don't have a parent namespace.


3. Specification
----------------

A library is a PSR-4 compliant "Autoloadable Library" if it satisfies the
following rules:

1. The library MUST document how to find the "corresponding directory" for at
   least one namespace. Such a namespace is called a "base namespace". The
   corresponding directory MAY be the library root itself.

   > *How* this correspondence is documented is up to the developer. Examples:
   >
   > * end user documentation
   > * composer.json
   > * PHP source code
   > * conventions (Drupal modules)

2. Each autoloadable namespace within a base namespace MUST have exactly one
   corresponding directory. That directory MUST be a subdirectory of the parent
   namespace's corresponding directory. The directory name MUST equal the
   namespace's unqualified name.

   > Inductive definition of the namespace->directory mapping.
   >
   > We focus on namespaces with autoloadable classes only. Frameworks can do
   > whatever they want with other namespaces.

3. Rule 2 does not apply to base namespaces within other base namespaces.

   > Allow the following:
   >
   > * Acme\ -> src/
   > * Acme\Test\ -> test/

4. Each autoloadable class MUST belong to one of the base namespaces.

5. Each autoloadable class MUST be contained in a file located in the
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

