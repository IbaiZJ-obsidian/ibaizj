
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

---
# 3. Ariketa

---

**Jonek finantza saileko datuak gainbegiratu behar ditu. Irakurtzeko baimen soila eman nahi diogu, ezin dezan ezer aldatu.**

---
# 4. Ariketa

---

**Unai proiektu kudeatzailea da. `proiektuak/` direktorioan lan egiten du eta bertan sortzen den fitxategi berri orok bere baimen osoa (`rwx`) izatea nahi dugu, herentzia bidez.**

---
# 5. Ariketa

---
**Datu-basea oso kritikoa da eta segurtasun politikak zehazten du hemen zerrendatutako erabiltzaile batek ere ezin duela ez idatzi ez exekutatu. Hau, ACL maskara baten bidez inplementatuko da. Dena den, Ederri segurtasun-politika ez diogu aplikatuko eta bertan irakurri eta idazteko baimena emango diogu. Zer gertatzen da?**

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