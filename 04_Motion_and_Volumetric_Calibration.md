# Guida alla Calibrazione Cinematica e Portata Volumetrica: FBG5 & Orbiter v2.5

Questo documento raccoglie le ricerche e le migliori pratiche della community di Klipper per calibrare in modo scientifico la velocità, l'accelerazione, la portata volumetrica dell'hotend e le risonanze dell'asse (Input Shaper) sulla tua Flying Bear Ghost 5 dotata di estrusore **Orbiter v2.5 in Direct Drive**.

---

## 1. Calibrazione della Portata Volumetrica Massima (Max Volumetric Flow)

La velocità massima a cui puoi stampare non è limitata solo dai motori, ma soprattutto da **quanta plastica l'hotend riesce a fondere al secondo**.

*   **L'Hotend Originale (Stock Ghost 5):** È un hotend stile V6 Bowden/Direct a bassa portata. Il suo limite fisico reale per il PLA è compreso tra **`8 e 12 mm³/s`** (una media sicura è **`10 mm³/s`**).
*   **La formula del flusso:**
    $$\text{Flusso (mm³/s)} = \text{Altezza Layer (mm)} \times \text{Larghezza Linea (mm)} \times \text{Velocità di Stampa (mm/s)}$$
    *   Stampa a `120 mm/s` (layer `0.2mm`, larghezza `0.42mm`) richiede un flusso di:
        $$0.2 \times 0.42 \times 120 = 10.08\text{ mm³/s}$$
        *Questo si trova esattamente al limite fisico dell'hotend originale!*

