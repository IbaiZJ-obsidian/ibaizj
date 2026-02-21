

```bash
ssh user@192.168.1.100 -p 2222
```

```
ssh-keygen -R "[localhost]:2205"
```

# SSH egin gako publiko/pribatuarekin

```
# Windows-ean .ssh karpetara sartu
cd .ssh

# Sortu gako publiko eta pribatua
ssh-keygen -f DatuIngenieritza -t rsa

# Sortu .ssh direktorioa zerbitzarian
ssh ibai@localhost -p 2205 "mkdir /home/ibai/.ssh"
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