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


#### Reflection


