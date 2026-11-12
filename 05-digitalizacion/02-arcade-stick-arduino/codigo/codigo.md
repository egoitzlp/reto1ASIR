#include <Keyboard.h>

// Palanca → flechas
const int pinesPalanca[] = {5, 4, 3, 2};
const char teclasPalanca[] = {KEY_UP_ARROW, KEY_DOWN_ARROW, KEY_LEFT_ARROW, KEY_RIGHT_ARROW};
bool estadoPalanca[4];

// Botones (B1, B2… están reservados en Arduino)
const int BOT1 = 6, BOT2 = 7, BOT3 = 8, BOT4 = 9;
bool e1, e2, e3, e4, eSalir;

void setup() {
  for (int i = 0; i < 4; i++) pinMode(pinesPalanca[i], INPUT_PULLUP);
  pinMode(BOT1, INPUT_PULLUP);
  pinMode(BOT2, INPUT_PULLUP);
  pinMode(BOT3, INPUT_PULLUP);
  pinMode(BOT4, INPUT_PULLUP);
  Keyboard.begin();
}

// Pulsa o suelta una tecla según el botón
void revisar(bool pulsado, char tecla, bool &estado) {
  if (pulsado && !estado) Keyboard.press(tecla);
  if (!pulsado && estado) Keyboard.release(tecla);
  estado = pulsado;
}

void loop() {
  for (int i = 0; i < 4; i++)
    revisar(digitalRead(pinesPalanca[i]) == LOW, teclasPalanca[i], estadoPalanca[i]);

  bool p1 = digitalRead(BOT1) == LOW;
  bool p2 = digitalRead(BOT2) == LOW;
  bool p3 = digitalRead(BOT3) == LOW;
  bool p4 = digitalRead(BOT4) == LOW;

  // Botones 3 + 4 a la vez → Esc
  bool salir = p3 && p4;
  if (salir && !eSalir) {
    Keyboard.releaseAll();
    e3 = e4 = false;
    Keyboard.write(KEY_ESC);
  }
  eSalir = salir;

  revisar(p1, KEY_LEFT_CTRL, e1);  // Disparo 1
  revisar(p2, KEY_LEFT_ALT, e2);   // Disparo 2

  if (!salir) {
    revisar(p3, '5', e3);          // Moneda
    if (p4 && !e4) { Keyboard.press('1'); Keyboard.press(KEY_RETURN); }
    if (!p4 && e4) { Keyboard.release('1'); Keyboard.release(KEY_RETURN); }
    e4 = p4;
  }

  delay(5);
}
