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
#include <Servo.h>
#include <LiquidCrystal.h>
#include <IRremote.h>
```

⭕Estas líneas son instrucciones de inclusión de bibliotecas.
Estas bibliotecas contienen definiciones y funciones predefinidas que te permiten utilizar y controlar los dispositivos específicos conectados a tu placa Arduino. En este caso, se incluyen las bibliotecas para el control de un servo motor, una pantalla LCD y un receptor de infrarrojos.

``` C++
#define Tecla_1 0xEF10BF00
#define Tecla_2 0xEE11BF00
```
Aquí se definen dos constantes con nombres "Tecla_1" y "Tecla_2". 
Estas constantes almacenan valores hexadecimales que representan códigos de señales infrarrojas. 
Esos códigos se utilizarán más adelante para identificar teclas específicas de un control remoto infrarrojo.

``` C++
Servo servo;
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);
const int pinTemperatura = A0;
const int umbralAlarma = 60;
int IR = 6;

```
Aquí se declaran varias variables. "servo" es un objeto de la clase "Servo" que se utilizará para controlar un servo motor. 
"lcd" es un objeto de la clase "LiquidCrystal" que se utilizará para controlar una pantalla LCD. 
"pinTemperatura" es una variable que almacena el número de pin analógico utilizado para leer la temperatura. 
"umbralAlarma" es una constante que establece el valor de temperatura a partir del cual se activará una alarma. 
"IR" es una variable que almacena el número de pin digital utilizado para recibir señales del receptor de infrarrojos.

``` C++
bool alarmaActivada = false;
int posicionServo = 0;
```

Estas líneas declaran dos variables adicionales. "alarmaActivada" es una variable booleana que indica si la alarma está activada (true) o desactivada (false).
"posicionServo" es una variable entera que almacenará la posición actual del servo motor.

``` C++
void setup() {
  Serial.begin(9600);
  servo.attach(13);
  lcd.begin(16, 2);
  pinMode(9, OUTPUT);
  pinMode(10, OUTPUT);
  IrReceiver.begin(IR, DISABLE_LED_FEEDBACK);
}

```
❗La función setup() se ejecuta una vez al inicio del programa. Aquí se realizan las configuraciones iniciales de los componentes y se establecen los pines de entrada/salida necesarios.

❗Serial.begin(9600) inicializa la comunicación serial a una velocidad de 9600 baudios, lo que permite enviar y recibir datos desde y hacia el puerto serial de la placa Arduino.

❗servo.attach(13) asocia el pin digital 13 a la instancia del objeto servo creado anteriormente, permitiendo así el control del servo motor.

❗lcd.begin(16, 2) inicializa la comunicación con la pantalla LCD de 16 columnas y 2 filas, que se conecta a los pines especificados en la biblioteca LiquidCrystal.

❗pinMode(9, OUTPUT) y pinMode(10, OUTPUT) configuran los pines 9 y 10 como salidas digitales, que se utilizarán para controlar algún tipo de dispositivo externo, como luces o alarmas.

❗IrReceiver.begin(IR, DISABLE_LED_FEEDBACK) inicializa el receptor de infrarrojos con el pin especificado (6 en este caso) y desactiva el LED de retroalimentación del recepto


``` C++
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
```

❗La función loop() se ejecuta continuamente después de la función setup(). Aquí se encuentran las instrucciones que se repetirán en bucle mientras el Arduino esté encendido.

❗if (IrReceiver.decode()) verifica si se ha recibido una señal infrarroja. Si es así, se ejecuta el bloque de código dentro del if.

❗Serial.println(IrReceiver.decodedIRData.decodedRawData, HEX) imprime por el puerto serial el código de la señal infrarroja recibida en formato hexadecimal.

❗if (IrReceiver.decodedIRData.decodedRawData == Tecla_1) verifica si el código de la señal infrarroja coincide con el valor de Tecla_1 (definido previamente). Si coincide, se llama a la función activarAlarmaControl().

❗if (IrReceiver.decodedIRData.decodedRawData == Tecla_2) verifica si el código de la señal infrarroja coincide con el valor de Tecla_2 (definido previamente). Si coincide, se llama a la función desactivarAlarmaControl().

❗IrReceiver.resume() reinicia el receptor de infrarrojos para recibir la siguiente señal.

❗delay(1) introduce una pausa de 1 milisegundo entre iteraciones del bucle principal para evitar que el programa se ejecute demasiado rápido.

❗float temperatura = leerTemperatura() llama a la función leerTemperatura() para obtener el valor de la temperatura actual.

❗lcd.setCursor(1, 1) y lcd.print(temperatura) establecen la posición del cursor en la pantalla LCD y muestran la temperatura.

❗String estacion = obtenerEstacion(temperatura) llama a la función obtenerEstacion() pasando la temperatura como argumento y obtiene la estación correspondiente basada en la temperatura.

❗lcd.setCursor(8, 1) y lcd.print(estacion) establecen la posición del cursor en la pantalla LCD y muestran la estación obtenida.

---
### 🧠Funciones Principales

``` C++
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

