---
layout: post
title: AI surveillance video system
author: [Richard Kuo]
category: [Lecture]
tags: [jekyll, ai]
---
## 系統功能介紹
`每一分鐘拍攝一次，比較與前一分鐘拍攝影像透過判斷的文字確認是否相同，是則不儲存影像，否則儲存影像並儲存影像文字`

---
## 系統方塊圖
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E7%B3%BB%E7%B5%B1%E6%96%B9%E5%A1%8A%E5%9C%96.jpg?raw=true)

---
## AI surveillance video system

![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E7%9B%A3%E8%A6%96%E9%8C%84%E5%BD%B11.jpg?raw=true)
```text
Time: 20250529_202703
Description: A young woman with glasses wearing a rust-colored sweater looks up with a slightly questioning expression. The lighting overhead is bright.
```
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E7%9B%A3%E8%A6%96%E9%8C%84%E5%BD%B12.jpg?raw=true)
```text
Time: 20250529_202803
Description: A person with glasses is making a peace sign in what appears to be a computer lab.  Bright overhead lights illuminate the room.
```
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E7%9B%A3%E8%A6%96%E9%8C%84%E5%BD%B13.jpg?raw=true)
```text
Time: 20250529_202903
Description: A woman with glasses and a red sweater gives a thumbs up, looking sideways. The background shows a ceiling with fluorescent lights.
```
---
## 系統編碼設計流程
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E6%B5%81%E7%A8%8B%E5%9C%96.png?raw=true)

