# Configurazione Driver Silenziosi e Estrusore Orbiter v2.0

Questo documento descrive la taratura fisica ed elettrica dei driver passo-passo silenziosi e la configurazione Klipper specifica per il tuo nuovo estrusore **Orbiter v2.0** montato in Direct Drive.

---

## 1. Taratura Elettrica dei Driver Silenziosi (Modalità Standalone)

Dato che i tuoi driver TMC2208/TMC2209 sono installati in modalità **Standalone** (nessuna comunicazione UART software), devi regolare fisicamente la corrente erogata ai motori impostando la tensione di riferimento (**Vref**) tramite il potenziometro a vite presente su ciascun driver.

### ⚠️ Avvertenze di Sicurezza Importanti:
1. Utilizza esclusivamente un **cacciavite ceramico** (non conduttivo) per girare la vite. Un cacciavite metallico potrebbe scivolare e cortocircuitare il driver, distruggendolo all'istante o danneggiando la scheda madre.
2. Esegui la regolazione della vite a stampante spenta, quindi accendila solo per misurare il valore con il multimetro.
3. Posiziona il multimetro in modalità **DC Voltage (tensione continua)**. Il puntale nero (negativo) va collegato a una massa (GND) sulla scheda madre o sull'alimentatore. Il puntale rosso (positivo) va appoggiato delicatamente sulla testa metallica della vite del potenziometro sul driver.

### Valori di Vref Raccomandati:
* **Assi X e Y (Motori Originali):** 
  * Corrente consigliata: ~0.7A - 0.8A RMS.
  * **Vref target: 1.0V - 1.1V**
  * *Nota: I motori originali possono lavorare fino a 1.2V (circa 0.85A RMS), ma tenderanno a scaldarsi parecchio, soprattutto a stampante chiusa. Rimanere su 1.0V garantisce silenziosità e temperature ottimali senza perdita di passi.*
* **Asse Z (Motore Originale):**
  * Corrente consigliata: ~0.7A RMS.
  * **Vref target: 1.0V**
* **Estrusore E0 (Motore LDO Pancake Orbiter v2.0):**
  * **ATTENZIONE!** Il motore LDO dell'Orbiter v2.0 (`LDO-36STH20-1004AHG`) è molto più piccolo di quelli originali. Se riceve troppa corrente, si surriscalda rapidamente superando i 75°C, trasmettendo il calore all'ingranaggio interno e sciogliendo il filamento di PLA prima dell'hotend, causando intasamenti continui.
  * Corrente consigliata per standalone: **0.5A - 0.6A RMS max**.
  * **Vref target: 0.80V - 0.85V** *(Non superare mai 0.85V in standalone!)*

---

## 2. Configurazione Klipper per Orbiter v2.0

L'Orbiter v2.0 ha un rapporto di trasmissione interno a ingranaggi molto elevato. Per questo motivo, i parametri di estrusione in Klipper differiscono totalmente dall'estrusore BMG originale.

Nel file `motors.cfg`, la sezione dell'estrusore deve essere configurata così:

```ini
[extruder]
step_pin: PD6
dir_pin: !PD3                           # Aggiungi o rimuovi "!" se estrude al contrario
enable_pin: !PB3
microsteps: 32                          # 32 microstep garantiscono la massima silenziosità e fluidità
rotation_distance: 4.637                # Valore standard raccomandato da LDO per Orbiter v2.0
nozzle_diameter: 0.400
filament_diameter: 1.750
max_extrude_only_distance: 500          # Consente estrusioni lunghe per caricamento/scaricamento filamento
max_extrude_only_velocity: 120
```

### Note Fondamentali:
* **rotation_distance:** Il valore **`4.637`** è già comprensivo del rapporto di riduzione dell'Orbiter v2.0. **NON inserire la riga `gear_ratio`** sotto la sezione `[extruder]` se usi questo valore. L'inserimento combinato di gear_ratio e rotation_distance a 4.637 provocherebbe una sovra-estrusione massiccia (circa 7 volte il necessario) rischiando di danneggiare l'hotend.
* **Direzione del Motore (`dir_pin`):** A causa del Direct Drive e dell'ingranaggio interno, la direzione di rotazione dell'Orbiter v2.0 potrebbe essere invertita. 
  * Se noti che durante il caricamento del filamento il motore spinge il filo verso l'alto (scaricamento) invece che verso l'ugello, apri il file `motors.cfg` e modifica `dir_pin: PD3` in `dir_pin: !PD3` (o viceversa). Salva e riavvia Klipper.

