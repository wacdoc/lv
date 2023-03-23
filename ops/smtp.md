# Izveidojiet savu SMTP pasta sūtīšanas serveri

## preambula

SMTP var tieši iegādāties pakalpojumus no mākoņa pakalpojumu sniedzējiem, piemēram:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Ali mākoņa e-pasta push](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Varat arī izveidot savu pasta serveri — neierobežota sūtīšana, zemas kopējās izmaksas.

Zemāk mēs soli pa solim demonstrējam, kā izveidot savu pasta serveri.

## Servera izvēle

Pašmitinātam SMTP serverim ir nepieciešams publisks IP ar atvērtiem portiem 25, 456 un 587.

Parasti izmantotie publiskie mākoņi ir bloķējuši šos portus pēc noklusējuma, un tos var atvērt, izdodot darba pasūtījumu, taču tas galu galā ir ļoti apgrūtinoši.

Es iesaku iegādāties no resursdatora, kuram ir atvērti šie porti un kas atbalsta apgriezto domēna nosaukumu iestatīšanu.

Šeit es iesaku [Contabo](https://contabo.com) .

Contabo ir mitināšanas pakalpojumu sniedzējs, kas atrodas Minhenē, Vācijā, dibināts 2003. gadā ar ļoti konkurētspējīgām cenām.

Izvēloties eiro kā pirkuma valūtu, cena būs lētāka (serveris ar 8GB atmiņu un 4 CPU gadā maksā aptuveni 529 juaņas, un sākotnējā instalācijas maksa vienu gadu ir bez maksas).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

Veicot pasūtījumu, atzīmējiet, ka `prefer AMD` , un serverim ar AMD centrālo procesoru būs labāka veiktspēja.

Tālāk es ņemšu Contabo VPS kā piemēru, lai parādītu, kā izveidot savu pasta serveri.

## Ubuntu sistēmas konfigurācija

Operētājsistēma šeit ir Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Ja ssh serverī tiek rādīts `Welcome to TinyCore 13!` (kā parādīts attēlā zemāk), tas nozīmē, ka sistēma vēl nav instalēta. Lūdzu, atvienojiet ssh un uzgaidiet dažas minūtes, lai vēlreiz pieteiktos.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Kad tiek parādīts `Welcome to Ubuntu 22.04.1 LTS` , inicializācija ir pabeigta, un jūs varat turpināt tālāk norādītās darbības.

### [Neobligāti] Inicializējiet izstrādes vidi

Šī darbība nav obligāta.

Ērtības labad es ievietoju ubuntu programmatūras instalēšanu un sistēmas konfigurāciju vietnē [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

Palaidiet šo komandu, lai instalētu ar vienu klikšķi.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Ķīniešu lietotāji, lūdzu, izmantojiet tālāk norādīto komandu, un valoda, laika josla utt. tiks iestatīta automātiski.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo iespējo IPV6

Iespējojiet IPV6, lai SMTP varētu arī nosūtīt e-pasta ziņojumus ar IPV6 adresēm.

rediģēt `/etc/sysctl.conf`

Mainiet vai pievienojiet šādas rindas

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Sekojiet līdzi [kontabo apmācībai: IPv6 savienojamības pievienošana serverim](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Rediģējiet `/etc/netplan/01-netcfg.yaml` , pievienojiet dažas rindiņas, kā parādīts zemāk esošajā attēlā (Contabo VPS noklusējuma konfigurācijas failā šīs rindas jau ir, vienkārši noņemiet komentārus).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Pēc tam `netplan apply` , lai modificētā konfigurācija stātos spēkā.

Kad konfigurācija ir veiksmīga, varat izmantot `curl 6.ipw.cn` , lai skatītu ārējā tīkla IPv6 adresi.

## Klonēt konfigurācijas repozitorija operācijas

```
git clone https://github.com/wactax/ops.soft.git
```

## Izveidojiet bezmaksas SSL sertifikātu savam domēna vārdam

Pasta sūtīšanai nepieciešams SSL sertifikāts šifrēšanai un parakstīšanai.

Mēs izmantojam [acme.sh](https://github.com/acmesh-official/acme.sh) , lai ģenerētu sertifikātus.

acme.sh ir atvērtā koda automatizēts sertifikātu parakstīšanas rīks,

Ievadiet konfigurācijas noliktavu ops.soft, palaidiet `./ssl.sh` , un **augšējā direktorijā** tiks izveidota `conf` mape.

Atrodiet savu DNS nodrošinātāju vietnē [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , rediģējiet `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Pēc tam palaidiet `./ssl.sh 123.com` , lai savam domēna vārdam ģenerētu `123.com` un `*.123.com` sertifikātus.

Pirmajā palaišanas reizē automātiski tiks instalēts [acme.sh](https://github.com/acmesh-official/acme.sh) un pievienots ieplānots uzdevums automātiskai atjaunošanai. Var redzēt `crontab -l` , tur ir šāda rinda.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

Ģenerētā sertifikāta ceļš ir kaut kas līdzīgs `/mnt/www/.acme.sh/123.com_ecc。`

Sertifikāta atjaunošana izsauks skriptu `conf/reload/123.com.sh` , rediģējiet šo skriptu, varat pievienot komandas, piemēram `nginx -s reload` lai atsvaidzinātu saistīto lietojumprogrammu sertifikātu kešatmiņu.

## Izveidojiet SMTP serveri ar chasquid

[chasquid](https://github.com/albertito/chasquid) ir atvērtā koda SMTP serveris, kas rakstīts Go valodā.

Kā aizstājēju senajām pasta serveru programmām, piemēram, Postfix un Sendmail, chasquid ir vienkāršāks un vieglāk lietojams, kā arī vieglāks sekundārajai izstrādei.

Palaidiet `./chasquid/init.sh 123.com` kas tiks instalēts automātiski ar vienu klikšķi (aizstāt 123.com ar savu sūtīšanas domēna nosaukumu).

## Konfigurējiet e-pasta paraksta DKIM

DKIM tiek izmantots, lai nosūtītu e-pasta parakstus, lai vēstules netiktu uzskatītas par surogātpastu.

Pēc veiksmīgas komandas izpildes jums tiks piedāvāts iestatīt DKIM ierakstu (kā parādīts tālāk).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Vienkārši pievienojiet TXT ierakstu savam DNS (kā parādīts zemāk).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Skatiet pakalpojuma statusu un žurnālus

 `systemctl status chasquid` Skatiet pakalpojuma statusu.

Normālas darbības stāvoklis ir tāds, kā parādīts zemāk esošajā attēlā

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` vai `journalctl -xeu chasquid` var skatīt kļūdu žurnālu.

## Apgrieztā domēna vārda konfigurācija

Apgrieztais domēna nosaukums ir paredzēts, lai IP adrese tiktu atrisināta līdz atbilstošajam domēna vārdam.

Iestatot apgrieztu domēna nosaukumu, e-pasta ziņojumi var tikt identificēti kā mēstules.

Kad pasts ir saņemts, saņēmēja serveris veiks apgrieztā domēna vārda analīzi sūtītāja servera IP adresei, lai pārbaudītu, vai sūtītājam ir derīgs apgrieztā domēna nosaukums.

Ja sūtītājam serverim nav apgrieztā domēna nosaukuma vai ja apgrieztais domēna nosaukums neatbilst sūtītāja servera IP adresei, saņēmēja serveris var atpazīt e-pastu kā surogātpastu vai to noraidīt.

Apmeklējiet [vietni https://my.contabo.com/rdns](https://my.contabo.com/rdns) un konfigurējiet, kā parādīts tālāk

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Pēc apgrieztā domēna nosaukuma iestatīšanas neaizmirstiet konfigurēt servera domēna nosaukuma ipv4 un ipv6 pārsūtīšanas izšķirtspēju.

## Rediģējiet chasquid.conf resursdatora nosaukumu

Modificējiet `conf/chasquid/chasquid.conf` uz apgrieztā domēna nosaukuma vērtību.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Pēc tam palaidiet `systemctl restart chasquid` lai restartētu pakalpojumu.

## Dublējiet konf. git krātuvē

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Piemēram, es dublēju mapi conf savā github procesā šādi

Vispirms izveidojiet privātu noliktavu

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Ievadiet conf direktoriju un iesniedziet to noliktavā

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Pievienot sūtītāju

palaist

```
chasquid-util user-add i@wac.tax
```

Var pievienot sūtītāju

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Pārbaudiet, vai parole ir iestatīta pareizi

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Pēc lietotāja pievienošanas `chasquid/domains/wac.tax/users` tiks atjaunināts, neaizmirstiet to iesniegt noliktavā.

## DNS pievieno SPF ierakstu

SPF (Sūtītāja politikas ietvars) ir e-pasta verifikācijas tehnoloģija, ko izmanto, lai novērstu krāpšanu e-pastā.

Tas pārbauda pasta sūtītāja identitāti, pārbaudot, vai sūtītāja IP adrese atbilst tā domēna vārda DNS ierakstiem, par kuru tas uzdodas, tādējādi neļaujot krāpniekiem sūtīt viltus e-pastus.

SPF ierakstu pievienošana var pēc iespējas novērst to, ka e-pasta ziņojumi tiek identificēti kā mēstules.

Ja jūsu domēna nosaukumu serveris neatbalsta SPF tipu, vienkārši pievienojiet TXT tipa ierakstu.

Piemēram, `wac.tax` SPF ir šāds

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Ņemiet vērā, ka šeit esmu `include:_spf.google.com` , jo ​​es vēlāk konfigurēšu `i@wac.tax` kā sūtīšanas adresi Google pastkastē.

## DNS konfigurācija DMARC

DMARC ir saīsinājums no (Domain-based Message Authentication, Reporting & Conformance).

To izmanto, lai tvertu SPF atlēcienus (iespējams, to izraisa konfigurācijas kļūdas vai kāds cits uzdodas par jums, lai sūtītu surogātpastu).

Pievienot TXT ierakstu `_dmarc` ,

Saturs ir šāds

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

Katra parametra nozīme ir šāda

### p (politika)

Norāda, kā rīkoties ar e-pasta ziņojumiem, kuriem neizdodas SPF (Sender Policy Framework) vai DKIM (DomainKeys Identified Mail) verifikācija. Parametram p var iestatīt vienu no trim vērtībām:

* nav: netiek veiktas nekādas darbības, tikai verifikācijas rezultāts tiek nosūtīts atpakaļ sūtītājam, izmantojot e-pasta ziņošanas mehānismu.
* Karantīna: e-pastu, kas nav izturējis verifikāciju, ievietojiet surogātpasta mapē, taču tas netiks tieši noraidīts.
* noraidīt: tieši noraidiet e-pasta ziņojumus, kuru verifikācija neizdodas.

### fo (kļūmes opcijas)

Norāda ziņošanas mehānisma atgrieztās informācijas apjomu. To var iestatīt uz vienu no šīm vērtībām:

* 0: ziņojiet par visu ziņojumu validācijas rezultātiem
* 1. Ziņojiet tikai par ziņojumiem, kuru pārbaude neizdodas
* d: ziņot tikai par domēna vārda verifikācijas kļūmēm
* s: ziņot tikai par SPF verifikācijas kļūmēm
* l: ziņot tikai par DKIM verifikācijas kļūmēm

### rua & ruf

* rua (Reporting URI for Aggregate Reports): e-pasta adrese apkopoto pārskatu saņemšanai
* ruf (Ziņojuma URI kriminālistikas ziņojumiem): e-pasta adrese detalizētu ziņojumu saņemšanai

## Pievienojiet MX ierakstus, lai pārsūtītu e-pastus uz Google Mail

Tā kā es nevarēju atrast bezmaksas korporatīvo pastkasti, kas atbalstītu universālās adreses (Catch-All, var saņemt visus e-pastus, kas nosūtīti uz šo domēna nosaukumu, bez ierobežojumiem attiecībā uz prefiksiem), es izmantoju chasquid, lai pārsūtītu visus e-pastus uz savu Gmail pastkasti.

**Ja jums ir sava maksas uzņēmuma pastkaste, lūdzu, nemodificējiet MX un izlaidiet šo darbību.**

Rediģēt `conf/chasquid/domains/wac.tax/aliases` , iestatīt pāradresācijas pastkasti

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` norāda visus e-pastus, `i` ir iepriekš izveidotais sūtītāja lietotāja e-pasta adreses prefikss. Lai pārsūtītu pastu, katram lietotājam ir jāpievieno rinda.

Pēc tam pievienojiet MX ierakstu (šeit es norādu tieši uz apgrieztā domēna nosaukuma adresi, kā parādīts zemāk esošā attēla pirmajā rindā).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Kad konfigurācija ir pabeigta, varat izmantot citas e-pasta adreses, lai sūtītu e-pastus uz `i@wac.tax` un `any123@wac.tax` , lai noskaidrotu, vai varat saņemt e-pasta ziņojumus pakalpojumā Gmail.

Ja nē, pārbaudiet chasquid žurnālu ( `grep chasquid /var/log/syslog` ).

## Nosūtiet e-pastu uz i@wac.tax, izmantojot Google Mail

Kad Google Mail saņēma pastu, es, protams, cerēju atbildēt ar `i@wac.tax` , nevis i.wac.tax@gmail.com.

Apmeklējiet vietni [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) un noklikšķiniet uz Pievienot citu e-pasta adresi.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Pēc tam ievadiet verifikācijas kodu, kas saņemts e-pastā, uz kuru tika pārsūtīts.

Visbeidzot, to var iestatīt kā noklusējuma sūtītāja adresi (kopā ar iespēju atbildēt ar to pašu adresi).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Tādā veidā esam pabeiguši SMTP pasta servera izveidi un tajā pašā laikā e-pastu sūtīšanai un saņemšanai izmantojam Google Mail.

## Nosūtiet testa e-pastu, lai pārbaudītu, vai konfigurācija ir veiksmīga

Ievadiet `ops/chasquid`

Palaist `direnv allow` instalēt atkarības (direnv tika instalēts iepriekšējā vienas atslēgas inicializācijas procesā, un apvalkam ir pievienots āķis)

tad skrien

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

Parametru nozīme ir šāda

* lietotājs: SMTP lietotājvārds
* caurlaide: SMTP parole
* kam: adresāts

Varat nosūtīt testa e-pastu.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Ieteicams izmantot Gmail, lai saņemtu testa e-pastus, lai pārbaudītu, vai konfigurācijas ir veiksmīgas.

### TLS standarta šifrēšana

Kā parādīts attēlā zemāk, ir šī mazā bloķēšana, kas nozīmē, ka SSL sertifikāts ir veiksmīgi iespējots.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Pēc tam noklikšķiniet uz "Rādīt sākotnējo e-pastu"

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Kā parādīts tālāk esošajā attēlā, Gmail sākotnējā pasta lapā tiek rādīts DKIM, kas nozīmē, ka DKIM konfigurācija ir veiksmīga.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Sākotnējā e-pasta galvenē pārbaudiet Saņemts, un jūs varat redzēt, ka sūtītāja adrese ir IPV6, kas nozīmē, ka arī IPV6 ir veiksmīgi konfigurēts.
