---
layout: page-fullwidth
subheadline: They look so similar - what's the difference!
title:  "Powershell -replace is not .replace"
teaser: "In Powershell there are (at least) two ways of doing string replacement..."
breadcrumb: true
header: no
tags:
    - dev-ops powershell regex
categories:
    - dev-ops
---
Just a friendly reminder that Powershell's -replace operator behaves differently from the string's replace method. The former does a regular expression replacement. While the latter does a plain text replacement. You can see this demonstrated below:

{% gist c00f22a3bbe7330f41cd %}

Note that in the use of the -replace operator, the regular expression is searching for 'p' or 'me' and thus replaces both the 'p' in 'Help' and the word 'me' with the replacement string.