### 🛠️ Azione su OrcaSlicer:
1.  Apri le impostazioni del Filamento in OrcaSlicer (`PLA GHOST5`).
2.  Trova il parametro **Max Volumetric Speed** (Velocità Volumetrica Massima). Attualmente era impostato a `15 mm³/s` (troppo alto per l'hotend originale).
3.  Impostalo a **`10 mm³/s`**.
    *   *Nota:* This parameter acts as a fundamental safety limit: if you try to print at higher speeds (e.g. 130-150 mm/s), OrcaSlicer will automatically and smoothly throttle the speed only in areas where you would exceed 10 mm³/s, preventing under-extrusion or extruder clogging.
4.  Per trovare il limite esatto del tuo specifico hotend, esegui il test integrato in OrcaSlicer: **Calibration -> Max Volumetric Speed**.

---

## 2. Ricalibrazione dell'Input Shaper (Risonanze)

Nel file `calibration.cfg` ereditato dal vecchio backup sono presenti questi valori:
*   `shaper_freq_x = 69.800` (con shaper `3hump_ei`)
*   `shaper_freq_y = 51.600` (con shaper `mzv`)

### ⚠️ Perché devi ricalibrarli:
L'estrusore **Orbiter v2.5 in Direct Drive** ha un peso e una distribuzione di massa completamente diversi rispetto all'estrusore originale a filo (Bowden) o ad altre modifiche. Avendo montato l'intero gruppo motore pancake + estrusore direttamente sul carrello X, la massa del carrello è aumentata.
Questo **cambia drasticamente la frequenza di risonanza dell'asse X**. Se utilizzi i vecchi valori, l'Input Shaper correggerà le risonanze su frequenze sbagliate, causando comunque ringing/ghosting.

### 🛠️ Azione di Calibrazione:
*   **Se hai un accelerometro (ADXL345):** Collegalo al carrello e lancia il comando:
    ```gcode
    SHAPER_CALIBRATE
    ```
    Klipper calcolerà automaticamente le nuove risonanze per X e Y e aggiornerà il file.
*   **Se non hai un accelerometro (Calibrazione Manuale):**
    1. Stampa il modello di test `ringing_tower.stl` (scaricabile dalla documentazione ufficiale di Klipper).
    2. Stampa a velocità elevata (80-100 mm/s) disattivando qualsiasi accelerazione dinamica nello slicer.
    3. Lancia il comando per variare l'accelerazione durante la stampa:
       ```gcode
       TUNING_TOWER COMMAND=SET_VELOCITY_LIMIT PARAMETER=ACCEL START=1500 FACTOR=100
       ```
    4. Misura con un calibro la distanza tra le onde (ghosting) sulla stampa e inserisci i dati nella formula di Klipper per calcolare le frequenze corrette.

---

## 3. Calibrazione dei Limiti di Velocità ed Accelerazione Massima

Non copiare parametri di accelerazione molto alti (es. 10000 mm/s²) dallo slicer se la stampante non è tarata fisicamente per sopportarli.

### 🛠️ La procedura di Ellis per i limiti meccanici:
Per trovare le massime prestazioni reali del tuo telaio senza rischiare la perdita di passi:
1.  Installa la macro **`TEST_SPEED`** (descritta nella celebre guida [Ellis' Print Tuning Guide](https://ellis3dp.com/Print-Tuning-Guide/)).
2.  Questa macro muove la stampante ad alta velocità e accelerazione in diagonale, per poi tornare al centro ed eseguire l'homing per verificare se i motori hanno perso passi (misurando se la posizione si è spostata).
3.  Lancia il test partendo da `max_accel: 3000` e aumenta gradualmente fino a trovare il limite della tua macchina. Per la Ghost 5 originale, una calibrazione sicura è:
    *   **Velocità Massima (`max_velocity`):** `150 - 200 mm/s`
    *   **Accelerazione Massima (`max_accel`):** `3000 - 4500 mm/s²` (con Input Shaper attivo).

---

## 4. Calibrazione fine della Rotation Distance (Estrusione 100mm)

Anche se abbiamo impostato il valore nominale di fabbrica di **`4.69`** per l'Orbiter v2.5, è caldamente consigliato eseguire la calibrazione fine dei 100mm:
1.  Riscalda l'ugello a 200°C.
2.  Fai un segno sul filamento a esattamente **120 mm** di distanza dall'ingresso dell'estrusore.
3.  Invia il comando da console per estrudere 100mm di filo a bassa velocità:
    ```gcode
    M83
    G1 E100 F100
    ```
4.  Misura la distanza rimasta dal segno all'ingresso dell'estrusore:
    *   Se è esattamente **20 mm**, la calibrazione è perfetta.
    *   Se è diversa (es. 24 mm, quindi ha estruso solo 96 mm), ricalcola il valore con la formula:
        $$\text{Nuovo rotation\_distance} = \text{Vecchio rotation\_distance} \times \frac{\text{Millimetri Estrusi Reali}}{100}$$

---

## 5. Livellamento del Piatto e Bed Mesh Manuale

Dato che la stampante non è dotata di sensore fisico (BLTouch/CRTouch) ma usa il finecorsa Z meccanico standard, Klipper gestisce la compensazione delle imperfezioni del piatto tramite una **Bed Mesh manuale** assistita tramite la console web.

### 🛠️ Step 1: Generare la Bed Mesh Manuale
1.  Riscalda il piatto alla tua temperatura d'uso abituale (es. 60°C per il PLA) e attendi 5 minuti affinché le dilatazioni termiche si stabilizzino.
2.  Su Mainsail, clicca o digita nella console il comando della macro personalizzata che abbiamo configurato:
    ```gcode
    CREA_MESH_MANUALE
    ```
3.  La stampante si muoverà in sequenza su **9 punti del piatto** (una griglia 3x3).
4.  Per ciascun punto, l'ugello si abbasserà vicino al piatto:
    *   Fai scorrere un foglio di carta da stampante standard sotto l'ugello.
    *   Usa l'interfaccia di Mainsail (pulsanti Z+ / Z-) per alzare o abbassare l'ugello finché la carta non fa una leggera resistenza (deve scorrere ma fare attrito, senza bloccarsi).
    *   Clicca su **Accept** per salvare la quota di quel punto e passare al successivo.
5.  Una volta completati tutti i 9 punti, digita in console per salvare permanentemente la mesh ed effettuare il restart del firmware:
    ```gcode
    SAVE_CONFIG
    ```
    *Questo salverà la mesh sotto il profilo denominato `default` nella sezione finale del tuo `printer.cfg`.*

### 🛠️ Step 2: Abilitare il Caricamento ad inizio stampa
Una volta salvata la mesh `default`, puoi fare in modo che OrcaSlicer la carichi automaticamente prima di ogni stampa:
1.  Apri **OrcaSlicer** e vai sulle impostazioni della stampante (`FlyingBear Ghost 5 0.4 nozzle New`).
2.  Nella scheda **Machine G-code**, individua il **Machine start G-code**.
3.  Trova la riga commentata:
    `; BED_MESH_PROFILE LOAD=default`
4.  **Rimuovi il punto e virgola `;`** per renderla attiva:
    `BED_MESH_PROFILE LOAD=default`
5.  Salva il profilo stampante. Adesso, Klipper applicherà la compensazione ad ogni stampa, correggendo le altezze Z in tempo reale durante i movimenti!

