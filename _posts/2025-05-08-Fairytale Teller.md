---
layout: post
title: Fairytale Teller
author: [Richard Kuo]
category: [Lecture]
tags: [jekyll, ai]

---
## 系統功能介紹
```
透過鏡頭拍攝照片，利用AI將圖片以故事敘述出來
```
---
## 系統方塊圖
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E7%B3%BB%E7%B5%B1%E6%96%B9%E5%A1%8A%E5%9C%96.jpg?raw=true)

---
## Fairytale Teller

<iframe width="337" height="599" src="https://www.youtube.com/embed/lq2bxWHAIII" title="2025年6月6日" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---
## 編碼設計流程
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E6%B5%81%E7%A8%8B%E5%9C%96.png?raw=true)

---
## 程式碼提示詞
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E7%9C%8B%E5%9C%96%E8%AA%AA%E6%95%85%E4%BA%8B.png?raw=true)
```

/*
This sketch captures an image when button is pressed, sends it to Gemini-Vision to generate a fairytale,
then uses Google TTS to speak the story.

Required libraries:
- WiFi
- GenAI
- VideoStream
- WiFiSSLClient
- GoogleTTS (you'll need to add this library)

Credit : ChungYi Fu (Kaohsiung, Taiwan)
Modified for fairytale generation and TTS
*/

String openAI_key = "";               // paste your generated openAI API key here
String Gemini_key = "";               // paste your generated Gemini API key here
String Llama_key = "";                // paste your generated Llama API key here
char wifi_ssid[] = "TCFSTWIFI.ALL";    // your network SSID (name)
char wifi_pass[] = "035623116";        // your network password

#include <WiFi.h>
#include "GenAI.h"
#include "VideoStream.h"
#include "GoogleTTS.h"  // Make sure you have this library installed
WiFiSSLClient client;
GenAI llm;
GoogleTTS tts;
VideoSetting config(768, 768, CAM_FPS, VIDEO_JPEG, 1);
#define CHANNEL 0

uint32_t img_addr = 0;
uint32_t img_len = 0;

// Modified prompt to ask for a fairytale
String prompt_msg = "Please look at this image and create a short, imaginative fairytale story based on what you see. Make it suitable for children with a happy ending.";

const int buttonPin = 1;          // the number of the pushbutton pin
const int speakerPin = 2;         // pin connected to speaker or audio output

void initWiFi()
{
    for (int i = 0; i < 2; i++) {
        WiFi.begin(wifi_ssid, wifi_pass);

        delay(1000);
        Serial.println("");
        Serial.print("Connecting to ");
        Serial.println(wifi_ssid);

        uint32_t StartTime = millis();
        while (WiFi.status() != WL_CONNECTED) {
            delay(500);
            if ((StartTime + 5000) < millis()) {
                break;
            }
        }

        if (WiFi.status() == WL_CONNECTED) {
            Serial.println("");
            Serial.println("STAIP address: ");
            Serial.println(WiFi.localIP());
            Serial.println("");
            break;
        }
    }
}

void setup()
{
    Serial.begin(115200);

    initWiFi();

    config.setRotation(0);
    Camera.configVideoChannel(CHANNEL, config);
    Camera.videoInit();
    Camera.channelBegin(CHANNEL);
    Camera.printInfo();
    
    pinMode(buttonPin, INPUT);
    pinMode(LED_B, OUTPUT);
    pinMode(LED_G, OUTPUT);
    pinMode(speakerPin, OUTPUT);
    
    // Initialize TTS (adjust parameters as needed)
    tts.setLanguage("en");  // Set to English
    tts.setSpeed(1.0);      // Normal speed
    tts.setPitch(1.0);      // Normal pitch
}

void loop()
{
    if ((digitalRead(buttonPin)) == HIGH) {
        // Visual feedback that button was pressed
        for (int count = 0; count < 3; count++) {
            digitalWrite(LED_B, HIGH);
            delay(500);
            digitalWrite(LED_B, LOW);
            delay(500);
        }

        // Capture image
        Camera.getImage(CHANNEL, &img_addr, &img_len);
        
        // Send to Gemini Vision to generate a fairytale
        digitalWrite(LED_G, HIGH);  // Indicate processing
        String story = llm.geminivision(Gemini_key, "gemini-1.5-pro", prompt_msg, img_addr, img_len, client);
        digitalWrite(LED_G, LOW);
        
        Serial.println("Generated Fairytale:");
        Serial.println(story);
        
        // Send story to Google TTS and play it
        if (story.length() > 0) {
            Serial.println("Playing story with TTS...");
            digitalWrite(LED_B, HIGH);  // Indicate audio playback
            
            // Split long text into chunks if needed (Google TTS may have character limits)
            int maxChunkSize = 200;  // Adjust based on TTS limitations
            for (int i = 0; i < story.length(); i += maxChunkSize) {
                String chunk = story.substring(i, min(i + maxChunkSize, story.length()));
                
                // Get TTS audio and play it
                tts.speak(chunk, speakerPin);
                
                // Wait for playback to finish (adjust delay as needed)
                delay(1000);
            }
            
            digitalWrite(LED_B, LOW);
        } else {
            Serial.println("No story was generated.");
        }
        
        // Small delay to prevent multiple triggers
        delay(1000);
    }
}
```
---
## 心得
```  
這次的實驗也是對學習英文很有用，只要透過一些材料組合起來就能擁有一個簡易的英文造句，而且會唸出來，只要自己會這些材料和懂得如何問AI就能有簡易便宜的工具
```
