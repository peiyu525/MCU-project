---
layout: post
title: Navigation system for the blind
author: [Richard Kuo]
category: [Lecture]
tags: [jekyll, ai]
---
## 系統功能介紹
```
透過鏡頭掃描QR code念出地點
```
---
## Navigation system for the blind

<iframe width="337" height="599" src="https://www.youtube.com/embed/5r9KPE47gI4" title="盲人導航系統" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---
## 系統方塊圖
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/image.png?raw=true)
```

---

/*
#undef DEFAULT
#include "VideoStream.h"
#include "QRCodeScanner.h"
#include "WiFi.h"
#include <WiFiClient.h>
#include <WiFiSSLClient.h>
#include "AmebaFatFS.h"
#include "GenAI.h"
// WiFi 設定
char ssid[] = "abcd"; // 請替換為您的 WiFi 名稱
char pass[] = "88888888"; // 請替換為您的 WiFi 密碼
// 視頻設定
#define CHANNEL 0
VideoSetting config(CHANNEL);
// 各功能模組
QRCodeScanner Scanner;
AmebaFatFS fs;
GenAI tts;
// 系統狀態
bool isScanning = true;
bool hasNewQRResult = false;
String lastQRResult = "";
String mp3Filename = "location_speech.mp3";
// 初始化 WiFi 連接
void initWiFi()
{
for (int i = 0; i < 2; i++) {
WiFi.begin(ssid, pass);
delay(1000);
Serial.print("正在連接到 ");
Serial.println(ssid);
uint32_t StartTime = millis();
while (WiFi.status() != WL_CONNECTED) {
delay(500);
if ((StartTime + 5000) < millis()) {
break;
}
}
if (WiFi.status() == WL_CONNECTED) {
Serial.println("WiFi 連接成功！");
Serial.print("IP 地址: ");
Serial.println(WiFi.localIP());
return;
}
}
Serial.println("WiFi 連接失敗！請檢查網路設定。");
}
// 初始化攝像頭與 QR 掃描器
void initCamera()
{
Camera.configVideoChannel(CHANNEL, config);
Camera.videoInit();
Scanner.StartScanning();
}
// 掃描 QR Code
void scanQRCode()
{
Scanner.GetResultString();
Scanner.GetResultLength();
if (Scanner.ResultString != nullptr && Scanner.ResultLength >
0) {
String currentResult = String(Scanner.ResultString);
if (currentResult != lastQRResult ||
lastQRResult.length() == 0) {
lastQRResult = currentResult;
hasNewQRResult = true;
}
}
}
// 處理掃描到的 QR Code 進行語音播報
void processLocationSpeech(String locationName)
{
if (WiFi.status() != WL_CONNECTED) {
Serial.println("WiFi 未連接，使用離線播報模式");
playDefaultMessage(locationName);
return;
}
String speechText = "Current location is " + locationName;
tts.googletts(mp3Filename, speechText, "en-US");
delay(500); // 等待語音檔案生成
// 播放語音檔案
sdPlayMP3(mp3Filename);
}
// 播放預設語音
void playDefaultMessage(String locationName)
{
Serial.println("播放預設訊息: " + locationName);
String defaultMessage = "此為預設訊息。當無法獲取 TTS 音訊時播放此訊
息。";
tts.googletts(mp3Filename, defaultMessage, "en-US");
delay(500); // 等待語音檔案生成
sdPlayMP3(mp3Filename);
}
// 播放 MP3 檔案
void sdPlayMP3(String filename)
{
fs.begin();
String filepath = String(fs.getRootPath()) + filename;
File file = fs.open(filepath, MP3);
file.setMp3DigitalVol(120); // 設定音量
file.playMp3();
file.close();
fs.end();
}
void setup()
{
Serial.begin(115200);
initWiFi();
initCamera();
fs.begin();
Serial.println("系統初始化完成，開始掃描 QR Code...");
}
void loop()
{
scanQRCode();
if (hasNewQRResult) {
Serial.println("掃描結果: " + lastQRResult);
processLocationSpeech(lastQRResult);
hasNewQRResult = false;
isScanning = false; // 停止掃描
Serial.println("掃描完成，停止掃描。");
}
if (!isScanning) {
Serial.println("已完成掃描，停止進行新的掃描。");
}
delay(500); // 減少延遲時間，增加掃描頻率
}
```
---

## 心得
