+++
title = "Update Headscale"
description = "How to update Headscale when there is a new release"
template = "docs/page.html"
sort_by = "weight"
weight = 10001
draft = false

[extra]
lang = "en"
translation = "docs/instrukser/oppdatere-headscale.md"
+++

It is a good idea to update the Headscale coordination server regularly to
receive new security updates.

1. Log in to fribyte@headscale
2. Find the latest Headscale release on [GitHub](https://github.com/juanfont/headscale/releases?page=2)
3. *Check for breaking changes between the old and new releases*
4. Back up the Headscale SQLite database `sudo cp /var/lib/headscale/db.sqlite /var/lib/headscale/db.sqlite.old`
5. Run the [installation process](https://headscale.net/stable/setup/install/official/#using-packages-for-debianubuntu-recommended) again by setting `HEADSCALE_VERSION` to the new version number and `HEADSCALE_ARCH="amd64"`
6. Restart the Headscale service
7. Profit
