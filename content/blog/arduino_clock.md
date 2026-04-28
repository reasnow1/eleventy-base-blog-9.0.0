---
title: Arduino手搓时钟 Arduino clock
description: This is a post on My Blog about agile frameworks.
date: 2026-04-08
tags: Arduino
---

![arduino_clock1]({% staticBase %}{% endstaticBase %}/arduino/arduino_clock1.jpg)

## 所需材料
- Arduino UNO
- 面包板
- 杜邦线若干
- 1KΩ 电阻 × 8
- 4位数码管（共阴极）

## 接线图
![arduino_clock2]({% staticBase %}{% endstaticBase %}/arduino/arduino_clock2.jpg)
![arduino_clock3]({% staticBase %}{% endstaticBase %}/arduino/arduino_clock3.jpg)

### 接线表（Arduino → 数码管）
| Arduino 引脚 | 电阻 | 数码管引脚 |
|--------------|------|-------------|
| 2            | 1KΩ  | 5           |
| 3            | 1KΩ  | 6           |
| 4            | 1KΩ  | 9           |
| 5            | 1KΩ  | 8           |
| 6            | 1KΩ  | 12          |
| 7            | 1KΩ  | 3           |
| 8            | 1KΩ  | 2           |
| 9            | 无   | 1           |
| 10           | 无   | 11          |
| 11           | 无   | 10          |
| 12           | 无   | 7           |
| 13           | 1KΩ  | 4           |

## 代码
```cpp
// 段码表（共阴极，高电平点亮，顺序：A B C D E F G 对应 D2~D8）
byte segCode[10] = {
  0b00111111, // 0
  0b00000110, // 1
  0b01011011, // 2
  0b01001111, // 3
  0b01100110, // 4
  0b01101101, // 5
  0b01111101, // 6
  0b00000111, // 7
  0b01111111, // 8
  0b01101111  // 9
};

// 引脚定义
int segPins[] = {2, 3, 4, 5, 6, 7, 8};   // A,B,C,D,E,F,G
int digitPins[] = {9, 10, 11, 12};       // 小时十位,小时个位,分钟十位,分钟个位
int colonPin = 13;                       // 冒号控制

int hour = 0;     // 此处输入当前时间
int minute = 0;   // 此处输入当前时间 
int second = 0;   // 内部秒计数
unsigned long lastSecond = 0;
unsigned long lastBlink = 0;
bool colonOn = true;

void setup() {
  for (int i = 0; i < 7; i++) pinMode(segPins[i], OUTPUT);
  for (int i = 0; i < 4; i++) pinMode(digitPins[i], OUTPUT);
  pinMode(colonPin, OUTPUT);
  digitalWrite(colonPin, HIGH);
}

void loop() {
  // 每1秒更新一次时间
  if (millis() - lastSecond >= 1000) {
    lastSecond += 1000;  // ← critical change (was: lastSecond = millis();)
    second++;
    if (second >= 60) {
      second = 0;
      minute++;
      if (minute >= 60) {
        minute = 0;
        hour++;
        if (hour >= 24) hour = 0;
      }
    }
  }

  if (millis() - lastBlink >= 500) {
    lastBlink = millis();
    colonOn = !colonOn;
    digitalWrite(colonPin, colonOn ? HIGH : LOW);
  }

  displayDigits(hour / 10, hour % 10, minute / 10, minute % 10);
}

void displayDigits(int d0, int d1, int d2, int d3) {
  int digits[4] = {d0, d1, d2, d3};
  for (int i = 0; i < 4; i++) {
    for (int j = 0; j < 4; j++) digitalWrite(digitPins[j], HIGH);
    byte code = segCode[digits[i]];
    for (int k = 0; k < 7; k++) {
      digitalWrite(segPins[k], bitRead(code, k));
    }
    digitalWrite(digitPins[i], LOW);
    delay(5);
  }
}
```

![arduino_clock4]({% staticBase %}{% endstaticBase %}/arduino/arduino_clock4.png)

## 常见问题：Debian 上传失败
如果因为权限不足无法上传：
```bash
sudo usermod -a -G dialout <username>
```
将 `<username>` 替换为当前用户名，**重启后生效**。