⏺float leerTemperatura(): Esta función lee el valor del sensor de temperatura conectado al pin analógico pinTemperatura. El valor leído se convierte a voltaje y luego se utiliza una fórmula para calcular la temperatura en grados Celsius. El resultado se devuelve como un valor de tipo float.

⏺String obtenerEstacion(float temperatura): Esta función toma la temperatura como argumento y devuelve una cadena de texto que representa la estación climática correspondiente a esa temperatura. Se realizan comparaciones en los rangos de temperatura para determinar la estación. Si no se cumple ninguna condición, se devuelve una cadena vacía.

⏺void activarAlarma(): Esta función se encarga de activar la alarma de incendio. Si la alarma no está ya activada, se realiza lo siguiente: se establece la variable alarmaActivada como true, se borra la pantalla LCD, se imprime el mensaje "Alarma Incendio" en la pantalla, se mueve el servo motor a 180 grados, se espera 1 segundo, se activan los pines 10 y 9 (que se asume que están conectados a elementos de alarma), y finalmente se vuelve a mover el servo motor a 180 grados.

⏺void desactivarAlarma(): Esta función se encarga de desactivar la alarma de incendio. Si la alarma está activada, se realiza lo siguiente: se establece la variable alarmaActivada como false, se borra la pantalla LCD, se desactivan los pines 10 y 9, y se mueve el servo motor a 90 grados.

⏺void activarAlarmaControl(): Esta función se ejecuta cuando se presiona la tecla 1 del control remoto. Muestra un mensaje en la pantalla LCD indicando que la alarma está activada, activa los elementos de alarma y mueve el servo motor a 180 grados.

⏺void desactivarAlarmaControl(): Esta función se ejecuta cuando se presiona la tecla 2 del control remoto. Muestra un mensaje en la pantalla LCD indicando que la alarma está desactivada, desactiva los elementos de alarma y mueve el servo motor a 90 grados.

⏺void moverServo(): Esta función se encarga de mover el servo motor de un extremo a otro. Si la variable posicionServo es 0, se cambia a 180 (derecha), y si es diferente de 0, se cambia a 0 (izquierda). Luego, se utiliza servo.write(posicionServo) para mover el servo motor a la posición deseada, y se introduce un retardo de 1 segundo para observar el movimiento.



---
## <img src="tinkercad-logo.png" alt="Tinkercad" height="32px"> Link al proyecto

- [Proyecto](https://www.tinkercad.com/things/7Iyod7Uu8om?sharecode=HWUrwhqvIBeIJMbOGsoJYb3Ep-zqGuxPZPnH1S97p7c)

### 📄Parcial:

[Consignas](https://github.com/magikboy/parcial-2-SPD/blob/672a8977f21a56b0f5e515bc6447ac401242fa1d/Consigna_%20Sistema%20de%20incendio%20con%20Arduino%2C%20Segundo%20parcial.pdf)

### 📄Fuentes

- [Youtube](https://www.youtube.com)
- [electrontools](https://www.electrontools.com/Home/WP/sensor-infrarojo-con-arduino/)
- [Utn](http://www.sistemas-utnfra.com.ar/#/home)
