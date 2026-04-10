# InkTime v6 — Smartwatch cu E-Paper

## 1. Diagramă Bloc

Proiectul este construit în jurul microcontrollerului `nRF52840`, care gestionează comunicația wireless (Bluetooth), afișajul și senzorii.

- Creier: `nRF52840` (MCU + BLE)
- Afișaj: E-Paper (consum ultra-redus)
- Senzori: IMU `BMA421` pentru detectarea mișcării și gesturilor
- Power Management: Încărcător LiPo, Fuel Gauge (monitorizare baterie) și convertoare DC/DC
- Interfață Utilizator: 3 butoane fizice și un motor haptic pentru notificări

## 2. Bill Of Materials (BOM)

| Componentă | Descriere                 | Link Achiziție (JLC) | Datasheet |
| ---------- | ------------------------- | -------------------- | --------- |
| nRF52840   | MCU Bluetooth 5.4         | JLC PartsLink        | Datasheet |
| BMA421     | Accelerometru 3 axe (IMU) | JLC PartsLink        | Datasheet |
| BQ25180    | Li-Ion Charger IC         | JLC PartsLink        | Datasheet |
| MAX17048   | Fuel Gauge (Baterie)      | JLC PartsLink        | Datasheet |
| DRV2605    | Haptic Driver (LRA/ERM)   | JLC PartsLink        | Datasheet |
| RT610AWSC  | Buck-Boost DC/DC          | JLC PartsLink        | Datasheet |

## 3. Descrierea Detaliată a Funcționalității Hardware

### Microcontroller (MCU)

Inima sistemului este `nRF52840`, ales pentru suportul nativ de Bluetooth 5, consumul extrem de mic în deep-sleep și arhitectura ARM Cortex-M4F capabilă pentru procesări complexe (de ex. algoritmi pentru fitness și procesare IMU).

### Afișaj (E-Paper)

- Conector de 24 pini pentru ecranul E-Paper.
- Ecranul este alimentat printr-un E-Paper Drive Circuit dedicat (vezi pagina 2 din schemă).
- Circuitul include un boost (L5, D2, D4, D5) pentru generarea tensiunilor necesare refresh-ului.
- Controlul afișajului se face printr-o interfață SPI dedicată pentru operații rapide de refresh.

### Managementul Energiei

- Încărcare: port USB-C gestionat de `BQ25180`.
- Monitorizare: `MAX17048` comunică prin I2C și raportează SoC fără rezistor extern de descărcare.
- Reglare: `RT610AWSC` asigură o tensiune stabilă de 3.3V când bateria LiPo scade sub prag critic.

### Senzori și Feedback

- `BMA421` (IMU): conectat prin I2C — folosit pentru numărare pași și detecție „wrist flip” (aprinderea ecranului la mișcare).
- `DRV2605` (Haptic Driver): oferă vibrații avansate (LRA/ERM) pentru notificări și experiență premium.

## 4. Mapare și descriere pini nRF52840

Mai jos este o descriere adaptată a pinilor principali ai `nRF52840` prezentați în schemă. Am reformulat puțin explicațiile pentru claritate.

- `VDD` — alimentarea de 3.3V pentru SoC.
- `DCC` — (semnal de control intern, documentația schemei trebuie consultată pentru detalii precise).
- `DEC4`, `DEC3`, `DEC5`, `DEC1` — pini de decuplare/masă folosiți de circuitele interne de reglare a tensiunii (folosiți pentru integritatea alimentării).
- `VSS` — masă (GND).
- `P0.02 / AIN0` — pin folosit pentru semnale de tip SPI: SCK (clock) și, în unele variante de rutare, MOSI (Master Out Slave In). În schemă acesta servește ca linie de ceas pentru tranzacțiile cu display-ul.
- `DEC2` — masă (GND) pentru etajele asociate.
- `XC2`, `XC1` — pini pentru cristalul/external oscillator X1 (32 MHz).
- `ANT` — conexiunea către amplificator/antena 2.4GHz (prin rețea de potrivire).
- `P0.10 / NFC2` — pin folosit, probabil, pentru semnalizarea Fuel Gauge-ului către MCU atunci când bateria scade sub un prag (interrupt/alert).
- `SWDIO`, `SWDCLK`, `SWO` — pini dedicati debugging/programare (SWD).
- `P1.18 / RESET` — pinul de reset (reset hardware al microcontroller-ului).
- `P0.17`, `P0.16`, `P0.15` — pini alocați comunicării și controlului display-ului (transfer de date / comenzi SPI sau semnale de control).
- `D-`, `D+` — liniile diferențiale de date USB pentru tranzacționarea pachetelor USB.
- `VBUS` — alimentarea USB (5V) pe portul USB-C.
- `DEC3V3` — etichetat în schemă lângă GND; pare legat de blocurile de decuplare ale rail-ului 3.3V (verificare recomandată în schematică).
- `P0.12` — enable/enable-driver pentru controlul driverului haptic (`DRV2605` sau MOSFET-ul asociat).
- `P0.11` — semnal de monitorizare/alertă venit de la LiPo charger (semnalizare a stării de încărcare/erori alimentare).
- `P1.08`, `P0.08` — pini folosiți ca intreruperi (IRQ) pentru modulul IMU (`BMA421`).
- `P0.07`, `P0.06` — linii folosite de protocolul I2C (SDA / SCL) în această implementare.
- `P0.05 / AIN3` — chip select (CS) pentru display.
- `XL2`, `XL1` — pini pentru cristalul/external oscillator X2 (32.768 kHz) folosit de RTC.
- `P0.13`, `P0.14`, `P1.02` — GPIO alocate pentru citirea butoanelor fizice.

Observație: pinii etichetați `DECx` sunt în principal legați de circuitele de decuplare/reglare internă ale SoC-ului — ei apar frecvent în schemă pentru stabilitatea alimentării.

## 5. Design Log și Note Review

- USB-C: implementat cu protecție ESD pe liniile de date (`USBLC6-2`) pentru a proteja MCU-ul de descărcări electrostatice.
- E-Paper VCC Control: utilizare MOSFET (`Q1`) pentru a întrerupe complet alimentarea etajului ecranului când nu este în refresh, maximizând autonomia bateriei.