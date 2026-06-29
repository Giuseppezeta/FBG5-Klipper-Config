# Calibrazione del Flusso (Flow Rate) e della Portata Volumetrica Massima (Max Volumetric Speed)

Questa guida tecnica documenta in modo dettagliato le procedure per eseguire, valutare e calcolare i valori ottimali del **Flusso del filamento** e della **Portata Volumetrica Massima** utilizzando OrcaSlicer e Klipper.

La guida è ottimizzata per la stampante **Flying Bear Ghost 5** equipaggiata con estrusore **Orbiter v2.5** (Direct Drive) e **ugello Volcano** con viti in rame per l'omogeneizzazione termica.

---

## 1. Calibrazione del Flusso (Flow Rate)

Il flusso (o moltiplicatore di estrusione) compensa l'espansione naturale della plastica all'uscita del nozzle. Tararlo previene sia la sotto-estrusione (fessure tra le linee) che la sovra-estrusione (superfici ruvide e grumi).

### 📋 Istruzioni Passo-Passo

#### **Passo 1: Test Preliminare (Pass 1)**
1. In OrcaSlicer, seleziona in alto a sinistra: **Calibration** -> **Flow Rate** -> **Pass 1**.
2. OrcaSlicer genererà un piatto con 9 piastrine rettangolari con valori da `-10` a `+5`.
3. *Ottimizzazione tempo/filamento:* Seleziona ed elimina con il tasto `Canc` i provini estremi `-10`, `-8` e `+5` (lascia solo quelli centrali da `-6` a `+4`).
4. Esegui lo Slice e avvia la stampa.

#### **Passo 2: Valutazione e Calcolo (Pass 1)**
Togli le piastrine dal piatto e passaci sopra il polpastrello, concentrandoti **solo sulla superficie piatta centrale** (ignora bordi e spigoli):
* **Sotto-estrusione (Ruvido/Fessurato):** Le linee non si toccano. Si avvertono fessure e vuoti al tatto.
* **Sovra-estrusione (Ruvido/Grumoso):** La plastica in eccesso crea piccole "creste" graffianti e accumuli di materiale ai bordi.
* **Flusso Ottimale:** La superficie è perfettamente liscia, compatta e uniforme al tatto.

Identifica il numero inciso sul provino migliore (es. `0`, `-2`, `+2`) e applica la formula:
$$\text{Nuovo Flow Ratio} = \text{Flow Ratio Attuale} \times \left(1 + \frac{\text{Valore Piastrina Migliore}}{100}\right)$$

*Esempio:* Con flusso attuale a `0.98` e piastrina migliore a `-2`:
$$\text{Nuovo Flow Ratio} = 0.98 \times (1 - 0.02) = \mathbf{0.96}$$

Inserisci questo valore in OrcaSlicer all'interno del profilo del filamento nella scheda **Filament** -> **Flow ratio**.

#### **Passo 3: Test di Rifinitura (Pass 2 - Opzionale)**
Se vuoi una precisione assoluta, imposta il nuovo valore calcolato nel filamento e seleziona **Calibration** -> **Flow Rate** -> **Pass 2**. Verranno stampate piastrine con variazioni piccolissime (da `-4` a `+4`). Ripeti la valutazione e la formula per centrare il valore perfetto.

---

### 🎯 Risultato Atteso (Flusso)
Superfici superiori dei pezzi stampati completamente speculari, piatte al tatto, prive di micro-fessure tra i passaggi dell'ugello e prive di graffi o grumi di plastica sollevata.

---

## 2. Portata Volumetrica Massima (Max Volumetric Speed)

La portata volumetrica massima stabilisce il limite termico reale del tuo hotend: quanta plastica al secondo riesce a fondere la stampante prima che l'estrusore inizi a slittare o a perdere passi.

> [!IMPORTANT]
> Avendo installato un **ugello Volcano** più lungo (con melt zone estesa) ed ottimizzato termicamente con **viti in rame**, la tua capacità di fusione è nettamente superiore a quella stock. Ci aspettiamo un limite intorno ai **18 - 22 mm³/s** (rispetto ai 10 mm³/s originali).

### 📋 Istruzioni Passo-Passo

1. In OrcaSlicer, seleziona in alto a sinistra: **Calibration** -> **Max Volumetric Speed**.
2. Configura i parametri per la rampa di test:
   * **Start Volumetric Speed (Inizio):** `5 mm³/s`
   * **End Volumetric Speed (Fine):** `25 mm³/s` (ideale per testare i limiti del Volcano)
   * **Step (Passo):** `0.5`
3. OrcaSlicer genererà una torre che sale in altezza. Man mano che sale, la stampante aumenterà progressivamente la velocità di estrusione.
4. Esegui lo Slice e avvia la stampa.

---

### 🔍 Come Misurare e Valutare il Risultato

Durante la stampa della torre, osserva attentamente le pareti esterne a diverse altezze. Ad un certo punto, a causa del limite di fusione dell'ugello, noterai dei difetti:
1. La superficie esterna passerà da **lucida** a **opaca**.
2. Salendo ancora, inizieranno a comparire **linee mancanti, buchi, sfibrature** o l'estrusore inizierà a fare dei "clic" saltando passi.

```text
    Altezza Torre
        ▲
        │  [SFIBRATO / BUCHI] ◄─── Limite superato (sotto-estrusione meccanica)
  H_max │  [OPACIZZAZIONE]    ◄─── INIZIO DEL LIMITE REALE (Misura l'altezza Z a questo punto)
        │  [LUCIDO / PERFETTO]◄─── Stampa corretta e fluida
        └───────────────────────
```

Prendi un calibro e misura l'altezza in millimetri ($h$) dal fondo della torre fino al punto preciso in cui è iniziato il difetto (opacizzazione o sfibratura). Applica la formula:

$$\text{Portata Limite} = \text{Portata Iniziale} + \left( \frac{h}{H_{\text{tot}}} \times (\text{Portata Finale} - \text{Portata Iniziale}) \right)$$

*Dove:*
*   $\text{Portata Iniziale} = 5$
*   $\text{Portata Finale} = 25$
*   $H_{\text{tot}}$ (Altezza totale della torre stampata) = Di solito `100 mm` (verifica sullo slicer).
*   $h$ = Altezza misurata col calibro al punto del difetto.

#### **Margine di Sicurezza (Fattore di Sicurezza)**
Una volta trovata la Portata Limite, applica un margine di sicurezza del **90%** per evitare di stampare al limite termico assoluto:
$$\text{Portata Max Sicura} = \text{Portata Limite} \times 0.90$$

*Esempio:* Se riscontri il difetto a $50\text{ mm}$ di altezza su una torre da $100\text{ mm}$:
$$\text{Portata Limite} = 5 + \left( \frac{50}{100} \times (25 - 5) \right) = 5 + (0.5 \times 20) = 15\text{ mm}^3/\text{s}$$
$$\text{Portata Max Sicura} = 15 \times 0.90 = \mathbf{13.5\text{ mm}^3/\text{s}}$$

Inserisci il valore calcolato in OrcaSlicer nel profilo del filamento sotto la voce **Filament** -> **Max volumetric speed**.

---

### 🎯 Risultato Atteso (Portata Massima)
OrcaSlicer limiterà automaticamente la velocità massima di stampa durante i riempimenti e le pareti interne per non superare mai la portata volumetrica sicura impostata, eliminando per sempre il rischio di sotto-estrusione o intasamenti dovuti a velocità eccessive.
