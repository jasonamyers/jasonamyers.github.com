---
layout: post
title: "Mavericks uses clang by default"
description: "Many of our tools are used to the standard gcc complier, and need
help"
tags: [Mavericks, clang, strlcpy]
comments: true
share: true
---

So many of our tools are accustom to and developed against the gcc compiler;
however, the preferred (default in some cases) is now clang on OS X 10.9
Mavericks.  One of the problems is that clang has a bunch of "builtin" function
definitions, and those builtins can conflict with the normal header files we are
used to using.

The full set, from clang/Basic/Builtins.def, is:

::

    LIBBUILTIN(calloc, "v*zz",        "f",     "stdlib.h", ALL_LANGUAGES)
    LIBBUILTIN(malloc, "v*z",         "f",     "stdlib.h", ALL_LANGUAGES)
    LIBBUILTIN(realloc, "v*v*z",      "f",     "stdlib.h", ALL_LANGUAGES)
    LIBBUILTIN(memcpy, "v*v*vC*z",    "f",     "string.h", ALL_LANGUAGES)
    LIBBUILTIN(memcmp, "ivC*vC*z",    "f",     "string.h", ALL_LANGUAGES)
    LIBBUILTIN(memmove, "v*v*vC*z",   "f",     "string.h", ALL_LANGUAGES)
    LIBBUILTIN(strncpy, "c*c*cC*z",   "f",     "string.h", ALL_LANGUAGES)
    LIBBUILTIN(strncmp, "icC*cC*z",   "f",     "string.h", ALL_LANGUAGES)
    LIBBUILTIN(strncat, "c*c*cC*z",   "f",     "string.h", ALL_LANGUAGES)
    LIBBUILTIN(strxfrm, "zc*cC*z",    "f",     "string.h", ALL_LANGUAGES)
    LIBBUILTIN(memchr, "v*vC*iz",     "f",     "string.h", ALL_LANGUAGES)
    LIBBUILTIN(strcspn, "zcC*cC*",    "f",     "string.h", ALL_LANGUAGES)
    LIBBUILTIN(strspn, "zcC*cC*",     "f",     "string.h", ALL_LANGUAGES)
    LIBBUILTIN(memset, "v*v*iz",      "f",     "string.h", ALL_LANGUAGES)
    LIBBUILTIN(strlen, "zcC*",        "f",     "string.h", ALL_LANGUAGES)
    LIBBUILTIN(snprintf, "ic*zcC*.",  "fp:2:", "stdio.h", ALL_LANGUAGES)
    LIBBUILTIN(vsnprintf, "ic*zcC*a", "fP:2:", "stdio.h", ALL_LANGUAGES)
    LIBBUILTIN(alloca, "v*z",         "f",     "stdlib.h", ALL_GNU_LANGUAGES)
    LIBBUILTIN(stpncpy, "c*c*cC*z",   "f",     "string.h", ALL_GNU_LANGUAGES)
    LIBBUILTIN(strndup, "c*cC*z",     "f",     "string.h", ALL_GNU_LANGUAGES)
    LIBBUILTIN(bzero, "vv*z",         "f",     "strings.h", ALL_GNU_LANGUAGES)
    LIBBUILTIN(strncasecmp, "icC*cC*z", "f",   "strings.h", ALL_GNU_LANGUAGES)
    LIBBUILTIN(strlcpy, "zc*cC*z",    "f",     "string.h", ALL_GNU_LANGUAGES)
    LIBBUILTIN(strlcat, "zc*cC*z",    "f",     "string.h", ALL_GNU_LANGUAGES)

