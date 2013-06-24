Path Matching
=============

This document describes an algorithm that finds the file system path(s) for a
logical path.

The main goal is to provide a foundation for future PSRs based on this
algorithm, such as an autoloader PSR, a resource location PSR and so on.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

[Meta Document](path-matching-meta.md)

1. Specification
----------------

### 1.1 Definitions

**Path Matcher**: A program implementing the path matching algorithm described
in section 1.2.

> This should neither be fixed to PHP code, nor C code, nor a method, nor a
> function. "Program" is a generic term that matches all of these concepts.

**Separator**: A single character chosen by the path matcher, for example a
slash ("/").

> Allows to use this algorithm for both autoloading (separator: "\") and
> resource location (separator: "/").

**Path Segment**: A sequence of one or more characters except for separators.

**(Logical) Path**: A sequence of zero or more path segments, divided by
separators and starting with a separator. Given the separator "/", then `/`,
`/A`, `/A/B` and `/A/B/C` are valid paths.

> E.g. a namespace (\Acme\Demo\Parser) or a URI path (/acme/demo-package/config)

**File System Path**: A path to a file or directory on the file system.

**Prefix Path**: Given a path, then a prefix path is any prefix of the path
that is followed by a separator. The single `/` is always a prefix path. For
example, given the separator "/" and the path `/A/B/C`, then `/`, `/A` and
`/A/B` are valid prefix paths.

**Path Mapping**: A set of logical paths, each of which is associated with one
or more file system paths. Given a path mapping, we call the logical paths in the
mapping *mapped*. We refer to the file system paths in the mapping as *base paths*.
The base paths MUST be provided such that PHP can read and include them from the
local file system.

> That is, the base paths must be loadable even if allow_url_fopen and
> allow_url_include are disabled.

### 1.2 Path Matching Algorithm

Compliant path matchers MUST implement this algorithm or an equivalent
algorithm that returns the same outputs for the same inputs.

> For example, a caching algorithm is implemented differently but behaves
> the same.

#### 1. Generating Potential Matches

Given a logical path and a path mapping which associates that path with one
or more base paths, then every base path is a *potential match*. For example,
given the separator "/", the path `/A/B/C/D` and a path mapping which associates
`/A/B/C/D` with `/src`, then `/src` is a potential match.

> The full path itself can be mapped. Needed in the resource location PSR to
> look up the directories that a namespace is mapped to.

Given a logical path and a path mapping which associates any of its prefix
paths with one or more base paths, then a potential match is generated for
each base path by applying the [Path Transformation algorithm described in PSR-?]
(https://github.com/pmjones/fig-leaf/blob/master/transform-logical-paths.md),
on the logical path, the prefix path, the separator and the base path. The result
is a potential match.

Separators in potential matches MUST be replaced by directory separators.

> Allows the use of other separators than slashes, e.g. backslashes ("\").

#### 2. Evaluating Potential Matches

A potential match is *evaluated* by checking whether it exists on the local
file system. If it does, it is called a *match*. For example, if both
`/src/C/D` and `/alt/C/D` are potential matches and only `/src/C/D` actually
exists on the file system, then only `/src/C/D` is a match.

> Matches must exist.

If both a full path and any of its prefix paths are mapped, potential matches
for the full path MUST be evaluated before those for the prefix paths. If a
mapped path contains multiple mapped prefix paths, potential matches for longer
prefix paths MUST be evaluated before those for shorter prefix paths.

> Define order of evaluation..

If a path is mapped to multiple base paths, the potential matches MUST be
evaluated in the order of the base paths.

> When multiple implementations of this algorithm (e.g. PSR-X and PSR-R)
> receive the same mapping, they should behave identically.

A path matcher MAY choose to abort this algorithm once a match has been found,
or continue in order to generate all matches.

> Find first vs. find all matches.

A path matcher MAY not find a match for a logical path. The result in this
case is undefined.

> Exception, returning null etc. can be chosen by the implementation.

### 1.3 EBNF

The following block defines the path syntax used in this PSR using the Extended
Backus-Naur Form (EBNF) specified in ISO/IEC 14977.

```
separator     = chosen separator character
path-symbol   = all characters - separator
path-segment  = path-symbol, {path-symbol}
prefix-path   = separator, {path-segment, separator}
path          = prefix-path, [path-segment, {separator, path-segment}]
```

2. Package
----------

The test suite to verify a path matcher implementation is provided as part of the
psr/path-matching package.

3. Example Implementation
-------------------------

The example implementation MUST NOT be regarded as part of the specification; it is
an example only. Path matchers MAY contain additional features and MAY differ in how
they are implemented. As long as a path matcher adheres to the rules set forth in
the specification it MUST be considered compatible with this PSR.

```php
<?php

/**
 * An example implementation of the above specification that finds a match
 * for a logical path when given a mapping of paths to base paths and a
 * separator character.
 *
 * Note that this is only an example, and is not a specification in itself.
 */
function match_path($path, array $path_mappings, $separator)
{
    // remember the length of the path
    $path_length = strlen($path);
    
    // first see if the complete path is mapped
    $prefix_path = $path;

    // the reverse offset of the separator following the prefix path
    $cursor = -1;
    
    while (true) {
        // are there any base paths for this prefix path?
        if (isset($path_mappings[$prefix_path])) {
            // look through base paths for this prefix path
            foreach ((array) $path_mappings[$prefix_path] as $base_path) {
                // create a potential match using a PSR-? transformation
                $potential_match = transform($path, $prefix_path, $separator, $base_path);
                
                // can we read the file from the file system?
                if (is_readable($potential_match)) {
                    // yes, we have a match
                    return $potential_match;
                }
            }
        }
        
        // once the cursor tested the first character, the
        // algorithm terminates
        if ($prefix_path === $separator) {
            return;
        }
        
        // place the cursor on the next separator to the left
        $cursor = strrpos($path, $separator, $cursor - 1) - $path_length;

        // the prefix path is the part left of the cursor, e.g. "/Acme/Demo"
        // "/" is also a prefix path
        $prefix_path = substr($path, 0, max($cursor, -$path_length + 1));
    }
}
```
