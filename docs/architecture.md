
# Current Infrastructure Overview

## Security Architecture

- **Primary Security Model**: CloudFlare Zero Trust with deny-all posture
- **Access Control**: All external access routed through CloudFlare Zero Trust Tunnels
- **DNS Strategy**: CloudFlare DNS with Zero Trust App Launcher
- **Network Isolation**: No direct internet exposure of services

## Core Infrastructure Components

### Mokena LAN
- Corporate network
- Windows Active Directory Domain 
  - Two physical Hyper-Visor servers running all domain controller and file serving roles
- Gateway Public IP: 50.171.29.67
- Gateway LAN IP: 10.26.0.1 
- LAN: 10.26.0.0/24
- VPN access via Fortinet

#### ProxMox based Molabs infrastructure
- **Hardware**: AMD Threadripper, 128GB DDR4 ECC, 2x8TB HDD + 4x960GB NVMe SSD in RAID 10
- **Network**: 1Gbit Intel I210 NIC
- **Hostname**: pm2
- **Purpose**: ProxMox virtualization host running multiple VMs and containers
- **Notes**: Uses zfs to enable easier expansion, resizing, and faster backups
- **Network Configuration**:
  - **Host Network**: 10.26.0.250
  - **Internal SDN**: ProxMox SDN "dhcpsnat" zone
  - **VM Network**: 10.51.0.0/24 (dhcpsnat-wan0 vnet)
  - **Gateway**: 10.51.0.1 (ProxMox host serves as gateway)
  - **DNS Hierarchy**: 
    1. 10.51.0.1 (local resolution for VM intercommunication)
    2. 1.1.1.1 (CloudFlare DNS)
    3. 8.8.8.8 (Google DNS)

### Molabs Cloud
- Cloud based infrastructure
- ProxMox virtualization host installed on bare-metal server
- Public IP: 65.108.120.62/32
- Provider: Hetzner dedicated server

#### ProxMox Host (pm1)
- **Hardware**: Intel Xeon W-2295, 128GB DDR4 ECC, 2x16TB HDD + 2x960GB NVMe SSD
- **Network**: 1Gbit Intel I210 NIC
- **Hostname**: pm1
- **Purpose**: ProxMox virtualization host running multiple VMs and containers
- **Notes**: Uses zfs to enable easier expansion, resizing, and faster backups
- **Network Configuration**:
  - **Host Network**: 65.108.120.62 (Hetzner public IP)
  - **Internal SDN**: ProxMox SDN "dhcpsnat" zone
  - **VM Network**: 10.50.0.0/24 (dhcpsnat-wan0 vnet)
  - **Gateway**: 10.50.0.1 (ProxMox host serves as gateway)
  - **DNS Hierarchy**: 
    1. 10.50.0.1 (local resolution for VM intercommunication)
    2. 185.12.64.1 (Hetzner nameserver)  
    3. 1.1.1.1 (CloudFlare DNS)
    4. 8.8.8.8 (Google DNS)
  - **Firewall Rules**: 
    - **Incoming (Restrictive by Design)**:
      - ROB: IPv4 | * | 76.16.186.2/32 | NULL | NULL | NULL | Accept (Personal IP whitelist)
      - ICMP: IPv4 | icmp | NULL | NULL | NULL | NULL | Accept
      - TCP EST: IPv4 | tcp | NULL | NULL | 32768-65535 | ack | Accept
      - UDP EST: IPv4 | udp | NULL | NULL | 53 | NULL | Accept
      - NTP: IPv4 | udp | NULL | NULL | 123 | 1024-65535 | Accept
    - **Outgoing**:
      - Block mail: IPv4 | tcp | NULL | NULL | NULL | NULL | Discard
      - Allow all: * | NULL | NULL | NULL | NULL | NULL | Accept

## Tunnel Architecture

### Tunnel 1: pm1-tunnel
- **Name**: pm1-tunnel
- **Type**: Cloudflared
- **Purpose**: Direct access to ProxMox host management
- **Tunnel Host**: pm1 (10.50.0.1)
- **Routes**: None
- **Public Hostnames**:
  - **pm1.molabs.cloud**
    - Subdomain: pm1
    - Domain: molabs.cloud
    - Path: NULL
    - Service Type: https
    - URL: 10.50.0.1:8006
    - Purpose: Access to ProxMox web admin 
  - **pm1-ssh.molabs.cloud**
    - Subdomain: pm1-ssh
    - Domain: molabs.cloud
    - Path: NULL
    - Service Type: ssh
    - URL: 10.50.0.1:22
    - Purpose: SSH access to ProxMox host
  - **pbs1.molabs.cloud**
    - Subdomain: pbs1
    - Domain: molabs.cloud
    - Path: NULL
    - Service Type: https
    - URL: 10.50.0.1:8007
    - Purpose: Access to ProxMox Backup Server web admin

