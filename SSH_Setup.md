Setting up SSH:
Need to use another network adapter set to "Nat" to acess the internet so I can install and update packages.
once booted my netplan config didn't include the Nat adapter so I had to add the line myself, after that I could install the openssh-server package with ease.
I then ran "systemctl enable ssh --now" to make sure it runs on boot so I don't need to boot it every time myself.
once running I used "ssh-copy-id <user>@10.10.10.3" to send the clients public key to the server to be copied into the ssh file making so if I ever need "remote" control of the server I can just run "ssh <user>@10.10.10.3". After that I changed the "PasswordAuthentication" from yes to no in "ssh_config"
to bypass any need to use the system password when trying to connect. This also removes the ability for any malicious user to try and brute force the servers typical password because from now on the only way to access the server is by having access to the shh public key file, which I would have to share myself.
I did have to restart the ssh process by running "sudo systemctl restart ssh" to make sure the config changes were applied and after that everything ran smoothly.
