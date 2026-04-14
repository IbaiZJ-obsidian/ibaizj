
```
ip route show
```

```
# 1. Borramos la ruta por defecto problemática
sudo ip route del default via 192.168.100.1 dev enp0s8

# 2. Añadimos una ruta solo para llegar a la VM2 (asumiendo que VM2 está en la red 192.168.20.0/24 como sugerimos antes)
sudo ip route add 192.168.20.0/24 via 192.168.100.1 dev enp0s8
```