# Configurazione del Raspberry Pi (MainsailOS)

Questa guida ti guiderà passo dopo passo nella preparazione della nuova micro SD per il tuo Raspberry Pi e nell'installazione del software necessario per gestire la stampante.

---

## 1. Scrittura del Sistema Operativo su Micro SD

La scelta consigliata per Klipper è **MainsailOS** (o in alternativa **FluiddPI**), un'immagine del sistema operativo Debian pre-configurata che contiene già Klipper, Moonraker e l'interfaccia web Mainsail.

1. Scarica e installa sul tuo PC il programma ufficiale **[Raspberry Pi Imager](https://www.raspberrypi.com/software/)**.
2. Inserisci la micro SD nel computer.
3. Apri Raspberry Pi Imager e configura le seguenti impostazioni:
   * **Dispositivo Raspberry Pi:** Seleziona il tuo modello (es. Raspberry Pi 3, 4 o Zero 2 W).
   * **Sistema Operativo:** Fai clic su *Scegli S.O.* -> vai su **Other specific-purpose OS** -> **3D Printing** -> **Mainsail OS** -> seleziona la versione consigliata (solitamente MainsailOS a 32 o 64 bit).
   * **Memoria:** Seleziona la tua micro SD.
4. Fai clic sul pulsante delle **impostazioni (ingranaggio)** prima di scrivere per configurare i parametri iniziali:
   * **Abilita SSH:** Scegli "Usa autenticazione con password".
   * **Imposta nome utente e password:** (Ad esempio, utente `pi` e una password a tua scelta. Ricorda queste credenziali!).
   * **Configura Wi-Fi:** Inserisci il nome della tua rete Wi-Fi (SSID) e la password. Imposta il codice del paese (es. `IT` per l'Italia).
5. Salva e fai clic su **Scrivi**. Attendi il completamento della scrittura e della verifica.

---

## 2. Primo Avvio e Identificazione dell'IP

1. Rimuovi la micro SD dal PC e inseriscila nel Raspberry Pi.
2. Collega il Raspberry Pi all'alimentatore per avviarlo. Attendi circa 2-3 minuti per il primo avvio.
3. Trova l'indirizzo IP locale assegnato al Raspberry Pi. Puoi farlo nei seguenti modi:
   * Accedendo alla pagina di amministrazione del tuo router Wi-Fi.
   * Utilizzando un software di scansione di rete (es. *Fing* da smartphone o *Angry IP Scanner* da PC).
   * Provando a connetterti all'indirizzo locale predefinito digitando sul browser: `http://mainsailos.local` o `http://mainsail.local`.

---

## 3. Connessione SSH e Aggiornamenti

Per configurare Klipper e compilare il firmware, dobbiamo connetterci al terminale del Raspberry Pi tramite SSH.

1. Apri un terminale (PowerShell su Windows, o terminale su Mac/Linux) e digita il comando:
   ```bash
   ssh pi@<IP_DEL_RASPBERRY>
   ```
   *(Sostituisci `<IP_DEL_RASPBERRY>` con l'IP trovato prima, ad esempio `ssh pi@192.168.1.50`).*
2. Accetta l'avviso di sicurezza digitando `yes` e premi Invio.
3. Inserisci la password impostata in Raspberry Pi Imager (i caratteri digitati non saranno visibili, premi Invio al termine).
4. Una volta effettuato l'accesso, aggiorna il sistema operativo eseguendo questi due comandi in sequenza:
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```
5. Riavvia il Raspberry Pi per rendere effettivi gli aggiornamenti:
   ```bash
   sudo reboot
   ```
   La connessione SSH verrà interrotta. Attendi 1 minuto e riconnettiti nuovamente tramite SSH.

---

## 4. Installazione tramite KIAUH (Opzionale / Alternativa)

Se preferisci installare tutto manualmente su un'installazione pulita di Raspberry Pi OS Lite, puoi usare **KIAUH** (Klipper Installation And Update Helper).

1. Esegui la connessione SSH.
2. Installa Git se non è presente:
   ```bash
   sudo apt install git -y
   ```
3. Scarica KIAUH sul Raspberry Pi:
   ```bash
   git clone https://github.com/dw-0/kiauh.git
   ```
4. Esegui il tool:
   ```bash
   ./kiauh/kiauh.sh
   ```
5. Tramite l'interfaccia a riga di comando di KIAUH, installa nell'ordine:
   * **Klipper** (opzione 1)
   * **Moonraker** (opzione 2)
   * **Mainsail** o **Fluidd** (opzione 3 o 4)

Una volta terminato, sarai pronto per compilare il firmware per la tua scheda madre.
