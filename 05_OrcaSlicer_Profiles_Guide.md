# Guida ai Profili di OrcaSlicer per Flying Bear Ghost 5 e Orbiter v2.5

Questo documento descrive la configurazione, i parametri e le ottimizzazioni applicate ai profili di stampa di OrcaSlicer per la **Flying Bear Ghost 5** equipaggiata con estrusore **Orbiter v2.5 in Direct Drive** e firmware **Klipper**.

I profili sono suddivisi per due scenari d'uso: stampa standard con ugello da **`0.4 mm`** e stampa ad alta precisione con ugello da **`0.2 mm`**.

---

## 1. Mappa dei Profili Attivi e Backup

I profili modificati sono attivi in locale nella cartella AppData di Windows ed è presente una copia di backup nel repository GitHub all'interno della cartella:
📂 **`orcaslicer_profiles/`**

| Tipo Profilo | Ugello 0.4 mm (Standard/Fast) | Ugello 0.2 mm (Alta Risoluzione) |
| :--- | :--- | :--- |
| **Stampante (Machine)** | `FlyingBear Ghost 5 0.4 nozzle New.json` | `FlyingBear Ghost 5 0.2 nozzle.json` |
| **Filamento (Filament)** | `FlyingBear Generic PLA - Copy.json` | `FlyingBear Generic PLA - Copy @0.2 nozzle.json` |
| **Processo (Slicing)** | `0.20mm Standard ... -new.json` / `FAST PLA.json` | `0.08 mm @FlyingBear Ghost 5 0.2 nozzle.json` |

---

## 2. Dettagli dei Parametri Ottimizzati

### A. Estrusore e Ritrazione (Direct Drive LDO Orbiter v2.5)
Gli estrusori Direct Drive hanno un percorso del filamento cortissimo. Valori di ritrazione elevati (tipici del sistema Bowden originale) tirano la plastica fusa nella zona fredda dell'hotend, provocando intasamenti.
*   **Distanza di Ritrazione:**
    *   *Ugello 0.4 mm:* **`1.0 mm`** (Valore ottimale di compromesso).
    *   *Ugello 0.2 mm:* **`0.8 mm`** (Distanza ridotta per prevenire vuoti nella punta stretta).
*   **Velocità di Ritrazione:** **`40 mm/s`** (Sia ritrazione che deretrazione).
    *   *Motivazione:* L'Orbiter ha una riduzione di 7.5:1. Velocità di ritrazione superiori (es. 90 mm/s) forzano il motore pancake LDO a girare a frequenze eccessive, causando surriscaldamento o perdita di passi.

### B. Portata Volumetrica Massima (Max Volumetric Flow)
La portata volumetrica limita la velocità di stampa in base alla capacità di fusione termica dell'hotend.
*   **Ugello 0.4 mm:** **`10.0 mm³/s`** (Hotend originale Ghost 5).
    *   *Calcolo:* A `120 mm/s` con layer `0.2 mm` e larghezza `0.42 mm`, la portata richiesta è `10.08 mm³/s`. Lo slicer rallenterà dinamicamente solo se la portata volumetrica rischia di superare il limite termico.
*   **Ugello 0.2 mm:** **`2.0 mm³/s`**.
    *   *Motivazione:* Il foro da 0.2 mm è 1/4 rispetto a quello da 0.4 mm. Tentare di stampare a portate elevate genera una contropressione che blocca l'estrusione. Impostare `2.0 mm³/s` è un limite di sicurezza invalicabile che protegge l'estrusore.

### C. Velocità, Accelerazioni e Jerk (SCV)
I limiti dello slicer sono stati allineati ai limiti reali del firmware Klipper (`max_accel: 3000`) per evitare conflitti e vibrazioni strutturali.
*   **Velocità Primo Layer:** **`30 mm/s`** (0.4 mm) / **`20 mm/s`** (0.2 mm) per garantire la massima adesione iniziale al piatto.
*   **Velocità Spostamenti (Travel):** Alzata a **`150 mm/s`** per ridurre i tempi morti sfruttando i driver silenziosi TMC2209.
*   **Accelerazioni:**
    *   *Pareti esterne (Qualità):* **`1500 mm/s²`** (0.4 mm) / **`1000 mm/s²`** (0.2 mm) per eliminare del tutto il ghosting/ringing.
    *   *Pareti interne / Spostamenti:* **`3000 mm/s²`** (0.4 mm) / **`1500 - 2500 mm/s²`** (0.2 mm).
*   **Jerk (Square Corner Velocity):**
    *   Ridotto da `15 mm/s` a **`8 mm/s`** (0.4 mm) e **`5 - 6 mm/s`** (0.2 mm) per ammorbidire i cambi di direzione negli spigoli stretti ed evitare colpi meccanici violenti sul telaio.

---

## 3. Configurazione dello Start & End G-Code (Machine Profile)

Lo Start G-code è stato ottimizzato per Klipper per evitare conflitti con la calibrazione del piatto (Bed Mesh manuale):

### Start G-code (Ugello 0.4 mm)
```gcode
M104 S[nozzle_temperature_initial_layer] ; Imposta temp ugello provvisoria (evita colature)
M140 S[heatbed_temperature_initial_layer] ; Imposta temp piatto
G90 ; Coordinate assolute
M83 ; Estrusore in modalità relativa
G28 ; Homing di tutti gli assi
; BED_MESH_PROFILE LOAD=default ; Commentato - Scommenta solo dopo aver salvato la mesh in Klipper
G1 Z10 F3000 ; Alza l'ugello per sicurezza
M190 S[heatbed_temperature_initial_layer] ; Attendi temp piatto stabili
M109 S[nozzle_temperature_initial_layer] ; Attendi temp ugello stabili
; --- Linea di Spurgo (Purge Line) ---
G1 X2 Y10 Z0.28 F3000 ; Vai all'inizio
G1 Y140 E10 F1200 ; Traccia prima riga
G1 X2.4 F3000 ; Spostamento laterale
G1 Y10 E20 F1200 ; Traccia seconda riga indietro
G92 E0 ; Azzera estrusore
```
*(Per il profilo da 0.2 mm la linea di spurgo estrude solo `6mm` e `12mm` per evitare sovrapressioni nell'ugello stretto).*

---

## 4. Manutenzione Futura e Calibrazioni

### Come ripristinare o importare i profili in OrcaSlicer
Se installi OrcaSlicer su un nuovo PC:
1.  Scarica i file JSON dal tuo repository GitHub (cartella `orcaslicer_profiles`).
2.  In OrcaSlicer, seleziona **File -> Import -> Import Configs...** e seleziona i file scaricati.

### Come abilitare il livellamento automatico della Bed Mesh manuale
Quando calibrerai il piatto con la macro `CREA_MESH_MANUALE` e salverai con `SAVE_CONFIG`:
1.  Apri le impostazioni della stampante in OrcaSlicer.
2.  Nel **Machine start G-code**, individua la riga `; BED_MESH_PROFILE LOAD=default`.
3.  Rimuovi il punto e virgola `;` per scommentarla: `BED_MESH_PROFILE LOAD=default`.
4.  Salva il profilo. Da quel momento, Klipper caricherà e applicherà la compensazione ad ogni stampa.
