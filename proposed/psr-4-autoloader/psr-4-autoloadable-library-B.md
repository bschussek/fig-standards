PSR-4: Autoloadable Code Base
=============================

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

> Text written in a block quote (like this) is a comment and *not* part of the
> specification. It will be removed from the final text.

1. Introduction
---------------

The goal for this specification is two-fold:

1. As an extension to PSR-0, and to provide an alternative option for
   applications to determine the location of a file resource on a medium,
   as supported by PHP, when a "Fully Qualified Class Name" is provided.

2. As a specification for software maintainers on how to name and structure
   namespaces in an application so that a uniform representation develops.

   This will aid developers by

   - improving navigation and searchability of PHP source code due to
     recognizability.
   - preventing conflicts and assisting with naming and structuring for
     maintainers.
   - assisting applications to automatically find classes without the need for
     include or require statements, also known as 'autoloading'.

The location of a file resource is often determined by a specialized library,
or component, to which we refer as an autoloader. This name provides no further
significance in the context of this document but is provided to clarify when
and where file resource locations are determined in real world situations.

2. Scope
--------

This specification is partly backwards incompatible with PSR-0. Where a
conflict occurs the rules in this specification precede, or override, the
rules in PSR-0.

3. Definitions
--------------

- A "Code Base" is a directory which contains PHP code. A code base can contain
  nested code bases. In practice, multiple other terms exist for this or similar
  concepts, such as "Application", "Library", "Package" or "Module".

- "Class" refers to a PHP class, interface or trait.

- "Namespace" refers to a PHP namespace, as is accepted after the
  [PHP "namespace" keyword](http://www.php.net/manual/en/language.namespaces.definition.php).
  `\` is not a valid namespace.

- "Namespace Separator" refers to the symbol `\` (backslash).

- A "Fully Qualified Name" refers to the full identifier of a namespace or a
  class, as defined by [PHP's name resolution rules](http://php.net/manual/en/language.namespaces.rules.php).
  For the scope of this specification, fully qualified names always include the
  leading namespace separator.

- The "Unqualified Name" is the part of a fully qualified name that succeeds the
  last namespace separator. Given a fully qualified name of `\Foo\Bar\Baz\Qux`,
  the unqualified name is `Qux`.

- The "Parent Namespace" of a namespace is the part of a fully qualified
  namespace name that precedes the last namespace separator. Given a namespace
  of `\Foo\Bar\Baz\Qux`, the parent namespace is `\Foo\Bar\Baz`. Namespaces with
  only one namespace separator don't have a parent namespace.

- An "Autoloader" is a piece of PHP code that, when given the fully qualified
  name of a class, finds and loads the PHP file that contains the definition
  of that class.

- An "Autoloadable Class" is a class which is intended to be loaded by an
  autoloader ("autoloaded"). Classes which are not intended to be autoloaded
  by their developer are not covered by this term.

- An "Autoloadable Namespace" is a namespace which contains autoloadable
  classes.

- The "Corresponding Directory" of a namespace is the directory that contains
  the PHP files defining the autoloadable classes within that namespace. The
  location of that directory is given as a relative path from the root of the
  containing code base.

- A "Base Namespace" within the scope of a code base is a namespace for which a
  corresponding directory is explicitly defined. The base namespaces of nested
  code bases are not covered by this definition.

4. Specification
----------------

This specification establishes a relationship between the fully qualified class
names of a code base and the PHP files that contain their class definition. It
does so by first associating the autoloadable namespaces of the code base with
their corresponding directories (rules 1-5). Then the specification describes
where a file that contains an autoloadable class should be located and how it
should be named (rules 6-7).

A code base is a PSR-4 compliant "Autoloadable Code Base" if it satisfies the
following rules:

1. The code base MUST document how to find one or more corresponding directories
   for at least one namespace. Such a namespace then becomes a base namespace.
   The specific approach of the documentation is left to the developer. Valid
   examples are narrative text, meta-files, PHP source code, project-specific
   conventions, or some other approach.

2. A base namespace MAY correspond to the root directory of the code base.

3. To prevent conflicts, different base namespaces SHOULD NOT correspond to the
   same directory.

4. Each autoloadable namespace within a base namespace MUST have exactly one
   corresponding directory. That directory MUST be a subdirectory of the parent
   namespace's corresponding directory. The directory name MUST equal the
   namespace's unqualified name.

   > Inductive definition of the namespace->directory mapping.
   >
   > We focus on namespaces with autoloadable classes only. Frameworks can do
   > whatever they want with other namespaces.

5. Rule 4 does not apply to base namespaces within other base namespaces.

   > Allow the following:
   >
   > * \Acme\ -> src/
   > * \Acme\Test\ -> test/

6. Each autoloadable class MUST belong to one of the base namespaces.

7. Each autoloadable class MUST be contained in a file located in the
   corresponding directory of the class' namespace. The file name MUST equal the
   class' unqualified name suffixed with `.php`.

   > Again we focus on autoloadable classes. A code base may contain other
   > classes that don't satisfy this rule.

5. Example Algorithm
--------------------

Autoloaders are free to choose any algorithm of their choice to locate files
for the classes within an autoloadable code base. The following is an example
algorithm for transforming a fully qualified class name into a file location:

1. Remove any preceding namespace separator of the given fully qualified class
   name.

2. Match the beginning of the fully qualified class name against the documented
   base namespaces.

3. When a matching base namespace is found, replace it by one of the namespace's
   corresponding directories.

3. Replace all remaining namespace separators with the directory separators of
   the respective operating system.

4. Suffix the result with the string `.php`

The result is the path to a PHP file relative to the root of the code base. If
the autoloader knows the location of that code base, it can successfully load
the file now.
