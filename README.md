# Manuel
by Senna Hoving 

## Table of Contents
![Introduction](##introduction)
![Stap1](##stap1)
![Stap2](##stap2)
![Stap3](##stap3)
![Stap4](##stap4)

## Introduction
This is a manual for instructing your NodeMCU via Telegram.

attributes required: 
- NodeMCU Arduino board esp8266
- LED strip
- Hotspot
- Arduino IDE

## Stap 1: Telegram instellen
Download [Telegram](https://web.telegram.org/a/). 

Maak een account aan op Telegram 

Zoek naar de gebruiker 'Botfather'. 
![Screenshot Botfather](/IoT/assets/Screenshot_20241003_094644_Telegram.jpg) 

Open een chat met Botfather, klik vervolgens op beginnen en voer het volgende command uit in de chat `/newbot`

Dan moet je in dezelfde chat deze bot een naam, en unieke username geven

Als dan alles goed is gegaan heb je nu een eigen bot, deze zou je in een apparte chat moeten zien. 
![Screenshot Telegram chats]()

## Stap 2: Opzetten van de Arduino IDE
Open Arduino IDE, en open de library manager (crl + shift + i)

zoek vervolgens naar de volgende twee libraries en installeer deze: 
> Universaltelegrambot - Brian Lough
> ArduinoJson - Benoit Blanchon

Na het installeren van deze twee libraries kun je de example 'echobot' uit de universaltelegrambot openen. Klik op **Files -> Examples -> Universaltelegrambot (misschien nodig om naar beneden te scrollen) -> Echobot**
![Screenshot Echobot example](/IoT/assets/Schermafbeelding%202024-10-03%20095805.png)

Als het goed is heb je nu de volgende code: 
![Screenshot code example](/IoT/assets/Schermafbeelding%202024-10-03%20095805.png)

Pas vervolgens op lijn 27 & 28 de waardes aan naar jouw netwerk (naam en wachtwoord)

Vraag de token van jouw bot aan door de command `/token` in de Botfather chat te zetten, en selecteerd dan de bot die jij hiervoor hebt aangemaakt. 

Zet deze token dan neer op lijn 30. 

Dan als alles correct is ingevuld, kun je de bot testen door een bericht te sturen naar jouw eigen bot. Let hierbij op dat je met hetzelfde wifi netwerk verbonden ben als de NodeMCU.
Ook nog iets over de BAUS 
Als dan alles goed is krijg je het volgende bericht te zien in de serial monitor: 
> got response

## stap 3 Telegram met NodeMCU verbinden
Verander het antwoord van jouw bot door `bot.messages[i].text` te vervangen door bijvoorbeeld `"Duurt even maar komt eraan!"` 

Zet `pinMode(LED_BUILTIN, OUTPUT);` in de void setup

Hierna kun je met de volgende regels het bericht ophalen wat jij naar de bot stuurt
```
      for (int i = 0; i == numNewMessages; i++) {
        String bericht = bot.messages[i].text;
      }
```

Dit kun je testen door `Serial.println(bericht);` hieraan toe te voegen, waardoor je jouw berichten in de serial monitor krijgt te zien


Voeg vervolgens de volgende if / else if statement toe binnen de bovenstaande for loop. 
```
        if (bericht == "aan") {
            digitalWrite(LED_BUILTIN, LOW);
        } else if (bericht == "uit") {
            digitalWrite(LED_BUILTIN, HIGH);
        }
```

## Stap 4 Aansturen van de LED strip
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

## Fouten / Errors 
Dit zijn de errors die je tegen kan komen tijdens dit process 

* denk deze gewoon tijdens het stappenplan schrijven * 

## Code review
```
#include <ESP8266WiFi.h>
#include <WiFiClientSecure.h>
#include <UniversalTelegramBot.h>
#include <Adafruit_NeoPixel.h>

// Wifi network station credentials
#define WIFI_SSID "AndroidAP"
#define WIFI_PASSWORD "Nodeboard"
// Telegram BOT Token (Get from Botfather)
#define BOT_TOKEN "7563993381:AAG7Udj5LDuLdyyWEjnFwGo3Jpvb1Wwy6pA"

#define PIN D1
#define NUMPIXELS 15 
Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

const unsigned long BOT_MTBS = 1000; // mean time between scan messages

X509List cert(TELEGRAM_CERTIFICATE_ROOT);
WiFiClientSecure secured_client;
UniversalTelegramBot bot(BOT_TOKEN, secured_client);
unsigned long bot_lasttime; // last time messages' scan has been done

void handleNewMessages(int numNewMessages)
{
  for (int i = 0; i < numNewMessages; i++)
  {
    bot.sendMessage(bot.messages[i].chat_id, "Duurt even maar komt eraan!", "");
  }
}

void setup()
{
  Serial.begin(115200);
  Serial.println();

  // attempt to connect to Wifi network:
  Serial.print("Connecting to Wifi SSID ");
  Serial.print(WIFI_SSID);
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  secured_client.setTrustAnchors(&cert); // Add root certificate for api.telegram.org
  
  while (WiFi.status() != WL_CONNECTED)
  {
    Serial.print(".");
    delay(500);
  }
  Serial.print("\nWiFi connected. IP address: ");
  Serial.println(WiFi.localIP());

  Serial.print("Retrieving time: ");
  configTime(0, 0, "pool.ntp.org"); // get UTC time via NTP
  time_t now = time(nullptr);
  while (now < 24 * 3600)
  {
    Serial.print(".");
    delay(100);
    now = time(nullptr);
  }
  Serial.println(now);

  pixels.begin();
}

void loop()
{
  if (millis() - bot_lasttime > BOT_MTBS)
  {
    int numNewMessages = bot.getUpdates(bot.last_message_received + 1);

    while (numNewMessages)
    {
      Serial.println("got response");
      handleNewMessages(numNewMessages);
      numNewMessages = bot.getUpdates(bot.last_message_received + 1);

      for (int i = 0; i == numNewMessages; i++)
      {
        String bericht = bot.messages[i].text;
        Serial.println(bericht);

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
      }
    }

    bot_lasttime = millis();
  }
}`
```

## Bronnen
DfETsr IOT (https://icthva.sharepoint.com/:w:/s/FDMCI_ORG__CMD-Amsterdam/Eb7Jd27yWphMuVFbMHV_9WoBEg5_zqAQilsb6Q3gPSKueg?e=f5PM7l)