---
title: Docker-compose ENV guide
description: Official guide to install lancache.net
permalink: /docs/installation/docker-compose/env/
---

There are several options specified in the default `.env` for you to work through. We've provided a basic summary here, for further detail check out the Environment Variables sections on the right. 

## Settings
<div class="note info">      
<h5>Don't forget the basics</h5>   
<p>
You *MUST* set at least <code><a href="#LANCACHE_IP">LANCACHE_IP</a></code> and <code><a href="#DNS_BIND_IP">DNS_BIND_IP</a></code>. It is highly recommended that you change <code><a href="CACHE_ROOT">CACHE_ROOT</a></code> to a folder of your choosing, and set <code><a href="cache_disk_size">CACHE_DISK_SIZE</a></code> to a value that suits your storage capacity.
</p>
</div>                          

### USE_GENERIC_CACHE
This controls IP assignment within the DNS service - it assumes that every service is reachable by default on every IP given in `LANCACHE_IP`. See the [lancache-dns](https://github.com/lancachenet/lancache-dns) project for documentation on customising the behaviour of the DNS service.

### LANCACHE_IP
This provides one or more IP addresses to the DNS service to advertise the cached services. If your cache host has exactly one IP address (e.g. `192.168.0.10`), specify that here. If your cache host has more IP addresses, you can list all of them, separated by spaces (e.g. `192.168.0.10 192.168.0.11 192.168.0.12`) - DNS entries will be configured for all services and all IPs by default.

> **Note:** unless your cache host is at `10.0.39.1`, you will want to change this value.

### DNS_BIND_IP
This sets the IP address that the DNS service will listen on. If your cache host has exactly one IP address (eg. `192.168.0.10`), specify that here. If your cache host has multiple IPs, specify exactly one and use that. This compose stack does not support the DNS service listening on multiple IPs by default.

> **Note:** unless your cache host is at `10.0.39.1`, you will want to change this value.

Remember to advertise the IP given in `DNS_BIND_IP` via DHCP to your network as a nameserver. In this scenario, all clients configured to use the nameservers from DNS will use the `lancache-dns` service.
  This allows the `lancache-dns` service to provide clients with the appropriate local IPs for cached services, and all other requests will be passed to `UPSTREAM_DNS`.

### UPSTREAM_DNS
This allows you to choose one or more IP addresses for upstream DNS resolution if a name is not matched by the `lancache-dns` service (e.g. non-cached services, local hostname resolution).

Whichever resolver you choose depends on your network's requirements - if you don't need to provide internal DNS names, you can point `UPSTREAM_DNS` directly to an external resolver (the default is Google's DNS at `8.8.8.8`).

If you run internal services on your network, you can set `UPSTREAM_DNS` to be your internal DNS resolver(s), semicolon separated (e.g. `192.168.0.1; 192.168.0.2`).

#### Example external resolvers
- Google DNS:
  - `8.8.8.8`
  - `8.8.4.4`
- Cloudflare
  - `1.1.1.1`
- OpenDNS
  - `208.67.222.222`
  - `208.67.220.220`

### CACHE_ROOT
This will be used as the base directory for storing cached data (as `CACHE_ROOT/cache`) and logs (as `CACHE_ROOT/logs`).

The `CACHE_ROOT` should either be on a separate partition, or ideally on separate storage devices entirely, from your system root.

> **Note:** this setting defaults to `./lancache`. Unless your cache storage lives here, you probably want to change this value.

### CACHE_DISK_SIZE
This setting will constrain the upper limit of space used by cached data. You generally want to leave a small gap (10-20GB at least) between the size listed here and the available storage space used for the cached data, just in case.

The cache server will automatically cached data when the total stored amount approaches this limit, in a least-recently-used fashion (oldest data, least accessed deleted first).

> **Note:** that this must be given in megabytes with an `m` suffix (e.g. the default value, `1000000m`).

## `CACHE_INDEX_SIZE`
Change this to allow sufficient index memory for the nginx cache manager (default 500m)
We recommend 250m of index memory per 1TB of CACHE_DISK_SIZE 

> **Note:** this setting does not limit the amount of memory that the Linux host will use for page caches, only what the cache server will use itself - see the Docker documentation on limiting memory consumption for a container if you wish to constrain the total memory consumption of the cache server, but generally you want as much memory as possible on your cache server to be used to store hot data.

### CACHE_MAX_AGE
This setting allows you to control the maximum duration cached data will be kept for. The default should be fine for most use cases - the `CACHE_DISK_SIZE` setting will generally be used before this for aging out data.

> **Note:** this must be given as a number of days in age before expiry, with a `d` suffix (e.g. the default value, `3650d`).

