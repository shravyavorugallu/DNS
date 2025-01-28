
# DNS Lab Project: Setting Up Primary and Secondary DNS Servers

## Overview
In this lab project, you will configure two DNS servers—R1 as the primary DNS server and R2 as the secondary DNS server—using BIND9. The goal is to enable DNS resolution within the network, allowing each machine to be reachable by its hostname (e.g., `ping Kali`). The primary zone, "cn.", will be hosted on R1, while the secondary zone, "second.cn.", will be hosted on R2. This configuration will allow seamless name resolution between machines in the network.

## Requirements
- **R1**: Primary DNS Server (Hosts primary zone "cn.")
- **R2**: Secondary DNS Server (Hosts secondary zone "second.cn.")
- **Machines in Area 0**: R1, R2, Kali
- **Machines in Area 1**: R2, R3, R4, Ubuntu
- **BIND9** installed on R1 and R2
- **Network Connectivity**: Ensure that all machines are connected to the network and can communicate with each other.

## Configuration Steps

### Part 1: Setup DNS Resolution
1. On each machine in Area 0, modify the `/etc/resolv.conf` file to include the following configuration:
   ```bash
   nameserver <eth1 interface address of R1>
   domain <primary zone name>
   search <primary zone name>
   ```

### Part 2: Configuring the Primary Zone on R1
1. On R1, configure the primary DNS zone by editing `/etc/bind/named.conf.local` to include both forward and reverse zone definitions:
   ```bash
   zone "cn." {
       type master;
       file "/etc/bind/db.cn";
   };
   
   zone "10.10.10." {
       type master;
       file "/etc/bind/db.10.10.10";
   };
   ```

2. Copy the template for the forward zone file:
   ```bash
   sudo cp /etc/bind/db.local /etc/bind/db.cn
   ```
3. Edit the forward zone file `/etc/bind/db.cn` to add A records for R1, R2, and Kali.

4. Create the reverse zone file `/etc/bind/db.10.10.10` and add PTR records corresponding to the A records in the forward zone.

5. Restart the BIND9 service to apply the configuration:
   ```bash
   sudo systemctl restart bind9.service
   ```

### Part 3: Configuring the Secondary Zone on R2
1. On R2, configure the secondary DNS zone by editing `/etc/bind/named.conf.local` to include the secondary zone definition:
   ```bash
   zone "second.cn." {
       type slave;
       file "/etc/bind/db.second.cn";
       masters { <R1's IP address>; };
   };
   ```

2. Edit the forward zone file `/etc/bind/db.second.cn` to include A records for R2, R3, R4, and Ubuntu.

3. Create the reverse zone file `/etc/bind/db.10.10.20` and add PTR records corresponding to the A records in the forward zone.

4. Add a NS record in the primary zone file (`/etc/bind/db.cn`) to link the secondary zone:
   ```bash
   @ IN NS <R2's IP address>;
   ```

5. Restart the BIND9 service on R2:
   ```bash
   sudo systemctl restart bind9.service
   ```

## Verification
- After configuring both primary and secondary DNS zones, you should be able to ping machines by their hostnames.
- Verify that you can ping the following machines from each respective area:
  - **From R1**: Ping R2 and Kali.
  - **From R2**: Ping R3, R4, and Ubuntu.

This lab project simulates a real-world DNS configuration for a network. By setting up both primary and secondary DNS servers, you ensure fault tolerance and redundancy in DNS resolution. The primary DNS server handles the main zone, while the secondary DNS server ensures continued name resolution in case the primary server becomes unavailable. This setup is critical for maintaining reliable network communication in larger environments, such as corporate or data center networks.

## Additional Resources
- [BIND9 Documentation](https://www.isc.org/bind/)
