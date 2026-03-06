
Sarbide-kontrol aurreratua AppArmor erabiliz
Ariketa honen bigarren zatian AppArmor erabiliz segurtasun politika bat sortu da.
Helburua izan da nc (netcat) aplikazioaren sare-erabilera mugatzea.
Politika honen arabera aplikazioak honako konexioak soilik egin ditzake:

-  IPv4 bidezko UDP konexioak
-  IPv6 bidezko TCP konexioak
- 
Beste edozein konbinazio debekatuta geratzen da.
Lehenik eta behin egiaztatu da zein den netcat programaren benetako bidea. Horretarako
erabili da:

```
realpath /bin/nc.openbsd
```

Komando horrek erakutsi du sistemak /usr/bin/nc.openbsd erabiltzen duela. Horregatik
AppArmor profila bide horretarako sortu da.
Sortutako profilaren barruan honako arauak definitu dira:
-  Beharrezko AppArmor abstractions
-  Netcat exekutagarriaren baimena
-  Baimendutako sare motak
-  Gainerako sare jarduerak blokeatzea
Profilak behar bezala funtzionatzen duela egiaztatzeko hainbat konexio saiakera egin dira.
Adibidez:
```
nc 127.0.0.1 443
```
eta beste konbinazio batzuk.
Debekatutako konexio bat egiten denean, AppArmor-ek jarduera hori blokeatzen du eta
kernel log-etan mezu bat agertzen da.
Log mezu horietan ikus daiteke konexioa ukatu dela eta zein profil aplikatu den.

**Zergatik ezin da politika hau DAC edo RBAC sistemekin egin?**
DAC (Discretionary Access Control) sistemetan sarbide-kontrola fitxategien baimenetan
oinarritzen da (rwx). Sistema honek fitxategien irakurketa, idazketa edo exekuzioa
kontrolatzen du, baina ez du aplikazio baten sare-portaera kontrolatzeko aukerarik.

Bestalde, RBAC (Role-Based Access Control) sisteman baimenak rolei esleitzen zaizkie eta
rol horiek erabiltzaileei ematen zaizkie. Hala ere, rol batek ematen dituen baimenak
orokorrak dira eta ez dute aplikazio baten socket mota edo sare protokoloa bezalako
xehetasunak kontrolatzen.

Aitzitik, AppArmor MAC (Mandatory Access Control) ereduan oinarritzen da, eta aplikazio
bakoitzaren jarduera zehatz-mehatz mugatzeko aukera ematen du.

Horregatik, ariketan eskatutako politika —sare familiaren eta socket motaren arabera
konexioak baimentzea edo blokeatzea— AppArmor bezalako sistema batekin bakarrik ezar
daiteke.
