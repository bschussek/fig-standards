PSR-4: Autoloader
=================

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).


1. Overview
-----------

The following describes the mandatory requirements that must be adhered to
for autoloader interoperability, by mapping namespaces to file system paths.


2. Definitions
--------------

- **Class**: The term "class" refers to PHP classes, interfaces, and traits.

- **Fully Qualified Class Name**: The full namespace and class name. The
  "fully qualified class name" MUST NOT include the leading namespace
  separator.

- **Namespace Name**: Given a "fully qualified class name" of
  `Acme\Log\Formatter\LineFormatter`, the "namespace names" are `Acme`, `Log`, and `Formatter`. A
  namespace name MUST NOT include a leading or trailing namespace separator.

- **Namespace Prefix**: Given a "fully qualified class name" of
  `Acme\Log\Formatter\LineFormatter`, the "namespace prefix" could be `Acme\`, `Acme\Log\`, or
  `Acme\Log\Formatter\`. The "namespace prefix" MUST NOT include a leading namespace
  separator, but MUST include a trailing namespace separator.

- **Relative Class Name**: The parts of the "fully qualified class name" that
  appear after the "namespace prefix". Given a "fully qualified class name" of
  `Acme\Log\Formatter\LineFormatter` and a "namespace prefix" of `Acme\Log\`, the "relative
  class name" is `Formatter\LineFormatter`. The "relative class name" MUST NOT include a
  leading namespace separator.

- **Base Directory**: The directory path in the file system where the files
  for "relative class names" have their root. Given a namespace prefix of
  `Acme\Log\`, the "base directory" could be `./src/`.
  The "base directory" MUST include a trailing directory separator.

  > This rule couples base directories to directories in the file system and
  > disallows loading from remote file systems and .phars.

- **Mapped File Name**: The path in the file system resulting from the
  transformation of a "fully qualified class name". Given a "fully qualified
  class name" of `Acme\Log\Formatter\LineFormatter`, a namespace prefix of `Acme\Log\`, and a
  "base directory" of `./src/`, the "mapped file name"
  MUST be `./src/Formatter/LineFormatter.php`.

  > Again, "path in the file system".


3. Specification
----------------

### 3.1. General

1. A fully-qualified namespace and class must have the following
  structure `\<Vendor Name>\(<Namespace>\)*<Class Name>`

  > Which syntax is used here? BCNF? This rule is superfluous (see next comment).

2. Each namespace must have a top-level namespace ("Vendor Name").

  > A PHP namespace (anything after the "namespace" keyword) automatically has
  > at least one "namespace name". This rule is superfluous.
  
  >> If you are suggesting combining the first two, I think that would be a nice improvement.
  >> But, in reality, the first doesn't come right out and say -- you must have a unique
  >> vendor-name for all classes. So, combining would be fine, but the point is important
  >> enough it should be stated.

3. Each namespace can have as many sub-namespaces as it wishes.

  > This rule is superfluous. Of course it can.
  
  >> Agreed. (But, on the other hand! ;-) It doesn't hurt to say there is no limit.)

4. Each namespace separator is converted to a `DIRECTORY_SEPARATOR` when
  loading from the file system.

  > Who does that? Must that happen or is it an option? What if I'm loading
  > from something else than the file system?
  
  >> Can you provide an example of this? IMO, this makes sense -- the interface
  >> could certainly be a filesytem package [like](https://github.com/KnpLabs/Gaufrette)
  >> or PHP filesystem commands.  An example of a limitation would help, though.

5. The "fully qualified class name" MUST begin with a "namespace name", which
MAY be followed by one or more additional namespace names, and MUST end in
a class name.

  > Repetition of rule 1, which - as I have noted - is superfluous.
  
  >> Agree -- funny thing is -- you are driving to PSR-X -- Paul had the most
  >> beautiful terse language but others added lots and lots of words. =)
  >> This one _can_ go.

6. A "namespace prefix" MUST correspond to at least one "base directory".

  > Each namespace prefix? So for a namespace `Acme\Log\Formatter\LineFormatter`,
  > each of
  >
  > * `Acme\`
  > * `Acme\Log\`
  > * `Acme\Log\Formatter\`
  >
  > MUST correspond to one or more base directories? A pretty heavy restriction.
  
  >> No, but, LOL on that. Here's what I read that as -- any namespace prefix
  >> identified above as those values mapped to base directories -- MUST
  >> correspond to at least one base directory. It's an awkward wording, for sure.
  >> So, what is it trying to say?  Basically, I think the intention is to say 
  >> Namespace prefixes map to base directory(ies).

7. A "namespace prefix" MAY correspond to more than one "base directory". The
order in which an autoloader will attempt to map the file is not in the scope
of this specification, but the consumer should be aware that different
approaches may be used and should refer to the autoloader documentation.

>> I would change this to combine to 6 and say "Namespace prefix map to one or more base directories."
>> Not so sure I would add the condition about scope -- not speaking to it should suffice. But
>> it's not a problem there, either.

### 3.2. Registered Autoloaders

1. A relationship MUST be present between a "namespace prefix" and the "base
   directory". This relationship allows a registered autoloader to know where to
   identify the location of the class. To establish this relationship, the
   registered autoloader MUST transform the "fully qualified class name" into a
   "mapped file name" as follows:

   > This rule is talking about a relationship between namespaces prefixes and
   > base directories...

   1.1. The "namespace prefix" portion of the "fully qualified class name"
        MUST be replaced with the corresponding "base directory".

        > ... and this rule about establishing this relationship. But namespace
        > prefixes are not FQCNs.
        >
        > Further, why must I replace them? Can't I choose another algorithm than
        > string replacement?
        
        >> This is an important aspect to PSR-4. It's simply saying the namespace prefix
        >> and the base directory are interchangeable. When referencing the filesystem, 
        >> the base directory is used. When referencing the FQCN, the namespace prefix is used.
        >> I have no problem with this rule.

   1.2. Namespace separators in the "relative class name" portion of the
        "fully qualified class name" MUST be replaced with directory separators
        for the respective operating system.

        > Again, why can't a choose a different algorithm?
        
        >> The alogorithm is PSR-4. If you used a different approach, then the consistency
        >> between different software would be gone. And that means no one would know how 
        >> to resolve your namespaces. Nope, this is the basics of PSR-4. Gotta comply here. 

   1.3. The result MUST be suffixed with `.php`.

2. If the "mapped file name" exists in the file system, the registered
   autoloader MUST include or require it.

   > Again coupling to the file system.
   
   >> You *have* to map the file system and the namespace or the autoloader has no
   >> way of resolving which file to choose based on the namespace. 
   >> The coupling you want to *avoid* is rules requiring all files and all folders
   >> have a certain structure. THAT is bad coupling. 
   
3. The registered autoloader MUST NOT throw exceptions, MUST NOT raise errors
   of any level, and SHOULD NOT return a value.

   > Implementation details. I (library developer) don't care about this, that's
   > end-user business (i.e. if I choose a specific autoloader for an application,
   > I may choose one which throws exception or which doesn't - my responsibility).
   
   >> Disagree -- and this one got so much talk it could have had it's own podcast.
   >> Discourage anyone from touching this. Or talking about it. Or breathing in it's 
   >> general direction. Or looking at it like bratty teenagers do at their parents. 
   >> Bury it - and bury the shovel it was buried with. 


4. Implementations
------------------

For example implementations, please see the [examples file][]. Example
implementations MUST NOT be regarded as part of the specification. They are
examples only, and MAY change at any time.

[examples]: psr-4-autoloader-examples.php
