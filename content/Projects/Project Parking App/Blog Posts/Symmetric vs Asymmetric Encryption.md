### Purpose of the Implementation

From the beginning of the project, I knew we needed some form of security around our data.  
Even though a license plate by itself isn't sensitive, **plate + timestamp + GPS location + an image** _is absolutely sensitive data_.

If that information leaked, someone could:
- track where a person parked
- see movement patterns
- identify vehicles at specific times

So I started looking into encryption.  
Not because we were going to fully implement it in our MVP (minimal viable product) — but because I needed to _understand the fundamentals_ to make the right decisions as the system grows.

Two examples helped me:
**AES**: fast symmetric encryption
**RSA**: secure asymmetric encryption

Learning these two approaches gave me a good idea for how real-world systems protect sensitive files, communication, and stored data.

### What I built

To explore the technologies hands-on, I implemented small AES and RSA demos in C#.  
These weren’t meant to be part of our production code — they were learning experiments to help me understand how the encryption process actually works.

#### AES – Symmetric Encryption

AES uses **one key** for both encryption and decryption.  
It’s extremely fast and is what you use when encrypting larger files such as images.

Here is the AES example I used:

~~~
public void AesTest()

{

	string text = "Encrypt me";
	
	using var aes = Aes.Create();
	aes.GenerateKey();
	aes.GenerateIV();
	
	byte[] plainBytes = Encoding.UTF8.GetBytes(text);
	using var encryptor = aes.CreateEncryptor();
	byte[] encrypted = encryptor.TransformFinalBlock(plainBytes, 0, plainBytes.Length);
	
	using var decryptor = aes.CreateDecryptor();
	
	byte[] decrypted = decryptor.TransformFinalBlock(encrypted, 0, encrypted.Length);
	
	string decryptedText = Encoding.UTF8.GetString(decrypted);
	
}
~~~

This demo showed me:
- How AES generates a random key + IV
- How data is transformed into encrypted bytes
- How the same key decrypts the data again

This is the technique systems would use to encrypt **images, video, GPS metadata**, etc.

#### RSA – Asymmetric Encryption

SA works with **two keys**:
- Public key → you can share this freely
- Private key → keep this secret

Anything encrypted with the _public_ key can only be decrypted with the _private_ key.

Here is the RSA example I used:

~~~
public string AskerOfData()

{
	using var rsa = RSA.Create();
	
	byte[] publicKey = rsa.ExportRSAPublicKey();
	
	byte[] encrypted = DataHolder(publicKey);
	
	byte[] decrypted = rsa.Decrypt(encrypted, RSAEncryptionPadding.Pkcs1);
	
	string secretData = Encoding.UTF8.GetString(decrypted);
	
	return secretData;
}
~~~

And the part that encrypts the data using the public key:

~~~
public byte[] DataHolder(byte[] publicKey)

{
	string secretData = "omg, an RSA secret";
	
	using var rsa = RSA.Create();
	
	int bytesRead;
	
	rsa.ImportRSAPublicKey(publicKey, out bytesRead);
	
	byte[] plainBytes = Encoding.UTF8.GetBytes(secretData);
	
	byte[] encrypted = rsa.Encrypt(plainBytes, RSAEncryptionPadding.Pkcs1);
	
	return encrypted;
}
~~~

This demo showed me:
- How to export/import RSA public keys
- How to encrypt with the public key
- How only the original private key holder can decrypt

This is the foundation for secure key exchange.
### What I learned

AES and RSA solve different problems.
- **AES** → great for encrypting _large_ data (photos, payloads, logs)
- **RSA** → great for _sharing a key securely_ or verifying identity

Real systems combine them (hybrid encryption):

1. Encrypt data with AES
2. Encrypt AES key with RSA
3. Send both together

This gives the speed of AES with the security of RSA.

During this exploration, I also learned that **we don’t need to manually implement RSA or AES for data-in-transit**, because HTTPS already handles that job for us.

HTTPS uses a mix of cryptographic techniques under the hood:

- **RSA (or ECDHE)** during the TLS handshake to agree on a shared session key
- **AES** for encrypting the actual HTTP request/response body

So if our app enforces HTTPS, manually implementing RSA for API traffic would be redundant and add unnecessary complexity.

I originally planned to implement **AES encryption for data-at-rest** in PostgreSQL (for images or GPS data stored in the database), but I didn’t have time to explore it fully. If I had more time, this is the part of encryption I would have focused on, since HTTPS only protects data _in transit_, not when it’s stored on disk.