# Lab #2, 22110072, NTT, INSE330380E_24_1_02FIE
# Task 1: Public-key based authentication
**Question 1**: Implement public-key based authentication step-by-step with openssl according the following scheme.
![scheme](https://cdn.discordapp.com/attachments/502382381399801856/1310776517734699049/image-1.png?ex=67467342&is=674521c2&hm=eb14585df28097b1ab735bdcd9f234acec09df1204263dea2f0333b872bc0e1c&)<br>
**Answer 1**:
## 1. Set up the docker environments:
*First, we create a Docker volume to share files between the client and the server:*

```bash
docker volume create shared-data
```

*Second, we create the client and server containers which use the shared volume:*

```bash
docker run -it --name client --mount source=shared-data,target=/shared ubuntu:latest
docker run -it --name server --mount source=shared-data,target=/shared ubuntu:latest
```

*Third, we update and install openssl in both containers:*

```bash
apt update && apt install -y openssl
```

## 2. [Client] Generate keys:
*Inside the client container, generate its private and public keys:*

```bash
openssl genrsa -out client_private_key.pem 2048
openssl rsa -in client_private_key.pem -pubout -out client_public_key.pem
```

*Then, move the public key to the shared folder:*

```bash
mv client_public_key.pem /shared/client_public_key.pem
```

![public_key](https://raw.githubusercontent.com/ByrnorOCount/Subs2/refs/heads/main/public_key.png)

## 3. [Server] Generate and encrypt the challenge message:
*Inside the server container, create the challenge message:*

```bash
echo "I keep floating down the river but the ocean never comes. Since the operation, I heard you're breathing just for one. Now everything's imaginary, especially what you love. You left another message, said it's done. It's done." > /shared/challenge.txt
``` 

*Then, we encrypt the challenge message using the client's public key:*

```bash
openssl pkeyutl -encrypt -inkey /shared/client_public_key.pem -pubin -in challenge.txt -out /shared/encrypted_challenge.bin
```

![encrypted](https://raw.githubusercontent.com/ByrnorOCount/Subs2/refs/heads/main/encrypted.png)

## 4. [Client] Decrypt and sign challenge:
*Inside the client container, we decrypt the message using its private key:*

```bash
openssl pkeyutl -decrypt -inkey client_private_key.pem -in /shared/encrypted_challenge.bin -out decrypted_challenge.txt

![decrypted](https://raw.githubusercontent.com/ByrnorOCount/Subs2/refs/heads/main/decrypted.png)

```
*Then, we sign the decrypted message using the its private key again:*

```bash
openssl dgst -sha256 -sign client_private_key.pem -out /shared/signed_challenge.sig decrypted_challenge.txt
```

![signed](https://raw.githubusercontent.com/ByrnorOCount/Subs2/refs/heads/main/signed.png)

## 5. [Server] Verify the signature:

*We verify the signature using the client's public key:*

```bash
openssl dgst -sha256 -verify /shared/client_public_key.pem -signature /shared/signed_challenge.sig challenge.txt
```

![verified](https://raw.githubusercontent.com/ByrnorOCount/Subs2/refs/heads/main/verified.png)

*Thus, the authentication is successful, and the scheme is complete.*


# Task 2. Encrypting large message 
Create a text file at least 56 bytes. 
![text](https://raw.githubusercontent.com/ByrnorOCount/Subs2/refs/heads/main/text.png)
<br>

**Question 1**: Encrypt the file with aes-256 cipher in CFB and OFB modes. How do you evaluate both cipher as far as error propagation and adjacent plaintext blocks are concerned. 

**Answer 1**:

**Question 2**: Modify the 8th byte of encrypted file in both modes (this emulates corrupted ciphertext).
Decrypt corrupted file, watch the result and give your comment on Chaining dependencies and Error propagation criteria.
**Answer 2**:

