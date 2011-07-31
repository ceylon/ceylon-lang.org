---
layout: documentation
title: Building the website
tab: documentation
author: Emmanuel Bernard
---
# How to build the ceylon-lang.org website

Building the website and work the documentation is fairly simple. It is based
on `Markdown` or `haml` based files and use Git as a rudimentary CMS.

## infrastructure

You need to:

* get Ruby
* if on Mac OS, get XCode (needed for native gems)
* `gem install awestruct` or `sudo gem install awestruct`
* `git clone git@github.com:ceylon/ceylon-lang.org.git;cd ceylon-lang.org`

## Serve the site locally

* Go in your `~/ceylon-lang.org` directory.  
* Run  `awestruct -d`
* Open your browser to <http://localhost:4242>

Any change will be automatically picked up except for `_partials` files 
(eg. this footer) and for `_base.css`.

### If your changes are not visible...

If for whatever reason you make some changes which don't show up, you can
completely regenerate the site:

    rm -fR _site/ .sass-cache/
    awestruct -d

### If serving the site is slow...

Sometimes on Linux, serving the file is atrociously slow 
(something to do with WEBRick).

Use the following alternative:

* Go in your `~/ceylon-lang.org` directory.  
* Run  `awestruct --auto -P development`
* In parallel, go to the `~/ceylon-lang.org/_site` directory
* Run `python -m SimpleHTTPServer 4242`

You should be back to millisecond serving :) 
