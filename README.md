# ProjectFarm_switch-timer

# Code
```C++
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <IRremote.h>
LiquidCrystal_I2C lcd(0x27, 16, 2);
int RECV_PIN = 11;

IRrecv irrecv(RECV_PIN);

decode_results results;
unsigned long timer;
unsigned int i = 0;
unsigned int r;
unsigned int a;
unsigned int b;
bool n = false;
bool m = false;
void setup() {
  // put your setup code here, to run once:
  digitalWrite(5 , 1);
  pinMode(10, INPUT);
  pinMode(3, INPUT);
  pinMode(7, INPUT);
  pinMode(5, OUTPUT);
  lcd.begin();
  lcd.backlight();
  lcd.home();
  Serial.begin(9600);
  irrecv.enableIRIn(); // Start the receiver
}

void loop() {
  // put your main code here, to run repeatedly:
  digitalWrite(5 , 1);

  while (m == false) {
    lcd.setCursor(0, 0);
    lcd.print("00:00");
      a = 0,b = 0;
    Serial.println(digitalRead(10));
    if (digitalRead(10) == 1 || digitalRead(3) == 1) {

      m == true;
      goto start;
    }
  }
  if (m == true) {

    goto start2;
  }
///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
start2:
  while (true) {
    digitalWrite(5 , 1);
    timer = millis();
    while (millis() - timer < 500 && n == false) {//เช็ค n ว่าการกระพริบเลขล่าสุดเป็นการซ่อนเลข
      lcd.setCursor(2, 0);
      lcd.print(":");
      lcd.setCursor(i, 0);
      lcd.print(r);//แสดงเลข0.5วินาที เนื่องจาก n เป็นการซ่อนเลข
      //Serial.println(digitalRead(3));
      if ((digitalRead(10) == 1 || digitalRead(3) == 1) && millis() - timer > 200) {//เช็คเวลากดปุ่มว่าไม่ใช่การกดใน start ป้องกันการทำงานต่อเนื่องจาก start
        lcd.setCursor(i, 0);
        if (i == 0) {
          lcd.print(a);
        }
        else if (i == 3) {
          lcd.print(b);
        }
        n = true;
        goto start;
      }
      else  if (digitalRead(7) == 1) {
        goto start3;
      }
    }
    n = true;
    timer = millis();
    while (millis() - timer < 500 && n == true) {//เช็ค n ว่าการกระพริบเลขล่าสุดเป็นการแสดงเลข
      lcd.setCursor(2, 0);
      lcd.print(":");
      lcd.setCursor(i, 0);
      lcd.print("  ");//ซ่อนเลข0.5วินาที เนื่องจาก n เป็นการแสดงเลข
      //Serial.println(digitalRead(3));
      if ((digitalRead(10) == 1 || digitalRead(3) == 1) && millis() - timer > 200) {
        lcd.setCursor(i, 0);
        if (i == 0) {
          lcd.print(a);
        }
        else if (i == 3) {
          lcd.print(b);
        }

        n = false;//ให้การกระพริบถัดไปเป็นการแสดงเลข
        goto start;
      }
      else  if (digitalRead(7) == 1) {
        goto start3;
      }
    }
    n = false;
  }

///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
start :
  digitalWrite(5 , 1);
  if (digitalRead(10) == 1  && i == 0) {
    Serial.println(i);
    while (digitalRead(10) == 1) {
      //////wait รอให้ปล่อยปุ่มจากการกดใน start2 ป้องกันการทำงานต่อเนื่องใน start2
    }
    i = 0;
    a++;
    r = a;
    goto start2;
  }
  else if (digitalRead(10) == 1  && i == 3) {

    while (digitalRead(10) == 1) {
      //////wait
    }
    i = 0;
    r = a;
    goto start2;
  }
  else if (digitalRead(3) == 1 && i == 3) {
    while (digitalRead(3) == 1) {
    //////wait
    }
    if (b == 59) {
      b = 0;
    }
    else {
      i = 3;
      b++;
    }
    r = b;
    goto start2;
  }
  else if (digitalRead(3) == 1  && i == 0) {
    Serial.println(i);
    while (digitalRead(3) == 1) {
      //////wait
    }
    i = 3;
    r = b;
    goto start2;
  }

///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
start3 :
  while (a > 0 || b > 0) {
    while (b > 0) {
      digitalWrite(5 , 0);
      lcd.setCursor(3, 0);
      lcd.print("  ");
      lcd.setCursor(3, 0);
      lcd.print(b);
      b = b - 1;
      delay(1000);
    }
    if(a == 0 && b == 0){
      break;
    }
      lcd.setCursor(3, 0);
      lcd.print("  ");
      lcd.setCursor(3, 0);
      lcd.print(b);
      delay(1000);
          b = 59;
    a = a - 1;
    lcd.setCursor(0, 0);
    lcd.print("  ");
    lcd.setCursor(0, 0);
    lcd.print(a);
  }
 
}
```
<br>


https://user-images.githubusercontent.com/61747927/147899965-a3e4733f-2676-4344-ba45-a44849711292.mp4

```
ปุ่มการทำงานมี 3 ปุ่ม คือ ปุ่ม นาที(ปุ่มซ้าย) ปุ่มวินาที(ปุ่มกลาง) ปุ่มเริ่ม(ปุ่มขวา)
โดยการทำงานคือ เมื่อกดปุมนาทีระยะหน่วยนาทีจะเพิ่มขึ้น ปุ่มวินาทีก็เช่นกัน เมื่อระยะเวลาในหน่วยนาทีหรือวินาทีมากกว่า 60 จะเปลี่ยนมาเป็น 0 เหมือนเดิม
และเมื่อกดปุ่มเริ่ม สวิตส์จะทำการเปลี่ยนสถานะจนกว่าจะครบเวลาตามที่ตั้งไว้และกลับคืนสถานะเดิม(ในรูปใช้ LED ในการจำลองให้เห็นภาพ)
```


goto simulation https://www.tinkercad.com/things/b91UUV6lfqz-projectfarm

![Circuits](https://github.com/boss2546th/ProjectFarm_switch-timer/blob/main/projectfram/ProjectFarm.png)



ตัวอย่างการใช้งานจริง

https://user-images.githubusercontent.com/61747927/147900666-67e3b142-b92b-465b-a34d-12b1f66c5127.mp4


