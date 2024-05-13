# P2

## Objetivo: El objetivo de la practica es comprender el funcionamiento de las interrupciones.

## Materiales: 
- ESP32-S1
- GPIO

## Programa A

**Código**
```cpp
    struct Button {
        const uint8_t PIN;
        uint32_t numberKeyPresses;
        bool pressed;
    };
    Button button1 = {18, 0, false};
    // Instancia de la estructura Button con valores iniciales
    void IRAM_ATTR isr() {
        button1.numberKeyPresses += 1;
        button1.pressed = true;
    }
    void setup() {
        Serial.begin(115200); 
        // Inicia la comunicación serial para la depuración
        pinMode(button1.PIN, INPUT_PULLUP);
        // Configura el pin del botón como entrada con pull-up interno
        attachInterrupt(button1.PIN, isr, FALLING);
        //Funcion integrada de Arduiono, no necesario inicializar.
        //Función de interrupción al pin del botón
    }
    
    void loop() {
        if (button1.pressed) {
            Serial.printf("Button 1 has been pressed %u times\n",
            button1.numberKeyPresses);
            button1.pressed = false;
            }
        //Detach Interrupt after 1 Minute
        static uint32_t lastMillis = 0;
        // Variable estática para rastrear el tiempo transcurrido
        if (millis() - lastMillis > 60000) {
            lastMillis = millis(); // Reinicia el contador de tiempo
            detachInterrupt(button1.PIN); 
            // Desvincula la interrupción del pin del botón
            Serial.println("Interrupt Detached!");
            }
    }
```
## Descripción
Este código permite contar las pulsaciones de un botón físico y enviar esta información a través del puerto serial, además de desvincular la interrupción del botón después de un minuto de inactividad para conservar recursos. 

## Programa B

**Código**
```cpp
    #include <Arduino.h>

    volatile int interruptCounter;
    int totalInterruptCounter;
    hw_timer_t * timer = NULL;
    portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;
    void IRAM_ATTR onTimer() {
        portENTER_CRITICAL_ISR(&timerMux);
        interruptCounter++;
        portEXIT_CRITICAL_ISR(&timerMux);
    }
    void setup() {
        Serial.begin(115200);
        timer = timerBegin(0, 80, true);
        timerAttachInterrupt(timer, &onTimer, true);
        timerAlarmWrite(timer, 1000000, true);
        timerAlarmEnable(timer);
    }
    void loop() {
        if (interruptCounter > 0) {
        portENTER_CRITICAL(&timerMux);
        interruptCounter--;
        portEXIT_CRITICAL(&timerMux);
        totalInterruptCounter++;

        Serial.print("An interrupt as occurred. Total number: ");
        Serial.println(totalInterruptCounter);
        }
    }
```
## Descripción
Este código implementa un temporizador que genera interrupciones cada segundo, y cada vez que ocurre una interrupción, se incrementa un contador y se imprime el número total de interrupciones en el puerto serial. La sálida se veria usando el print "An interrupt as occurred. Total number: x" así que se vería de esta manera:

```
    An interrupt as occurred. Total number: 1
    An interrupt as occurred. Total number: 2
    An interrupt as occurred. Total number: 3
    An interrupt as occurred. Total number: 4
    An interrupt as occurred. Total number: 5
    etc...
```






