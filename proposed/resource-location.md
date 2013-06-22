> This proposal is annotated with comments in quote blocks like this one. These
> annotations should help to clarify why certain passages exist and what other
> alternatives exist or have been tried.
>
> The final PSR will not contain these annotations, but a copy could be preserved
> for documentation purposes.
>
> **TL;DR**
>
> This specification proposes to refer to files and directories through URIs, e.g.:
>
> * classpath:///Acme/Demo/Parser/resources/config.yml
> * classpath:///Acme/Demo/Parser/resources
> * classpath:///Acme/Demo/Parser.php
> * file:///var/www/project/favicon.ico
> * view:///acme/demo-package/show.php
>
> These URIs can have different schemes ("classpath", "file" etc.), but only the
> schemes "file" and "classpath" are explicitly specified in this document.
>
> The resource locator is able to turn URIs into file paths which can be read
> or included by PHP code, e.g.:
>
> ```php
> // autoloading
> include $locator->findResource('classpath:///Acme/Demo/Parser.php');
>
> // loading of configuration files
> $config = Yaml::parse($locator->findResource('classpath:///Acme/Demo/config.yml'));
>
> // loading of templates
> // internally prepends the "view://" scheme
> // loads "show.php" from the Composer package "acme/demo-package"
> render('/acme/demo-package/show.php');
> ```
>
> How to configure such a resource locator depends on the implementation.
> A very simple implementation would feature a method `addPath()` which maps a
> URI scheme and a URI path prefix to one or more directories:
>
> ```php
> // paths in the "classpath" scheme are prefixed by PHP namespaces with
> // forward slashes
> $locator->addPath('classpath', '/Acme/Demo/', '/path/to/src');
>
> // paths in other schemes (except for "file") are prefixed by the
> // Composer package name (by convention, not enforced)
> $locator->addPath('view', '/acme/demo-package/', '/path/to/resources/views');
> ```
>
> **Main Goals:**
>
> The general goal of this PSR is to locate files (PHP, XML, YAML, INI, JPG, etc.)
> and directories in a generic way. For example, there should be a unified
> notation to refer to *the file* of a class `\A\B\C\D` and other files located in
> the same directory (or nested directories).
>
> The secondary goal is to locate files in redistributable (e.g. Composer)
> packages in a generic fashion.
>
> The tertiary goal is to provide a foundation for the new autoloader PSR.
>
> **Requirements:**
>
> 1. **Locate files relative to classes**
>
>    If a file is in the same directory (or a subdirectory of) as a class
>    `\A\B\C\D`, allow to locate its path by providing (a) the namespace
>    `\A\B\C\` and (b) the relative location of the file, e.g.
>    `config/settings.yml`.
>
>    ```php
>    $locator->findResource('classpath:///Acme/Demo/config/settings.yml');
>    ```
>   Since this is now going to be based on URI namespaces, it may or may not
>   be true that the implementation elected to map folders in this manner. 
>   Unless we are stating all classes and class subfolders and files must be 
>   mapped a certain way, I think this is a "it depends on what they do" 
>   situation. Does that make sense?
>
> 2. **Locate both directories and files**
>
>    ```php
>    $locator->findResource('classpath:///Acme/Demo/config/settings.yml');
>    $locator->findResource('classpath:///Acme/Demo/config');
>    ```
>   I realize this is an example prior to taking the URI namespace route, but  
>   just to confirm we understand it the same, the path in the namespace will 
>   not specify an extension. The scheme will be defined to allow one or more
>   extension values, but the namespace itself has file extension. 
> 
>   Of course, that has implications in that multiple files could share the 
>   same FQNS: example.txt and example.php -- the scheme file extensions are
>   tested with the node to find the appropriate match for that type. The
>   possibility would still exist, of course, that multiple files, each with
>   different extensions, defined to the same scheme, could result in the 
>   wrong match. That's something I'd ignore in the standard and leave to the
>   implementation to edit against. But, I want to raise the issue for your 
>   consideration.
>
> 3. **Short identifiers when the context is known**
>
>    If similar files are always located in the same location, provide
>    a short, redundancy-free notation. For example, if a namespace
>    is `\A\B\C\` and template files are always located in a subdirectory
>    `resources/views/`, make it possible to skip this redundant information.
>
>    ```twig
>    {% include 'classpath:///Acme/Demo/resources/views/show.html.twig' %}
>    {% include 'view:///acme/demo-package/show.html.twig' %}
>    {% include '/acme/demo-package/show.html.twig' %}
>    ```
>   This is another example of simply defining the namespaces to do so. 
>   I'll give matching examples of how I would use tags to do this, but it 
>   is something the developer would have to build into the implementation
>   when building (manually or in an automated fashion) addNamespace statements.
>
> 4. **Locate resources independent from PHP classes**
>
>    Even though this is not the main goal, support location of resources
>    independent from PHP classes, interfaces and traits.
>
>    ```php
>    $locator->addPath('view', '/app/', '/path/to/app/views');
>    $locator->findResource('view:///app/layout');
>    ```
>
> 5. **Support resource overriding**
>
>    Look for a resource in multiple directories to support overriding.
>
>    ```php
>    $locator->addPath('classpath', '/Acme/Demo/', array(
>        '/path/to/overridden/src',
>        '/path/to/acme/demo/src',
>    ));
>
>    include $locator->findResource('classpath:///Acme/Demo/Parser');
>
>   I do not advise adding the scheme to the addPath signature. The scheme 
>   will define file extensions matching, so that will match directly to your Parser.php
>   file. Then, selectivity can be acheived by your how those addNamespace paths
>   are defined. We can also use tagging to create custom paths based on file and folder
>   search criteria if more selectivity is needed. 
>   
>   The problem is in adding in the scheme, it now excludes other schemes from reusing the
>   the namespace. Those location-based relationships will benefit from keeping it open.
>
>      Ex. '/Acme/Demo/' is a namespace for the class loader to load Parser.php
>        The developer can affix 'config/something' to that Class NS to find it's config file.
>          $locator->findResource('config:///Acme/Demo/Config/Something');
>
> **Performance**:
>
> Resource location performance can be optimized by mirroring the URI
> paths in a cache directory. Then location is reduced to one simple
> `file_exists()` check in that cache directory. For example, the
> URIs
>
> * view:///acme/demo/show.php
> * classpath:///Acme/Demo/Parser.php
>
> would be cached through symbolic links in
>  and, in fact, it can be cached using a namespace (going to play with that, cool example)
>
> ```
> /cache
>     /view
>         /acme
>             /demo
>                 show.php -> /path/to/show.php
>     /classpath
>         /Acme
>             /Demo
>                 Parser.php -> /path/to/Parser.php
> ```
>
> **Common Rules of Resource Location and Autoloading**
>
> Since both the autolading PSR (PSR-X) and this PSR (PSR-R) define how
> to map PHP namespaces to directories, and how to locate files in these
> directories, they need common logical rules for specifying this mapping.
>
> We have the following possibilities:
>
> 1. Include common rules in PSR-R, refer to PSR-R from PSR-X
> 2. Include common rules in PSR-X, refer to PSR-X from PSR-R
> 3. Include common rules in both
> 4. Move common rules to a separate PSR, refer to that PSR from both
>    PSR-X and PSR-R
>
> I want to briefly outline the implications of these solutions:
>
> **1. Include common rules in PSR-R, refer to PSR-R from PSR-X**
>
> In this case, the logical formulation of the PSRs will essentially be:
>
> PSR-R: Given the URI "classpath:///A/B/C/D" and a prefix `/A/B` mapped
> to some path `/src`, then `/src/C/D` must be an existing *directory or
> file*. If `/src/C/D` is a file with PHP class definitions, one of them must
> have the FQCN `\A\B\C\D`.
>
> PSR-X: The autoloader must turn the loaded class into a PSR-R compatible
> classpath URI (trivial), use the PSR-R locator to find its path and include
> that path.
>
> Advantages:
>
> * (comparatively) simple logical constructs
> * simple implementation
>
> Disadvantages:
>
> * PSR-X will be delayed after PSR-R
> * PSR-X cannot be implemented without either
>   - using a `ResourceLocatorInterface` instance, or
>   - understanding and (partially) implementing PSR-R
>
> Implementing an autoloader is then as simple as:
>
> ```php
> spl_autoload_register(function ($class) use ($locator) {
>     try {
>         include $locator->findResource('classpath:///'.strtr($class, '\\', '/').'.php');
>     } catch (\Exception $e) {
>     }
> });
> ```
>
> **2. Include common rules in PSR-X, refer to PSR-X from PSR-R**
>
> In this case, the logical formulation of the PSRs will essentially be:
>
> PSR-X: Given a FQCN `\A\B\C\D` and a prefix `\A\B` mapped to some path
> `/src`, then `/src/C/D/` must be a file containing PHP class definitions.
> One of these definitions must have the FQCN `\A\B\C\D`.
>
> PSR-R: Given the URI "classpath:///A/B/C/D", then one of the prefixes, when
> turned into a namespace, must be mapped by PSR-X to some path `/src`. Then
> `/src/C/D` must be an existing *directory or file*. If `/src/C/D` is a file
> with PHP class definitions, then autoloading `\A\B\C\D` per PSR-X must result
> in loading `/src/C/D`.
>
> Advantages:
>
> * PSR-X can be released now
> * PSR-X can be implemented independently from whether PSR-R is successful or not
>
> Disadvantages:
>
> * PSR-R will be more complicated to formulate (see the example above)
> * if it turns out that today's formulation of PSR-X is not adequate/sufficient
>   for PSR-R, we will either have to
>   - release a suboptimal PSR-R spec or
>   - release a PSR-R spec that is incompatible with PSR-X
>
> **3. Include common rules in both**
>
> Advantages:
>
> * PSR-X can be released now
> * PSR-X can be implemented independently from whether PSR-R is successful or not
>
> Disadvantages:
>
> * PSR-X and PSR-R implementations are not necessarily compatible
> * if it turns out that today's formulation of PSR-X is not adequate/sufficient
>   for PSR-R, we will either have to
>   - release a suboptimal PSR-R spec or
>   - release a PSR-R spec that is incompatible with PSR-X
>
> **4. Move common rules to a separate PSR, refer to that PSR from both
> PSR-X and PSR-R**
>
> Advantages:
>
> * PSR-X can be implemented independently from whether PSR-R is successful or not
>
> Disadvantages:
>
> * PSR-X is delayed after that separate PSR
> * three different PSRs for resource location and autoloading might confuse people

