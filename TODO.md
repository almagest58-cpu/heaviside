# Heaviside – poluprovodnici, posebne diode, transformator i osciloskop

Stanje: 23. 10. 2026. Poslednji APK je V2.5 (bez posebnih dioda i transformatora). Nova verzija APK-a **nije pravljena** – čeka odluku korisnika.
`app.html` = izvorni kod, `projekat.zip` = Android projekat (testovi u `tests/`).

## Novo u ovoj fazi
- **Diode po vrstama:** obična (Si/Ge), Zener, Šotki, LED (crvena, žuta, zelena, plava; svetli na šemi kada struja pređe 1 mA), varikap.
  - Zener: probojni napon BV i struja IBV; proboj se računa kao u ngspice-u (iteracija za xbv).
  - Idealni režim: samo jednosmerna karakteristika (r_d u modelu za male signale); varikap uvek ima C_j(V).
  - Realni režim: R_S (unutrašnji čvor), C_j(V) sa VJ i M (M ograničen na 0,9 kao u ngspice-u) i difuziona kapacitivnost τ_T·g_d.
- **Transformator (T):** odnos n = N₁/N₂, tačke označavaju početke namotaja, dugme za rotaciju obrće fazu sekundara (krajevi ostaju na mestu, žice se ne pomeraju).
  - Idealni: V₁ = n·V₂, I₂ = n·I₁ (jedna dodatna nepoznata po transformatoru u svim rešavačima: AC, DC radna tačka, vreme, polovi/nule, simboličko izvođenje).
  - Realni: udžbenička T šema – R₁, L_σ1, L_m paralelno primaru, idealni n:1, L_σ2, R₂. Isto je kao spregnuti kalemovi L₁ = L_σ1 + L_m, L₂ = L_σ2 + L_m/n², M = L_m/n.
  - Gotovi tipovi: opšti 2:1, mrežni 230 V → 12 V, audio 600 Ω, impulsni (gejt drajver), RF 1:4, SMPS 10:1.
  - Provera: svaki namotaj mora imati svoju vezu sa masom (jasna poruka).
- **Info kartice, korak „Model transformatora“ u izvođenju, PDF, primeri** (Zener stabilizator, LED, varikap, prilagođenje impedanse, mrežni transformator), prevodi na 6 jezika.

- **Osciloskop (simulacija u vremenu):** 6 generatora signala (DC, sinus, kvadratni, trougaoni, testerasti, impulsni), voltmetar i ampermetar.
  - Sa generatorom u kolu: Bode/Nyquist/Smith/Ω/polovi/formule se isključuju, crta se do 5 kanala (napon/struja bilo koje komponente).
  - Simulator `tran.js`: puni nelinearni modeli, BDF promenljivog koraka sa kontrolom greške na naelektrisanjima, Njutn sa SPICE ograničavanjem, pomoćne metode za teške slučajeve (prigušeni Njutn, pseudo-tranzijent, preskok preko „preklopa“).
  - Op. pojačavač: napajanje ±V_CC (klizač, standardne vrednosti u E-režimu); izlaz zasićuje na naponima napajanja.
  - Kanali se čuvaju sa projektom; 5 primera.
  - Testovi: `trtest` 200 kola (193 upoređeno, 0 grešaka aplikacije), `ostest` 150 kola (148 upoređeno). Na strmim ivicama razlika do nekoliko %, zbog vremenske mreže oba simulatora.

- **Realni tranzistor i MOSFET (realni mod, prekidač Idealan/Realan u panelu):**
  - BJT: r_b, r_c, r_e (unutrašnji čvorovi B′, C′, E′), C_je(V), C_jc(V), τ_F (difuziono naelektrisanje τ_F·I_CC/q_b), τ_R, I_KF; transkapacitivnost C_x = ∂Q_BE/∂V_BC (SPICE geqcb) je poseban element u svim rešavačima (AC, polovi, formule).
  - MOSFET: R_G, R_D, R_S, C_gs stalna, C_gd(V_GD) glatka (oblik spoja ispod 0, C_gd0 iznad), unutrašnja dioda S→D sa svojim parametrima.
  - Gotovi tipovi: opšti, BC547, 2N2222, 2N3904 (objavljeni SPICE modeli, uprošćeni), 2N7000, IRF540 (prema katalogu); Reset vraća vrednosti izabranog tipa; izbor tipa i pri dodavanju (meni).
  - Stari projekti ostaju idealni (realni samo kada je deo izričito prebačen na Realan).
  - Info kartice: okvir „Realni model“ sa šemom i formulama (i za diodu). Izvođenje: C_π, C_μ, C_x, f_T, C_gd(V) u radnoj tački.
  - Primeri: tranzistor kao prekidač (2N2222, vreme pražnjenja), MOSFET kao prekidač (IRF540, Milerov plato).
  - Testovi: `stest QREAL` (OP do 2e-8, AC do 5e-7, formule do 3e-9), `trtest QREAL`, `ostest QREAL` (gotovi tipovi kroz aplikaciju).

## Rezultati testova (posle ove faze)
- Transformatori prema ngspice-u (`stest.js` sa `XFT=1`): 145 + 112 + 109 kola.
  - Radna tačka do 9e-13, AC do 4e-7, formule do 8e-12.
  - Kod realnog transformatora sa k blizu 1 ngspice gubi tačnost na visokim učestanostima (do 11 %); proračun sa razlomcima (tačna aritmetika) potvrđuje aplikaciju (greška 5e-10).
  - Dva kola sa razlikom 1e-3: ngspice drži GMIN = 1e-12 S između baze i kolektora, a aplikacija takve provodnosti izostavlja (namerno, `SSALL=1` ih zadržava). Sa `SSALL=1` razlika je 3e-11.
  - Kola u kojima ngspice pada: petlje kalemova ili kalem paralelno izvoru pri DC (aplikaciji radna tačka tu ne treba).
- Diode prema ngspice-u: `ztest` 1152 tačke do 2e-12; `stest REAL` 150 kola: OP 6e-11, AC 3e-7, formule 2e-10.
- Stari testovi – identični kao pre: `rtest` (6e-15), `otest` (2,31e-8; 23/23), `otest GEN2` (2,74e-10; 141/141), `ptest` (0 problema), `spicetest` (0 problema), `spiceex` (stari primeri nepromenjeni, novi do 2e-12), `devtest` 60/60, `stest` 300 (OP 6e-10, AC 5e-7, formule 3e-9).
- `ftest`: stari lažni nalaz „R₁=0“ i isti takav „n=0“ za mrežni transformator (uslov, ne jednakost). Zaokružene vrednosti L₁, L₂, M, k sada se pišu sa ≈.
- `optest` (SEED=7, N=400): jedno kolo (321) daje radnu tačku koja se razlikuje 4 % od ngspice-a – isto i sa starim rešavačem, nije posledica ove faze; ostaje za ispitivanje.

## Ostalo
1. Ispitati `optest` kolo 321 (prihvatanje rešenja na „podu zaokruživanja“ kod velikog pojačanja).
2. Provera na pravom telefonu.
3. APK (V2.6) i glavna verzija artefakta – tek kad korisnik kaže.
4. PDF izveštaj u režimu osciloskopa; žute tačke za tok struje (ako korisnik želi).
5. Sledeće ideje korisnika (za kasnije): relej, prekidač sa debouncing-om, šum.