---

## 3. Calibrazione Precisa dell'Estrusore (Test dei 100mm)

Anche se `4.637` è il valore teorico ideale, ogni stampante presenta tolleranze meccaniche. Ti consigliamo di calibrare la reale quantità di plastica estrusa seguendo questi passaggi:

1. Scalda l'ugello alla temperatura di stampa del tuo filamento (es. 200°C per il PLA).
2. Prendi un calibro o un righello e fai un segno sul filamento a **120 mm** di distanza dall'ingresso dell'estrusore Orbiter.
3. Tramite l'interfaccia web di Klipper (Mainsail/Fluidd), vai in console ed esegui i seguenti comandi per estrudere esattamente 100 mm di filamento a bassa velocità:
   ```gcode
   G92 E0
   G1 E100 F100
   ```
4. Al termine dell'estrusione, misura lo spazio rimasto tra l'ingresso dell'estrusore e il segno fatto in precedenza sul filamento.
   * Se l'estrusione è perfetta, dovrebbero rimanere esattamente **20 mm** (120mm totali - 100mm estrusi).
   * Se rimangono più di 20 mm (es. 25 mm), significa che l'estrusore ha sotto-estruso (ha estruso solo 95 mm).
   * Se rimangono meno di 20 mm (es. 15 mm), significa che l'estrusore ha sovra-estruso (ha estruso 105 mm).
5. Se la lunghezza estrusa non è corretta, calcola la nuova `rotation_distance` usando la formula:
   \[
   \text{Nuova rotation\_distance} = \text{Vecchia rotation\_distance} \times \frac{\text{Lunghezza Estrusa Effettiva}}{100}
   \]
   *Esempio:* Se hai impostato `4.637`, hai chiesto 100 mm ma l'estrusore ha consumato solo 95 mm di filo:
   \[
   \text{Nuova rotation\_distance} = 4.637 \times \frac{95}{100} = 4.405
   \]
6. Sostituisci il valore calcolato in `motors.cfg`, salva, riavvia Klipper e ripeti il test per conferma.

---

## 4. Analisi del file Excel di terze parti (Marlin vs Klipper come esempio)

Nel file Excel che hai scaricato (`Calibrazioni GH5 originale.xlsx`), che appartiene a un altro utente, sono presenti calibrazioni per un estrusore BMG basate sul firmware Marlin (che utilizza i passi per millimetro, o `step/mm`). 

Puoi usare questo file **esclusivamente come esempio metodologico** per capire come convertire e calcolare i valori, ma **NON devi applicare questi parametri sulla tua stampante**.

Ecco come interpretare i calcoli a scopo illustrativo:
* **Formula di conversione teorica:** In Klipper non si usano gli `step/mm`, ma la `rotation_distance`. La formula di conversione è:
  \[
  \text{rotation\_distance} = \frac{\text{passi\_del\_motore\_per\_giro} \times \text{microstep}}{\text{step/mm}}
  \]
* **Esempio di calcolo del BMG di terzi:**
  L'estrusore BMG di quell'utente è stato calibrato a **`417.60 step/mm`**. A 16 microstep (con motore standard da 200 passi/giro), questo equivaleva in Klipper a:
  \[
  \text{rotation\_distance} = \frac{200 \times 16}{417.60} \approx 7.662 \text{ mm}
  \]
* **Per il tuo nuovo Orbiter v2.0 (a 32 microstep):**
  Il valore nominale di partenza è **`4.637`**. Se volessi calcolare gli step/mm equivalenti per scopi di confronto, la formula sarebbe:
  \[
  \text{step/mm} = \frac{200 \times 32}{4.637} \approx 1380.20 \text{ step/mm}
  \]

### ⚠️ Nota importante sul Flusso (Flow Rate)
Nel file Excel di terze parti viene calcolato un flusso ottimizzato del **68.57%**. **Questo valore è estremamente basso e specifico per la stampante di quell'utente (o probabilmente errato/influenzato da altri problemi meccanici).**
* Con il tuo nuovo **Orbiter v2.0 in Direct Drive**, ti consigliamo caldamente di lasciare il flusso al **100%** (o 1.0 nello slicer) all'inizio.
* Esegui successivamente una calibrazione indipendente del flusso stampando un cubetto a singola parete (o usando i test nativi di OrcaSlicer) per trovare il valore corretto specifico per la tua macchina e il tuo filamento.
