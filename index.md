---
# tell Jekyll “this is the homepage”
layout: single            # or “home” or “single”—see below
title: "About"           # whatever makes sense
permalink: /              # ensure it becomes your “/”
author_profile: true
---

# <!-- Or, to pull in your existing About page: -->
{% include_relative _pages/about.md %}
