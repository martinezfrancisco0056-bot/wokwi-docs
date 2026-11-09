// Pines Semáforo Vehicular 1 (Vía A)
const int v1Rojo = 13;
const int v1Amarillo = 12;
const int v1Verde = 11;

// Pines Semáforo Vehicular 2 (Vía B)
const int v2Rojo = 10;
const int v2Amarillo = 9;
const int v2Verde = 8;

// Pines Semáforo Peatonal 1 (Cruza Vía A)
const int p1Rojo = 7;
const int p1Verde = 6;

// Pines Semáforo Peatonal 2 (Cruza Vía B)
const int p2Rojo = 5;
const int p2Verde = 4;

// Pines de Botones
const int btnPeaton1 = 2;
const int btnPeaton2 = 3;

// Variables para recordar si alguien presionó el botón
volatile bool solicitudP1 = false;
volatile bool solicitudP2 = false;

void setup() {
  // Configurar LEDs como salidas
  for (int pin = 4; pin <= 13; pin++) {
    pinMode(pin, OUTPUT);
  }

  // Configurar botones con resistencia interna
  pinMode(btnPeaton1, INPUT_PULLUP);
  pinMode(btnPeaton2, INPUT_PULLUP);

  // Interrupciones para detectar los botones en cualquier momento
  attachInterrupt(digitalPinToInterrupt(btnPeaton1), pedirCruce1, FALLING);
  attachInterrupt(digitalPinToInterrupt(btnPeaton2), pedirCruce2, FALLING);

  // Estado inicial: Todos los peatones en Rojo
  digitalWrite(p1Rojo, HIGH);
  digitalWrite(p2Rojo, HIGH);
}

void loop() {
  // --- TURNO VÍA A ---
  digitalWrite(v1Rojo, LOW);
  digitalWrite(v1Verde, HIGH);
  digitalWrite(v2Rojo, HIGH);
  esperarConChequeo(5000); // 5 segundos en verde

  // Cambio a amarillo Vía A
  digitalWrite(v1Verde, LOW);
  digitalWrite(v1Amarillo, HIGH);
  delay(2000);
  digitalWrite(v1Amarillo, LOW);
  digitalWrite(v1Rojo, HIGH);
  delay(1000); // Tiempo de seguridad todo en rojo

  // Revisar si el Peatón 1 pidió cruzar la Vía A
  if (solicitudP1) {
    activarPasoPeatonal(p1Rojo, p1Verde);
    solicitudP1 = false;
  }

  // --- TURNO VÍA B ---
  digitalWrite(v2Rojo, LOW);
  digitalWrite(v2Verde, HIGH);
  esperarConChequeo(5000); // 5 segundos en verde

  // Cambio a amarillo Vía B
  digitalWrite(v2Verde, LOW);
  digitalWrite(v2Amarillo, HIGH);
  delay(2000);
  digitalWrite(v2Amarillo, LOW);
  digitalWrite(v2Rojo, HIGH);
  delay(1000); // Tiempo de seguridad todo en rojo

  // Revisar si el Peatón 2 pidió cruzar la Vía B
  if (solicitudP2) {
    activarPasoPeatonal(p2Rojo, p2Verde);
    solicitudP2 = false;
  }
}

// Funciones de interrupción para los botones
void pedirCruce1() {
  solicitudP1 = true;
}

void pedirCruce2() {
  solicitudP2 = true;
}

// Secuencia segura para el peatón
void activarPasoPeatonal(int pinRojo, int pinVerde) {
  digitalWrite(pinRojo, LOW);
  digitalWrite(pinVerde, HIGH);
  delay(4000); // Tiempo fijo para cruzar

  // Parpadeo de advertencia antes de cerrar el paso
  for (int i = 0; i < 4; i++) {
    digitalWrite(pinVerde, LOW);
    delay(300);
    digitalWrite(pinVerde, HIGH);
    delay(300);
  }

  digitalWrite(pinVerde, LOW);
  digitalWrite(pinRojo, HIGH);
  delay(1000); // Pausa de seguridad antes de dar verde a los autos
}

// Mantiene el verde vehicular pero permite avanzar normal
void esperarConChequeo(unsigned long tiempo) {
  unsigned long inicio = millis();
  while (millis() - inicio < tiempo) {
    // El sistema sigue escuchando interrupciones automáticamente
  }
}
