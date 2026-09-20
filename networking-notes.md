# Day 2 — Networking Fundamentals

These notes cover IP addresses, ports, TCP, UDP, and DNS.
My goal is to understand how devices communicate and how these
concepts help with SOC investigations.

## 1. IP Addresses

An IP address is an address used to send traffic to a device’s
network interface.

Example: 192.168.1.10

A laptop can have different IP addresses for Wi-Fi and Ethernet.
An IP address can change, so it does not permanently identify
a device or person.

### Private IP
A private IP is used inside a home or company network.

Private IPv4 ranges:
- 10.0.0.0 to 10.255.255.255
- 172.16.0.0 to 172.31.255.255
- 192.168.0.0 to 192.168.255.255

Different homes can use the same private addresses because
their networks are separate.

### Public IP
A public IP is used for communication over the internet.

Several devices in a home commonly share one public IPv4
address through the router.

### NAT
NAT stands for Network Address Translation.

A home router commonly translates private addresses and ports
so multiple devices can share a public IPv4 address. It tracks
the connections to send replies back to the correct device.

Memory tip:
Private = inside the network.
Public = internet-facing address.

## 2. Subnet Mask

A subnet mask helps a device determine whether a destination
is on its local network.

Example:
IP address: 192.168.1.10
Subnet mask: 255.255.255.0
This can also be written as 192.168.1.10/24.

With this mask:
- 192.168.1.20 is on the same subnet.
- 192.168.2.20 is on a different subnet.

Memory tip:
Subnet mask = tells me which addresses are local.

## 3. Default Gateway

The default gateway is usually the router.

My laptop uses it to reach destinations outside its local
network when there is no more specific route.

Example:
Laptop: 192.168.1.10
Gateway: 192.168.1.1

Memory tip:
Gateway = the way out of my local network.

## 4. DHCP

DHCP automatically gives devices their network settings.

These settings commonly include:
- IP address
- Subnet mask
- Default gateway
- DNS servers

At home, the router often provides DHCP.

Memory tip:
DHCP gives me network settings.
DNS helps me look up names.

## 5. Ports

An IP address helps traffic reach the correct device.
A port helps traffic reach the correct application or service.

Think of:
IP address = building address.
Port = department inside the building.

Example: 192.168.1.20:443
- 192.168.1.20 is the IP address.
- 443 is the port.

TCP and UDP port numbers range from 0 to 65535.
TCP and UDP have separate port spaces.

### Source and Destination Ports

When my browser connects to an HTTPS website:
- My laptop normally uses a temporary source port.
- The website normally uses destination port 443.

Example:
Laptop:51524 → Website:443

Replies return from Website:443 to Laptop:51524.

### Ports to Memorize

22   — SSH   — Secure remote command-line access
53   — DNS   — Domain record lookups
80   — HTTP  — Unencrypted web traffic
443  — HTTPS — Encrypted web traffic
3389 — RDP   — Remote desktop access

These are common default ports. Services can run on other ports.

A port number is a clue, not proof of the application or whether
traffic is safe.

## 6. TCP

TCP stands for Transmission Control Protocol.

TCP establishes a connection and provides reliable, ordered
delivery of data to an application.

If data is lost, TCP can retransmit it.
If data arrives out of order, TCP puts it in order before
delivering it to the application.

### TCP Three-Way Handshake

1. SYN: The client requests a connection.
2. SYN-ACK: The server acknowledges and responds.
3. ACK: The client acknowledges the response.

The connection is then established.

Common uses include SSH and web traffic using HTTP/1.1 or HTTP/2.

TCP does not encrypt data by itself.
HTTPS uses TLS to protect the web traffic.

Memory tip:
TCP = connection, tracking, and ordered delivery.

## 7. UDP

UDP stands for User Datagram Protocol.

UDP sends individual messages called datagrams without a
TCP-style connection handshake.

UDP itself does not:
- Guarantee delivery.
- Retransmit lost data.
- Guarantee the order of arrival.

Applications can add these features if they need them.

Common uses include:
- Many DNS queries
- Voice calls
- Online games

UDP has less built-in transport overhead, but it is not always
faster in every situation.

HTTP/3 uses QUIC over UDP. QUIC adds encryption and reliable
delivery of streams.

Memory tip:
UDP = sends datagrams without built-in delivery guarantees.

## 8. DNS

DNS stands for Domain Name System.

DNS lets my computer look up records for domain names.
One common purpose is finding a website’s IP addresses.

Example:
I enter example.com.
My computer looks up its address.
My browser uses the address to connect to the website.

### Basic Lookup Process

1. My computer checks for a cached answer.
2. If needed, it asks a DNS resolver.
3. The resolver answers from its cache or finds the answer.
4. My computer receives the requested records.

A cached answer is stored temporarily to avoid repeated lookups.
TTL controls how long a DNS record can normally be cached.

### Common DNS Records

A     — IPv4 address
AAAA  — IPv6 address
CNAME — Alias for another domain name
MX    — Mail server information
TXT   — Text information, often for verification or email policies

Traditional DNS uses both UDP and TCP port 53.
Many ordinary queries use UDP.

Memory tip:
DNS = look up a name to find its records.

## 9. What Happens When I Open an HTTPS Website?

For a typical HTTPS connection using TCP:

1. My laptop has its network settings.
2. DNS provides the website’s IP address, unless already cached.
3. My laptop sends internet-bound traffic through its gateway.
4. My browser establishes a TCP connection to port 443.
5. TLS protects the connection with encryption.
6. My browser requests the page.
7. The server sends the content back.

My router commonly performs NAT for this IPv4 traffic.

HTTP/3 uses QUIC over UDP instead of TCP.

## 10. Windows Commands

### ipconfig
Shows my network configuration.

Look for:
- IPv4 address
- Subnet mask
- Default gateway

### nslookup example.com
Looks up DNS information for example.com.

The DNS server shown at the top is the resolver answering.
The returned addresses below belong to the domain being queried.

“Non-authoritative answer” is normally not an error.

### netstat -an
Shows connections and listening endpoints using numeric
addresses and ports.

Important TCP states:
- LISTENING: Waiting for connection requests.
- ESTABLISHED: A connection has been established.
- TIME_WAIT: Temporarily retaining state after a connection closes.

A listening port is not automatically accessible from the internet.
Firewalls and router settings also affect access.

UDP entries do not have TCP connection states.

## 11. Why This Matters in a SOC

When reviewing network activity, I should ask:
- Which IP address started the communication?
- Which IP address received it?
- What were the source and destination ports?
- Was TCP or UDP used?
- Was the connection successful?
- Is this activity expected for this device?

Examples:
- Port 3389 may indicate Remote Desktop activity.
- Port 22 may indicate SSH access.
- DNS logs may show domains a device tried to resolve.
- Port 443 traffic can be legitimate or malicious.

One IP address, port, or DNS query is not enough to prove an attack.

ToBeRemember

IP address: Where should the traffic go?
Port: Which application or service should receive it?
Subnet mask: Is the destination local?
Gateway: Where do I send traffic for other networks?
DHCP: How does my device get network settings?
TCP: Reliable, ordered transport.
UDP: Datagrams without built-in delivery guarantees.
DNS: Look up domain records.
TLS: Protect communication with encryption.

Ports:
22 = SSH
53 = DNS
80 = HTTP
443 = HTTPS
3389 = RDP
