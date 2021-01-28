---
layout: post
title: "[Django] Django auto reload"
subtitle: "with gunicorn '--reload' option"
author: qwlake
categories: Django
tags: Django gunicorn
---

When you work on django in docker, sometimes it's not might be autoload. I have been shutdown and restart the `docker-compose` process, but, there is cool way.

1. Mount code volumes into django image ([https://stackoverflow.com/a/32029035](https://stackoverflow.com/a/32029035))
2. Set `--reload` option on the command line executor of gunicorn

```
gunicorn myapp.dev.wsgi -b 0:8080 --reload
```