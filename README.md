Below is a complete user guide you can publish on GitHub (as a `README.md` or in the project wiki). It’s tailored to your app’s actual features as seen in the code.

---

# Siglent Remote – User Guide

**Siglent Remote** is a macOS‑native application that lets you control a Siglent SDS1200X‑E (or compatible) oscilloscope over TCP/IP.  
You can acquire screenshots, read live measurements, change timebase and channel settings, capture raw waveform data, and send arbitrary SCPI commands – all from a clean, native SwiftUI interface.

---

## Table of Contents

1. [Connecting to the Oscilloscope](#connecting-to-the-oscilloscope)  
2. [Main Interface Overview](#main-interface-overview)  
   - [Acquisition Panel](#acquisition-panel)  
   - [Horizontal (Timebase) Panel](#horizontal-timebase-panel)  
   - [Channel 1 & 2 Panels](#channel-1--2-panels)  
   - [Screenshot Capture](#screenshot-capture)  
   - [Live Monitor](#live-monitor)  
   - [Measurement Panel](#measurement-panel)  
   - [Manual Command Console](#manual-command-console)  
   - [Log Panel](#log-panel)  
3. [Working with Waveforms](#working-with-waveforms)  
4. [Saving Data](#saving-data)  
5. [Troubleshooting](#troubleshooting)  
6. [SCPI Command Examples](#scpi-command-examples)  

---

## Connecting to the Oscilloscope

1. Make sure your oscilloscope is connected to the same local network as your Mac.  
2. On the oscilloscope, go to `Utility` → `I/O` → `LAN` and note its **IP address**.  
3. In the app, enter the IP address in the top‑right text field.  
4. Click **Connect**.  
   - The status label turns green and shows “Connesso”.  
   - A short log message confirms the connection.

> **Tip:** If connection fails, check that the oscilloscope’s LAN settings are correct and that no firewall blocks port 5025 (the default SCPI port).

---

## Main Interface Overview

The window is split into two main columns.

### Left Column – Oscilloscope Control

#### Acquisition Panel

- **Avvia (Run)** – sets trigger mode to `AUTO`.  
- **Stop** – stops the acquisition (`STOP` mode).  
- **Normal** – normal trigger mode.  
- **Single** – single trigger mode (captures one waveform then stops).  
- **Auto Setup** – sends `ASET` to automatically set vertical/horizontal scales.  
- **Default** – sends `*RST` to reset the oscilloscope to its factory settings.

#### Horizontal (Timebase) Panel

- **Scala (Scale)** – choose a time/div value from the dropdown. The current value is shown with a small purple icon.  
- **Posizione (Position)** – set the horizontal position in nanoseconds and click **Imposta**. Use **Azzera** to reset to zero.

#### Channel 1 & 2 Panels

- **ON / OFF** – enable/disable the channel.  
- **Scala (Scale)** – dropdown with voltage steps (10V … 500µV). The active scale is shown with a yellow (CH1) or cyan (CH2) icon.  
- **Offset** – set the vertical offset in volts.

#### Screenshot Capture

- Click **Cattura Screenshot** – the app requests a BMP screenshot from the scope.  
- When the image appears, you can:  
  - **Visualizza** – open the image in a zoomable window (⌘+, ⌘-, ⌘0).  
  - **Salva…** – save as BMP file.  
  - **Copia** – copy the image to the clipboard.

---

### Right Column – Monitoring & Tools

#### Live Monitor

- **Avvia / Stop** – starts or stops periodic polling of:  
  - Frequency (CYMT?)  
  - Trigger status (SAST?)  
  - Timebase (TDIV?)  
  - Channel V/div values  
- **Refresh interval** – choose 1s, 2s, 5s, or 10s.  
- **⟳ Button** – manually refresh all values once (stops monitoring while pushing).

#### Measurement Panel

- **Aggiorna misure** – reads the selected measurement for CH1 and CH2.  
- **📈 WF** – acquires a full waveform from CH1 (see [Working with Waveforms](#working-with-waveforms)).  
- For each channel you can choose a measurement type:  
  `FREQ` (frequency), `PKPK` (peak‑to‑peak voltage), `RMS`, `PER` (period), `MEAN`, `MIN`, `MAX`.  
- The current measurement value is displayed in large type.

#### Manual Command Console

- Type any SCPI command into the text field and press **Invia** or hit **Return**.  
- Quick buttons provide shortcuts for common queries:  
  `*IDN?`, `SARA?`, `TRDL?`, `TDIV?`, `SCDP` (screenshot), `C1:PRE` (waveform preamble), `C1:DAT2` (waveform data).  
- All command responses appear in the **Log** panel below.

#### Log Panel

- Shows raw responses from the oscilloscope.  
- **Pulisci** clears the log.

---

## Working with Waveforms

The app can retrieve, decode, and plot waveform data from **Channel 1** (support for CH2 can be added easily).

### How to acquire a waveform

1. Make sure CH1 is ON and the oscilloscope is running (trigger mode AUTO or NORMAL).  
2. Click the **📈 WF** button in the Measurement panel.  
   - The app temporarily stops any running monitoring.  
   - It then reads `C1:VDIV`, `C1:OFST`, `TDIV`, and `SARA` to obtain the necessary scaling parameters.  
   - Finally it requests the binary data with `C1:WF? DAT2`.  
3. The waveform is decoded using the formula from the Siglent manual:  
   `voltage = code × (vdiv / 25) - voffset`.  
4. A new window opens with the waveform plotted using Swift Charts.  

### Waveform window features

- **Zoomable chart** – you can pinch or scroll.  
- **Channel colour** – yellow for CH1, cyan for CH2.  
- **Save CSV** – exports time (in seconds) and voltage (in volts) as a CSV file.  
- The window title shows the channel name and the applied V/div and offset.

### Important notes

- The app automatically **downsamples** the waveform to a maximum of 10 000 points using a min‑max decimation algorithm – this preserves peaks while keeping the chart responsive.  
- If the binary transfer fails, check the connection and try again.

---

## Saving Data

- **Screenshot** → BMP file (Save… button).  
- **Waveform** → CSV file (Save CSV button in the waveform window).  
- **Manual responses** – you must copy the log manually (the app does not auto‑save logs).

---

## Troubleshooting

| Problem                       | Possible solution                                           |
|-------------------------------|-------------------------------------------------------------|
| Cannot connect to the scope   | Verify IP address, port 5025, and that the scope is not firewalled. |
| Screenshot doesn’t appear     | Check that the scope supports the `SCDP` command. Increase timeout? (The app waits 10s). |
| Waveform window shows no data | Make sure CH1 is ON and the scope is not in STOP mode. Try running the acquisition first. |
| Live monitor values show “--”  | The scope may not respond to that query; try sending the command manually from the console to see the raw reply. |
| Binary transfer hangs          | The network might be slow; try disconnecting and reconnecting. If the problem persists, restart the app. |

If you encounter a bug, please open an issue on the GitHub repository with the **log output** and a description of the steps.

---

## SCPI Command Examples

You can type any command supported by the oscilloscope. Below are some useful ones:

| Command            | Description                                          |
|--------------------|------------------------------------------------------|
| `*IDN?`            | Returns instrument identification string.            |
| `C1:VOLT_DIV?`     | Read CH1 V/div (equivalent to `C1:VDIV?`).           |
| `CHDR ON` / `CHDR OFF` | Turn command headers on/off in responses.        |
| `C1:WF? PRE`       | Read the waveform preamble (not used by the WF button, but useful for debugging). |
| `C1:WF? DAT2`      | Fetch binary waveform points (the app uses this).    |
| `TRIG_MODE?`       | Read current trigger mode.                           |
| `SARA?`            | Sample rate (Sa/s).                                  |

> **Note:** The app’s **📈 WF** button already handles the full sequence – you normally don’t need to type these manually.

---

## Support & Contributions

- **Bug reports & feature requests** → GitHub Issues  
- **Source code** – The app is written entirely in SwiftUI. Feel free to fork and adapt it for other Siglent models (e.g., SDS1104X‑E).  
- **Localisation** – Current UI is in Italian; English labels can be added by editing the strings.

---

**Enjoy remote controlling your oscilloscope from your Mac!**  
— *MK4001 Remote Team*
