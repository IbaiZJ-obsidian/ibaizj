
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