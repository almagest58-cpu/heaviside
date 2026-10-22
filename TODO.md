# Heaviside – poluprovodnici (grana `poluprovodnici`)

Stanje: 23. 10. 2026. Glavna objava i APK (V2.4) **nisu urađeni** – čekaju odluku korisnika. `main` je netaknut.
`app.html` = izvorni kod, `projekat.zip` = Android projekat (testovi u `tests/`).

## Urađeno
- **Novi elementi:** dioda (Si/Ge); NPN/PNP tranzistor, Gummel–Poon podskup (IS, β, β_R, V_A, C_π, C_μ); N/P MOSFET, SPICE level 1 (V_th, k, λ, C_gs, C_gd); DC naponski i DC strujni izvor; DC nivo ulaza Vin.
- **Jednosmerna radna tačka:** Njutnova metoda sa SPICE ograničavanjem napona, zatim gmin-stepping, pa source-stepping; GMIN raspoređen kao u ngspice-u.
  - Jasne poruke za plivajuće čvorove, petlje naponskih izvora i nerealne struje.
- **Model za male signale iz tačnog Jakobijana:** g_m je VCCS u svim rešavačima i u simboličkom izvođenju.
- **UI, primeri i prevodi:** simboli, paneli, oblast rada na šemi, info kartice, vodič, 5 primera, prevodi na 6 jezika.
- **Izvođenje i PDF:** korak „Radna tačka i model za male signale“ (tačna vrednost, pa udžbenička aproksimacija u posebnom redu) i tabela radne tačke u PDF-u.

## Izmena u pencilEig (stara greška, postojala i pre poluprovodnika)
- **Problem:** polovi daleko od pomaka gubili su relativnu tačnost, pa su se bliski različiti polovi spajali u lažni višestruki. Primer `cs`: trostruko −24 umesto −6,84, −31,19 i −34,26.
- **Ispravka:** `pencilEigR` takvu grupu ponovo računa sa pomakom oko njene veličine. Novi rezultat prihvata samo ako je broj vrednosti isti i ako ne spaja vrednosti koje su bile razdvojene; bez ovog uslova kolo `act43` je dobilo novi lažni dvostruki pol.

## Rezultati testova (posle svih izmena)
- **Stari testovi (identični ili bolji nego pre):**
  - `rtest` 100: formule do 6e-15.
  - `otest` GEN2 200: grafici 2,74e-10, formule 141/141.
  - `otest` nasumični 150: grafici 2,31e-8, formule 23/23.
  - `ptest` 100: 308 oznaka na graficima, 0 problema.
  - `gtest`: Bode, Ω, Nyquist, Smith i polovi bez greške; vremenski tab je i ranije bio nepouzdana metrika.
  - `ftest` 14.007 jednačina: 1 lažni nalaz – uslov „Re{Z}=0 ⇔ R₁=0“; kod koji ga piše nije menjan.
  - `spicetest` 132: 0 problema; na spornim tačkama aplikacija je bliža proračunu od 72 cifre.
  - `spiceex`: praktično identično; razlike samo na spornim tačkama, gde je greška aplikacije ≤ 7e-11.
  - `tdtest` 200: 177 tačno (bilo 176, `cs` sada tačan), 11 simulirano. 6 grešaka: `cmp` je očekivan, a u 5 kola ngspice zauzme preko 3 GB memorije i bude prekinut. Lista promašenih polova ista kao pre, osim što je `cs` ispravljen.
- **Novi testovi:**
  - `devtest`: 60/60 elemenata se poklapa sa ngspice-om.
  - `optest` 400: 308/308 se poklapa.
  - `stest` 980 kola: radna tačka do 3e-9, AC do 1e-6, sve formule do 1e-9.
  - Z-mode kola sa 1 GΩ ka masi na ulazu (`ZG=1`): 240 kola, 238 upoređeno i sva se poklapaju (OP 1e-10, AC 1,5e-7, formule 238/238). Jedno kolo je nerealno (aplikacija ga ispravno odbija), a u jednom padaju i aplikacija i ngspice.

## Provereno
- **MathJax:** iscrtava formule kada je `assets/mathjax/tex-svg.js` prisutan; CI ga ugrađuje, a lokalno je uzet iz ranijeg APK-a.
- **PDF:** naslovna strana sa šemom i tabelom radne tačke; izvođenje se pravilno iscrtava.
- **Telefon (390 px):** svetla i tamna tema, meni, paneli i primeri. Raspored primera „emitorski sledilac“ je popravljen.

## Ostalo (čeka korisnika)
1. Provera na pravom telefonu (dodir, rotacija tranzistora).
2. Objava glavne verzije artefakta i APK V2.4 – tek kad korisnik kaže.
