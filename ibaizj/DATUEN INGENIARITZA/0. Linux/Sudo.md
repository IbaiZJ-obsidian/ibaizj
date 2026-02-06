
# Install sudo

```
su -
apt update
apt install sudo
```

# Add sudo to user

```
usermod -aG sudo ibai
```
```
groups ibai
```
ibai : ibai cdrom floppy sudo audio dip video plugdev users netdev bluetooth
