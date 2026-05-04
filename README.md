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

## Kod programu

```cpp
#include <Servo.h>
#include <LiquidCrystal.h>

LiquidCrystal lcd(2, 3, 4, 5, 6, 7);

Servo szlaban;

// czujniki
const int trigWejscie = 8;
const int echoWejscie = 9;
const int trigWyjscie = 10;
const int echoWyjscie = 11;

// parking
int max_miejsc = 3;
int wolne_miejsca = 3;

// blokady
bool autoWjazd = false;
bool autoWyjazd = false;

// komunikat LCD
String status = "START";

// pomiar odległości
long zmierzOdleglosc(int trig, int echo) {
  digitalWrite(trig, LOW);
  delayMicroseconds(2);
  digitalWrite(trig, HIGH);
  delayMicroseconds(10);
  digitalWrite(trig, LOW);

  long czas = pulseIn(echo, HIGH, 30000);
  long odleglosc = czas * 0.034 / 2;

  if (odleglosc == 0) odleglosc = 999;

  return odleglosc;
}

void otworzSzlaban() {
  szlaban.write(90);
  delay(2000);
  szlaban.write(0);
}

void pokazLCD() {
  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("Miejsca: ");
  lcd.print(wolne_miejsca);
  lcd.print("/");
  lcd.print(max_miejsc);

  lcd.setCursor(0, 1);
  lcd.print(status);
}

void setup() {
  pinMode(trigWejscie, OUTPUT);
  pinMode(echoWejscie, INPUT);
  pinMode(trigWyjscie, OUTPUT);
  pinMode(echoWyjscie, INPUT);

  szlaban.attach(12);
  szlaban.write(0);

  lcd.begin(16, 2);
  lcd.print("START PARKING");

  delay(1500);
  lcd.clear();
}

void loop() {

  long wejscie = zmierzOdleglosc(trigWejscie, echoWejscie);
  long wyjscie = zmierzOdleglosc(trigWyjscie, echoWyjscie);

  status = "OK";

  // RESET BLOKAD
  if (wejscie > 15) autoWjazd = false;
  if (wyjscie > 15) autoWyjazd = false;

  // WJAZD
  if (wejscie < 10 && !autoWjazd) {
    autoWjazd = true;

    if (wolne_miejsca > 0) {
      status = "WJAZD";
      otworzSzlaban();
      wolne_miejsca--;
    } else {
      status = "BRAK MIEJSC";

      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("BRAK MIEJSC");

      lcd.setCursor(0, 1);
      lcd.print("Czekaj...");

      delay(3000);
    }
  }

  // WYJAZD
  if (wyjscie < 10 && !autoWyjazd) {
    autoWyjazd = true;

    status = "WYJAZD";

    if (wolne_miejsca < max_miejsc) {
      otworzSzlaban();
      wolne_miejsca++;
    }
  }

  // LCD UPDATE
  pokazLCD();

  delay(200);
}
```

---

## Autor

Projekt edukacyjny Arduino – system parkingowy automatyczny.

Można dowolnie rozwijać i modyfikować 🚀
