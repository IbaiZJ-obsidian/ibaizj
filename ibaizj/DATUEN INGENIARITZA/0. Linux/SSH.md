

```bash
ssh user@192.168.1.100 -p 2222
```

```
ssh -p 2205 -i C:\Users\iibai\.ssh\DatuIngenieritza ibai@localhost
```

```
ssh-keygen -R "[localhost]:2205"
```

# SSH gako publiko/pribatuarekin

```
# Windows-ean .ssh karpetara sartu
cd .ssh

# Sortu gako publiko eta pribatua
ssh-keygen -f DatuIngenieritza -t rsa

# Sortu .ssh direktorioa zerbitzarian
ssh ibai@localhost -p 2205 "mkdir /home/ibai/.ssh"

# Kopiatu gakoak zerbitzarian
scp -P 2205 DatuIngenieritza* ibai@localhost:/home/ibai/.ssh/

# Sartu gakoa authorized_keys barruan
ssh -p 2205 ibai@localhost "cat /home/ibai/.ssh/DatuIngenieritza.pub >> /home/ibai/.ssh/authorized_keys"

# ssh egin
ssh -p 2205 -i C:\Users\iibai\.ssh\DatuIngenieritza ibai@localhost
```
# Root usuarioari ssh egiteko pausuak

```
root@debian:~# nano /etc/ssh/sshd_config
```

```
PermitRootLogin yes
```

```
systemctl restart ssh
```