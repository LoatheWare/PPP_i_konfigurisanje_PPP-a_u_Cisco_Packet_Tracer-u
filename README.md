# PPP i konfigurisanje PPP protokola u Cisco Packet Tracer-u
## Šta je ovo?
Ovo je vežba za administratore računarskih mreža koju sam radio tokom mog školovanja. Nalazi se jedan .pkt fajl koji se preuzme i to će Vam biti početno stanje. Zadak Vam je dat, kao i uputstvo za njegovo rešavanje. Ukoliko Vam nešto nije jasno tokom rešavanja ovih zadataka obratite mi se direktnom porukom na aplikaciji LinkedIn, www.linkedin.com/in/nikola-karanović-397185390

## Pitanja i odgovori — PPP (Point-to-Point Protocol)

### 1. Šta je PPP?
**PPP** (Point-to-Point Protocol) je protokol **2. (data link) sloja** koji se koristi za uspostavljanje direktne veze između dva uređaja preko serijske linije (npr. WAN veza između dva rutera). PPP je standardizovan protokol (definisan u **RFC 1661**), za razliku od starijih, proizvođački-zavisnih protokola kao što je Cisco-ov **HDLC**, koji je default enkapsulacija na Cisco serijskim interfejsima ali ne radi sa opremom drugih proizvođača.

---

### 2. Koje su osnovne prednosti PPP protokola?
PPP nam omogućava:
1. **Interoperabilnost** — radi između opreme različitih proizvođača (za razliku od HDLC-a koji je Cisco-proprietary).
2. **Autentifikaciju veze** — podržava PAP i CHAP autentifikaciju, čime se obezbeđuje da se sa udaljenim uređajem poveže samo ovlašćena strana.
3. **Detekciju greški i kontrolu kvaliteta linka** — putem LCP-a (Link Control Protocol).
4. **Podršku za više protokola 3. sloja istovremeno** (multiprotokolska podrška) preko **NCP-a** (Network Control Protocol), npr. IP, IPv6 i drugi mogu da koegzistiraju na istoj vezi.
5. **Kompresiju i Multilink PPP** — mogućnost spajanja više fizičkih linkova u jedan logički (veći propusni opseg).

---

### 3. Od kojih se komponenti (podslojeva) sastoji PPP?
PPP se sastoji od dva ključna podprotokola:
- **LCP (Link Control Protocol)** — zadužen za uspostavljanje, konfigurisanje, testiranje i raskidanje same veze (link-a), kao i za pregovaranje parametara poput autentifikacije, kompresije i Multilink-a.
- **NCP (Network Control Protocol)** — zadužen za konfigurisanje i pregovaranje parametara protokola **3. sloja** koji se prenosi preko linka (npr. **IPCP** za IP saobraćaj).

---

### 4. Koje vrste autentifikacije podržava PPP?
PPP podržava dva tipa autentifikacije:
- **PAP (Password Authentication Protocol)** — jednostavnija, **manje sigurna** metoda. Lozinka se šalje u **plain text** formatu prilikom uspostavljanja veze (samo na početku, two-way handshake).
- **CHAP (Challenge Handshake Authentication Protocol)** — **sigurnija** metoda. Koristi **three-way handshake** i razmenjuje *hash* vrednost (na osnovu challenge-a i lozinke), umesto same lozinke u čistom tekstu. Provera identiteta se može ponavljati i tokom trajanja same veze, ne samo na početku.

---

### 5. Kako radi CHAP autentifikacija (three-way handshake)?
CHAP se odvija u tri koraka:
1. Ruter A (koji inicira proveru) šalje **challenge** poruku ruteru B.
2. Ruter B na osnovu primljenog challenge-a i svoje lozinke računa **hash** vrednost i vraća je kao odgovor.
3. Ruter A samostalno računa istu hash vrednost (na osnovu iste lozinke koju i on poseduje) i poredi je sa primljenom. Ako se hash vrednosti poklapaju, veza se uspostavlja (**success**), u suprotnom autentifikacija ne uspeva (**failure**).

Važno je da **oba** rutera imaju **isto korisničko ime i lozinku** konfigurisane (lozinke moraju biti identične na obe strane).

---

### 6. Kako se konfiguriše PPP enkapsulacija na serijskom interfejsu?
Osnovna komanda za promenu enkapsulacije sa default HDLC na PPP:

"Router(config)# interface Serial0/0/0"

"Router(config-if)# encapsulation ppp"

---

### 7. Kako se konfiguriše PAP autentifikacija?
Na oba rutera potrebno je definisati korisničko ime/lozinku i aktivirati PAP:

"Router(config)# username R2 password cisco123"

"Router(config)# interface Serial0/0/0"

"Router(config-if)# encapsulation ppp"

"Router(config-if)# ppp authentication pap"

"Router(config-if)# ppp pap sent-username R1 password cisco123"

*(`username R2` predstavlja hostname suprotnog rutera, a `sent-username` je identitet koji ovaj ruter šalje drugoj strani.)*

---

### 8. Kako se konfiguriše CHAP autentifikacija?
Konfiguracija je slična kao kod PAP-a, samo se koristi `ppp authentication chap` umesto `pap`, a **username mora biti hostname suprotnog rutera**, dok je lozinka identična na oba rutera:

"Router(config)# username R2 password cisco123"

"Router(config)# interface Serial0/0/0"

"Router(config-if)# encapsulation ppp"

"Router(config-if)# ppp authentication chap"

*(Ovde nije potreban `sent-username`, jer CHAP automatski koristi hostname lokalnog rutera kao identitet.)*

---

### 9. Koje su komande za proveru PPP konfiguracije i stanja veze?
Najvažnije komande za verifikaciju PPP-a:

"Router# show interfaces serial0/0/0"

"Router# show ppp"

"Router# debug ppp authentication"

"Router# debug ppp negotiation"

- `show interfaces serial0/0/0` — prikazuje status interfejsa, enkapsulaciju (PPP), kao i status LCP-a (`LCP Open` znači da je veza uspešno uspostavljena).
- `debug ppp authentication` — prikazuje detaljan tok autentifikacije (PAP/CHAP) u realnom vremenu, korisno za troubleshooting.
- `debug ppp negotiation` — prikazuje pregovaranje LCP/NCP parametara prilikom uspostavljanja veze.

---

### 10. Šta je Multilink PPP (MLPPP) i čemu služi?
**Multilink PPP** je proširenje PPP protokola koje omogućava da se **više fizičkih serijskih linkova** logički spoje u **jedan virtuelni link** veće propusne moći. Saobraćaj se raspoređuje (load-balansira) preko svih linkova u grupi, a sa strane viših slojeva mreže to izgleda kao da postoji samo jedna veza. Ovo se koristi kada je potrebno povećati propusni opseg WAN veze bez prelaska na skuplji, brži medijum.

---

### 11. Šta se dešava ako autentifikacija (PAP ili CHAP) ne uspe?
Ukoliko autentifikacija ne uspe (npr. zbog pogrešne lozinke ili pogrešno unetog username-a), **LCP veza se ne uspostavlja** (ostaje u stanju npr. `LCP: state = Stopped` ili sl.), a interfejs ostaje u stanju **"up/down"** — fizički (Layer 1) je aktivan, ali protokol (Layer 2) nije, pa nema razmene saobraćaja preko te veze.

---
