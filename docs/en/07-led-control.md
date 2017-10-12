# LEDs control

Our next task - навчитись керувати трьохкольоровим світлодіодом через Firebase.

![Led pins](https://github.com/snipter/firebase-iot-codelab/blob/master/docs/assets/image32.png)

Ми будемо використовувати трьохкольоровий світлодіод з загальним анодом. Його можна представити собі у вигляді трьох світлодіодів, у котрих з’єднаний "+", але роз’єднаний "-". Тобто, щоб  засвітився необхідний світлодіод, треба подати на його вивід "-", а на всі інші "+". Найдовший вивід - це загальний "+", коротший справа - червоний, перший зліва - синій, другий зліва - зелений. Встановіть світлодіод на монтажну плату та підключіть чотири проводи. Для зручності в нашому прикладі ми підключили жовтий провід до загального "+" (найдовший вивід), а червоний, зелений і синій - до відповідних виводів світлодіода. Те ж саме радимо зробити і вам.

![Led connection](https://github.com/snipter/firebase-iot-codelab/blob/master/docs/assets/image15.png)

Подайте живлення на вашу плату. Підключіть загальний вивід (жовтий) до будь-якого виводу плати з позначкою `3V` (живлення 3.3V). Підключіть по черзі кожен з інших дротів світлодіода до виводу `G` (мінус) плати. Якщо все було зроблено правильно, то засвітяться по черзі різні кольори.

Тепер переходимо до задачі. Для початку нам треба підключити світлодіод до нашої плати. Ви можете обрати для цього три будь-які виводи GPIO. У прикладі ми обрали `D5` (`GPIO14`), `D6` (`GPIO12`), `D7` (`GPIO13`). Загальний провід (жовтий) ми підключили на сусідній вивід `3V`.

![Led connection](https://github.com/snipter/firebase-iot-codelab/blob/master/docs/assets/image23.png)

Напишемо невелику програму щоб перевірити, чи вірно ми все підключили.

```c++
#include <Arduino.h>

// Constans with the numbers of pins
#define RED_PIN 13
#define GREEN_PIN 14
#define BLUE_PIN 12

// допоміжна функція для включення світлодіода
// так як у нас світлодіод з загальним анодом то
// для його включення необхідно встановити на
// ньому мінус (нуль)
void ledOn(byte pin){
    digitalWrite(pin, LOW);
}

// відповідно для виключення необхідно встановити
// плюс (один)
void ledOff(byte pin){
    digitalWrite(pin, HIGH);
}

// функція для виключення всіх світлодіодів
void ledsOff(){
    ledOff(RED_PIN);
    ledOff(GREEN_PIN);
    ledOff(BLUE_PIN);
    Serial.print("All leds OFF");
}

// ввімкнути червоний
void redLedOn(){
    ledOn(RED_PIN);
    Serial.println("Red led ON");
}

// ввімкнути зелений
void greenLedOn(){
    ledOn(GREEN_PIN);
    Serial.println("Green led ON");
}

// ввімкнути синій
void blueLedOn(){
    ledOn(BLUE_PIN);
    Serial.println("Blue led ON");
}

void setup(){
    // Налаштовуємо послідовний порт
    Serial.begin(115200);
    // Налаштовуємо піни на вихід
    pinMode(RED_PIN, OUTPUT);
    pinMode(GREEN_PIN, OUTPUT);
    pinMode(BLUE_PIN, OUTPUT);
    // на всяк випадок вимикаємо всі піни
    ledsOff();
}

void loop(){
    // вмикаємо червоний і чекаємо 1с
    redLedOn();
    delay(1000);
    // вимикаємо всі
    ledsOff();
    // вмикаємо зелений і чекаємо 1с
    greenLedOn();
    delay(1000);
    // вимикаємо всі
    ledsOff();
    // вмикаємо синій і чекаємо 1с
    blueLedOn();
    delay(1000);
    // вимикаємо всі
    ledsOff();
    // починаємо все з початку
}
```

Запустіть програму і перевірте, чи все працює.

Тепер час підключити девайс до Firebase і отримувати колір світлодіода звідти. Відкрийте адмін-панель Firebase та додайте новий запис:

![Adding new record](https://github.com/snipter/firebase-iot-codelab/blob/master/docs/assets/image44.png)

Код підключення до WiFi та Firebase ви можете скопіювати з минулого проекту або з папки `projects/firebase-rgb-led` скачаного з репозиторію. Основний код буде виглядати так:

```c++
// Тут ми будемо зберігати попереднє значення
// отримане з Firebse щоб не перемикати світлодіоди
// щоразу як ми отримаємо дані з Firebase
String prevLedVal;

void loop(){
    // Отримаємо рядок, котрий зберігається
    // за шляхом /led
    String val = Firebase.getString("led");
    // Перевіряєм чи не сталось помилки
    if (Firebase.failed()) {
        Serial.println("Getting data failed");
        delay(2000);
        return;
    }
    // Якщо нове значення відрізняється від попереднього
    // то оновлюємо стан світлодіода
    if(val != prevLedVal){
        Serial.print("New value: ");
        Serial.println(val);
        // Вимикаємо всі світлодіоди
        ledsOff();
        // Перевіряємо отримане значення та вмикаємо необхідний
        // світлодіод
        if(val == "red"){
            redLedOn();
        }else if(val == "green"){
            greenLedOn();
        }else if(val == "blue"){
            blueLedOn();
        }
        // Зберігаємо отримане значення щоб не повторюватись
        prevLedVal = val;
    }
}
```

Змініть значення запису в базі на `red`, `blue` та `off` і перевірте чи все працює вірно.