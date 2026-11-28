# IP.MD File Format Specifications (v0.1)

## Overview
Inside a project, each device is described once in its own `.md` file. IP.MD will draw different views from the same data.

| View | Display |  Definitions |
| :---: | :---: | :---: |
| Physical | Racks, devices by rack unit, cables between physical ports | folders, `rack_unit`, `links`  |
| Logical | Network/VLAN zones, devices and their services, allowed traffic between zones   | `networks`, `interfaces`, `services`, `rules`  |

Commands: `ipmd watch homelab --view physical`/`--view /logical`

## Project Layout 
```
homelab/                    # project root folder
|――ipmd.yaml                # project settings & network definitions (i.e. VLANs)
|――rack1/                   # top level folder defines racks
|   |――_rack.md             # _index.md is the rack describer
|   |――router.md            # .md files are device definitions
|   |――switch.md        
|   |――proxmox1.md
|   |――proxmox1/            # folders with the same name as a device define its insides
|       |――vm-ntp.md        # .md files inside a device folder defines virtual machines or containers
|       |――vm-dns.md        
|――_closet/                 # _ defines a special folder for unracked devices like modems, clients, cloud, etc.
|   |――_index.md            # _index.md is the group describer
|   |――isp-modem.md    
|――_clients/
|   |――laptop.md
```

**Layout Specifications**
1. Top level folders are racks. Folder names starting with `_` are for unracked devices.
2. Each `.md` file is one device. It's name without `.md` is its ID.
3. A folder named after a device id holds any VMs or containers. Any file inside the folder implies it runs on the device ID. (i.e. `proxmox1.md` and `proxmox1/`).
4. A file named `_index.md` describes the folder it is placed in (rack or unracked group). It does not represent a device.
5. All ID's must be unique.
6. `README.md` and any non-`.md` file is ignored.

## Networks (`ipmd.yaml`)
Networks are defined in `ipmd.yaml`. They are referenced by their `id`. 

```
name: Homelab
networks:
    - id: mgmt
      name: Management
      vlan: 10
      subnet: 10.0.10.0/24
    - id: servers
      vlan: 20
      subnet: 10.0.20.0
    - id: iot
      vlan: 30
      subnet: 10.0.30.0/24
    - id: wan
      name: Internet
```   

| Fields | Required |  View | Definition |
| :---: | :---: | :---: | :---: |
| `id` | yes | logical | Referenced by devices and rules |
| `name` | no | logical | Display name |
| `vlan` | no | logical | Vlan tag |
| `subnet` | no | logical | CIDR subnet |
| `color` | no | logical | Zone color in diagram |

## Device file
Markdown with a YAML frontmatter notated by `---`. Body is for free notes and will have future additions.

```
---
type: server
name: Proxmox 01
rack_unit: 10
height: 2
interfaces:
  - name: eno1
    network: servers
    ip: 10.0.20.11
  - name: ipmi
    network: mgmt
    ip: 10.0.0.11
links:
  - to: switch
    port: eno1
    to_port: port2
    label: 10G
services:
  - name: proxmox-ui
    port: 8006
    protocol: https
---
Primary Proxmox hypervisor
```

| Fields | Required |  View | Definition |
| :---: | :---: | :---: | :---: |
| `type` | yes | both | `router`, `firewall`, `switch`, `server`, `nas`, `ap`, `vm`, `container`, `pdu`, `patch-panel`, `other` |
| `name` | no | both | Display name. Defaults as the `id` |
| `id` | no | both | Overrides the default file-name `id` |
| `rack_unit` | no | physical | Occupying unit on the rack from the bottom U position |
| `height` | no | physical | Height in rack units |
| `links` | no | physical | Cables to other devices | 
| `interfaces` | no | logical | Networks that the device sits on | 
| `ip` | no | logical | ip address for the interface | 
| `services` | no | logical | What the device is running | 
| `rules` | no | logical | traffic rules, placed typically on router/firewall | 
| `tags` | no | both | Free labels |

