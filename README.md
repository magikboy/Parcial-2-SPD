### 📄Documentación de la parte práctica del segundo parcial para la materia Sistemas de procesamiento de datos - UTN Tecnicatura Superior en Programación.

### Nombre: Bosco Mascaro Massimo Ariel

### **Sistema de Procesamiento de Datos (SPD)**

![Arduino](https://github.com/magikboy/Parcial-1/blob/30c7b791849ce1d70de15ec52cb6a92ac3aec450/ArduinoTinkercad.jpg)

### Modelo de Alarma Contra Incendios

![Alarma Contra Incendios](https://github.com/magikboy/parcial-2-SPD/blob/35b1e881b319ae2d823b6017096cff1e721dc344/alarma.PNG)
### 🦴Esquema de Alarma Contra Incendios

![Esquema de Alarma Contra Incendios](https://github.com/magikboy/parcial-2-SPD/blob/35b1e881b319ae2d823b6017096cff1e721dc344/esquema.PNG)

## 📄Consigna de la Alarma:
Objetivo:
El objetivo de este proyecto es diseñar un sistema de incendio utilizando Arduino que pueda
detectar cambios de temperatura y activar un servo motor en caso de detectar un incendio.
Además, se mostrará la temperatura actual y la estación del año en un display LCD.

Componentes necesarios:

✔Arduino UNO

✔Sensor de temperatura

✔Control remoto IR (Infrarrojo)

✔Display LCD (16x2 caracteres)

✔Servo motor

✔Cables y resistencias según sea necesario

✔Protoboard para realizar las conexiones

✔Dos leds.


• Control remoto:
Configura el control remoto IR para recibir señales.
Define los comandos necesarios para activar y desactivar el sistema de incendio.
Utiliza un algoritmo para determinar la estación del año (por ejemplo, rangos de temperatura
para cada estación).

• Detección de temperatura:
Configura el sensor de temperatura y realiza la lectura de la temperatura ambiente.
Muestra la temperatura actual en el display LCD.

• Sistema de alarma:
Define un umbral de temperatura a partir del cual se considera que hay un incendio (por
ejemplo, temperatura superior a 60 grados Celsius).
Cuando se detecta un incendio (temperatura por encima del umbral), se activa el servo
motor para simular una respuesta del sistema de incendio.

• Mensajes en el display LCD:
Muestra la temperatura actual y la estación del año en el display LCD.
Cuando se detecta un incendio, muestra un mensaje de alarma en el display LCD.
Punto libre:
Se deberá agregar dos leds y darle una funcionalidad de su elección, acorde al
proyecto previamente detallado.


### 🚀Codigo del proyecto
``` C++
#include <Servo.h>
#include <LiquidCrystal.h>
#include <IRremote.h>

#define Tecla_1 0xEF10BF00
#define Tecla_2 0xEE11BF00

Servo servo;
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);
const int pinTemperatura = A0;
const int umbralAlarma = 60;
int IR = 6;

bool alarmaActivada = false;
int posicionServo = 0;

void setup() {
  Serial.begin(9600);
  servo.attach(13);
  lcd.begin(16, 2);
  pinMode(9, OUTPUT);
  pinMode(10, OUTPUT);
  IrReceiver.begin(IR, DISABLE_LED_FEEDBACK);
}

void loop() {
  if (IrReceiver.decode()) {        
    // Se imprime el código de la señal infrarroja recibida
    Serial.println(IrReceiver.decodedIRData.decodedRawData, HEX); 
    // Verificar si se presionó la tecla 1 del control remoto
    if (IrReceiver.decodedIRData.decodedRawData == Tecla_1) {
      activarAlarmaControl();
    }
    // Verificar si se presionó la tecla 2 del control remoto
    if (IrReceiver.decodedIRData.decodedRawData == Tecla_2) {
      desactivarAlarmaControl();
    }
    IrReceiver.resume();
  }
  delay(1);

  // Leer la temperatura y mostrarla en la pantalla LCD
  float temperatura = leerTemperatura();
  lcd.setCursor(1, 1);
  lcd.print(temperatura);
  String estacion = obtenerEstacion(temperatura);
  lcd.setCursor(8, 1);
  lcd.print(estacion);

  // Activar o desactivar la alarma según la temperatura
  if (temperatura > umbralAlarma) {
    activarAlarma();
  } else {
    desactivarAlarma();
  }

  // Si la alarma está activada, mover el servo motor
  if (alarmaActivada) {
    moverServo();
  }
}

// Leer la temperatura del sensor
float leerTemperatura() {
  int valorRaw = analogRead(pinTemperatura);
  float voltaje = valorRaw * (5.0 / 1023.0);
  float temperatura = (voltaje - 0.5) * 100.0;
  return temperatura;
}

// Obtener la estación climática según la temperatura
String obtenerEstacion(float temperatura) {
  if (temperatura >= -5 && temperatura <= 10) {
    return "Winter";
  } else if (temperatura > 20 && temperatura <= 30) {
    return "Spring";
  } else if (temperatura > 30 && temperatura <= 45) {
    return "Summer";
  } else if (temperatura > 10 && temperatura <= 20) {
    return "Autumn";
  } else {
    return "      ";
  }
}

// Activar la alarma de incendio
void activarAlarma() {
  if (!alarmaActivada) {
    alarmaActivada = true;
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Alarma");
    lcd.setCursor(7, 0);
    lcd.print("Incendio");
    servo.write(180);
    delay(1000);
    digitalWrite(10, HIGH);
    delay(1000);
    digitalWrite(9, HIGH);
    delay(1000);
    servo.write(180);
  }
}

// Desactivar la alarma de incendio
void desactivarAlarma() {
  if (alarmaActivada) {
    alarmaActivada = false;
    lcd.clear();
    digitalWrite(10, LOW);
    digitalWrite(9, LOW);
    servo.write(90);
  }
}

// Activar la alarma de incendio mediante el control remoto
void activarAlarmaControl() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Alarma");
  lcd.setCursor(7, 0);
  lcd.print("Incendio");
  lcd.setCursor(0, 1);
  lcd.print("Activada");
  digitalWrite(9, HIGH);
  servo.write(180);
  delay(1000);
  lcd.clear();
  delay(1000);
}

// Desactivar la alarma de incendio mediante el control remoto
void desactivarAlarmaControl() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Alarma");
  lcd.setCursor(7, 0);
  lcd.print("Incendio");
  lcd.setCursor(0, 1);
  lcd.print("Desactivada");
  digitalWrite(10, HIGH);
  servo.write(90);
  delay(1000);
  lcd.clear();
  delay(1000);
}

// Mover el servo motor de un extremo a otro
void moverServo() {
  if (posicionServo == 0) {
    posicionServo = 180; // Cambiar la posición del servo motor a 180 grados (derecha)
  } else {
    posicionServo = 0; // Cambiar la posición del servo motor a 0 grados (izquierda)
  }
  servo.write(posicionServo); // Mover el servo motor a la posición deseada
  delay(1000); // Retardo de 1 segundo para observar el movimiento
}

```
### 🤖Funcionando
![Alarma](https://github.com/magikboy/parcial-2-SPD/blob/291f70dad8f28e1de76b9b239e56fb4a9caaebaa/2023-06-19-23-43-59.gif)

### 🧠Explicacion

``` C++
#define BOTON_SUBIR 2
#define BOTON_BAJAR 3
#define BOTON_PAUSAR 4
#define led_Verde 5
#define led_Rojo 6
#define A 7
#define B 8
#define C 9
#define D 10
#define E 11
#define F 12
#define G 13
const int TIEMPO_ESPERA_BOTON = 10; // Tiempo de espera entre lecturas de botones en milisegundos.
const int TIEMPO_POR_PISO = 3000; // Tiempo que tarda el montacargas en llegar a cada piso en milisegundos.
const int TIEMPO_ESPERA_MOVIMIENTO = 3000; // Tiempo de espera después de que se mueve el montacargas en milisegundos.

boolean botonSubir = false;
boolean botonBajar = false;
boolean botonPausa = false;
boolean enMovimiento = false;

int contador = 0; //INICIALIZO EL CONTADOR EN 0
String mensaje = ""; //PARA PODER ESCRIBIR EN EL MONITOR

const char* mensajesPisos[] = {
  "Llego al piso 0.",
  "Llego al piso 1.",
  "Llego al piso 2.",
  "Llego al piso 3.",
  "Llego al piso 4.",
  "Llego al piso 5.",
  "Llego al piso 6.",
  "Llego al piso 7.",
  "Llego al piso 8.",
  "Llego al piso 9."
};
```

Las constantes definen los pines que se usan para los botones, LEDs y segmentos de un display de siete segmentos. También define los tiempos que tarda el montacargas en llegar a cada piso y el tiempo que se espera después de que se mueve el montacargas.

Las variables booleanas son para almacenar el estado de los botones y el estado de movimiento del montacargas.

El contador se inicializa en 0 y se utiliza para indicar el piso actual en el que se encuentra el montacargas. El array de mensajes de pisos se usa para almacenar mensajes de texto que indican el piso al que se mueve el montacargas.


``` C++
// FUNCIONES
void displayOff() // Apago display al salir del switch
{
  digitalWrite(A, LOW);
  digitalWrite(B, LOW);
  digitalWrite(C, LOW);
  digitalWrite(D, LOW);
  digitalWrite(E, LOW);
  digitalWrite(F, LOW);
  digitalWrite(G, LOW);
}

void cero(int on)
{
  digitalWrite(A, HIGH);
  digitalWrite(B, HIGH);
  digitalWrite(C, HIGH);
  digitalWrite(D, HIGH);
  digitalWrite(E, HIGH);
  digitalWrite(F, HIGH);
  digitalWrite(G, LOW);
}

void uno(int on)
{
  digitalWrite(A, LOW);
  digitalWrite(B, HIGH);
  digitalWrite(C, HIGH);
  digitalWrite(D, LOW);
  digitalWrite(E, LOW);
  digitalWrite(F, LOW);
  digitalWrite(G, LOW);
}

void dos(int on)
{
  digitalWrite(A, HIGH);
  digitalWrite(B, HIGH);
  digitalWrite(C, LOW);
  digitalWrite(D, HIGH);
  digitalWrite(E, HIGH);
  digitalWrite(F, LOW);
  digitalWrite(G, HIGH);
}

void tres(int on)
{
  digitalWrite(A, HIGH);
  digitalWrite(B, HIGH);
  digitalWrite(C, HIGH);
  digitalWrite(D, HIGH);
  digitalWrite(E, LOW);
  digitalWrite(F, LOW);
  digitalWrite(G, HIGH);
}

void cuatro(int on)
{
  digitalWrite(A, LOW);
  digitalWrite(B, HIGH);
  digitalWrite(C, HIGH);
  digitalWrite(D, LOW);
  digitalWrite(E, LOW);
  digitalWrite(F, HIGH);
  digitalWrite(G, HIGH);
}

void cinco(int on)
{
  digitalWrite(A, HIGH);
  digitalWrite(B, LOW);
  digitalWrite(C, HIGH);
  digitalWrite(D, HIGH);
  digitalWrite(E, LOW);
  digitalWrite(F, HIGH);
  digitalWrite(G, HIGH);
}

void seis(int on)
{
  digitalWrite(A, HIGH);
  digitalWrite(B, LOW);
  digitalWrite(C, HIGH);
  digitalWrite(D, HIGH);
  digitalWrite(E, HIGH);
  digitalWrite(F, HIGH);
  digitalWrite(G, HIGH);
}

void siete(int on)
{
  digitalWrite(A, HIGH);
  digitalWrite(B, HIGH);
  digitalWrite(C, HIGH);
  digitalWrite(D, LOW);
  digitalWrite(E, LOW);
  digitalWrite(F, LOW);
  digitalWrite(G, LOW);
}

void ocho(int on)
{
  digitalWrite(A, HIGH);
  digitalWrite(B, HIGH);
  digitalWrite(C, HIGH);
  digitalWrite(D, HIGH);
  digitalWrite(E, HIGH);
  digitalWrite(F, HIGH);
  digitalWrite(G, HIGH);
}

void nueve(int on)
{
  digitalWrite(A, HIGH);
  digitalWrite(B, HIGH);
  digitalWrite(C, HIGH);
  digitalWrite(D, HIGH);
  digitalWrite(E, LOW);
  digitalWrite(F, HIGH);
  digitalWrite(G, HIGH);
}

void todos(int on)
{
  digitalWrite(A, HIGH);
  digitalWrite(B, HIGH);
  digitalWrite(C, HIGH);
  digitalWrite(D, HIGH);
  digitalWrite(E, HIGH);
  digitalWrite(F, HIGH);
  digitalWrite(G, HIGH);
}

void actualizarDisplay(int piso) {
  switch (piso) {
    case 1:
      digitalWrite(A, HIGH);
      digitalWrite(B, HIGH);
      digitalWrite(C, HIGH);
      digitalWrite(D, HIGH);
      digitalWrite(E, HIGH);
      digitalWrite(F, HIGH);
      digitalWrite(G, LOW);
      break;
    case 2:
      digitalWrite(A, LOW);
      digitalWrite(B, HIGH);
      digitalWrite(C, HIGH);
      digitalWrite(D, LOW);
      digitalWrite(E, LOW);
      digitalWrite(F, LOW);
      digitalWrite(G, LOW);
      break;
    case 3:
      digitalWrite(A, HIGH);
      digitalWrite(B, HIGH);
      digitalWrite(C, HIGH);
      digitalWrite(D, HIGH);
      digitalWrite(E, LOW);
      digitalWrite(F, LOW);
      digitalWrite(G, HIGH);
      break;
    case 4:
      digitalWrite(A, LOW);
      digitalWrite(B, HIGH);
      digitalWrite(C, HIGH);
      digitalWrite(D, LOW);
      digitalWrite(E, LOW);
      digitalWrite(F, HIGH);
      digitalWrite(G, HIGH);
      break;
    case 5:
      digitalWrite(A, HIGH);
      digitalWrite(B, LOW);
      digitalWrite(C, HIGH);
      digitalWrite(D, HIGH);
      digitalWrite(E, LOW);
      digitalWrite(F, HIGH);
      digitalWrite(G, HIGH);
      break;
    case 6:
      digitalWrite(A, HIGH);
      digitalWrite(B, LOW);
      digitalWrite(C, HIGH);
      digitalWrite(D, HIGH);
      digitalWrite(E, HIGH);
      digitalWrite(F, HIGH);
      digitalWrite(G, HIGH);
      break;
    case 7:
      digitalWrite(A, HIGH);
      digitalWrite(B, HIGH);
      digitalWrite(C, HIGH);
      digitalWrite(D, LOW);
      digitalWrite(E, LOW);
      digitalWrite(F, LOW);
      digitalWrite(G, LOW);
      break;
    case 8:
      digitalWrite(A, HIGH);
      digitalWrite(B, HIGH);
      digitalWrite(C, HIGH);
      digitalWrite(D, HIGH);
      digitalWrite(E, HIGH);
      digitalWrite(F, HIGH);
      digitalWrite(G, HIGH);
      break;
    case 9:
      digitalWrite(A, HIGH);
      digitalWrite(B, HIGH);
      digitalWrite(C, HIGH);
      digitalWrite(D, HIGH);
      digitalWrite(E, LOW);
      digitalWrite(F, HIGH);
      digitalWrite(G, HIGH);
      break;
  }
}
// FIN FUNCIONES
```
La función **displayOff()** se utiliza para apagar todos los segmentos del display cuando se sale del switch o se necesita apagar el display.

Las funciones **cero() a nueve()**se utilizan para mostrar los dígitos del 0 al 9 en el display. Cada función enciende los segmentos necesarios para mostrar el dígito correspondiente. Por ejemplo, la función cero() enciende todos los segmentos excepto el segmento G.

La función **todos()** enciende todos los segmentos del display, lo que resulta en la visualización del número 8.

La función **actualizarDisplay()** se utiliza para mostrar el número del piso en el que se encuentra un elevador, por ejemplo. Se utiliza un switch para seleccionar el número del piso y luego se llama a esta función para actualizar el display con el número correspondiente. La función toma como argumento el número del piso y utiliza los comandos digitalWrite() para encender los segmentos necesarios para mostrar el número en el display.

``` C++
void mostrarPiso(int piso) {
switch(piso) {
  case 0:
  	cero(1);
  	break;
  case 1:
  	uno(1);
  	break;
  case 2:
    dos(1);
    break;
  case 3:
    tres(1);
    break;
  case 4:
    cuatro(1);
    break;
  case 5:
    cinco(1);
    break;
  case 6:
    seis(1);
    break;
  case 7:
    siete(1);
    break;
  case 8:
    ocho(1);
    break;
  case 9:
    nueve(1);
    break;
}
mensaje = mensajesPisos[piso];
}

void cambiarPiso(String direccion) {
  if (direccion == "subir" && contador < 9) {
    contador++;
  }
  else if (direccion == "bajar" && contador > 0) {
    contador--;
  }
}

int moverPiso(String subirBajar, int tiempoDelay)
{
  digitalWrite(led_Rojo, 0);
  cambiarPiso(subirBajar);
  mostrarPiso(contador);
  digitalWrite(led_Verde, 1);
  delay(tiempoDelay);
  digitalWrite(led_Verde , 0);
  displayOff();
  Serial.println(mensaje);
  return contador;
}
```


**las funciones principales son**

**moverPiso()** La función principal del programa es moverPiso(), que se encarga de mover el montacargas a un piso determinado. Esta función recibe dos parámetros: subirBajar, que indica si el montacargas debe subir o bajar, y tiempoDelay, que indica cuánto tiempo debe esperar el programa después de que el montacargas se mueve.

Dentro de la función, primero se apaga el led rojo, se cambia el piso actual utilizando la función cambiarPiso(), se muestra el piso actual en el display utilizando la función mostrarPiso(), se enciende el led verde y se espera el tiempo indicado por el parámetro tiempoDelay. Luego se apaga el led verde, se apaga el display y se muestra un mensaje en el monitor serie indicando el piso al que se ha llegado.

**cambiarPiso()** La función cambiarPiso() se encarga de cambiar el piso actual del montacargas en función del parámetro subirBajar. Si subirBajar es "subir" y el contador actual es menor que 9, el contador se incrementa en 1. Si subirBajar es "bajar" y el contador actual es mayor que 0, el contador se decrementa en 1.

**mostrarPiso()** La función mostrarPiso() se encarga de mostrar el piso actual en el display utilizando las funciones para encender los diferentes segmentos correspondientes al número que indica el piso actual.

``` C++
void setup() {
pinMode(BOTON_SUBIR, INPUT_PULLUP);
pinMode(BOTON_BAJAR, INPUT_PULLUP);
pinMode(BOTON_PAUSAR, INPUT_PULLUP);
pinMode(led_Rojo, OUTPUT);
pinMode(led_Verde, OUTPUT);
pinMode(A, OUTPUT);
pinMode(B, OUTPUT);
pinMode(C, OUTPUT);
pinMode(D, OUTPUT);
pinMode(E, OUTPUT);
pinMode(F, OUTPUT);
pinMode(G, OUTPUT);
Serial.begin(9600);
mostrarPiso(contador);
}

void loop() {
// Leer el estado de los botones
botonSubir = digitalRead(BOTON_SUBIR);
botonBajar = digitalRead(BOTON_BAJAR);
botonPausa = digitalRead(BOTON_PAUSAR);

// Si se presiona el botón de subir, mover hacia arriba
if (botonSubir == LOW) {
moverPiso("subir", TIEMPO_POR_PISO);
}

// Si se presiona el botón de bajar, mover hacia abajo
if (botonBajar == LOW) {
moverPiso("bajar", TIEMPO_POR_PISO);
}

// Si se presiona el botón de pausa, detener el movimiento
if (botonPausa == LOW) {
  mensaje = "El montacargas se detuvo";
  Serial.println(mensaje);
  digitalWrite(led_Verde, 0);
  digitalWrite(led_Rojo, 1);
  delay(TIEMPO_ESPERA_MOVIMIENTO);
  mostrarPiso(contador);
}
}

```

La función **setup()** es una función que se ejecuta una sola vez al inicio del programa. En ella se inicializan los pines que se van a utilizar como entradas o salidas, y se establece la velocidad de comunicación para la interfaz serial (Serial.begin(9600)). Además, se llama a la función mostrarPiso() para que muestre el piso en el que se encuentra el montacargas en ese momento.

La variable **botonSubir** es una variable que se utiliza para almacenar el estado del botón de subir. Se lee su estado utilizando la función digitalRead(), que devuelve un valor HIGH o LOW dependiendo de si el botón está pulsado o no.

La variable **botonBajar** es una variable que se utiliza para almacenar el estado del botón de bajar. Se lee su estado utilizando la función digitalRead().

la variable **botonPausa** es una variable que se utiliza para almacenar el estado del botón de pausa. Se lee su estado utilizando la función digitalRead().

---
## <img src="tinkercad-logo.png" alt="Tinkercad" height="32px"> Link al proyecto

- [Proyecto](https://www.tinkercad.com/things/cn31fUhwE2b)

### 📄Parcial:

[Consignas](https://github.com/magikboy/parcial-2-SPD/blob/672a8977f21a56b0f5e515bc6447ac401242fa1d/Consigna_%20Sistema%20de%20incendio%20con%20Arduino%2C%20Segundo%20parcial.pdf)

### 📄Fuentes

- [Youtube](https://www.youtube.com)
- [electrontools](https://www.electrontools.com/Home/WP/sensor-infrarojo-con-arduino/)
- [Utn](http://www.sistemas-utnfra.com.ar/#/home)