**Important**: https services on ispc1 use self-signed certificates and as a result, all public hostnames for https have tls ignore enabled in their Public Hostname settings. 

**Status**: Running as service on pm1, CloudFlare status healthy with days of uptime

### Tunnel 2: molabs-cloud-tunnel 

- **Name**: molabs-cloud-tunnel
- **Type**: Cloudflared
- **Purpose**: Access to VM-hosted services on 10.50.0.0/24 network
- **Tunnel Host**: cft-mocloud1 VM (10.50.0.200)
- **Routes**: 
  - 10.50.0.8/32
- **Public Hostnames**:
  - **test-website.molabs.tools**
    - Subdomain: test-website
    - Domain: molabs.tools
    - Path: NULL
    - Service Type: https
    - URL: 10.50.0.90
    - TLS ignore error: ENABLED
    - Target Host: ispc1.local.lan
    - Purpose: Serves vhost file test-website.molabs.tools defined on ISPConfig
    - Status: Properly serving site
  - **test-website2.molabs.tools**
    - Subdomain: test-website2
    - Domain: molabs.tools
    - Path: NULL
    - Service Type: https
    - URL: 10.50.0.90
    - TLS ignore error: ENABLED
    - Target Host: ispc1.local.lan
    - Purpose: Serves vhost file test-website2.molabs.tools defined on ISPConfig
    - Status: Properly serving site
  - **n8n.molabs.cloud**
    - Subdomain: n8n
    - Domain: molabs.cloud
    - Path: NULL
    - Service Type: https
    - URL: 10.50.0.50
    - TLS ignore error: ENABLED
    - Target Host: n8n1.local.lan
    - Purpose: Serves n8n
    - Status: Properly serving site

**Status**: CloudFlare status healthy with days of uptime

### Tunnel 3: moco-mokena

- **Name**: moco-mokena
- **Type**: Cloudflared
- **Purpose**: Access to PM2 hosted VMs on 10.51.0.0/24 network
- **Tunnel Host**: pm2 (10.51.0.1)
- **Routes**: 
  - 10.51.0.8/32
- **Public Hostnames**:
  - **n8n.molabs.cloud**
    - Subdomain: n8n
    - Domain: molabs.cloud
    - Path: NULL
    - Service Type: https
    - URL: 10.50.0.50
    - TLS ignore error: ENABLED
    - Target Host: n8n1.local.lan
    - Purpose: Serves n8n
    - Status: Properly serving site

**Status**: CloudFlare status healthy with days of uptime

### Tunnel 4: wireshark

- **Name**: pg-wireshark
- **Type**: Wireshark point-to-point
- **Purpose**: Allows Postgres replication between PG1 and PG2 
- **Tunnel Host**: 
  - PG1 (Postgres replica)
    - SNAT IP: 10.50.0.8
    - Wireshark IP: 10.100.0.2
  - PG2 (Postgres primary)
    - SNAT IP: 10.51.0.8
    - Wireshark IP: 10.100.0.1

**Status**: Healthy with days of uptime

## Production VMs and Services

### cft-mocloud1 (CloudFlare Tunnel Host)
- **ProxMox Host**: pm1
- **IP**: 10.50.0.200
- **Purpose**: Dedicated CloudFlare tunnel daemon host for both tunnels
- **Search Domain**: molabs.lan
- **DNS Config**: `/etc/systemd/resolved.conf.d/molabs.conf`
  ```
  [Resolve]
  Domains=local.lan
  DNSStubListener=no
  ```
- **CloudFlared Service**: Configured for auto-start, always connected
- **Status**: VM functional with internet access, tunnel connections healthy

### ispc1.local.lan (ISPConfig Web Hosting)
- **ProxMox Host**: pm1
- **IP**: 10.50.0.90/24
- **Hostname**: ispc1.local.lan (FQDN required for ISPConfig installation)
- **Purpose**: Multi-tenant web hosting with ISPConfig 3.2 control panel
- **DNS Config**: `/etc/systemd/resolved.conf.d/molabs.conf`
  ```
  [Resolve]
  Domains=local.lan
  DNSStubListener=no
  ```
- **Features**: 
- HTTP→HTTPS redirects managed through ISPConfig
- LetsEncrypt SSL (currently buggy, using self-signed certificates)
- Multiple vhost configurations
- Serves multiple sites through CloudFlare tunnel exposure
- **Search Domain**: molabs.lan

### n8n1 (Workflow Automation)
- **ProxMox Host**: pm1
- **IP**: 10.50.0.50/24
- **Hostname**: n8n1
- **Purpose**: n8n workflow automation platform
- **Architecture**: Docker Swarm (molabs-n8n-swarm) with a main n8n instance and 2 workers
- **Dependencies**: PostgreSQL and Redis containers
- **Status**: Not yet exposed through CloudFlare tunnels

