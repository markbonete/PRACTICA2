# PRACTICA2
## Codi sencer
```cpp
#include <Arduino.h>

struct Button {
  const uint8_t PIN;
  uint32_t numberKeyPresses;
  bool pressed;
};

Button button1 = {18, 0, false};

void IRAM_ATTR isr() {
  button1.numberKeyPresses += 1;
  button1.pressed = true;
}

void setup() {
  Serial.begin(115200);
  pinMode(button1.PIN, INPUT_PULLUP);
  attachInterrupt(button1.PIN, isr, FALLING);
}

void loop() {
  if (button1.pressed) {
    Serial.printf("Button 1 has been pressed %u times\n", button1.numberKeyPresses);
    button1.pressed = false;
  }
  //Detach Interrupt after 1 Minute
  static uint32_t lastMillis = 0;
  if (millis() - lastMillis > 60000) {
    lastMillis = millis();
    detachInterrupt(button1.PIN);
    Serial.println("Interrupt Detached!");
  }
}
```

## Explicació del codi per parts
### Definició estructura Button
```cpp
#include <Arduino.h>

struct Button {
  const uint8_t PIN;
  uint32_t numberKeyPresses;
  bool pressed;
};
```
L'estructura botó esta composada per tres variables:
- PIN: Pin al que estarà conectat el botó. Es una variable const perquè no es modificarà.
- numberKeyPresses: Comptador de la quantitat de vegades que premem el botó.
- pressed: Variable booleana que indica si s'ha premut el botó o no.

### Creació d'un objecte Button
```cpp
Button button1 = {18, 0, false};
```
S'implementa al codi un objecte d'estructura Button anomenat button1, que esta indicat al PIN 18 del ESP32, inicialitzat el contador numberKeyPressed en 0 i pressed declarat com false(no s'ha premut encar el botó)

### Rutina d' interrupció
```cpp
void IRAM_ATTR isr() {
  button1.numberKeyPresses += 1;
  button1.pressed = true;
}
```
La funció isr() es una rutina de interrupció (ISR).Cada vegada que es detecti una interrupció en el pin del botó,la funció incrementarà el contador(numberKeyPressed) i estableix pressed en true.

### Setup
```cpp
void setup() {
  Serial.begin(115200);
  pinMode(button1.PIN, INPUT_PULLUP);
  attachInterrupt(button1.PIN, isr, FALLING);
}
```
S'estableix la configuració inicial del programa:
- Serial.begin(115200): Inicialitza la comunicació sèrie en 115200 bauds.
- pinMode: Es configura button1.PIN com un pin d'entrada amb una resistència pull-up.
- attachInterrupt: Configura una interrupció al pin del botó que es dispara en detectar un flanc descendent (quan el botó passa de no estar pressionat a estar pressionat).
  
### Loop
```cpp
void loop() {
  if (button1.pressed) {
    Serial.printf("Button 1 has been pressed %u times\n", button1.numberKeyPresses);
    button1.pressed = false;
  }
  //Detach Interrupt after 1 Minute
  static uint32_t lastMillis = 0;
  if (millis() - lastMillis > 60000) {
    lastMillis = millis();
    detachInterrupt(button1.PIN);
    Serial.println("Interrupt Detached!");
  }
}
```
Si es pren el botó s'imprimeix per pantalla el número de vegades que s'ha premut el botó i reseteja l'estat de pressed a false cada vegada.
Es crea una variable lastMillis per fer el recompte del temps transcorregut.
Quan es passi de 60.000 ms (1 minut) des de l'última comprovació de l'estat del botó, es deshabilita l'interrupció en el botó i imprimeix per pantalla el missatge: Interrupció desconnectada.
