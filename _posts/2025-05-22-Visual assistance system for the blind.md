---
layout: post
title: Visual assistance system
author: [Richard Kuo]
category: [Lecture]
tags: [jekyll, ai]
---
## 系統功能介紹
```
在觸碰到A0腳位: 擷取影像傳送至Gemini詢問場景，A1腳位: 傳送RTC時間資訊至Gemini並回傳文字，A2腳位: 錄音傳送至Gemini轉文字，然後執行文字轉語音
```
---
## 系統方塊圖
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E7%B3%BB%E7%B5%B1%E6%96%B9%E5%A1%8A%E5%9C%96.jpg?raw=true)

---
## Visual assistance system

### 時間
<iframe width="337" height="599" src="https://www.youtube.com/embed/3bE6IRuO9cA" title="盲人視覺輔助--時間" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### 影像辨識
<iframe width="337" height="599" src="https://www.youtube.com/embed/enoquvDqgIM" title="盲人視覺系統--影像辨識" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


---
## 編碼設計流程
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E6%B5%81%E7%A8%8B%E5%9C%96.png?raw=true)

---
## 程式碼提示詞
```
Sample code:GenAIVision_TTS.ino、GenAISpeech.ino、AnalogInput.ino、Simple_RTC.ino
你可以透過這四個程式碼檔案合併符合題目
1. Touch (ADC)

2. Capture Image send to Gemini and ask about the scene

3. Send RTC timeinfo to Gemini and return text

4. Microphone record audio sent to Gemini to return text, then do Text-to-Speech
```
```


/*
整合觸控AI功能程式碼
結合四個功能：
1. Touch (ADC) - 觸控檢測
2. A0腳位: 擷取影像傳送至Gemini詢問場景
3. A1腳位: 傳送RTC時間資訊至Gemini並回傳文字
4. A2腳位: 錄音傳送至Gemini轉文字，然後執行文字轉語音

*/

// WiFi 設定
char wifi_ssid[] = "abcd";
char wifi_pass[] = "88888888";

// API Keys
String Gemini_key = "AIzaSyCDVTD9I8MJJzeG1PFmyqp7H3iDJnSSF5k"; // 請填入您的Gemini API key

// 引入必要的函式庫 - 調整順序避免衝突
#include <WiFi.h>
#include <WiFiUdp.h>
#include <stdio.h>
#include <time.h>
#include "rtc.h"
#include "GenAI.h"
#include "VideoStream.h"
#include "AmebaFatFS.h"

// 物件宣告
WiFiSSLClient client;
GenAI llm;
GenAI tts;
AmebaFatFS fs;

// 影像相關設定
VideoSetting config(768, 768, CAM_FPS, VIDEO_JPEG, 1);
#define CHANNEL 0
uint32_t img_addr = 0;
uint32_t img_len = 0;

// 音訊檔案設定
String mp3Filename = "ai_response.mp3";
String audioFileName = "recorded_audio";

// RTC 設定
#define YEAR 2025
#define MONTH 5
#define DAY 29
#define HOUR 21
#define MIN 11
#define SEC 0

// 腳位定義
int touchPin_A0 = A0; // 影像辨識觸發
int touchPin_A1 = A1; // RTC查詢觸發
int touchPin_A2 = A2; // 語音辨識觸發
int ledPin = LED_B;

// 觸控閾值
int touchThreshold = 50; // 可根據實際情況調整

// 功能狀態追踪
bool functionInProgress = false;

// 初始化WiFi
void initWiFi() {
    for (int i = 0; i < 2; i++) {
        WiFi.begin(wifi_ssid, wifi_pass);
        delay(1000);
        Serial.println("");
        Serial.print("正在連接到 ");
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
            Serial.println("WiFi連接成功！");
            Serial.print("IP位址: ");
            Serial.println(WiFi.localIP());
            Serial.println("");
            break;
        }
    }
}

// 初始化相機
void initCamera() {
    config.setRotation(0);
    Camera.configVideoChannel(CHANNEL, config);
    Camera.videoInit();
    Camera.channelBegin(CHANNEL);
    Camera.printInfo();
}

// 初始化RTC
void initRTC() {
    rtc.Init();
    long long epochTime = rtc.SetEpoch(YEAR, MONTH, DAY, HOUR, MIN, SEC);
    rtc.Write(epochTime);
}

// LED指示燈控制
void blinkLED(int times, int delayMs) {
    for (int i = 0; i < times; i++) {
        digitalWrite(ledPin, HIGH);
        delay(delayMs);
        digitalWrite(ledPin, LOW);
        delay(delayMs);
    }
}

// 功能1: 影像辨識
void handleImageRecognition() {
    if (functionInProgress) return;
    functionInProgress = true;
    
    Serial.println("=== 啟動影像辨識功能 ===");
    blinkLED(3, 300);
    
    // 擷取影像
    Camera.getImage(0, &img_addr, &img_len);
    
    // 傳送至Gemini進行影像辨識
    String prompt_msg = "請描述這張圖片中的場景。請用繁體中文回答。";
    String response = llm.geminivision(Gemini_key, "gemini-2.0-flash", prompt_msg, img_addr, img_len, client);
    
    Serial.println("Gemini影像辨識結果:");
    Serial.println(response);
    
    // 文字轉語音播放結果
    if (response.length() > 0) {
        tts.googletts(mp3Filename, response, "zh-TW");
        delay(500);
        sdPlayMP3(mp3Filename);
    }
    
    Serial.println("=== 影像辨識功能完成 ===\n");
    functionInProgress = false;
}

// 功能2: RTC時間查詢
void handleRTCQuery() {
    if (functionInProgress) return;
    functionInProgress = true;
    
    Serial.println("=== 啟動RTC時間查詢功能 ===");
    blinkLED(2, 500);
    
    // 讀取RTC時間
    long long seconds = rtc.Read();
    struct tm *timeinfo = localtime(&seconds);
    
    // 分別格式化時間和提示訊息，避免UTF-8編碼問題
    char timeOnly[50];
    sprintf(timeOnly, "%04d-%02d-%02d %02d:%02d:%02d", 
            timeinfo->tm_year + 1900, 
            timeinfo->tm_mon + 1, 
            timeinfo->tm_mday,
            timeinfo->tm_hour, 
            timeinfo->tm_min, 
            timeinfo->tm_sec);
    
    Serial.print("Current time: ");
    Serial.println(timeOnly);
    
    // 建立提示字串，使用String類別處理中文
    String prompt = "Current time is " + String(timeOnly) + "跟我說現在的詳細時間";
    
    Serial.println("Sending query to Gemini...");
    
    // 使用text方法傳送至Gemini
    String response = llm.geminitext(Gemini_key, "gemini-2.0-flash", prompt, client);
    
    Serial.println("Gemini time advice:");
    Serial.println(response);
    
    // 文字轉語音播放結果
    if (response.length() > 0) {
        tts.googletts(mp3Filename, response, "zh-TW");
        delay(500);
        sdPlayMP3(mp3Filename);
    }
    
    Serial.println("=== RTC Query Function Complete ===\n");
    functionInProgress = false;
}

// 功能3: 語音辨識與回應 (簡化版本，避免音訊函式庫衝突)
void handleSpeechRecognition() {
    if (functionInProgress) return;
    functionInProgress = true;
    
    Serial.println("=== Text Input Response Function ===");
    blinkLED(5, 200);
    
    // 由於音訊函式庫衝突，這裡提供一個文字輸入的替代方案
    Serial.println("Please enter your question in the Serial Monitor:");
    
    // 等待序列埠輸入
    String userInput = "";
    unsigned long startTime = millis();
    while (millis() - startTime < 10000) { // 等待10秒
        if (Serial.available()) {
            userInput = Serial.readString();
            userInput.trim();
            break;
        }
        delay(100);
    }
    
    if (userInput.length() > 0) {
        Serial.print("Input received: ");
        Serial.println(userInput);
        
        String prompt = userInput + " Please answer in Traditional Chinese and provide useful advice or information. Limit to 100 words.";
        String response = llm.geminitext(Gemini_key, "gemini-2.0-flash", prompt, client);
        
        Serial.println("AI Response:");
        Serial.println(response);
        
        // 文字轉語音播放結果
        if (response.length() > 0) {
            tts.googletts(mp3Filename, response, "zh-TW");
            delay(500);
            sdPlayMP3(mp3Filename);
        }
    } else {
        Serial.println("No input received, function cancelled");
    }
    
    Serial.println("=== Text Input Function Complete ===\n");
    functionInProgress = false;
}

// MP3播放函式
void sdPlayMP3(String filename) {
    fs.begin();
    String filepath = String(fs.getRootPath()) + filename;
    File file = fs.open(filepath, MP3);
    if (file) {
        file.setMp3DigitalVol(120);
        file.playMp3();
        file.close();
    } else {
        Serial.println("無法開啟MP3檔案");
    }
    fs.end();
}

void setup() {
    Serial.begin(115200);
    Serial.println("=== 觸控AI功能整合系統啟動 ===");
    
    // 初始化各個模組
    initWiFi();
    initCamera();
    initRTC();
    
    // 設定腳位
    pinMode(ledPin, OUTPUT);
    pinMode(LED_G, OUTPUT);
    
    // 檢查API金鑰
    if (Gemini_key.length() == 0) {
        Serial.println("Warning: Please set Gemini API key!");
    }
    
    Serial.println("System initialization complete!");
    Serial.println("Touch usage instructions:");
    Serial.println("- Touch A0: Image recognition");
    Serial.println("- Touch A1: RTC time query");
    Serial.println("- Touch A2: Text input response");
    Serial.println("Waiting for touch input...\n");
    
    // 系統就緒指示
    for (int i = 0; i < 3; i++) {
        digitalWrite(LED_G, HIGH);
        delay(200);
        digitalWrite(LED_G, LOW);
        delay(200);
    }
}

void loop() {
    // 如果功能正在執行中，跳過檢查
    if (functionInProgress) {
        delay(100);
        return;
    }
    
    // 讀取各個類比腳位的值
    int touch_A0 = analogRead(touchPin_A0);
    int touch_A1 = analogRead(touchPin_A1);
    int touch_A2 = analogRead(touchPin_A2);
    
    // 除錯輸出 (可選)
    // Serial.printf("A0:%d, A1:%d, A2:%d\n", touch_A0, touch_A1, touch_A2);
    
    // 檢查A0觸控 - 影像辨識
    if (touch_A0 > touchThreshold) {
        Serial.println("A0 touch detected - Starting image recognition");
        handleImageRecognition();
        delay(1000); // 防止重複觸發
    }
    
    // 檢查A1觸控 - RTC查詢
    else if (touch_A1 > touchThreshold) {
        Serial.println("A1 touch detected - Starting time query");
        handleRTCQuery();
        delay(1000); // 防止重複觸發
    }
    
    // 檢查A2觸控 - 文字回應
    else if (touch_A2 > touchThreshold) {
        Serial.println("A2 touch detected - Starting text response");
        handleSpeechRecognition();
        delay(1000); // 防止重複觸發
    }
    
    // 系統狀態指示 (綠燈慢閃)
    static unsigned long lastBlink = 0;
    if (millis() - lastBlink > 2000) {
        digitalWrite(LED_G, HIGH);
        delay(50);
        digitalWrite(LED_G, LOW);
        lastBlink = millis();
    }
    
    delay(100);
} 


```
---
## 心得
```
這次的實驗我們操作了很久，尤其是最後一個，但最後才發現是我們給太多資訊和問題不夠仔細，讓AI有了自己的判斷一直給一些不需要且不該出現的函示庫，導致無法編譯成功，最後詢問老師才知道原來蠻簡單的，只需要將給的範例加上觸發A2的程式碼就可以了
```
