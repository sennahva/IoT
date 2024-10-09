# Manuel
by Senna Hoving 

## Introduction
Dit is een manual voor het aansturen van een NodeMCU via Telegram

Benodigdheden: 
- NodeMCU Arduino board esp8266
- LED strip
- Hotspot / internet verbinding
- Arduino IDE

## Stap 1: Telegram instellen
Download [Telegram](https://web.telegram.org/a/)

Maak een account aan op Telegram 

Zoek naar de gebruiker 'Botfather'. 
![Screenshot_20241003_094644_Telegram](https://github.com/user-attachments/assets/af1b77d9-483a-4e8d-ba63-47d81ceb9a27)

Open een chat met Botfather, klik vervolgens op beginnen en voer het volgende command uit in de chat `/newbot`
![Screenshot_20241003_094740_Telegram](https://github.com/user-attachments/assets/573b4bf4-75ca-4146-aace-c1b80c03001b)

Dan moet je in dezelfde chat deze bot een naam, en unieke username geven
![Screenshot_20241003_094841_Telegram](https://github.com/user-attachments/assets/95ffcc39-c6d6-481b-b865-5097ca5f7b22)

Als dan alles goed is gegaan heb je nu een eigen bot, deze zou je in een apparte chat moeten zien. 
![Screenshot_20241004_135047_Telegram](https://github.com/user-attachments/assets/96e28c2a-09e1-4ca5-8d6f-efeb9055d814)


## Stap 2: Opzetten van de Arduino IDE
Open Arduino IDE, en open de library manager (crl + shift + i)

zoek vervolgens naar de volgende twee libraries en installeer deze: **Universaltelegrambot - Brian Lough** en **ArduinoJson - Benoit Blanchon**

Na het installeren van deze twee libraries kun je de example 'echobot' uit de universaltelegrambot openen. Klik op **Files -> Examples -> Universaltelegrambot (misschien nodig om naar beneden te scrollen) -> Echobot**
![Schermafbeelding 2024-10-03 095805](https://github.com/user-attachments/assets/db858f92-8747-43ba-825e-e93730238a64)

Als het goed is heb je nu de volgende code: 
![Schermafbeelding 2024-10-03 155257](https://github.com/user-attachments/assets/da9fb3c4-7ab3-4ecf-bcc6-5dd80f3cb469)

Pas vervolgens op lijn 27 & 28 de waardes aan naar jouw netwerk (naam en wachtwoord)

Vraag de token van jouw bot aan door de command `/token` in de Botfather chat te zetten, en selecteerd dan de bot die jij bij stap 1 hebt aangemaakt. 

Zet deze token dan neer op lijn 30

Dan als alles correct is ingevuld, kun je de bot testen door een bericht te sturen naar jouw eigen bot. Let hierbij op dat je met hetzelfde wifi netwerk verbonden ben als de NodeMCU.
En iets waar ik zelf tegenaan liep is dat de baud van mijn serial monitor verkeerd stond, dus controlleer of die op **115200** staat. 
(screenshot baud monitor)
Als dan alles goed is krijg je dan "got response" te zien in de serial monitor: 

## stap 3: Telegram met NodeMCU verbinden
Verander het antwoord van jouw bot door `bot.messages[i].text` te vervangen door bijvoorbeeld `"Duurt even maar komt eraan!"` 

Zet `pinMode(LED_BUILTIN, OUTPUT);` in de void setup

Hierna kun je met de volgende regels het bericht ophalen wat jij naar de bot stuurt
```
      for (int i = 0; i == numNewMessages; i++) {
        String bericht = bot.messages[i].text;
      }
```

Gebruik hierbij echt de for loop, anders krijg je een error terug: 
(screen shot missing 'i')

Dit kun je testen door `Serial.println(bericht);` hieraan toe te voegen, waardoor je jouw berichten in de serial monitor krijgt te zien

Let hierbij ook weer op dat de hierbij je baud nog steeds op 115200 staat.


Voeg vervolgens de volgende if / else if statement toe binnen de bovenstaande for loop. 
```
        if (bericht == "aan") {
            digitalWrite(LED_BUILTIN, LOW);
        } else if (bericht == "uit") {
            digitalWrite(LED_BUILTIN, HIGH);
        }
```

Een error die ik hier ben tegengekomen is dat ik enkele aanhalingstekens heb gebruikt, maar dat werkte niet. Dus let hierbij op dat je dubbele aanhalingstekens gebruikt. 
(screenshot ")

## Stap 4: Aansturen van de LED strip
Installeer de library `Adafruit NeoPixel - Adafruit

Voeg de library toe door middel van `#include <Adafruit_NeoPixel.h>` boven in de code te zetten

Voeg ververvolgens ook jouw configuraties toe door middel van het onderstaand, let hierbij op dat je de waardes bij PIN en NUMPIXELS vervangd voor jouw LED strip:
```
#define PIN D1
#define NUMPIXELS 15 
Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);
``` 

Als dat gelukt is kun je de `pinMode(LED_BUILTIN, OUTPUT);` in de setup vervangen door `pixels.begin();` 

En als laatste kun je de led functie bij 'aan' en 'uit' aanpassen zo get als je wilt. Bijvoorbeeld: 
```
        if (bericht == "aan") {
          while (bericht == "aan") {
            for (int i = 0; i < NUMPIXELS; i++) {
              pixels.setPixelColor(i, pixels.Color(random(0, 255), random(0, 255), random(0, 255)));
              pixels.show();
              delay(100);
            }
          }
        } else if (bericht == "uit") {
          pixels.clear();
          pixels.show();
        }
```

## Bronnen
DfETsr IOT (https://icthva.sharepoint.com/:w:/s/FDMCI_ORG__CMD-Amsterdam/Eb7Jd27yWphMuVFbMHV_9WoBEg5_zqAQilsb6Q3gPSKueg?e=f5PM7l)
