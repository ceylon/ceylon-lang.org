---
layout: reference11
title_md: '`tagged` annotation'
tab: documentation
unique_id: docspage
author: Tom Bentley
doc_root: ../../..
---

# #{page.title_md}

Marks a declaration with an arbitrary tag.

## Usage

<!-- try: -->

    tagged("thread-safe")
    class Example() {
        tagged("blocks")
        void m() {
        }
    }

## Description

The `tagged` annotation is processed by the `ceylon doc` tool.

Its content should be a short keyword or identifier, and
*not* [Markdown formatted](../markdown/) text.

## See also

* API documentation for [`tagged`](#{site.urls.apidoc_1_1}/index.html#tagged)
* Reference for [annotations in general](../../structure/annotation/)

