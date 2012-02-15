---
layout: reference
title: `ceylond` - The ceylon documentation compiler
tab: documentation
unique_id: docspage
author: Tom Bentley
---

# #{page.title}

## Usage 

<!-- lang: none -->
    ceylond [options] <module-names>...

Options include:

* `-out` Specifies the output module repository (which must be publishable).
* `-src` Specifies a source directory. XXX can be repeated?
* `-rep` specifies a module repository containing dependencies. XXX can be repeated?
* `-non-shared` Includes documentation for package-private declarations.
* `-source-code` Includes source code to the generated documentation.   
* `-d` Disable the default module repositories and source directory.

## Description

The documentation compiler generates XHTML-format documentation from Ceylon 
source files.

## See also

* [`ceylond`](#{site.urls.spec}#thedocumentationcompiler) in the language specification
* [module names and versions](#{site.urls.spec}#modulenamesandversionidentifiers) in the language specification.
