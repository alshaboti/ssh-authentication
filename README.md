# SSH Authentication using Certificate. 
This repo to illustrate how to use ssh certificate based authentication. 

Assumptions:
Server:  already running ssh server, its name is myserver
Client:  none
CA authority: none 

1- At CA authority 
Create public and private key, if not exists. 
```
ssh-keygen -f CA_key
```
you will get private key CA_key and public key CA_key.pub. 


2- Use CA authority private key to sign the server host public key 
Copy the `server` host key `/etc/ssh/ssh_host_rsa_key.pub` to the CA authority host to sign it. 

Note that this is not a user ssh key, the ones in `~/.ssh/`, rather this is the host key. This host keys are generated when openssh installed [see here](https://www.ssh.com/ssh/host-key). 

```
ssh-keygen -h -s CA_key -n myserver -I mydataserver -V +52w ssh_host_rsa_key.pub
```
Options `-h` for signing host key, `-s` private key to be use for signing, `-n` is a comma-separated list of the domain 
names by which the Server is accessed (e.g. myserver). 

You will get new file ssh_host_rsa_key-cert.pub.  copy this file and to the Server at /etc/ssh/ directory

3- Tell the Server to use the certificate. 
add this line to `/etc/ssh/sshd_config` file. 
```
HostCertificate /etc/ssh/ssh_host_rsa_key-cert.pub
```

This line will tell the server to present this signed certificate for any client, so client should not ask the user
to verify the signature of the server anymore as it does with ssh-key based. 

Now restart the sshd 
```
sudo systemctl restart ssh 
```

4- Let clients trust the CA key. 
Last step add CA public key to all client `/etc/ssh/ssh_known_hosts` file.  
```
@cert-authority <list of principals> <past the CA-key.pub content here>
```
Note that the list of principals, should be the same as in step 2 (i.e. myserver in our case). 
You can remove all non CA entry from user's `~/.ssh/known_hosts` and `/etc/ssh/ssh/ssh_known_hosts` files, and only let clients trust the CA key. 

5- congrates!
Now you can ssh from the client to the server, and you should not need to verify the server signature. 

```
ssh <user>@<servername>
```

You should be prompted to the password directly without this verification
```
The authenticity of host '10.0.2.7 (10.0.2.7)' can't be established.
ECDSA key fingerprint is SHA256:p1zAio6c1bI+8HDp5xa+eKRi561aFDaPE1/xq1eYzCI.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```
As the Server should provide its certificate to the client, and the client verified it using the CA-key that it trusts. 
