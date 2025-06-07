# P2_Alex_Marti-
Participantes: Alexandre Pascual i Marti Frigola

# Practica2-A
## Interrupció de botó en ESP32
### Descripció del Codi

El codi utilitza una **interrupció de hardware** que detecta si el botó esta sent pulsat o no. Es basa en un struct que guarda informació del botó i una deshabilitació d'aquesta interrupció si ha passat 1 minut.


### Funcionamient
1. Configurem un botó a un pin (18 en aquest cas) amb resistencia pull-up interna.
2. Si el botó es presionat, es genera una interrupció que augmenta el contador de pulsacions.
3. En un bucle s'imprimeix el numero de cops que el polsador ha sigut presionat.
4. Després d'un minut, la interrucpió es desactiva.

#### Estructura `Button`
```cpp
struct Button {
    const uint8_t PIN;
    uint32_t numberKeyPresses;
    bool pressed;
};
```
 `PIN`: Guarda a quin pin es connecta el botó.
`numberKeyPresses`: Compta les cops que es presiona.
`pressed`: Detecta i avisa si es polsat.

#### Configuració del Botó e Interrupció 
```cpp
void setup() {
    Serial.begin(115200); //Inicia una comunicació serial
    pinMode(button1.PIN, INPUT_PULLUP); //Configura el pin 18 com a `entrada pull-up`
    attachInterrupt(button1.PIN, isr, FALLING); // Interrupció que s'activa en 'FALLING'
}
```


#### Funció d`interrupció
```cpp
void IRAM_ATTR isr() { //Inicia si detecta una interrupció
    button1.numberKeyPresses += 1; //comptador de cops presionats
    button1.pressed = true; //booleà que ens indicarà si ha sigut presionat
}
```

#### Bucle Principal 
```cpp
void loop() {
    if (button1.pressed) { //Si el booleà anterior es 'true' imprimirà el nombre total
        Serial.printf("Button 1 has been pressed %u times\n", button1.numberKeyPresses);
        button1.pressed = false;
    }
    
    // Desactivar l'interrupció després de 1 minut
    static uint32_t lastMillis = 0;
    if (millis() - lastMillis > 60000) {
        lastMillis = millis();
        detachInterrupt(button1.PIN);
        Serial.println("Interrupt Detached!");
    }
}
```


### Sortida esperada en Serial Monitor
```
Button 1 has been pressed 1 times
Button 1 has been pressed 2 times
...
Interrupt Detached!
```

# Practica2-B

## _Timer Interrupt_ en ESP32

###  Descripció del Codi

Configura un temporitzador utilitzant la API de 'FreeRTOS' i funcions de la ESP32. La interrupció que es genera es controlada a la 'RAM intenra'.

###  Funcionamient

Inicia el 'serial monitor', després es configurarà el **temporitzador de hardware** el qual generara interrupcions cada segon. A cada interrupció augmentara un contador.
Si ocurreix una interrupció, al bucle es reduirà `interruptCounter` i s'incrementa `totalInterruptCounter`. Es mostra el total en pantalla.

`volatile int interruptCounter;` Actua com a comptador d'interrupcions.
- `int totalInterruptCounter;` Actua com a comptador TOTAL d'interrupcions.
- `hw_timer_t * timer = NULL;` Actua com a punter al temporitzador. 
- `portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;` Actua com a Mutex.

####  Funció d'Interrupció 

```cpp
void IRAM_ATTR onTimer() { //Incia quan es genera una interrupció
    portENTER_CRITICAL_ISR(&timerMux); //Protegeix la variable 'interruptCounter'
    interruptCounter++; //Comptador
    portEXIT_CRITICAL_ISR(&timerMux);//Protegeix la variable 'interruptCounter'
//Es guardarà a 'IRAM_ATTR'
}
```


####  Configuració en `setup()`

```cpp
void setup() {
    Serial.begin(115200);//Inicia comunicació serial
    timer = timerBegin(0, 80, true); //Configura un temporitzador
    timerAttachInterrupt(timer, &onTimer, true);//Adjunta la interrupció
    timerAlarmWrite(timer, 1000000, true);//Configura una alarma per dispararse cada 1 segons
    timerAlarmEnable(timer);//Habilita la alarma
}
```

####  Bucle Principal 

```cpp
void loop() {
    if (interruptCounter > 0) { //Comproba si hi ha hagut una interrupció
        portENTER_CRITICAL(&timerMux);
        interruptCounter--; //Redueix `interruptCounter` dins una secció critica per evitar condicions de carrera
        portEXIT_CRITICAL(&timerMux);
        totalInterruptCounter++; //Augmenta `totalInterruptCounter` i imprimeix el nombre total
        Serial.print("An interrupt has occurred. Total number: ");
        Serial.println(totalInterruptCounter);
    }
}
```

###  Sortida esperada en Serial Monitor

```
An interrupt has occurred. Total number: 1
An interrupt has occurred. Total number: 2
An interrupt has occurred. Total number: 3
...
```

Cada linea apareixierà cada 1 segundo.

---
