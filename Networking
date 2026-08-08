Setting Static IPs:
Once both VM's were running I captured both of their Ip's by using "ip address", once I did that I could test if they could see eachother by pinging one IP from the other.
I noticed both VM's shared the same IP address, this most likely being due to using the same iso image to install linux, to fix this I will use "ls /etc/netplan/ to navigate to my netplan config file and 
manually set a static ip for the client device. In future I'd like to create a simple DHCP system but for now that would be overkill.
I then used nano to edit the config file and simply set the ip to 10.10.10.2/24 and 10.10.10.3/24 respectively.

Setting up SSH:
Need to use another network adapter set to "Nat" to acess the internet so I can install and update packages.
once booted my netplan config didn't include the Nat adapter so I had to add the line myself, after that I could install the openssh-server package with ease.
I then ran "systemctl enable ssh --now" to make sure it runs on boot so I don't need to boot it every time myself.
once running I used "ssh-copy-id <user>@10.10.10.3" to send the clients public key to the server to be copied into the ssh file making so if I ever need "remote" control of the server I can just run "ssh <user>@10.10.10.3". After that I changed the "PasswordAuthentication" from yes to no in "ssh_config"
to bypass any need to use the system password when trying to connect.
