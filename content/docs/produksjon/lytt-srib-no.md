+++
title = "lytt.srib.no"
description = "Forklaring på hvordan den statiske nettsiden som serverer nettradioen"
date = 2022-02-25T08:00:00+00:00
updated = 2024-01-11T08:00:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

### Statisk side i Docker

For å hoste en statisk side må du servere filene gjennom en http-server som nginx. For eksempel [lytt.srib.no](https://lytt.srib.no) blir hostet ved at en statisk `index.html` ligger i mappen `/var/www/html/srib.no/lytt`. Vi kjører en nginx container som kun serverer filer fra denne mappen ved å mounte mappen med følgende kommando:

```sh
sudo docker run -d --restart always \
    --name lytt.srib.no \
    -v /var/www/srib.no/lytt:/usr/share/nginx/html \
    --network azuracast_frontend \
    --env VIRTUAL_HOST=lytt.srib.no \
    --env LETSENCRYPT_HOST=lytt.srib.no \
nginx:latest
```

### HTML iframe-tag

Azuracast genererer en `iframe`-tag for de offentlige "mountpoints"-ene til IceCast2-maskineriet. En "mountpoint" er på en måte en krok du kan henge lydstrømmer på; en mottaker for lydstrømmen din, som videresender den ut på nettet gjennom sitt eget domene. 

For radioen sitt tilfelle, ser den slik ut:

```html
<iframe src="https://radio.srib.no/public/studentradioen_i_bergen/embed?theme=light" frameborder="0" allowtransparency="true" style="width: 100%; min-height: 150px; border: 0;"></iframe>
```

Dette er limt rett inn i `index.html`, som nevnt tidligere.