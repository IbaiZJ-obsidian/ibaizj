
# Install OpenTofu

```
https://opentofu.org/docs/intro/install/deb/
```
## Installing tooling

```
sudo apt update
```

```
sudo apt install -y apt-transport-https ca-certificates curl gnupg
```

## Set up the OpenTofu repository

```
sudo install -m 0755 -d /etc/apt/keyrings
```

```
curl -fsSL https://get.opentofu.org/opentofu.gpg | sudo tee /etc/apt/keyrings/opentofu.gpg >/dev/null
```

```
curl -fsSL https://packages.opentofu.org/opentofu/tofu/gpgkey | sudo gpg --no-tty --batch --dearmor -o /etc/apt/keyrings/opentofu-repo.gpg >/dev/null
```

```
sudo chmod a+r /etc/apt/keyrings/opentofu.gpg /etc/apt/keyrings/opentofu-repo.gpg
```

Now you have to create the OpenTofu source list.

```
echo \  "deb [signed-by=/etc/apt/keyrings/opentofu.gpg,/etc/apt/keyrings/opentofu-repo.gpg] https://packages.opentofu.org/opentofu/tofu/any/ any maindeb-src [signed-by=/etc/apt/keyrings/opentofu.gpg,/etc/apt/keyrings/opentofu-repo.gpg] https://packages.opentofu.org/opentofu/tofu/any/ any main" | \  sudo tee /etc/apt/sources.list.d/opentofu.list > /dev/nullsudo chmod a+r /etc/apt/sources.list.d/opentofu.list
```
