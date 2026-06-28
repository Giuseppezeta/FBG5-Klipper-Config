# Istruzioni Eseguibili per Agenti: Installazione Klipper Server via KIAUH

Questo documento contiene istruzioni strutturate specificamente per un **Agente di Codifica AI** per eseguire l'installazione e la configurazione del server Klipper su un Mini PC (Host Linux Debian/Ubuntu-based) tramite **KIAUH** (Klipper Installation And Update Helper).

---

## 🤖 Istruzioni per l'Agente Esecutore

Caro Agente, il tuo compito è configurare il server Klipper, Moonraker e l'interfaccia web Mainsail su questo Mini PC. Esegui le fasi in sequenza, monitorando attentamente gli output del terminale.

---

### Fase 1: Verifica del Sistema Operativo e Privilegi
Prima di iniziare, verifica che l'ambiente sia compatibile (Debian/Ubuntu/Raspbian) ed esegui i comandi di aggiornamento iniziale.

1. **Esegui il comando di aggiornamento e installazione dipendenze:**
   ```bash
   sudo apt update && sudo apt upgrade -y && sudo apt install git curl python3 python3-pip -y
   ```
2. **Aggiungi l'utente corrente ai gruppi dialout e tty** per garantire l'accesso alle porte seriali USB della stampante:
   ```bash
   sudo usermod -a -G dialout,tty $USER
   ```
   *Nota per l'agente: L'utente potrebbe dover rieffettuare il login o riavviare la sessione SSH per rendere effettivi i gruppi.*

---

### Fase 2: Installazione di KIAUH
1. **Clona il repository ufficiale di KIAUH:**
   ```bash
   git clone https://github.com/dw-0/kiauh.git
   ```

---

### Fase 3: Esecuzione Interattiva di KIAUH
Poiché KIAUH è un'interfaccia a menu basata su terminale, l'agente deve avviare lo script ed inviare input specifici in risposta alle schermate del terminale.

#### Workflow di Interazione:
1. **Avvia KIAUH:**
   ```bash
   ./kiauh/kiauh.sh
   ```
2. **Schermata Principale (Main Menu):**
   * *Output atteso:* Un menu numerato con opzioni come `1) [Install]`, `2) [Uninstall]`, ecc.
   * *Azione:* Invia `1` e premi Invio per andare al menu di installazione.
3. **Menu di Installazione:**
   * *Azione 3.1 (Klipper):* Invia `1` (Klipper) e premi Invio.
     * Alla domanda sull'interprete python, seleziona **Python 3** (solitamente inserendo `1` o premendo Invio).
     * Alla domanda sul numero di istanze da installare, invia `1`.
     * Se richiesto di confermare l'installazione delle dipendenze, invia `Y`.
     * Attendi il completamento (l'output mostrerà il riepilogo dell'installazione).
   * *Azione 3.2 (Moonraker):* Invia `2` (Moonraker) e premi Invio.
     * Se richiesto di procedere con l'installazione, invia `Y`.
     * Attendi che finisca.
   * *Azione 3.3 (Mainsail):* Invia `3` (Mainsail) e premi Invio.
     * Se richiesto di procedere, invia `Y`.
     * Se richiesto di installare i macro consigliati o configurazioni aggiuntive per Mainsail, conferma con `Y`.
     * Attendi il completamento.
4. **Uscita dallo script:**
   * Quando tutte le installazioni sono completate con successo (vedrai la spunta `[V]` verde accanto a Klipper, Moonraker e Mainsail nel menu di installazione):
     * Invia `B` (Back) per tornare al menu principale.
     * Invia `Q` (Quit) per uscire da KIAUH.

---

### Fase 4: Verifica dei Servizi
Verifica che tutti i servizi di Klipper siano attivi e funzionanti in background.

1. **Verifica lo stato di Klipper:**
   ```bash
   systemctl status klipper
   ```
   *Output atteso:* Lo stato deve mostrare `active (running)`.
2. **Verifica lo stato di Moonraker:**
   ```bash
   systemctl status moonraker
   ```
   *Output atteso:* Lo stato deve mostrare `active (running)`.

---

### Fase 5: Identificazione della Stampante (MCU)
1. **Collega la stampante Flying Bear Ghost 5 al Mini PC tramite cavo USB** e accendila.
2. **Esegui il comando per trovare l'ID univoco del dispositivo USB:**
   ```bash
   ls /dev/serial/by-id/*
   ```
   *Output atteso:* Una stringa simile a `/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0`.
   *Azione per l'agente:* Copia questo indirizzo e incollalo nella sezione `[mcu]` all'interno del file `printer.cfg`.

---

### Fase 6: Copia dei File di Configurazione
Una volta installato Mainsail, le configurazioni Klipper si trovano in `/home/<utente>/printer_data/config/`.

1. **Copia tutti i file di configurazione generati per la FBG5** (`printer.cfg`, `motors.cfg`, `calibration.cfg`, `printing.cfg`, `others.cfg`) in quella directory.
2. **Riavvia Klipper** per caricare la nuova configurazione:
   ```bash
   sudo systemctl restart klipper
   ```
3. L'interfaccia web sarà ora accessibile all'indirizzo IP del Mini PC (es. `http://<IP_MINI_PC>`).
