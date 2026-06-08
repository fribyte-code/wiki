+++
title = "lytt.srib.no"
description = "Explanation of how the static website that serves the web radio is set up"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/produksjon/lytt-srib-no.md"
+++

### Static site in Docker

To host a static site, you need to serve the files through an HTTP server such
as nginx. For example, [lytt.srib.no](https://lytt.srib.no) is hosted by having
a static `index.html` in the `/var/www/html/srib.no/lytt` directory. We run an
nginx container that only serves files from this directory by mounting the
directory with the following command:

```sh
sudo docker run -d --restart always \
    --name lytt.srib.no \
    -v /var/www/srib.no/lytt:/usr/share/nginx/html \
    --network azuracast_frontend \
    --env VIRTUAL_HOST=lytt.srib.no \
    --env LETSENCRYPT_HOST=lytt.srib.no \
nginx:latest
```

### HTML iframe tag

Azuracast generates an `iframe` tag for the public "mountpoints" of the
IceCast2 setup. A "mountpoint" is, in a way, a hook you can hang audio streams
on; a receiver for your audio stream that forwards it onto the internet through
its own domain.

For the radio, it looks like this:

```html
<iframe
  src="https://radio.srib.no/public/studentradioen_i_bergen/embed?theme=light"
  frameborder="0"
  allowtransparency="true"
  style="width: 100%; min-height: 150px; border: 0;"></iframe>
```

This is pasted directly into `index.html`, as mentioned earlier.
