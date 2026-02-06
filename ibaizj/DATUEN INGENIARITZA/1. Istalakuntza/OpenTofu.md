
# Install OpenTofu

```
https://opentofu.org/docs/intro/install/deb/
```

- Install curl
```
sudo apt install curl
```

```
curl --proto '=https' --tlsv1.2 -fsSL https://get.opentofu.org/install-opentofu.sh -o install-opentofu.sh
```

```
chmod +x install-opentofu.sh
```

```
./install-opentofu.sh --install-method deb
```

```
rm -f install-opentofu.sh
```
