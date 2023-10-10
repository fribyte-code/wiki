# friByte intranett

## Hva er dette?

Dette er friByte sin interne wiki-side. Den bruker rammeverket Zola til å
generere en statisk side.

## Sett opp lokalt

```ssh
git clone git@github.com:fribyte-code/wiki.git
git submodule update --init --recursive
```

## Zola

Zola er en Statisk Side Generator, skrevet i Rust. Du finner dokumentasjon på
https://getzola.org.

## Miljø-krav

Zola versjon >=0.13.0 (se zola-dokumentasjon instrukser)

## Problemstillingen

Vi ville ha en nettside som kunne holde på masse markdown-filer, og unngå bruk
av databaser. I disse markdown-filene vil da all wiki-informasjonen stå. Vi
ønsket at alle medlemmer kunne ha hele wikien lokalt lagret, samtidig som den er
serveres ut til nettet fra en virtuell server. Vi ville at nettsiden skulle
oppdateres hver gang vi oppdaterer repoet.

## Hvordan er wikien implementert?

For å implementere dette, bruker vi GitHub actions som bygger den statiske siden
ved merge til master, og rsync-er resultatet til server.

Det er også en action for å validere endringer som er gjort ved opprettelse av 
pull request til master, slik at vi ikke merger inn ødeleggende endringer.

## Kommandoer

For å kjøre wikien lokalt: `zola serve` Dette er det samme som livereload, hvor
nettsiden på din lokale server oppdaterer seg umiddelbart etter en endring i
filene.

For å bygge de statiske nettsidefilene: `zola build` Dette lager en mappe som
heter public, som inneholder alle filene man trenger for å servere en statisk
side.

## Notater

- I skrivende stund bruker wikien en ferdiglaget utforming som heter adidoks.
  Den er å finne på zola sine nettsider. Følg instruksene gitt av adidoks for
  hvordan man legger til nytt innhold.