---
## 程式生成提示詞
```
/*
Sample code:GenAIVision.ino、CaptureJPG_SDcard.ino、Simple_RTC.ino 
Function:
1) capture image per minute and send to Gemini Vision (1分鐘拍一張)
2) if replied text has no change, then dont store the jpg and text
    if replied text are different from the previous scene, then store the jpg and text 
    (use date+time for the filename)

Sample codes:
1. GenAIVision.ino
2. CaptureJPG_SDcard.ino

Code Design: 請AI幫你寫出以上功能的程式碼
```  
---
``` 
/*
#include <WiFi.h>
#include "GenAI.h"
#include "VideoStream.h"
#include "AmebaFatFS.h"
#include <stdio.h>
#include <time.h>
#include "rtc.h"

char ssid[] = "abcd";    // WiFi SSID
char password[] = "88888888";    // WiFi 密碼
String Gemini_key = "AIzaSyC8QKRD3FnpU68O-PIWbDe9viPey2-RoP0";           // 請填入您的 Gemini API 金鑰

WiFiSSLClient client;
GenAI llm;
VideoSetting config(768, 768, CAM_FPS, VIDEO_JPEG, 1);
#define CHANNEL 0

/* 設定初始日期和時間 - 請修改為當前時間 */
#define YEAR  2025
#define MONTH 5
#define DAY   29
#define HOUR 20
#define MIN  41
#define SEC  31

AmebaFatFS fs;
File file;

uint32_t img_addr = 0;
uint32_t img_len = 0;

// 簡潔的提示詞用於場景比較
String prompt_msg = "Describe this image in 1-2 simple sentences. Focus on main objects, people, actions, or text content only.";

unsigned long last_capture_time = 0;
const unsigned long capture_interval = 60000; // 每分鐘拍攝一次 (60秒)

String previous_text = "";  // 儲存前一次的文字結果用於比較
long long seconds = 0;
struct tm *timeinfo;

// 初始化 Wi-Fi 連線
void initWiFi() {
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
}

// 初始化 RTC
void initRTC() {
  rtc.Init();
  long long epochTime = rtc.SetEpoch(YEAR, MONTH, DAY, HOUR, MIN, SEC);
  rtc.Write(epochTime);
  Serial.println("RTC initialized");
}

// 取得當前時間字串格式：YYYYMMDD_HHMMSS
String getCurrentTimeString() {
  seconds = rtc.Read();
  timeinfo = localtime(&seconds);
  
  char timeStr[32]; // 增加緩衝區大小到32
  sprintf(timeStr, "%04d%02d%02d_%02d%02d%02d", 
          timeinfo->tm_year + 1900,
          timeinfo->tm_mon + 1,
          timeinfo->tm_mday,
          timeinfo->tm_hour,
          timeinfo->tm_min,
          timeinfo->tm_sec);
  
  return String(timeStr);
}

// 儲存圖片到SD卡
bool saveImageToSD(String timeStr) {
  String filename = "IMG_" + timeStr + ".jpg";
  
  fs.begin();
  file = fs.open(String(fs.getRootPath()) + String(filename));
  
  if (file) {
    file.write((uint8_t *)img_addr, img_len);
    file.close();
    fs.end();
    Serial.println("Image saved: " + filename);
    return true;
  } else {
    Serial.println("Failed to save image: " + filename);
    fs.end();
    return false;
  }
}

// 儲存文字到SD卡
bool saveTextToSD(String timeStr, String text) {
  String filename = "TXT_" + timeStr + ".txt";
  
  fs.begin();
  File txtFile = fs.open(String(fs.getRootPath()) + String(filename));
  
  if (txtFile) {
    txtFile.print("Time: ");
    txtFile.print(timeStr);
    txtFile.print("\nDescription: ");
    txtFile.print(text);
    txtFile.close();
    fs.end();
    Serial.println("Text saved: " + filename);
    return true;
  } else {
    Serial.println("Failed to save text: " + filename);
    fs.end();
    return false;
  }
}

// 檢查文字是否有變化
bool hasTextChanged(String newText, String oldText) {
  // 移除前後空白並轉為小寫比較
  String newTrimmed = newText;
  String oldTrimmed = oldText;
  newTrimmed.trim();
  oldTrimmed.trim();
  newTrimmed.toLowerCase();
  oldTrimmed.toLowerCase();
  
  // 如果是第一次拍攝(沒有舊文字)，視為有變化
  if (oldTrimmed.length() == 0) {
    return true;
  }
  
  // 檢查新文字是否為空或太短
  if (newTrimmed.length() < 5) {
    return false;
  }
  
  // 比較文字是否相同
  return !newTrimmed.equals(oldTrimmed);
}

// 每分鐘自動拍攝功能
void autoCapture() {
  Serial.println("=== Auto Capture (Every Minute) ===");
  
  // 1) 拍攝圖片
  Camera.getImage(CHANNEL, &img_addr, &img_len);
  Serial.println("Image captured, sending to Gemini Vision...");

  // 檢查圖片是否成功拍攝
  if (img_len == 0) {
    Serial.println("Failed to capture image, skipping...");
    return;
  }

  // 2) 發送到 Gemini Vision 分析
  String current_text = llm.geminivision(Gemini_key, "gemini-2.0-flash", prompt_msg, img_addr, img_len, client);
  
  // 檢查API回應
  if (current_text.length() == 0) {
    Serial.println("No response from Gemini Vision, skipping...");
    return;
  }
  
  Serial.println("Gemini Vision Analysis:");
  Serial.println("Current: " + current_text);
  Serial.println("Previous: " + previous_text);

  // 3) 比較文字是否有變化
  if (hasTextChanged(current_text, previous_text)) {
    Serial.println(">>> SCENE CHANGED - Saving files...");
    
    String timeStr = getCurrentTimeString();
    
    // 儲存圖片和文字檔案
    if (saveImageToSD(timeStr)) {
      if (saveTextToSD(timeStr, current_text)) {
        previous_text = current_text;  // 只有成功儲存後才更新比較基準
        Serial.println("Files saved with timestamp: " + timeStr);
      }
    }
  } else {
    Serial.println(">>> NO CHANGE - Skipping save");
  }
  
  Serial.println("=====================================\n");
}

void setup() {
  Serial.begin(115200);
  
  // 初始化各項功能
  initWiFi();
  initRTC();

  // 初始化攝影機
  config.setRotation(0);
  Camera.configVideoChannel(CHANNEL, config);
  Camera.videoInit();
  Camera.channelBegin(CHANNEL);
  Camera.printInfo();
  
  Serial.println("\n=== System Ready ===");
  Serial.println("Function: Auto capture every minute");
  Serial.println("Logic: Only save when scene changes");
  Serial.println("File naming: date+time format");
  
  // 顯示當前時間
  String currentTime = getCurrentTimeString();
  Serial.println("Current time: " + currentTime);
  Serial.println("====================\n");
  
  // 設定開始時間
  last_capture_time = millis();
}

void loop() {
  // 檢查是否到了每分鐘拍攝時間
  unsigned long current_time = millis();
  
  if (current_time - last_capture_time >= capture_interval) {
    autoCapture();  // 執行自動拍攝和分析
    last_capture_time = current_time;  // 重設計時器
  }
  
  // 短暫延遲以避免過度消耗CPU
  delay(100);
}

```
---
## 心得

