# System Dwukanałowego Sniffera CAN - Płytka (Hardware)

![Render 3D zmontowanej płytki](Photos/Front%20Render.png)

## Opis projektu

Repozytorium zawiera projekt sprzętowy (hardware / PCB) dwukanałowego sniffera CAN, zaprojektowanego na potrzeby telemetrii bolidu Formuły Student.
Głównym zadaniem układu jest bezinwazyjny nasłuch (tryb _silent / bus monitoring_) dwóch niezależnych magistral CAN w pojeździe i przekazywanie zagregowanych ramek przez szybki interfejs UART do zewnętrznego modułu komunikacyjnego (np. modułu radiowego).

Układ został oparty o wydajny mikrokontroler **STM32G484RET6**. Na płytce przewidziano również dedykowany interfejs komunikacyjny dla modułu ESP32, który docelowo ma pełnić rolę nadrzędnego kontrolera filtrującego i konfigurującego.

> **Uwaga:** To repozytorium zawiera wyłącznie projekt płytki drukowanej (schematy, pliki Gerber, BOM). Oprogramowanie układowe (firmware) odpowiedzialne za formatowanie ramek CAN do 12-bajtowych rekordów binarnych znajduje się w osobnym repozytorium.

## Główne parametry sprzętowe

- **Mikrokontroler:** STM32G484RET6 (ARM Cortex-M4, taktowanie 170 MHz)
- **Interfejsy CAN:** 2x kanał sprzętowy dla magistrali Classical CAN (taktowanie do 1 Mbps)
- **Transceivery CAN:** 2x SN65HVD230DR
  - Zasilanie i logika 3.3V
  - Wbudowana terminacja magistrali 120 Ω
- **Interfejs telemetryczny (Radio):** USART2 z obsługą sprzętowej kontroli przepływu RTS/CTS (optymalizowany dla 921600 baud)
- **Interfejs konfiguracyjny (ESP32):** USART1
- **Sygnalizacja stanu:** Dioda LED wyprowadzona na pin PA5
- **Zasilanie układu:** 3.3V

![Schemat blokowy układu](Photos/Block%20Scheme.png)

## Pinout i połączenia zewnętrzne

### Interfejsy CAN (Transceivery)

| Funkcja STM32 | Pin  | Opis połączenia                                                                    |
| ------------- | ---- | ---------------------------------------------------------------------------------- |
| FDCAN1_RX     | PA11 | Odbiór ramek z sieci CAN1                                                          |
| FDCAN1_TX     | PA12 | Nadawanie CAN1 (w pierwszej wersji programowej sprzęt działa tylko jako odbiornik) |
| FDCAN2_RX     | PB12 | Odbiór ramek z sieci CAN2                                                          |
| FDCAN2_TX     | PB13 | Nadawanie CAN2                                                                     |

### Złącze wyjściowe telemetryczne (Moduł Radiowy)

Współpraca z modułem radiowym realizowana jest przez USART2:
| Funkcja | Pin STM32 | Opis |
|---|---|---|
| USART2_TX | PA2 | Główny strumień danych wysyłany przez DMA do modułu zewn. |
| USART2_RX | PA3 | Odbiór danych z modułu (opcjonalny) |

### Złącze modułu konfigurującego (ESP32)

Połączenie pomiędzy STM32 i ESP32 jest krzyżowe. Wymagane jest też połączenie masy.
| Funkcja STM32 | Pin | Podłączenie po stronie ESP32 |
|---|---|---|
| USART1_TX | PC4 | ESP32 RX |
| USART1_RX | PC5 | ESP32 TX |
| GND | GND | GND |

### Sygnalizacja statusu

| Funkcja    | Pin STM32 | Opis sprzętowy                                                                  |
| ---------- | --------- | ------------------------------------------------------------------------------- |
| LED_STATUS | PA5       | Dioda LED; stan wysoki na pinie załącza diodę (szeregowy rezystor 330Ω do GND). |

---

## Galeria PCB

![Zdjęcie PCB Top](Photos/PCB%20Front.png)
![Zdjęcie PCB Bottom](Photos/PCB%20Back.png)

## Uwagi montażowe i uruchomieniowe

1. **Terminacja CAN:** Płytka posiada zintegrowane rezystory terminujące 120Ω dla obydwu magistral CAN. Upewnij się co do topologii sieci w bolidzie, aby uniknąć zdublowanej terminacji w jednym węźle.
2. **Kontrola przepływu:** Złącze USART2 wspiera linie RTS/CTS. Jeśli podłączany zewnętrzny moduł radiowy nie posiada tych linii sprzętowych, zewrzyj lub skonfiguruj odpowiednio wejścia, aby nie blokować transmisji (zgodnie z ustawieniami peryferiów).
3. **Złącze SWD:** Na płytce wyprowadzono standardowe złącze SWD do programowania i debugowania mikrokontrolera STM32.
