---
layout: reference11
title_md: '`throws` annotation'
tab: documentation
unique_id: docspage
author: Tom Bentley
doc_root: ../../..
---

# #{page.title_md}

The `throws` annotation documents an exception that can be thrown by a
function, value, or class. 

## Usage

The annotation accepts a [program element reference 
expression](/documentation/current/tour/annotations/#the_metamodel)
identifying a class of `Exception` that is thrown, along with a
description of the conditions that cause the exception.

<!-- try: -->
    throws (`class AssertionError`, "if `i < 0`")
    void example(Integer i) {
        assert (i>=0);
    }


## Description

The `throws` annotation is processed by the 
[`ceylon doc`](#{site.urls.ceylon_tool_current}/ceylon-doc.html) tool. 
Its `description` is assumed to contain [Markdown formatted](../markdown/) text.

## See also

* API documentation for [`throws`](#{site.urls.apidoc_1_1}/index.html#throws)
* Reference for [annotations in general](../../structure/annotation/)

