# Heaviside – poluprovodnici, posebne diode, transformator i osciloskop

Stanje: 23. 10. 2026. Poslednji objavljeni APK je V2.6 (realni tranzistori). Posle njega: prekidač sa odskakivanjem, PDF na osciloskopu i ispravka polova (vidi dole) – novi APK čeka odluku korisnika.
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

## Posle V2.6
- **Ispravka (greška u V2.6):** ispravka za izgubljeni pol (pencilEig) zadržavala je i vrednosti koje ne prođu proveru iznad pomaka – to su numeričke slike beskonačnih sopstvenih vrednosti, pa su se pojavljivali lažni polovi/nule ~1e12 rad/s i odziv u vremenu je u ~20/200 kola prelazio na simulaciju. Sada se zadržava samo pojedinačna vrednost bar dve dekade ispod pomaka (tamo je sama provera nepouzdana). pas19R: 100/100 rasporeda tačno; pas1, pas9R, pas28, pas29, act16, act19: isto kao pre (tačni model).
- **optest kolo 321:** nije greška – kolo ima dve radne tačke (provera u 60 cifara, `tests/op321_hp.py`).
- **Prekidač (SW):** otvoren/zatvoren, dodir na šemi ga prebacuje dok osciloskop radi (bez ponovnog pokretanja). Kolo sa prekidačem radi na osciloskopu. Realni mod: R_on, t_b, N_b; odskakivanje nasumično (dodiri sve duži, prekidi sve kraći), pri otvaranju ~N_b/3 odskoka u t_b/4. U `tran.js` prekidač je grana sa strujom: zatvoren V = R_on·I, otvoren I = 0; promene su prelomne tačke (korak se završava na njima, posle ide unazad Ojler). Info kartica, primer „Odskakivanje prekidača i RC debouncing“ (RC + Šmitov okidač sa op. pojačavačem). Test `tests/swtest.js` (RC i RL analitički, do 6e-5).
- **PDF na osciloskopu:** šema, elementi, slika osciloskopa u trenutku pauze sa vrednostima, merenja na ekranu (min, max, od vrha do vrha, srednja, RMS), proračun u vremenu (jednačine elemenata, radna tačka, BDF2, Njutn), događaji prekidača, pa cela analiza malih signala istog kola: generator → Vin (oko DC nivoa), izlaz = Vout ili prvi napon prema masi; ostali generatori → DC, prekidači u trenutnom stanju. f(x) na osciloskopu prikazuje proračun u vremenu.
- Osciloskop: kanal koji je praktično nula (manje od 1e-4 ostalih) više se ne uvećava ×10ⁿ.
- Odziv na odskočnu pobudu: skok u t = 0 se crta do y(0+) (tačan model), pa kriva ide od uzoraka.
- gtest: provera vremenskih grafika preskače tačke van okvira grafika i uzima uzorke u koloni u obzir (lažni nalazi kod oscilacija bržih od piksela).
- **Tiristori: SCR, trijak, dijak, quadrac** (idealni i realni mod, gotovi tipovi BT151, 2N5064, BT136, DB3, Q4008LT). Modeli od postojećih elemenata: SCR = PNP + NPN + R_GK (R_GK i β_P se tačno rešavaju iz I_GT i I_H); realni: dioda sa probojem V_BO, R_T, C_J2 (dV/dt), vremena t_q. Trijak = dve SCR ćelije u antiparalelu + kola gejta za kvadrante I, II, III. Dijak = dve ćelije bez gejta sa zener diodama za V_BO i ΔV. Quadrac = dijak u gejtu trijaka. Sonde V, I, V_G, I_G; info kartice; primeri „SCR zakačivač sa tasterima“, „Dimer za sijalicu (dijak + trijak)“, „Dimer sa quadrac-om“. PDF: modeli tiristora sa izračunatim vrednostima u proračunu u vremenu i u izvođenju. Test `tests/thytest.js` (16 slučajeva prema ngspice-u, izvodi do 3e-3).
- **Dijak: novi model (tačka 3).** Umesto dve Šoklijeve ćelije sa zener diodama (koje su se „zakačile“ već pri ~1,1·I_BO, pa relaksacioni oscilator sa DC napajanjem nije oscilovao) dijak je jedan element V = f(I) sa strujom kao nepoznatom (u `tran.js` i `semi.js`): blokira do (I_BO, V_BO), pa negativna otpornost logaritamski do V_BO − ΔV pri 10 mA (kataloška definicija), pa R_on. Realni: C_j i τ (struja prati napon u ~τ, ψ = τ·A·ln(1 + |I|/I_BO)). Dijak koji DC radna tačka postavi u oblast negativne otpornosti (nestabilna ravnoteža) kreće iz blokiranja, da bi oscilator krenuo. Novi primer „Relaksacioni oscilator sa dijakom“. PDF dimera kraći (62 → 50 strana).
- **Trijak: kvadrant IV (tačka 4).** Struja gejta ulazi u p-bazu ćelije A kao emitorska struja PNP-a čiji kolektor kroz diodu stiže do p-baze ćelije B: deo β₄/(1 + β₄) okida B. Novi klizač I_GT (IV) (∞ = trijak sa tri kvadranta); BT136 (E): 11 mA. Kvadranti I, II, III ostaju isti kao pre (provereno rampom struje gejta).
- **Trijak: kvadrant II.** Ranije osetljivost u kvadrantu II nije bila parametar i mnogo je odstupala od kataloga (BT136E: 0,54 mA umesto 4 mA; kod osetljivih tipova nije ni okidao). Sada: NPN g2 (baza T1, emitor G) povlači bazu malog pilot PNP-a sa emitorom na T2, koji ubacuje β_p·I_C2 u p-bazu ćelije A; QIII: PNP g3 ubacuje β₃·I_G u p-bazu B. Zasićeni par uzima deo struje gejta po tačnoj formuli (GP, β_R = 0,1), pa se β-e računaju iz I_GT (II) i I_GT (III) zatvorenim formulama. Novi klizač I_GT (II); BT136 (E): 4 mA, opšti 8 mA. Provera rampom struje gejta: sva četiri kvadranta pogađaju zadatu vrednost na 0,1 % (I_GT od 50 µA do 50 mA, I_H/I_GT od 0,9 do 20).

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
1. Provera na pravom telefonu (korisnik).
2. Novi APK (posle V2.6) i glavna verzija artefakta – tek kad korisnik kaže.
3. Sledeće ideje korisnika (za kasnije): relej, šum.
