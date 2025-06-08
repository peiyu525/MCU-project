---
layout: post
title: AI-assisted Educational System
author: [Richard Kuo]
category: [Lecture]
tags: [jekyll, ai]
---
## 系統功能介紹
```
透過鏡頭拍攝文字，利用拍攝的文字造句並唸出來
```
---
## 系統方塊圖
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E7%B3%BB%E7%B5%B1%E6%96%B9%E5%A1%8A%E5%9C%96.jpg?raw=true)

---
## AI-assisted Educational System

![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E8%BC%94%E5%8A%A9%E8%8B%B1%E6%96%87%E6%95%99%E5%AD%B81.jpg?raw=true)

![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E8%BC%94%E5%8A%A9%E8%8B%B1%E6%96%87%E6%95%99%E5%AD%B82.jpg?raw=true)

---
## 編碼設計流程

---
## 程式設計提示詞
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E8%AE%80%E5%AD%97%E5%8D%A1.png?raw=true)
```
/*
Combined sketch that:
1. Press button to capture an image
2. Send Image to Gemini-Vision to read the word card
3. Send Text1 to Google-TTS and play mp3 file to speak 
4. Send Text2 to Gemini-LLM to make a sentence
5. Send Text2 to Google-TTS and play mp3 file to speak

Combines functionality from:
- GenAIVision.ino
- TextToSpeech.ino
- ILI9341_TFTLCD_Text.ino
*/

// Configuration section
String Gemini_key = "";               // paste your generated Gemini API key here
char wifi_ssid[] = "TCFSTWIFI.ALL";   // your network SSID (name)
char wifi_pass[] = "035623116";       // your network password

// Includes
#include <WiFi.h>
#include <WiFiUdp.h>
#include "GenAI.h"
#include "VideoStream.h"
#include "AmebaFatFS.h"
#include "SPI.h"
#include "AmebaILI9341.h"

// TFT LCD Configuration
#define TFT_RESET       5
#define TFT_DC          4
#define TFT_CS          SPI_SS
AmebaILI9341 tft = AmebaILI9341(TFT_CS, TFT_DC, TFT_RESET);
#define ILI9341_SPI_FREQUENCY 20000000
#define FONTSIZE 2
#define TEXTCOLOR ILI9341_GREEN
#define INITIAL_TEXT "AMB82-mini with GenAI"

// Video and AI Configuration
WiFiSSLClient client;
GenAI llm;
VideoSetting config(768, 768, CAM_FPS, VIDEO_JPEG, 1);
#define CHANNEL 0
uint32_t img_addr = 0;
uint32_t img_len = 0;

// Text-to-Speech Configuration
AmebaFatFS fs;
GenAI tts;
String mp3Filename1 = "text1.mp3";
String mp3Filename2 = "text2.mp3";

// Button and LED Configuration
const int buttonPin = 1;          // the number of the pushbutton pin

// Prompts
String vision_prompt = "Please read the text in this image clearly and accurately. If there are multiple words or sentences, read them all.";
String llm_prompt = "Make a simple sentence using this word: ";

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

void init_TFTLCD(int textcolor, int fontsize)
{
    tft.clr();
    tft.setCursor(0, 0);
    tft.setForeground(textcolor);
    tft.setFontSize(fontsize);
}

void sdPlayMP3(String filename)
{
    fs.begin();
    String filepath = String(fs.getRootPath()) + filename;
    File file = fs.open(filepath, MP3);
    file.setMp3DigitalVol(120);
    file.playMp3();
    file.close();
    fs.end();
}

void displayText(String text) {
    init_TFTLCD(TEXTCOLOR, FONTSIZE);
    tft.setRotation(0);
    tft.println(text);
}

void setup()
{
    Serial.begin(115200);
    
    // Initialize WiFi
    initWiFi();
    
    // Initialize TFT LCD
    SPI.setDefaultFrequency(ILI9341_SPI_FREQUENCY);
    tft.begin();
    displayText(INITIAL_TEXT);
    
    // Initialize Camera
    config.setRotation(0);
    Camera.configVideoChannel(CHANNEL, config);
    Camera.videoInit();
    Camera.channelBegin(CHANNEL);
    Camera.printInfo();
    
    // Initialize pins
    pinMode(buttonPin, INPUT);
    pinMode(LED_B, OUTPUT);
    pinMode(LED_G, OUTPUT);
}

void loop()
{
    if ((digitalRead(buttonPin)) == 1) {
        // Visual feedback that button was pressed
        for (int count = 0; count < 3; count++) {
            digitalWrite(LED_B, HIGH);
            delay(500);
            digitalWrite(LED_B, LOW);
            delay(500);
        }

        // Capture image
        Camera.getImage(0, &img_addr, &img_len);
        
        // Step 1: Send image to Gemini Vision
        String extracted_text = llm.geminivision(Gemini_key, "gemini-2.0-flash", vision_prompt, img_addr, img_len, client);
        Serial.println("Extracted text: " + extracted_text);
        displayText("Extracted: " + extracted_text);
        
        // Step 2: Convert extracted text to speech and play it
        tts.googletts(mp3Filename1, extracted_text, "en-US");
        delay(500);
        sdPlayMP3(mp3Filename1);
        delay(2000); // Wait for speech to finish
        
        // Step 3: Send text to Gemini LLM to make a sentence
        String full_llm_prompt = llm_prompt + extracted_text;
        String generated_sentence = llm.geminillm(Gemini_key, "gemini-2.0-flash", full_llm_prompt, client);
        Serial.println("Generated sentence: " + generated_sentence);
        displayText("Sentence: " + generated_sentence);
        
        // Step 4: Convert generated sentence to speech and play it
        tts.googletts(mp3Filename2, generated_sentence, "en-US");
        delay(500);
        sdPlayMP3(mp3Filename2);
        delay(2000); // Wait for speech to finish
    }
    
    delay(100); // Small delay to prevent button bounce
}
---

```  

---
## 心得

  

