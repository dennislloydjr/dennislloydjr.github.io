---
layout: page-fullwidth
subheadline: Install a Missing Module to Overcome this Build Error
title:  "Could not find C++ compiler 'g++' in system path!"
teaser: "Problems Compiling a C++ Module with Gradle"
breadcrumb: true
header: no
tags:
    - gradle linux c++ g++ genome-indexing
categories:
    - genome-indexing
---
While creating a Gradle build script for the Genome-Indexing project, I ran into the following error:
{% gist e24d494dce6fe83f87be %}

Google didn't help much in finding a solution, but a careful reading of the Gradle documentation turned up this [hint](https://bit.ly/GradleDoc1):

> Note that if you are using GCC then you currently need to install support for C++, even if you are not building from C++ source. This caveat will be removed in a future Gradle version.

Ah-ha! That's it - I probably had a C compiler installed, but not C++.

Here's how to fix it:
{% gist 15969d7dffa5c4e153f9 %}


