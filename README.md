ansible-role-unbound
====================

Ansible role for Unbound DNS Server and resolver


# Supports
- Add DNS entries
- Generation of DNS entries from ansible inventory (A entries and reverse)
- Forward to another dns
- IPv4 only for reverse

# Information :
- Test on Ubuntu
- Untested on debian and fedora

# Example :

## Simple forward on localhost :
```
# Activate forward (activate by default)
unbound_forward_zone_active : true
# Forward server to google DNS (activate by default)
unbound_forward_zone:
   - 8.8.8.8 #Google DNS 1
   - 8.8.4.4 #Google DNS 2
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

# Create entries from inventory (reverse  also created by default)
unbound_inventory_domain:
    all: 'internal.domain' # All hosts

# Create reverse entries from inventory
unbound_inventory_reverse_domain:
    all: 'internal.domain' # All hosts

# Activate forward (activate by default)
unbound_forward_zone_active : true
# Forward server to google DNS (activate by default)
unbound_forward_zone:
   - 8.8.8.8 #Google DNS 1
   - 8.8.4.4 #Google DNS 2

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
