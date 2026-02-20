

```bash
ssh admin@192.168.1.100 -p 2222
```

```
ssh-keygen -R "[localhost]:2205"
```

**Root usuarioari ssh egiteko pausuak** 
```
root@debian:~# nano /etc/ssh/sshd_config
```

```
PermitRootLogin yes
```

```
systemctl restart ssh
```