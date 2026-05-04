# Arduino Parking System 🚗🅿️

## Opis projektu

Projekt przedstawia prosty **system automatycznego parkingu** wykonany na platformie **Arduino**, wykorzystujący:

* serwomechanizm jako szlaban,
* 2 czujniki ultradźwiękowe HC-SR04 (wjazd / wyjazd),
* wyświetlacz LCD 16x2,
* logikę liczenia wolnych miejsc parkingowych.

System automatycznie:

* wykrywa samochód przy wjeździe,
* otwiera szlaban jeśli są wolne miejsca,
* zmniejsza liczbę dostępnych miejsc,
* wykrywa samochód przy wyjeździe,
* zwiększa liczbę wolnych miejsc po opuszczeniu parkingu,
* wyświetla aktualny status na LCD.

---

## Funkcjonalności

### 🚗 Wjazd

Jeśli czujnik przy wjeździe wykryje samochód:

* sprawdzana jest liczba wolnych miejsc,
* jeśli miejsce jest dostępne → szlaban otwiera się,
* liczba wolnych miejsc zostaje zmniejszona o 1.

Jeśli parking jest pełny:

* na LCD pojawia się komunikat:

```text
BRAK MIEJSC
Czekaj...
```

---

### 🚗 Wyjazd

Jeśli czujnik przy wyjeździe wykryje samochód:

* szlaban otwiera się,
* liczba wolnych miejsc zwiększa się o 1.

---

### 📺 Wyświetlacz LCD

LCD pokazuje:

```text
Miejsca: 2/3
WJAZD
```

lub inne statusy:

* `OK`
* `WJAZD`
* `WYJAZD`
* `BRAK MIEJSC`

---

## Wykorzystane komponenty

* Arduino UNO / Nano
* Serwomechanizm SG90
* LCD 16x2
* Potencjometr 10k (do kontrastu LCD)
* 2x czujnik ultradźwiękowy HC-SR04
* Rezystory
* Płytka stykowa + przewody połączeniowe

---

## Podłączenie pinów

### LCD

| LCD | Arduino |
| --- | ------- |
| RS  | 2       |
| E   | 3       |
| D4  | 4       |
| D5  | 5       |
| D6  | 6       |
| D7  | 7       |

---

### Czujniki ultradźwiękowe

#### Wjazd

| Element | Pin |
| ------- | --- |
| TRIG    | 8   |
| ECHO    | 9   |

#### Wyjazd

| Element | Pin |
| ------- | --- |
| TRIG    | 10  |
| ECHO    | 11  |

---

### Serwo

| Element | Pin |
| ------- | --- |
| Signal  | 12  |

---

## Logika działania

### Maksymalna liczba miejsc

```cpp
int max_miejsc = 3;
```

Możesz łatwo zmienić pojemność parkingu, edytując tę wartość.

---

## Zabezpieczenie przed wielokrotnym zliczaniem

Program wykorzystuje blokady:

```cpp
bool autoWjazd = false;
bool autoWyjazd = false;
```

Dzięki temu jedno auto nie zostanie policzone kilka razy podczas postoju przed czujnikiem.

---

## Biblioteki

Projekt wykorzystuje:

```cpp
#include <Servo.h>
#include <LiquidCrystal.h>
```

Biblioteki są standardowo dostępne w Arduino IDE.

---

## Uruchomienie

1. Podłącz wszystkie komponenty zgodnie ze schematem.
2. Wgraj kod do Arduino.
3. Uruchom zasilanie.
4. System automatycznie rozpocznie pracę.

Po starcie na LCD pojawi się:

```text
START PARKING
```

---

## Możliwe rozszerzenia projektu

### 🔥 Pomysły na rozwój

* diody LED (zielona / czerwona)
* buzzer
* RFID / karta dostępu
* moduł WiFi ESP8266
* aplikacja mobilna
* zapis danych do bazy
* system rezerwacji miejsc
* monitoring zajętości online

---

## Autorzy

- Amelia Kogutowska
- Klaudia Magrian
- Aleksandra Górska
- Aleksandra Strzelczyk


