---

title: "Unable to Obtain Stable Firefox Connection in 60 Seconds"
layout: doc
tags:
  - troubleshooting
  - ruby
---

This is an issue with the selenium-webdriver gem.
As Firefox updates to newer revisions, the interface used by selenium-webdriver can break.
Fortunately, the fix is pretty easy.

Update to a new version of the selenium-webdriver gem in your Gemfile, if
possible to the [latest version](http://rubygems.org/gems/selenium-webdriver).
