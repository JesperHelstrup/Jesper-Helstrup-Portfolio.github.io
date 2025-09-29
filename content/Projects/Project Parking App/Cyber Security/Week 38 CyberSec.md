### Learnings

#### Encryption basics (symmetric vs asymmetric)


#### The role of encryption in protecting database backups


### Tasks

#### Demo: Aes

```
using System;

using System.Security.Cryptography;

using System.Text;

  

class Program

{

    static void Main()

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

  

        Console.WriteLine(text);

        Console.WriteLine(Convert.ToBase64String(encrypted));

        Console.WriteLine(decryptedText);

    }

}
```
#### Demo: RSA

```
using System;

using System.Security.Cryptography;

using System.Security.Cryptography.X509Certificates;

using System.Text;

  

public class RsaExample

{

public string AskerOfData()

{

using var rsa = RSA.Create();

byte[] publicKey = rsa.ExportRSAPublicKey();

  

byte[] encrypted = DataHolder(publicKey);

  

byte[] decrypted = rsa.Decrypt(encrypted, RSAEncryptionPadding.Pkcs1);

  

string secretData = Encoding.UTF8.GetString(decrypted);

  

return secretData;

  

}

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

}
```


#### Reflection


