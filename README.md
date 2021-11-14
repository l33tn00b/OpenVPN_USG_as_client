# OpenVPN_USG_as_client
Setting up Ubiquiti's USG as OpenVPN client.
I spent quite some time on this... here we go.
But wait... 
This is by no means bullet proof. The USG just seems to be unable to work with non-static IP addresses. The tunnel will go stale without the USG taking note. 
So: If you have another machine capable of running an openvpn client and maybe doing some routing behind the USG: Use it instead of the USG.

# Prerequisites
- You need a machine to be used as OpenVPN server.
- Mine has Ubuntu 20.04.2 LTS
- You cannot use certificate-based authentication when going through the Web Interface of the USG.
- So we need to set up a shared-key authentication scheme.
- For whatever reason, keys and related stuff must be in ab subdirectory of `/etc/openvpn`. Else, OpenVPN will complain about not being able to find the files (even though they exist and are accessible).
- Create a static key as per OpenVPN doc: https://openvpn.net/community-resources/static-key-mini-howto/
- `cat` the key, copy it to your favorite editor and delete all newlines as well as the leading and trailing lines indiciating the content.
- Paste the modified key to the appropriate line in the Web Interface.
- It doesn't work (even though everythin else is set up correctly).

# TLDR:
- You need to set the server's cipher to BF-CBC (instead of something more modern).

# Long story:

## Server
- The server will throw errors (syslog):
```
Jul 23 14:29:43 spielwiese ovpn-server_static[8236]: Authenticate/Decrypt packet error: cipher final failed                                                      
Jul 23 14:29:53 spielwiese ovpn-server_static[8236]: Authenticate/Decrypt packet error: cipher final failed                                                      
Jul 23 14:30:02 spielwiese ovpn-server_static[8236]: Inactivity timeout (--ping-restart), restarting                                                             
Jul 23 14:30:02 spielwiese ovpn-server_static[8236]: SIGUSR1[soft,ping-restart] received, process restarting                                                     
Jul 23 14:30:02 spielwiese ovpn-server_static[8236]: Restart pause, 5 second(s)                                                                                  
Jul 23 14:30:07 spielwiese ovpn-server_static[8236]: Re-using pre-shared static key                                                                              
Jul 23 14:30:07 spielwiese ovpn-server_static[8236]: Preserving previous TUN/TAP instance: tun1  
```


## USG
- Everything seems fine, there is a green symbol behind the newly created network.
- Log on to the USG via ssh, take a look at `/var/log/messages`
- The USG will give an error when trying to connect to the server:
```
Jul 23 14:38:10 USG openvpn[1450]: Inactivity timeout (--ping-restart), restarting                                      
Jul 23 14:38:10 USG openvpn[1450]: SIGUSR1[soft,ping-restart] received, process restarting                             
Jul 23 14:38:10 USG openvpn[1450]: Restart pause, 2 second(s)             
```

- Just why?
```Jul 23 14:38:12 USG openvpn[1450]: Static Encrypt: Cipher 'BF-CBC' initialized with 128 bit key                         
Jul 23 14:38:12 USG openvpn[1450]: Static Encrypt: Using 160 bit message hash 'SHA1' for HMAC authentication            
Jul 23 14:38:12 USG openvpn[1450]: Static Decrypt: Cipher 'BF-CBC' initialized with 128 bit key                         
Jul 23 14:38:12 USG openvpn[1450]: Static Decrypt: Using 160 bit message hash 'SHA1' for HMAC authentication           
Jul 23 14:38:12 USG openvpn[1450]: Socket Buffers: R=[294912->131072] S=[294912->131072]                                
Jul 23 14:38:12 USG openvpn[1450]: UDPv4 link local (bound): [undef]                                                    
Jul 23 14:38:12 USG openvpn[1450]: UDPv4 link remote: [AF_INET]<remote IP>:<remote port>
```

- Ahh.. yes: BF-CBC is an old cipher (Blowfish, deemed too short)

# Have multiple servers/instances running?
`systemctl status openvpn@<instance name>`
