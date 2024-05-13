# P2

## Objetivo: El objetivo de la practica es comprender el funcionamiento de las interrupciones.

## Materiales: 
- ESP32-S1
- GPIO

## Programa

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