Devices can also be written in shorthand if it does not use multiple interfaces:
```
ip: 10.0.20.10
network: servers
```

 ## Links (physical connections)

 Write each link once on either end, IP.MD will treat it as two way.

```
links:
  - to: switch          # required: id of the other device
    port: eno1          # optional: port on THIS device
    to_port: port2      # optional: port on the OTHER device
    label: 10G          # optional: text on the line
    type: ethernet      # optional: ethernet (default), fiber, wifi, vpn
    vlans: [10, 20]     # optional: VLANs carried (trunk). Shown as a label in physical view.
```

## Interfaces

```
interfaces:
  - name: eno1          # optional label
    network: servers    # required: network id from ipmd.yaml
    ip: 10.0.20.11      # optional; "dhcp" allowed
```
Shorthand: `links: [switch, router]`

## Services
A service is defined as something running on a device (DNS, NTP, web apps, etc.)

#### Small Services
Small services can be written inline on a host's frontmatter. 

```
services:
    - name: pihole      # unique service name to the device
      app: dns          # common services like dns, http, https, smb, nfs, ssh and so on
      port: 53          # number, or list [80, 443]
      protocol: udp     # tcp or udp
      url: http://10.0.20.10/admin
```

#### Big services
Large services can be written in their own `.md` file, defined with `type: service` inside a host folder. 

```
|――proxmox1/
|   |――vm-media.md    # md file for a VM
|   |――vm-media/      # folder for the VM
|       |――plex.md    # service file
```

```
---
type: service
app: http
port: 32400                   # not http's usual 80, so it's written out for override
depends_on: [nas/smb, traefik] # for services from other devices do <device_id>/<service_name>. [ ] for lists
---
Plex server for streaming
```

#### Fields for Services
| Fields | Required |  View | Definition |
| :---: | :---: | :---: | :---: |
| `name` | inline: yes / file: no | logical | Unique to the host. Defaults to file name |
| `port` | no | logical | Number or list: `[80, 443]` |
| `app` | no | logical | Autofills port and protocol, and sets a label | 
| `protocol` | no | logical | `tcp` or `udp` |
| `url` | no | logical | Preview link | 
| `expose` | no | logical | Network the service is reachable from, like `wan` for internet exposed applications | 
| `deponds_on` | no | logical | Services required for this service |


#### Default services
Well-known default services that IP.MD recognizes.

| `app` | Port |  Protocol |
| :---: | :---: | :---: |
| `http` | 80 | tcp |
| `https` | 443 | tcp |
| `dns` | 53 | tcp + udp |
| `ssh` | 22 | tcp |
| `smb` | 445 | tcp |
| `dhcp` | 67 | udp | 
| `nfs` | 2049 | tcp + udp |


## Traffic Rules
Rules describe allowed traffic, usually written on the router/firewall device that enforces them. 

```
rules:
  - from: iot               # network id, device id, or device/service
    to: vm-dns/pihole
    port: 53
    protocol: udp
    action: allow           # allow (default) | deny
    note: IoT may only use DNS
  - from: servers
    to: wan
    action: allow
```

 - Rules are all one way
 - Deny rules override allow rules in traffic descriptions
 - Exclusion of `port` or `protocol` implies any
 - Exclusion of `action` implies allow

| Fields | Required |  View | Definition |
| :---: | :---: | :---: | :---: |
| `from` | yes | logical | From traffic |
| `to` | yes | logical | To traffic |
| `port` | no | logical | Traffic port | 
| `protocol` | no | logical | Traffic protocol `tcp` or `udp`
| `action` | no | logical | `allow` or `deny` | 
| `app` | no | logical | IP.MD default service |
| `note` | no | logical | Label/note for rule | 

## `_index.md`
#### Rack
```
---
name: Main Rack
height: 42          #total U
location: Storage Room
---
```

#### Group (Un-racked)
```
---
name: Network closet
location: Network Room
---
```

## Scope
- v0.1: Parse everything (logical and physical elements), render physical view
- v0.2: Render logical view