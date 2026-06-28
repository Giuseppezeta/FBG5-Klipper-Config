# Compilazione e Flash del Firmware MKS Robin Nano v1.3

La scheda madre della Flying Bear Ghost 5 è una **MKS Robin Nano v1.3** (basata principalmente sul microcontrollore **STM32F407**). Per far comunicare Klipper con la stampante, dobbiamo compilare il firmware corretto e caricarlo sulla scheda madre.

---

## 1. Compilazione del Firmware su Raspberry Pi

1. Collegati in SSH al tuo Raspberry Pi.
2. Entra nella cartella di Klipper:
   ```bash
   cd ~/klipper
   ```
3. Avvia il menu di configurazione:
   ```bash
   make menuconfig
   ```
4. Configura le opzioni esattamente come indicato di seguito per la **MKS Robin Nano v1.3 (STM32F407)**:
   * **[*] Enable extra low-level configuration options** (premi Spazio per abilitarlo)
   * **Micro-controller Architecture:** `STMicroelectronics STM32`
   * **Processor model:** `STM32F407`
   * **Bootloader offset:** `48KiB bootloader`
   * **Clock Reference:** `8 MHz crystal`
   * **Communication interface:** `USB (on PA11/PA12)`
5. Seleziona **Exit** (premi `Q`), conferma il salvataggio premendo `Y`.
6. Compila il firmware eseguendo:
   ```bash
   make clean
   ```
   ```bash
   make
   ```
   *Questo genererà il file compilato in `~/klipper/out/klipper.bin`.*

---

## 2. Preparazione del File di Flash per la Scheda Madre

Il bootloader della MKS Robin Nano non accetta il file standard `klipper.bin`. Dobbiamo rinominarlo affinché la scheda lo riconosca durante l'avvio.

1. Identifica il nome richiesto: per la Robin Nano v1.3 (e lo schermo TFT35), il file deve essere rinominato in **`Robin_nano35.bin`**.
2. Copia il file rinominato sul tuo computer. Puoi farlo usando un client SFTP (come *FileZilla* o *WinSCP*) connettendoti con le stesse credenziali SSH, oppure via riga di comando sul tuo computer Windows:
   ```bash
   scp pi@<IP_DEL_RASPBERRY>:~/klipper/out/klipper.bin C:\Users\Giuse\Desktop\Robin_nano35.bin
   ```
   *(Sostituisci `<IP_DEL_RASPBERRY>` con l'IP effettivo).*

---

## 3. Flash della Stampante

1. Prendi una micro SD (massimo 32GB, formattata in **FAT32** con dimensione unità di allocazione impostata su **4096 byte**).
2. Copia il file `Robin_nano35.bin` direttamente nella radice della micro SD (non dentro sottocartelle).
3. Spegni completamente la stampante 3D.
4. Inserisci la micro SD nello slot della stampante Flying Bear Ghost 5.
5. Accendi la stampante.
6. Lo schermo LCD potrebbe rimanere spento o mostrare una barra di aggiornamento / scritta "Booting..." per circa 10-15 secondi. Questo è normale, poiché Klipper disattiva la gestione dello schermo originale.
7. Spegni la stampante, rimuovi la micro SD e inseriscila nel computer. Se il flash è andato a buon fine, il file sulla micro SD sarà stato rinominato in **`Robin_nano35.CUR`** (Current).

---

## 4. Caso Alternativo: Chip STM32F103 (Robin Nano v1.1 / v1.2 o varianti v1.3)

Se il flash con STM32F407 fallisce (il file non viene rinominato in `.CUR` o Klipper non si connette), la tua scheda potrebbe montare un chip **STM32F103**. In tal caso, la procedura di compilazione cambia come segue:

1. Riapri `make menuconfig` in `~/klipper`:
   * **Micro-controller Architecture:** `STMicroelectronics STM32`
   * **Processor model:** `STM32F103`
   * **Bootloader offset:** `28KiB bootloader`
   * **Communication interface:** `Serial (on USART3 PB11/PB10)`
2. Salva e compila con `make`.
3. Per firmare il file per il bootloader STM32F103, devi eseguire lo script di Klipper prima di copiarlo:
   ```bash
   ./scripts/update_mks_robin.py out/klipper.bin out/Robin_nano35.bin
   ```
4. Carica il file `Robin_nano35.bin` risultante sulla micro SD e procedi al flash come descritto sopra.