Resource Location
=================

This document describes a common interface for resource location in PHP.

The main goal is to allow libraries to receive a
`Psr\ResourceLocation\ResourceLocatorInterface` object and locate file resources
in a simple and universal way. Frameworks and CMSs that have custom needs MAY
extend the interface for their own purpose, but SHOULD remain compatible with
this document. This ensures that the third-party libraries an application uses
can locate resources as specified in this document.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

1. Specification
----------------

### 1.1 Definitions

>> same comment for definitions as other document, where there is a definition in a 
>> a previous PSR, would be good to adopt it (would be nice if FIG had a list of terms...)

**Implementation**: An implementation of `Psr\ResourceLocation\ResourceLocatorInterface`.

**Consumer**: Code using a `Psr\ResourceLocation\ResourceLocatorInterface` implementation.

**Resource**: A common file or directory.

**Path Segment**: A path segment as defined by
[RFC 3986](http://tools.ietf.org/html/rfc3986), without leading or trailing
slash ("/").

**Path**: A sequence of zero or more path segments, separated by slashes
and starting with a slash. `/`, `/A`, `/A/` and `/A/B` are valid paths.

**Path Prefix**: Zero or more contiguous path segments that appear at the start
of a path, including the leading but excluding the trailing slash. Given
a path `/A/B/C/D`, the possible path prefixes are `/`, `/A`, `/A/B`, `/A/B/C`
and `/A/B/C/D`.

> If /A/B/C/D is mapped to /path/to/D, then
>
> * classpath:///A/B/C/D
>
> should resolve to /path/to/D. To keep the definition of the classpath scheme
> simple - which requires that at least one path prefix must be mapped - the
> full path is also considered a possible prefix.

**Relative Path**: Given a path `/A/B/C/D` and a prefix `/A/B`, the relative
path is `C/D`. Relative paths never start with a slash ("/").

**Class**: A PHP class, trait or interface.

**Fully Qualified Class Name (FQCN)**: A class identifier given as fully
qualified name as defined by the
[PHP Name Resolution Rules](http://php.net/manual/en/language.namespaces.rules.php).

>> We might to use use FQNS instead of FQCN

**Namespace**: Given a FQCN `\A\B\C\D`, the namespace of that class is `\A\B\C`.

### 1.2 Resource URIs

>> Would you agree to calling these Namespace URIs? 

Resources are identified by URIs that MUST conform to
[RFC 3986](http://tools.ietf.org/html/rfc3986), with the following restrictions.

> An alternative to URIs was proposed by @simensen. He proposed to locate
> resources through typical namespaced PHP identifiers, e.g.
>
> * \Acme\Demo\resources\views\show.php
>
> The disadvantage is that this method does not allow the same extensibility
> that URI schemes ("classpath://", "view://", "config://" etc.) do. In
> particular, the requirements 3 and 4 from above cannot be fulfilled.
>
> The advantage of URIs is that we can use PHP's native functions such as
> `dirname()` or `basename()` to work with them.
> 
> After more discussion with @AmyStephen, a way to use namespaces as the path within the URI
> was determined and peace ruled the land. ;-)
>

Resource URIs MUST contain at least a non-empty scheme, followed by a colon
(":"), a double slash ("//") and a non-empty path. Additional URI parts MAY be
interpreted by implementors, but their effect is undefined by this specification.

> i.e. scheme:///some/path
>  This is so awesome, BTW.
>
> Other URI parts are the authority (host:port) or the query string. We don't
> need them for the base functionality, but their presence doesn't hurt either
>
> This is one question I had for you, what would we use for host:port? The 
> value from the request? Seems like this is positioning for REST - and turning
> internal processing into lots of routes. Kinda cool.
>
> We have the alternatives of including or excluding the double slash after the
> scheme:
>
> * scheme:///some/path vs.
> My assumption is this one?
> * scheme:/some/path
>
> Both are valid URIs. The disadvantage of the latter is that PHP Stream
> Wrappers are not supported for it.
>
> "The URL syntax used to describe a wrapper only supports the scheme://...
> syntax. The scheme:/ and scheme: syntaxes are not supported."
> – http://www.php.net/manual/en/wrappers.php

The path of a resource URI SHOULD start with a slash ("/").

> Reason: The distinction between absolute and relative paths must be possible
> if the scheme is omitted, e.g. "Demo/Parser.php" vs. "/Demo/Parser.php".
>
> SHOULD and not MUST in order to support absolute Windows paths properly:
> file://C:/some/path
>
> However, using namespaces in path now we will not see those types of URIs

Valid path segments consist of alphanumeric characters (`A-Z`, `a-z`, `0-9`),
underscores (`_`), hyphens (`-`), colons (`:`) and dots (`.`). Implementors MAY
choose to support additional characters, but interoperability is not guaranteed
for such URIs.

> This prevents problems with invalid characters for file names, e.g. "*" or
> "<" on NTFS.
>
> Should percent encoding (%20 etc.) be allowed, as defined in the RFC?
> For clear security, we might want to describe the escaping and filtering requirements.

Paths MUST NOT contain dot segments (`.` and `..`).

> For security reasons.

The path structure MAY be further restricted by specifications of specific
schemes (for example the "file" scheme in section 1.5).

> E.g. the "classpath" scheme requires path prefixes to correspond to PHP
> namespaces with backslashes replaced by forward slashes.
>
> E.g. "\Acme\Demo\" -> "classpath:///Acme/Demo/config.yml
>
> Except we won't be doing these now that we are using namespaces

Examples of valid URIs:

- classpath:///Acme/Demo/Parser
- view:///acme/demo-package/template
- config:///acme/demo-package
- file:///
- file:///Project/settings

> Note how I changed above examples to confirm if we are both seeing this the 
> same now that namespaces are the base of the URI.

### 1.3 Resource Variants

The main task of the resource locator is to resolve resource URIs to existing
file paths. These are called *resource variants*. Each resource URI MAY resolve
to multiple variants. The concrete definition of how to obtain a variant for
a given URI is provided by scheme specifications (section 1.5 and 1.6

> Add definition above

> For overriding resources. See requirement 5.

The `ResourceLocatorInterface` exposes the method `findResourceVariants()` which
receives a resource URI as first argument and MUST return all variants for that
URI in descending order of priority. How to assign priorities MUST be chosen by
the implementation (e.g. FIFO). The method MUST throw a
`Psr\ResourceLocation\IllegalUriException` if the URI does not correspond to the
rules described in section 1.2.

> Again this is a difference due to namespaces. Recommend the method `findResourceVariants()` 
> is removed from the interface. One of the values of namespaces is the ability to map
> a namespace to the same resource many times. Each namespace could be used for different purposes
> I need to understand the use case -- what need does this provide for -- then we can 
> figure out a way to do so with namespaces. Do you have an example?

> If a scheme specification defines what a "variant" is for a given URI, by
> implication every such variant must be returned by findResourceVariants().

`findResourceVariants()` MUST return an array which MUST contain only strings,
i.e. the resource variants. If no variants exist for a resource URI, the array
MUST be empty

> same as above, use case will help clarify.

> No other return values allowed.
>
> If a scheme specification defines what a "variant" is but no such variant
> can be found for a given URI, by implication an empty array must be returned.

> Some namespaces will be to folders, so the portion just removed would limit results.
> Also, it is best to not include physical mapping in namespaces - if someone wants to use those values as scheme
> values, that's fine, but it's not a good practice, IMO, and certainly should not be required
> in the standard. It's just a big array of real paths to fake paths - no reason it
> must describe the protocol in there.

> Files must exist, otherwise the validity of resource variants cannot be
> determined. 
>
> Sure, but let's not add that to the standard. If a namespace is added that
> does not exist, that's a bug in the namespace loader. Standards get to pretend
> they live in a perfect world (unless defining edits or error handling, etc.)
> Perhaps though you are saying that "if the URI namespace of a file is requested
> then the process must return the full path to the file." (Rather than a path,
> or partial path to that value?)
>
> For example:
>
> ```php
> $locator->addPath('classpath', '/Acme/Demo/', array(
>     '/path/to/overridden/src',
>     '/path/to/acme/src',
> ));
>
> // returns /path/to/overridden/src/Parser.php if it exists,
> // /path/to/acme/src/Parser.php otherwise
> $locator->findResource('classpath:///Acme/Demo/Parser');
> ```
>
> As for the URI schemes, these are the only ones that are not restricted by
> allow_url_(fopen|include). Another one would be "glob://", which is not
> guaranteed to deliver a result. Certain variants of "php://" are also not
> restricted, but I'm not sure whether they should be allowed (e.g.
> "php://stdout").

Different resource URIs MAY be resolved to the same resource variants. They
MAY even be resolved to overlapping sets of variants, although this is NOT
RECOMMENDED. Two sets of variants are *overlapping* if they contain both
common and distinct variants. For example, the sets {V1, V2} and {V2, V3} are
overlapping.

> For clarification only. Not sure where this might be useful, but I'm also
> not sure that it never will be.
> I'm not loving it either - but I understand why you attempted it. Not sure.

`findResourceVariants()` MUST return the same variants in the same order
when called multiple times during the execution of a PHP application.

> Idempotence
> Want this to go away if possible, I think it's just a remant of non namespace approach.

### 1.4 Resource Location

The `ResourceLocatorInterface` exposes the method `findResource()` for
resolving a resource URI to a resource variant. It receives a resource URI as
first argument and MUST throw a `Psr\ResourceLocation\IllegalUriException`
if the URI does not correspond to the rules described in section 1.2.

`findResource()` MUST return a string, which MUST be equivalent to the first
entry of the array returned by `findResourceVariants()`. If no existing path
can be found, a `Psr\ResourceLocation\NoSuchResourceException` MUST be thrown.

> Whoa. Do not agree. We need to think this through very carefully. 
> The findResource response will vary. A class loader result will be the 
> included file. I am using handlers. There is a unique handler for each 
> unique treatment required to return the results in the manner expected by
> the application. In my approach, the file scheme defines what handler must
> be used as the response to the find. We need to fill this out a bit. 
> What do you think about including handlers in the spec? Not sure.


> The first variant is the highest priority one, as defined in section 1.3.

### 1.5 File Scheme

> For generic use cases (hacks) that cannot be achieved with other schemes.

Implementations of this PSR MUST support the scheme "file". If the URI path
corresponds to an existing path on the local file system, it MUST be considered
a resource variant for the given URI.

> Once it is defined what a resource variant is for the file scheme, the
> rules in section 1.3 and 1.4 guarantee that the locator behaves correctly.
> Makes sense.

### 1.6 Classpath Scheme

>> See I think all of this can come out, too, without namespaces this might 
>> be needed - OR - I just don't understand the purpose of this. Probably
>> need to talk?

Implementations of this PSR MUST support the scheme "classpath". The URI path
MUST then begin with a slash ("/"), followed by a top-level path segment (the
*vendor namespace*), which MUST be followed by zero or more sub-path segments.
Implementations MAY choose not to validate this rule.

> Valid examples:
>
> * classpath:///Acme
> * classpath:///Acme/Demo
> * classpath:///Acme/Demo/Parser.php
> * classpath:///Acme/Demo/config
> * classpath:///Acme/Demo/config/settings.ini

At least one path prefix of the URI path MUST be mapped to a directory
(the *base directory*) provided as absolute path or URI with one of the
schemes defined in section 1.3. That directory MUST exist on the local file
system, although implementations MAY choose not to validate this rule. A path
prefix MAY be mapped to more than one directory. How the mapping is specified
MUST be chosen by the implementation.

> Valid examples:
>
> * / => /path/to/src/
> * /Acme => /path/to/acme/
> * /Acme => phar://acme.phar/src
> * /Acme => [/path/to/acme/, /path/to/overridden/]
> * /Acme/Demo/Parser => /path/to/src/
> * /Acme/Demo/Parser.php => /path/to/src/ (valid, but not sensible in practice)
>
> Invalid examples (mapping of files is not supported):
>
> * /Acme/Demo/Parser.php => /path/to/src/Parser.php

Given a URI path `/A/B/C` that consists of the path prefixes {`/`, `/A`, `/A/B`,
`/A/B/C`} and the corresponding relative paths {`A/B/C`, `B/C`, `C` and `<empty>`},
the resulting string MUST be the concatentation of a base directory mapped to one
of the prefixes and the corresponding relative path, separated by a slash. If that
string is an existing path in the file system, it MUST be considered a resource
variant for the given URI.

> For example, if `/A/B` is mapped to `/src`, and the path `/src/C` exists, then
> `/src/C` is a valid resource variant for the URI `classpath:///A/B/C`.
>
> Once it is defined what a resource variant is for the classpath scheme, the
> rules in section 1.3 and 1.4 guarantee that the locator behaves correctly.

Variants for longer path prefixes MUST have a higher priority than variants for
shorter path prefixes.

> If both `/A` and `/A/B` are mapped, files found in a base directory mapped to
> `/A/B` should be preferred.

If a resource variant contains PHP class definitions that would be loaded when
[including](http://php.net/manual/en/function.include.php) the file, exactly
one of these classes MUST have a FQCN equivalent to the URI path with all
slashes ("/") replaced by backslashes ("\") and the file extension(s) removed.
This is the *primary class*. All other classes in the file MUST belong to the
same namespace as the primary class. The implementation MAY choose not to
validate this rule.

>> There is absolutely no reason to mention primary classes or anything of that
>> nature here. That's going to require application-specific knowledge. 
>> Each developer when building it into the framework will have to ensure this is
>> true if they want that selectivity around those classes. Maybe all they want
>> is to build a gallery? and this is not necessary. 
>> How those namespace pairs are designed and even built into a map is outside
>> of the scope, IMO.

> Make sure that when mapping
>
> * /Acme/Demo/Parser => /path/to/src/Parser.php
>
> then Parser.php must contain the class \Acme\Demo\Parser.
>
> IMO, that's a implementation concern
>
> If it contains further classes, these must belong to the \Acme\Demo\
> namespace.
>
> If that's how you need the namespaces, fine, but let's not get into that
> with this standard. We will not understand what the developer is trying to
> do or what files are best namespaced or how to do so. I can see blog posts
> and developers sharing namespacing strategies with one another around an
> application or framework. But, out of the standard, IMO.
>
> "PHP class definitions" are defined using include to avoid restrictions on
> the file extensions (".php", ".php5" etc.) or similar.

### 1.7 Further Schemes

Further schemes MAY freely be added by implementations, consumers and future
PSRs. However, the following schemes SHOULD be used for the corresponding
resource types listed in the table:

>> See that's a practice Symfony folks are accustomed to using. I don't have
>> any files for lang - I keep it in a database. I don't NS doc files
>> It would limit me to have to use public:// for those assets as I have 
>> processing needs that require I keep them separate and no value in 
>> combining them. So, this would preclude my use of the standard. 

>> Instead the standard should be more generic and define what a scheme is
>> and what it is used for. So, for me, a scheme is a way of categorizing 
>> namespaces so that valid file extensions can be defined and a handler 
>> can be assigned for responding in a manner with the findResource results
>> that the application intends. 

>> Ideal: ? : The consumer MUST define schemes which associate valid file extensions
>> search results handling requirements. These schemes are used as in the URI.
>> Examples:

| Scheme     | Resource Type                             |
|------------|-------------------------------------------|
| config://  | configuration files                       |
| doc://     | documentation files                       |
| lang://    | translation files                         |
| public://  | CSS, JS, images and similar public files   |
| view://    | template/view files                       |

2. Package
----------

The described interface as well as relevant exception classes and a test suite
to verify your implementation is provided as part of the psr/resource-location
package.

3. ResourceLocatorInterface
---------------------------

> TODO: Will be elaborated once there is agreement on the general direction.
> Yup. Ok, good stuff. If we can find time to get with Beau and get into
> some of this detail -- some areas need a lot less detail, some need more 
> fleshing out - this is an excellent start. Thanks much.

```php
<?php

namespace Psr\ResourceLocation;

interface ResourceLocatorInterface
{
    public function findResource($uri);

    public function findResourceVariants($uri);
}
```
