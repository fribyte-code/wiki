+++
title = "Oppdatere Headscale"
description = "Hvordan oppdatere headscale når det er en ny oppdatering"
template = "docs/section.html"
sort_by = "weight"
weight = 1
draft = false
+++

Det lønner seg å oppdatere headscale koordineringsserveren regelmessig for å få nye sikkerhetsoppdateringer.

1. Logg inn på fribyte@headscale
2. Finn nyeste release av headscale på [github](https://github.com/juanfont/headscale/releases?page=2)
3. *Sjekk for breaking changes mellom gamle og nye release*
3. Ta backup av headscale sqlite databasen `sudo cp /var/lib/headscale/db.sqlite /var/lib/headscale/db.sqlite.old`
4. Kjør installasjonsprosessen på nytt ved å sette `HEADSCALE_VERSION` til det nye versjonsnummeret og `HEADSCALE_ARCH="amd64"`
5. Restart headscale servicen
6. Profit
