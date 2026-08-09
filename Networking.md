Setting Static IPs:
Once both VM's were running I captured both of their Ip's by using "ip address", once I did that I could test if they could see eachother by pinging one IP from the other.
I noticed both VM's shared the same IP address, this most likely being due to using the same iso image to install linux, to fix this I will use "ls /etc/netplan/ to navigate to my netplan config file and 
manually set a static ip for the client device. In future I'd like to create a simple DHCP system but for now that would be overkill.
I then used nano to edit the config file and simply set the ip to 10.10.10.2/24 and 10.10.10.3/24 respectively.

Setting up SSH:
Need to use another network adapter set to "Nat" to acess the internet so I can install and update packages.
once booted my netplan config didn't include the Nat adapter so I had to add the line myself, after that I could install the openssh-server package with ease.
I then ran "systemctl enable ssh --now" to make sure it runs on boot so I don't need to boot it every time myself.
once running I used "ssh-copy-id <user>@10.10.10.3" to send the clients public key to the server to be copied into the ssh file making so if I ever need "remote" control of the server I can just run "ssh <user>@10.10.10.3". After that I changed the "PasswordAuthentication" from yes to no in "sshd_config"
to bypass any need to use the system password when trying to connect.


Network Routing:
learnt the usage of the "ip route" and "ip neigh" commands, route seems to give me the kernal's routing table which shows which route, interface, or gateway Linux will use to reach different destination networks. while neigh shows Linux's neighbour table, which contains information about nearby network endpoints, including IP to MAC address mappings. For IPv4, Linux can learn these mappings through ARP. I can use this information to more accurately identify the best way to connect and communicate with other devices throughout various networks. Using "ip route 8.8.8.8" returns "8.8.8.8 via 10.0.3.2 dev enp0s8 src 10.0.3.15 uid 1000" so for 8.8.8.8 it would use the NAT interface and route from my 10.0.3.15 ip through the 10.0.3.2 gateway onwards to 8.8.8.8. Then for "ip route get 10.10.10.2 (client) it returns "10.10.10.2 dev enp0s3 src 10.10.10.3 uid 1000" which is just my Interface Network ip going directly through the interface to the client. "src" is the source ip, "dev" is the interface to use, and "via" is just the gateway. The server does not need to send the packet to a router/gateway first as it can communicate directly over the 10.10.10.0/24 network. I noticed the client ip wasn't showing and neither was the interface used to communicate between client and server, this would have been because they hadn't communicated in a while so it "forgot" the clients details to save resources. By simply pinging the client from the server and then running "ip neigh show dev enp0s3" it shows that the server now stored the connection and identifier (MAC) for the client. This is IPv4 ARP in action, when the server pings 10.10.10.2 it checks the routing table and sees 10.10.10.0/24 is accesable over enp0s3. It then asks the devices on the network who the ip belongs to and the client responds with the MAC address corresponding to its IP, then the server saves it to the neigh table.

CIDR and Subnetting:
an IP is made of 32 bits. Every normal IPv4 address is associated with a network or prefix. CIDR is telling the host where the network/host boundary is. so you'd take the 32 bits and take away 24 of them for the network so your host has 8 bits meaning the host has 2 to the power of 8 possible bit combinations which would be 256, other than the dedicated .0 for network and .255 for broadcast. so if you use /25 instead we allocate one more of the 32 bits to the network which creates a subnet mask which is pretty much splitting one network into multiple. if we use /26 for example; 32 - 26 leaves 6 host bits instead meaning the host only has 62 usable host addresses.

CIDR     Addresses/subnet     Usable hosts*
------------------------------------------------
/24            256                  254
/25            128                  126
/26             64                   62
/27             32                   30
/28             16                   14
/29              8                    6
/30              4                    2


TCP, UDP, and Ports:
0.0.0.0 (all local IPv4 addresses)
:: (all local IPv6 addresses)
shh port is 22
so 0.0.0.0:22 is all local addresses listening on port 22
using "ss -tulpn" I can see all open sockets on the host and which addresses and ports they are listening on. If I use it with heightened privileges I can also see the direct process that is using the socket.
TCP is a reliable connection oriented transport between two endpoints. it cares about establishing a connection, sequencing data, acknowledging received data, retransmitting lost data, detecting certain connection problems, and closing the connection.
When a connection is made the sender will also have a unique port generated so the receiver can recognize the sender throughout the connection.
when using "ss -tn" after connecting the two machines through ssh I can see the socket has an "established" state meaning there is a connection happening. Whereas before it was showing listening which just means its waiting for a connection.

"State   Local Address:Port       Peer Address:Port
 ESTAB   10.10.10.2:51694        10.10.10.3:22"
 
TCP uses a threeway handshake where machine A will broadcast that it wishes to establish a TCP connection, then machine B will receive the request and agree, once agreed upon machine A will get acknowledge it got a response so both sides can begin exchanging data. This is useful because once a TCP handshake is make the receiver can indicate what data it has received and the sender can make sure it what it sent was all received, and if not it can resend what is missing. using "sudo tcpdump -i enp0s3 -n host 10.10.10.2" I can actually see all of the packets between client and server once a connection is formed.
TCP flags: P = PSH, . = ACK, S = SYN
[P.] means packet contains data and acknowledging previously received data.
[S] means I wish to establish a TCP connection with port 22 on this IP
("10.10.10.2:52336 > 10.10.10.3:22: Flags [S]")
[S.] means I have received your SYN and I'm willing to establish a connection
usually followed by a [.] packet which is acknowledging the response.

UDP skips the handshake and simply sends data directly to the ip and port without any confirmation that it arrived. This is useful in situations where old data becomes useless quickly, for example a players position in a game, if a packet is lost on TCP for a players position it might try resend the old position while the player is already far passed that point, but with UDP it wouldn't bother trying to resend the previous location instead it would just keep updating the most recent postion.

DNS:
DNS is a function that translates a name into an IP address. When running "ping example.com" the client doesn't know where example.com actually is however it can ask the DNS resolver and it will return the IP address of the requested service. DNS is an open socket usually open on the UDP port 53. When using DNS it pretty much checks the systemd-resolver cache and if the IP is not found relating to the service needed it will send a query out through the dns socket.

Application
    ↓
127.0.0.53:53
    ↓
systemd-resolved
    ↓
routing table
    ↓
enp0s8
    ↓
10.0.3.2 gateway
    ↓
192.168.1.1:53
