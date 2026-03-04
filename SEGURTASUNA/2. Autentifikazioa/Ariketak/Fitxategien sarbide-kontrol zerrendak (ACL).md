
---
# 1. Ariketa

---

**Mikel kanpoko aholkularia denez, `estrategia2026.txt` fitxategia irakurri behar du soilik. Ez dugu bera jabe egin nahi, ezta zuzendaritza taldean sartu ere.**

Lehenengo, fact table-a ikusiko dugu permisoak ikusteko 
```
root@Segurtasuna:/srv/euskaltech/zuzendaritza# getfacl estrategia2026.txt
# file: estrategia2026.txt
# owner: root
# group: root
user::rw-
group::r--
other::---
```

Mikeli permisoak eman directorioa exekutatzeko eta artxiboa irakurtzeko

```
setfacl -m u:mikel:x /srv/euskaltech/zuzendaritza
setfacl -m u:mikel:r /srv/euskaltech/zuzendaritza/estrategia2026.txt
```

 Fact table ondo dagoela ikusi

```
root@Segurtasuna:/srv/euskaltech# getfacl zuzendaritza
# file: zuzendaritza
# owner: root
# group: root
user::rwx
user:mikel:--x
group::r-x
mask::r-x
other::---

root@Segurtasuna:/srv/euskaltech/zuzendaritza# getfacl estrategia2026.txt
# file: estrategia2026.txt
# owner: root
# group: root
user::rw-
user:mikel:r--
group::r--
mask::r--
other::---
```

---
# 2. Ariketa

---
**Ikerketa sailean hainbat lagun egon daitezke (Ane barne). Banan-banan baimenak eman beharrean, `ikerlariak` talde osoari emango diogu `txostena.txt` fitxategia irakurtzeko eta idazteko baimena. Horrela, etorkizunean ikerlari berri bat datorrenean, taldean sartzearekin nahikoa izango da.**

Sortu ikerketa taldea
```
groupadd ikerketa
```

Gehitu usuarioak taldera
```
usermod -aG ikerketa ane
usermod -aG ikerketa jon
```
```
cat /etc/group

ikerketa:x:1009:ane,jon
```

Eman permisoak ikerketa taldeari ikerketa direktorioa exekutatzeko
```
root@Segurtasuna:/srv/euskaltech# setfacl -m g:ikerketa:x /srv/euskaltech/ikerketa

root@Segurtasuna:/srv/euskaltech# getfacl ikerketa/
# file: ikerketa/
# owner: root
# group: root
user::rwx
group::r-x
group:ikerketa:--x
mask::r-x
other::---
```

Eman permisoak ikerketa taldeari ikerketa/txostena.txt artxiboa irakurtzeko eta idazteko
```
root@Segurtasuna:/srv/euskaltech# setfacl -m g:ikerketa:rw /srv/euskaltech/ikerketa/txostena.txt

root@Segurtasuna:/srv/euskaltech# getfacl ikerketa/txostena.txt
# file: ikerketa/txostena.txt
# owner: root
# group: root
user::rw-
group::r--
group:ikerketa:rw-
mask::rw-
other::---
```

---
# 3. Ariketa

---

**Jonek finantza saileko datuak gainbegiratu behar ditu. Irakurtzeko baimen soila eman nahi diogu, ezin dezan ezer aldatu.**

```
root@Segurtasuna:/srv/euskaltech# setfacl -m u:jon:r /srv/euskaltech/finantzak/
root@Segurtasuna:/srv/euskaltech# getfacl finantzak/
# file: finantzak/
# owner: root
# group: root
user::rwx
user:jon:r--
group::r-x
mask::r-x
other::---
default:user::rwx
default:user:jon:r--
default:group::r-x
default:mask::r-x
default:other::---
```

---
# 4. Ariketa

---

**Unai proiektu kudeatzailea da. `proiektuak/` direktorioan lan egiten du eta bertan sortzen den fitxategi berri orok bere baimen osoa (`rwx`) izatea nahi dugu, herentzia bidez.**

d jarri behar da herentzia aktibatzeko
```
root@Segurtasuna:/srv/euskaltech# setfacl -m d:unai:rwx /srv/euskaltech/proiektuak/

root@Segurtasuna:/srv/euskaltech# getfacl /srv/euskaltech/proiektuak/
getfacl: Bide-izen absolutoen '/' aurrizkia kentzen
# file: srv/euskaltech/proiektuak/
# owner: root
# group: root
user::rwx
user:unai:rwx
group::r-x
mask::rwx
other::---
default:user::rwx
default:user:unai:rwx
default:group::r-x
default:mask::rwx
default:other::---
```

---
# 5. Ariketa

---
**Datu-basea oso kritikoa da eta segurtasun politikak zehazten du hemen zerrendatutako erabiltzaile batek ere ezin duela ez idatzi ez exekutatu. Hau, ACL maskara baten bidez inplementatuko da. Dena den, Ederri segurtasun-politika ez diogu aplikatuko eta bertan irakurri eta idazteko baimena emango diogu. Zer gertatzen da?**

```
setfacl -R -m mask::r-- datu_basea/
```

---
# 6. Ariketa

---
**Anek, ikerlaria izateaz gain, proiektu sekretu batean parte hartzen du. Dagokion fitxategian irakurri eta idatzi behar du**

---
# 7. Ariketa

---
**Xabierrek azpiegitura karpetan sortzen diren fitxategi guztien gaineko kontrola duela ziurtatu behar dugu defektuzko ACLak erabiliz.**

---
# 8. Ariketa

---
**Irati ikertzaileen taldeko kide izan daiteke, baina segurtasun arrazoiengatik, espresuki debekatu behar diogu txostena.txt fitxategira sarbide oro.  Taldeko politikak edo pertsonarenak du lehentasuna?**



---
# Emaitzak

---


Ondo: 1. Kasua: Mikel
Ondo: 2. Kasua: Ikerlariak Taldea
Ondo: 3. Kasua: Jon
Ondo: 4. Kasua: Unai (Herentzia)
Ondo: 5. Kasua: Eder (Maskara)
Ondo: 6. Kasua: Ane
Ondo: 7. Kasua: Xabier (Default)
Ondo: 8. Kasua: Irati