# Common Tasks & Troubleshooting Areas

## Infrastructure Expansion

1. **New VM Deployment**: Guide through ProxMox VM creation with proper network assignment to (10.50.0.0/24 for PM1, 10.51.0.0/24 for PM2)
2. **Service Exposure**: Configure new CloudFlare tunnel public hostnames with proper service types and TLS settings
3. **Network Routing**: Ensure proper connectivity between VMs and external access through tunnel architecture

## CloudFlare Tunnel Management

1. **Public Hostname Configuration**: Add new services with proper subdomain | domain | path | service type | URL format
2. **TLS Configuration**: Manage TLS ignore error settings for self-signed certificates
3. **Private Network Exposure**: Configure which internal networks are accessible through each tunnel
4. **Service Type Optimization**: Choose appropriate service types (https, ssh, tcp, etc.) for different applications

## Security & Access Control

1. **Zero Trust Policy Configuration**: Modify CloudFlare Access policies for new exposed services
2. **Firewall Rule Updates**: Coordinate Hetzner firewall with service requirements  
3. **DNS Resolution**: Troubleshoot internal hostname resolution between VMs using local.lan domain
4. **Tunnel Health**: Monitor and debug CloudFlare tunnel connectivity and uptime

## Performance & Optimization

1. **Resource Allocation**: Monitor ProxMox resource usage and optimize VM allocation
2. **Network Performance**: Analyze SDN performance and routing efficiency within 10.50.0.0/24 for PM1 SNAT or 10.51.0.0/24 for PM2. 
3. **Storage Management**: Balance workloads across HDD and NVMe storage tiers
4. **Service Scaling**: Plan horizontal scaling strategies within current tunnel architecture

# Key Architecture Principles

## Security-First Design

- No services directly exposed to the internet
- All external access mediated through CloudFlare Zero Trust tunnels and Public Hostnames
- Layered firewall approach (Hetzner + CloudFlare + application-level)
- Principle of least privilege in network access
- TLS ignore error only used for known self-signed certificate scenarios

## CloudFlare Tunnel Strategy

- **Separation of Concerns**: molabs-pm1-tunnel for infrastructure management, molabs-cloud-tunnel for application services
- **Private Network Exposure**: Only expose the 10.50.0.0/24 private network over Cloudflare tunnels
- **Dedicated Tunnel Host**: cft-mocloud1 serves as single point for tunnel daemon management
- **Service Type Granularity**: Specific service types (https, ssh) configured per exposed service

## Operational Resilience  

- Single ProxMox host with robust hardware specifications
- Centralized tunnel management through dedicated VM (cft-mocloud1)
- Standardized VM networking configuration with consistent DNS search domains
- Consistent domain naming conventions (molabs.cloud for infrastructure, molabs.tools for applications)

## Scalability Considerations

- Current architecture supports horizontal VM scaling within 10.50.0.0/24
- CloudFlare tunnel capacity for additional service exposure through public hostname additions
- Network capacity within dhcpsnat SDN zone
- Storage tier optimization (NVMe for high-IOPS, HDD for bulk storage)

# Troubleshooting Decision Trees

When addressing issues, follow this priority:
1. **Security Impact**: Ensure no security boundaries are compromised
2. **Service Availability**: Maintain uptime for production services accessible through tunnels
3. **Performance**: Optimize without affecting tunnel stability
4. **Future Scalability**: Consider long-term architectural impact on tunnel and network capacity

# Domain and Naming Conventions

- **Infrastructure Domain**: molabs.cloud (ProxMox management, SSH access)
- **Application Domain**: molabs.tools (web services, applications)
- **Internal Domains**: local.lan, molabs.lan
- **VM Naming**: Descriptive hostnames with service purpose (cft-mocloud1, ispc1, n8n1)
- **Tunnel Naming**: molabs-[purpose]-tunnel format
- **Public Hostname Format**: [service]-[descriptor].[domain] (test-website.molabs.tools)

# Critical Configuration Details

- **TLS Ignore Error on Cloudflare Public Hostnames**: Required for self-signed certificates on internal services
- **DNS Search Domains**: Configured via systemd-resolved for proper internal hostname resolution
- **Service Types**: Match CloudFlare tunnel capabilities (https, ssh, tcp, etc.)
- **Private Network Routing**: Only molabs-cloud-tunnel provides access to 10.50.0.0/24 subnet

This infrastructure represents a production environment with multiple active services and external dependencies. All recommendations should prioritize stability and security while enabling growth and optimization within the established CloudFlare Zero Trust tunnel architecture.RetryClaude can make mistakes. Please double-check responses.