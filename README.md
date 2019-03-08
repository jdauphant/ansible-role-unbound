ansible-role-unbound
====================

Ansible role for Unbound DNS Server and resolver


# Supports
- Add DNS entries (multiple record types per entry)
- Generation of DNS entries from ansible inventory (A entries and reverse)
- Forward to another dns
- IPv4/IPv6 for reverse

# Information :
- Tested on Ubuntu
- Tested on Debian Stretch (Use `forward-ssl-upstream` instead of `forward-tls-upstream`)
- Untested on Fedora

# Example :

## Simple forward on localhost :
```
# Activate forward (active by default)
unbound_forward_zone_active : true
# Activate DNS over TLS (active by default)
unbound_forward_zone_configuration:
    - forward-ssl-upstream: "yes" # `forward-ssl-upstream` for old version
# Forward server to Cloudflare DNS
unbound_forward_zone:
   - "1.1.1.1@853#cloudflare-dns.com"
   - "1.0.0.1@853#cloudflare-dns.com"
```

## Generate entries and reverse from the inventory (need ansible_ssh_host set on all host)
```
# Listen interface
unbound_interfaces:
    - 127.0.0.1
    - 192.168.0.10

# Authorized IPs
unbound_access_control:
    - 127.0.0.1 allow
    - 192.168.0.0/24 allow

# Create entries from inventory (reverse also created by default)
unbound_inventory_domain:
    all: 'internal.domain' # All hosts

# Create reverse entries from inventory
unbound_inventory_reverse_domain:
    all: 'internal.domain' # All hosts

# Activate forward (active by default)
unbound_forward_zone_active: true
# Activate DNS over TLS (active by default)
unbound_forward_zone_configuration:
    - forward-tls-upstream: "yes" # `forward-ssl-upstream` for old version
# Forward server to Cloudflare DNS
unbound_forward_zone:
   - "1.1.1.1@853#cloudflare-dns.com"
   - "1.0.0.1@853#cloudflare-dns.com"

```

## More complete example (need ansible_ssh_host set on all host)
```
# Listen interface
unbound_interfaces:
    - 127.0.0.1
    - 192.168.0.10

# Authorized IPs
unbound_access_control:
    - 127.0.0.1 allow
    - 192.168.0.0/24 allow

# Simple DNS entries
unbound_domains:
    - domain_name: "example.com"
      host1: IN A 127.0.0.1
      www: IN CNAME host1

# Create entry and reverse
unbound_domains_with_reverses:
    - domain_name: "reversed.example.com"
      host1: 127.0.0.1
      host2: 127.0.0.2
      host3: 127.0.0.3

# Create entries from inventory
unbound_inventory_domain:
    all: 'localdomain' # All hosts
    webserver: 'webserver.localdomain' # Hosts in webserver

# Create reverse entries from inventory
unbound_inventory_reverse_domain:
    dbserver: 'dbserver.localdomain' # Hosts in dbserver
    webserver: 'webserver.localdomain' # Hosts in webserver

# Type of local host (default : static )
unbound_local_zone_type:
    example.com: "transparent"
    reversed.example.com: "static"

```

### Create local domain data

For creating local domain data with the `unbound_domain` variable two variants can be used.
The simple one uses plain strings to create one resource record per host name.
With this variant no other resource records for the same name can be created.

The more complex version allows dict objects to set the following resource records: `A`,
`AAAA`, `CNAME`, `TXT`. Reverse records are automatically created for `A` and `AAAA` if needed.

Resource records for the domain itself may be set as a list with the `domain_rr` key.
Attention - the domain name is not automatically added, the string is taken as is!

##### Example for simple domain:
````yml
unbound_domain:
  domain_name: example.net
  domain_rr:
    - "MX 10 server1.example.net."
    - "IN A 1.2.3.5"
  www: "1.2.3.4"
  server1: "IN A 1.2.3.5"
  admin-contact: 'IN TXT "ask your neighbour"'
````

Generated unbound configuration:
````yml
    local-zone: "example.net." static
    local-data: "example.net. MX 10 server1.example.net."
    local-data: "example.net. IN A 1.2.3.5"
    local-data: "www.example.net. 1.2.3.4"
    local-data: "server1.example.net. IN A 1.2.3.5"
    local-data: 'admin-contact.example.net. IN TXT "ask your neighbour"'
````

##### Example for complex domain:

All fields (ip, ipv6, cnames, txt, reverse) are optional, only the attributes needed
should be set

````yml
unbound_domain:
  domain_name: example.net
  www:
    ip: "1.2.3.4"
    reverse: true
  server1:
    ip: "1.2.3.5"
    ipv6: "fe80::7"
    cnames:
      - mail
      - imap
      - smtp
    reverse: true
  admin-contact:
    txt: "ask your neighbour"
````

Generated unbound configuration:
````yml
    local-zone: "example.net." static
    local-data: "www.example.net. 1.2.3.4"
    local-data-ptr: "1.2.3.4 www.example.net."
    local-data: "server1.example.net. IN A  1.2.3.5"
    local-data: "server1.example.net. IN AAAA fe80::7"
    local-data: "mail.example.net. IN CNAME server1.example.net."
    local-data: "imap.example.net. IN CNAME server1.example.net."
    local-data: "smtp.example.net. IN CNAME server1.example.net."
    local-data-ptr: "1.2.3.5 server1.example.net."
    local-data-ptr: "fe80::7 server1.example.net."
    local-data: 'admin-contact.example.net. IN TXT "ask your neighbour"'
````

##### Example for mixed domain with both versions:
````yml
unbound_domain:
  domain_name: example.net
  www: "1.2.3.4"
  server1:
    ip: "1.2.3.5"
    ipv6: "fe80::7"
    cnames:
      - mail
      - imap
      - smtp
    reverse: true
````

Generated unbond configuration:
````yml
    local-zone: "example.net." static
    local-data: "www.example.net. 1.2.3.4"
    local-data: "server1.example.net. IN A  1.2.3.5"
    local-data: "server1.example.net. IN AAAA fe80::7"
    local-data: "mail.example.net. IN CNAME server1.example.net."
    local-data: "imap.example.net. IN CNAME server1.example.net."
    local-data: "smtp.example.net. IN CNAME server1.example.net."
    local-data-ptr: "1.2.3.5 server1.example.net."
    local-data-ptr: "fe80::7 server1.example.net."
````
