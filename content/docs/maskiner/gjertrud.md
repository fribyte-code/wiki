+++
title = "Gjertrud"
description = "Oversikt over Gjertrud"
template = "docs/page.html"
sort_by = "weight"
weight = 1
draft = false
+++

## Gjertrud

# Lagringsoppsett

Gjertrud er i dag satt opp med alle diskene satt i sin egen Raid 1 gjennom det
standard Raid kortet som sitter i hovedkortet på serveren. Dette skal
oppgraderes i fremtiden til et Dell H200 ( flashet til LSI 9211-8i IT-mode )

Alle diskene i denne serveren kobles opp i et ZFS pool slik at vi kan bruke
replication mellom Gjertrud og Konrad

# Ting lært

For å kunne bruke et ekstrern HBA kort i denne serven:

1. Disable det innebygde RAID kortet
2. Oppgradere BIOS
