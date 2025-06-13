---
layout: post
title: Emotional Music Player
author: [Richard Kuo]
category: [Lecture]
tags: [jekyll, ai]
---
## 系統功能介紹
```
透過鏡頭拍攝人物臉部表情，判斷人物情緒撥放不同歌曲
```
---
## 系統方塊圖
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E7%B3%BB%E7%B5%B1%E6%96%B9%E5%A1%8A%E5%9C%96.jpg?raw=true)

---
## Emotional Music Player

![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E6%83%85%E7%B7%92%E6%84%9F%E7%9F%A51.jpg?raw=true)

![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E6%83%85%E7%B7%92%E6%84%9F%E7%9F%A52.jpg?raw=true)

![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E6%83%85%E7%B7%92%E6%84%9F%E7%9F%A53.jpg?raw=true)

![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E6%83%85%E7%B7%92%E6%84%9F%E7%9F%A54.jpg?raw=true)

---
## 編碼設計流程
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/%E6%B5%81%E7%A8%8B%E5%9C%96.png?raw=true)

---
## 程式設計提示詞
![](https://github.com/peiyu525/MCU-project/blob/main/_posts/image.png?raw=true)
```

/*
    Emotion-to-MP3 Player with Gemini Vision
    功能：
    - 拍照分析情緒（喜怒哀樂）
    - 根據情緒播放對應 MP3 檔案（需存於 /mp3/ 資料夾中）
    - 顯示在 LCD 螢幕上
*/
String openAI_key = &quot;&quot;;  // 如果有用 openAI 可放 key，但此程式未用
String Gemini_key = &quot;AIzaSyAak7JerV3mz1e2h6XZmFSyKeJvw-5TALA&quot;;
String Llama_key = &quot;&quot;;
char wifi_ssid[] = &quot;Xiaomi 12X&quot;;
char wifi_pass[] = &quot;0902367789&quot;;
#include &lt;WiFi.h&gt;
#include &lt;WiFiUdp.h&gt;
#include &quot;GenAI.h&quot;
#include &quot;VideoStream.h&quot;
#include &quot;SPI.h&quot;
#include &quot;AmebaILI9341.h&quot;
#include &quot;TJpg_Decoder.h&quot;
#include &quot;AmebaFatFS.h&quot;
WiFiSSLClient client;
GenAI llm;
AmebaFatFS fs;
String mp3BasePath = &quot;/mp3/&quot;;
VideoSetting config(768, 768, CAM_FPS, VIDEO_JPEG, 1);
#define CHANNEL 0
uint32_t img_addr = 0;
uint32_t img_len = 0;
const int buttonPin = 1;
String prompt_msg = &quot;請判斷這張圖片代表的情緒，如果是喜輸出APT, 怒輸出
AstroBunny, 哀輸出gTTS, 樂輸出IBelieve&quot;;
#define TFT_RESET 5
#define TFT_DC    4
#define TFT_CS    SPI_SS
AmebaILI9341 tft = AmebaILI9341(TFT_CS, TFT_DC, TFT_RESET);
#define ILI9341_SPI_FREQUENCY 20000000

bool tft_output(int16_t x, int16_t y, uint16_t w, uint16_t h, uint16_t
*bitmap) {
    tft.drawBitmap(x, y, w, h, bitmap);
    return 1;
}
void initWiFi() {
    for (int i = 0; i &lt; 2; i++) {
        WiFi.begin(wifi_ssid, wifi_pass);
        delay(1000);
        Serial.print(&quot;Connecting to &quot;);
        Serial.println(wifi_ssid);
        uint32_t StartTime = millis();
        while (WiFi.status() != WL_CONNECTED) {
            delay(500);
            if ((StartTime + 5000) &lt; millis()) {
                break;
            }
        }
        if (WiFi.status() == WL_CONNECTED) {
            Serial.println(&quot;STAIP address: &quot;);
            Serial.println(WiFi.localIP());
            break;
        }
    }
}
void init_tft() {
    tft.begin();
    tft.setRotation(2);
    tft.clr();
    tft.setCursor(0, 0);
    tft.setForeground(ILI9341_GREEN);
    tft.setFontSize(2);
    tft.println(&quot;GenAIVision_TTS_LCD&quot;);
}
void setup() {
    Serial.begin(115200);
    SPI.setDefaultFrequency(ILI9341_SPI_FREQUENCY);
    initWiFi();
    config.setRotation(0);
    Camera.configVideoChannel(CHANNEL, config);
    Camera.videoInit();

    Camera.channelBegin(CHANNEL);
    Camera.printInfo();
    pinMode(buttonPin, INPUT);
    pinMode(LED_B, OUTPUT);
    init_tft();
    TJpgDec.setJpgScale(2);
    TJpgDec.setCallback(tft_output);
}
void loop() {
    tft.setCursor(0, 1);
    tft.println(&quot;press button to capture image&quot;);
    if ((digitalRead(buttonPin)) == 1) {
        tft.println(&quot;Capture Image&quot;);
        for (int count = 0; count &lt; 3; count++) {
            digitalWrite(LED_B, HIGH);
            delay(500);
            digitalWrite(LED_B, LOW);
            delay(500);
        }
        // 拍照
        Camera.getImage(0, &amp;img_addr, &amp;img_len);
        // 顯示照片
        TJpgDec.getJpgSize(0, 0, (uint8_t *)img_addr, img_len);
        TJpgDec.drawJpg(0, 0, (uint8_t *)img_addr, img_len);
        // 圖像辨識
        String text = llm.geminivision(Gemini_key, &quot;gemini-2.0-flash&quot;,
prompt_msg, img_addr, img_len, client);
        Serial.print(&quot;Gemini 回傳：&quot;);
        Serial.println(text);
        // 顯示結果
        tft.clr();
        tft.setCursor(0, 0);
        tft.setForeground(ILI9341_CYAN);
        tft.println(&quot;Playing Music:&quot;);
        tft.setForeground(ILI9341_WHITE);
        tft.println(text);
        // 撥放對應音樂

        String musicFile = mp3BasePath + text + &quot;.mp3&quot;;
        sdPlayMP3(musicFile);
    }
}
void sdPlayMP3(String filename) {
    fs.begin();
    String filepath = String(fs.getRootPath()) + filename;
    if (!fs.exists(filename)) {
        Serial.println(&quot;MP3 file not found: &quot; + filepath);
        tft.setForeground(ILI9341_RED);
        tft.println(&quot;MP3 not found!&quot;);
        fs.end();
        return;
    }
    File file = fs.open(filepath, MP3);
    file.setMp3DigitalVol(175);
    file.playMp3();
    file.close();
    fs.end();
}

```
---

## 心得
```
這次的實驗在讀取SD卡中的音樂就嘗試了很多次，透過轉換了各種AI嘗試不同的程式碼，最後才成功，只是最後拍攝出來發現情緒會顯示亂碼，但歌曲是正確的
```
