---
layout: post
title: AI-assisted Recycle System
author: [Richard Kuo]
category: [Lecture]
tags: [jekyll, ai]
---



---
## AI-assisted Recycle System

![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E8%BC%94%E5%8A%A9%E5%9B%9E%E6%94%B61.jpg?raw=true)

---
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E8%BC%94%E5%8A%A9%E5%9B%9E%E6%94%B62.jpg?raw=true)

---
<iframe width="442" height="785" src="https://www.youtube.com/embed/o9eQNXdhivI" title="心跳血氧偵測器" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

---
## 系統方塊圖
![](https://github.com/peijia0809/MCU-project/blob/main/_posts/thinkspeak.png?raw=true)
```

---

  
---
/*
 * 整合 GenAI Vision, TTS 和 LCD 顯示的 Arduino 程式碼
 * 功能：
 * 1. 按下按鈕拍攝照片
 * 2. 發送到 Google-Gemini 進行圖像識別
 * 3. 將識別結果顯示在 LCD 上
 * 4. 使用 Google-TTS 將結果轉為語音播放
 */

// WiFi 設定
char wifi_ssid[] = "TCFSTWIFI.ALL";    // 你的 WiFi SSID
char wifi_pass[] = "035623116";        // 你的 WiFi 密碼

// API 金鑰
String Gemini_key = "";               // 貼上你的 Gemini API 金鑰
String google_tts_key = "";           // 如果需要，貼上你的 Google TTS API 金鑰

// 硬體設定
const int buttonPin = 1;              // 按鈕接腳
#define CHANNEL 0                     // 攝影機通道
#define TFT_RESET 5                   // LCD 重置接腳
#define TFT_DC 4                      // LCD 資料/命令接腳
#define TFT_CS SPI_SS                 // LCD 片選接腳
#define ILI9341_SPI_FREQUENCY 20000000 // LCD SPI 頻率
#define FONTSIZE 2                    // 字體大小
#define TEXTCOLOR ILI9341_GREEN       // 文字顏色

// 包含必要的庫
#include <WiFi.h>
#include <WiFiSSLClient.h>
#include "GenAI.h"
#include "VideoStream.h"
#include "AmebaFatFS.h"
#include "SPI.h"
#include "AmebaILI9341.h"

// 創建物件實例
WiFiSSLClient client;
GenAI llm;
VideoSetting config(768, 768, CAM_FPS, VIDEO_JPEG, 1);
AmebaFatFS fs;
AmebaILI9341 tft = AmebaILI9341(TFT_CS, TFT_DC, TFT_RESET);

// 全局變數
uint32_t img_addr = 0;
uint32_t img_len = 0;
String prompt_msg = "Please describe the image, and if there is a text, please summarize the content";
String mp3Filename = "gemini_response.mp3";

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

void displayTextOnLCD(String text)
{
    init_TFTLCD(TEXTCOLOR, FONTSIZE);
    tft.setRotation(0); // 固定方向
    tft.println("Gemini Response:");
    tft.println("----------------");
    tft.println(text);
}

void setup()
{
    Serial.begin(115200);

    // 初始化 WiFi
    initWiFi();

    // 初始化攝影機
    config.setRotation(0);
    Camera.configVideoChannel(CHANNEL, config);
    Camera.videoInit();
    Camera.channelBegin(CHANNEL);
    Camera.printInfo();
    
    // 初始化按鈕和 LED
    pinMode(buttonPin, INPUT);
    pinMode(LED_B, OUTPUT);
    pinMode(LED_G, OUTPUT);
    
    // 初始化 LCD
    SPI.setDefaultFrequency(ILI9341_SPI_FREQUENCY);
    tft.begin();
    init_TFTLCD(TEXTCOLOR, FONTSIZE);
    tft.println("System Ready");
    tft.println("Press button to");
    tft.println("capture & analyze");
}

void loop()
{
    if ((digitalRead(buttonPin)) == 1) {
        // 指示按鈕按下
        for (int count = 0; count < 3; count++) {
            digitalWrite(LED_B, HIGH);
            delay(500);
            digitalWrite(LED_B, LOW);
            delay(500);
        }

        // 拍攝照片
        Camera.getImage(0, &img_addr, &img_len);
        
        // 顯示狀態
        init_TFTLCD(TEXTCOLOR, FONTSIZE);
        tft.println("Processing image...");
        
        // 發送到 Gemini 進行圖像識別
        String response = llm.geminivision(Gemini_key, "gemini-2.0-flash", prompt_msg, img_addr, img_len, client);
        Serial.println("Gemini Response:");
        Serial.println(response);
        
        // 在 LCD 上顯示結果
        displayTextOnLCD(response);
        
        // 使用 TTS 播放結果
        tft.println("Generating speech...");
        llm.googletts(mp3Filename, response, "en-US");
        delay(500);
        sdPlayMP3(mp3Filename);
        
        // 完成指示
        digitalWrite(LED_G, HIGH);
        delay(1000);
        digitalWrite(LED_G, LOW);
    }
    
    delay(100); // 防止按鈕彈跳
}

  
<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*

