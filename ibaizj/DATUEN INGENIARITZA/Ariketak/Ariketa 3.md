
# Adibidea 1

![[7 Datuen Ingeniaritza - 2.3.1 Datu masiboentzako azpiegitura oinarriak - Oinarrizko teknologiak - Kontenedoreak.pdf#page=19]]

```
ls -l /sys/fs/cgroup/
```

```
systemctl status
```

```
find /sys/fs/cgroup/ -name cgroup.procs | xargs -I % sh -c 'echo %; cat %'
```

```
mkdir /sys/fs/cgroup/st
```

```
ls -l /sys/fs/cgroup/st
```


# Ariketa
