---
layout: post
title: Emotional Music Player
author: [Richard Kuo]
category: [Lecture]
tags: [jekyll, ai]
---



---
## Emotional Music Player

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
#include "AmebaFatFS.h"
#include <YourGeminiVisionLibrary.h>  // Placeholder for Gemini Vision library

AmebaFatFS fs;
String filename;

void setup() {
    // Initialize Serial and SD card
    Serial.begin(115200);
    fs.begin();

    // Capture image and analyze emotion using Gemini Vision
    String emotion = analyzeEmotionFromImage();

    // Map the emotion to a corresponding song
    if (emotion == "happy") {
        filename = "mp3/APT.mp3";  // 喜 - APT.mp3
    } else if (emotion == "joyful") {
        filename = "mp3/BirdsOfAFeather.mp3";  // 樂 - BirdsOfAFeather.mp3
    } else if (emotion == "angry" || emotion == "sad") {
        filename = "mp3/ThePowerOfGoodBye.mp3";  // 怒和哀 - ThePowerOfGoodBye.mp3
    } else {
        // Default song for unknown emotion or neutral
        filename = "mp3/APT.mp3";
    }

    // Play the MP3 file based on emotion
    playMP3File(filename);
}

String analyzeEmotionFromImage() {
    // Placeholder for actual image capture and emotion analysis logic
    // Example: Capture an image, send to Gemini Vision, and analyze the emotion
    String emotion = "happy";  // Dummy response for now
    return emotion;
}

void playMP3File(String filename) {
    File file = fs.open(String(fs.getRootPath()) + filename, MP3);
    if (file) {
        // Start playing MP3 file
        Serial.println("Playing: " + filename);
        // MP3 playback code here
        // e.g., use a library function to play the file through the audio module
    } else {
        Serial.println("Error opening file: " + filename);
    }
}

void loop() {
    // Placeholder for any repeated tasks (if needed)
}


  
<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*

