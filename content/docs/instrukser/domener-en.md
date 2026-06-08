+++
title = "Domains"
description = "How to set up domains"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/instrukser/domener.md"
+++

## How to set up a new domain:

1. Go to bestemor `ssh root@bestemor.s.fribyte.no`
2. `ssh fribyte@ns1.fribyte.no`
3. Update the relevant domain by running the script `./update_zone.sh fribyte.no`
   (use something other than fribyte.no to edit domains that are not
   .fribyte.no)
4. Add the appropriate entry
5. Update the serial `{år}{måned}{dato}{hh}` -> `2022020618`

- It is important that the serial is increased!
  - But the serial cannot be larger than 32 bits

6. `rndc reload fribyte.no` -> Executed automatically by `update_zone.sh`
7. `rndc reload` -> Executed automatically by `update_zone.sh`
8. In theory it can take up to 2 hours before it works, but it usually takes
   10-20 minutes
9. We use Cloudflare's dns `1.1.1.1`, so it can help to visit
   https://1.1.1.1/purge-cache/ and then enter your new domain name and press
   "Purge Cache". Then Cloudflare will update that domain for everyone in the
   world who uses Cloudflare.

For more detailed information, see [/docs/nettverk/dns](/docs/nettverk/dns)
