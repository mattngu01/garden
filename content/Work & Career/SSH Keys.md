# How they work
- keys come in pairs 
	- public (in `authorized_keys`) -> copied to the SSH server; anyone w/ public can encrypt data with public key & is decrypted w/ private key
	- private (`id_rsa`) -> stays with user

```
 1.) Client says "Hey Server! I got the private key of that public key!" (Client -> Server)

2.) Server is like "Oh yeah, prove it!". And the Server generates a random number, and uses the public key pair installed in authorized_keys to encrypt it and send it to the client. (Client <- Server)

3.) The Client receives the encrypted message "challenge", decrypts it with the private key, and sends back the value to the Server as proof of possessing the key. (Client -> Server)

4.) Server checks that it matches the random number it generated, and accepts that as proof, and tells the client that it is allowed in. (Client <- Server) 
```

You generate the keypair on a client, and give the servers your public ssh key.
## Best Practices
- generate ssh keypair for every machine