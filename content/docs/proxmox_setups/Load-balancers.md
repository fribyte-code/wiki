+++
title = "Load-balancers"
description = "Simple walkthough of our current HA setup"
date = 2022-03-06T18:53:00+00:00
updated = 2022-03-06T18:53:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 1
draft = false
+++

## Hva er load balancing?

Load balancing gjøres ved å ta in spørringer på en IP/endepunkt og sende de videre til et distribuert system balansert. 

Dette brukes gjerne for f.eks Kubernetes, da alle tjenester er tilgjengelig fra alle Master nodene (hutre, petter ..)

## Hva har vi?

I dag har vi 2 load-balancere. De er hostet på Pluto og Fergus respektivt. 

load-balancer-1 er masteren og load-balancer-2 er slaven

Vi bruker i dag et system med haproxy og keepalived for å proxye adresser videre. 

Samt begge load-balancerene kjører en wireguard server som også er koblet opp mot keepalived, dette gir oss tilgang til interne tjenester som proxmox UI og Rancher og evt tjenester vi 
skal ha i fremtiden som vi ikke nødvendigvis vil åpne ut på internett

I teorien skal alle offentlige tjenester ligge som en ingress i kubernetes clusteret automatisk ville være tilgjengelig gjennom disse 2 load-balancerene. 

## Synkronisere configurasjon mellom lb-1 og lb-2

Dette gjøres i dag ved et script som ligger på load-balancer-1 

Dette betyr at alle endringer bør gjøres på load-balancer-1 også kjører du scriptet for å migrere endringene over på load-balancer-2

Scriptet finner du i `/home/fribyte/propagate_haproxy_wg.sh` på load-balancer-1



## Keepalived

Keepalived bruker en protokoll som heter Vrrp for å ha en flytende IP mellom de to serverene.

Dette ligger på IP 158.37.6.28. 

Load-balancer-1 er konfigurert som master, og vil kontinuerlig sjekke at haproxy er oppe og går. 

Dersom haproxy går ned, eller load-balancer-1 går ned, vil load-balancer-2 ta over IPen de deler

Dette er helt sømløst for brukeren

Her er configen brukt i dag: 


```
MASTER: 

vrrp_script chk_haproxy {
    script 'killall -0 haproxy' # faster than pidof
    interval 2
}

vrrp_instance VI_1 {
  state MASTER
  interface eth0
  virtual_router_id 55
  priority 150
  advert_int 1
  unicast_src_ip MASTER_IP
  unicast_peer {
    SLAVE_IP
  }
  authentication {
    auth_type PASS
    auth_pass """INSERT PASSWORD HERE"""
  }
  virtual_ipaddress {
    SHARED_IP
  }
  track_script {
    chk_haproxy
  }
}

SLAVE: 

vrrp_script chk_haproxy {
    script 'killall -0 haproxy' # faster than pidof
    interval 2
}

vrrp_instance VI_1 {
  state BACKUP
  interface eth0
  virtual_router_id 55
  priority 100
  advert_int 1
  unicast_src_ip SLAVE_IP
  unicast_peer {
    MASTER_IP
  }
  authentication {
    auth_type PASS
    auth_pass """INSERT PASSWORD HERE"""
  }
  virtual_ipaddress {
    SHARED_IP
  }
  track_script {
    chk_haproxy
  }
}
```

## Haproxy

Haproxy brukes for å proxye trafikken videre. 

Planen er å bruke denne både som proxy for eksterne tjenester i kubernetes clusteret, men også på interne tjenester

Dette gjøres ved at begge load-balancer VMene er koblet til samme Wireguard VPN nettverk. Så alle interne tjenester bindes til VPN nettverket
mens alle eksterne tjenester bindes direkte til den flytende IPen

Configs for HAproxy finner du under `/etc/haproxy/haproxy.cfg` på enten load-balancer-1 eller load-balancer-2

## Wireguard

Denne er helt lik oppsette pleide å være på Skaftetrynet

Info om hvordan koble seg til denne finner du under `nytt_medlem`

Config for denne finner du i `/etc/wireguard/wg0.conf` på load-balancer-1 eller load-balancer-2

