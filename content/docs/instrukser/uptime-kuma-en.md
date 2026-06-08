+++
title = "Uptime Kuma"
description = "How to add a new service in Uptime Kuma"
template = "docs/page.html"
sort_by = "weight"
weight = 10003

[extra]
lang = "en"
translation = "docs/instrukser/uptime-kuma.md"
+++

friByte uses [Uptime Kuma](https://github.com/louislam/uptime-kuma) to keep
track of the uptime of the services we operate. It is available at
[uptime.fribyte.no](https://uptime.fribyte.no). You can log in to the admin
panel by going to
[uptime.fribyte.no/dashboard](https://uptime.fribyte.no/dashboard).

This service is connected to Mattermost through Uptime Kuma's "Mattermost
notification" feature.

### How to add a new service

1. Log in
2. Click "Add New Monitor"
3. Fill in the following:

- Friendly Name: What the service should be called in the list
- Retries: How many times Uptime should retry before notifying about an error
- Certificate Expiry Notification: Notifies you if Let's Encrypt has expired
- Oppetidroboten: whether this service should send alerts to Mattermost

_There are other settings as well, but this is the minimum you need_

### How to add a service to the status page

1. Log in
2. Go to the [status page](https://uptime.fribyte.no).
3. Click "Edit Status Page"
4. In the "Add Monitor" field, enter the name of the service
5. Move the service to an appropriate group
