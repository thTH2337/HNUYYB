#include <LiquidCrystal_I2C.h>
#define but 3
#define tri 13
#define ech 12
#define buz 11
#define red 10
#define yel 9
#define gre 8
LiquidCrystal_I2C lcd(0x27,16,2);
void setup() {
  Serial.begin(9600);
  lcd.init();
  pinMode(but, INPUT_PULLUP);
  pinMode(tri, OUTPUT);
  pinMode(ech, INPUT);
  pinMode(buz, OUTPUT);
  pinMode(red, OUTPUT);
  pinMode(yel, OUTPUT);
  pinMode(gre, OUTPUT);
  lcd.print("AgPt Hacking...");
  delay(1000);
  lcd.clear();
}
int mode;
bool bep=true;
bool ibep=true;
String sta;
unsigned long llit=0;
unsigned long lbep=0;
unsigned long ltim=0;
unsigned long d;
bool lit=true;
void loop() {
  // put your main code here, to run repeatedly:
  digitalWrite(red, LOW);
  digitalWrite(yel, LOW);
  digitalWrite(gre,LOW);
  digitalWrite(buz, LOW);
  unsigned long nowmil=millis();
  unsigned long time;
  digitalWrite(tri, LOW);
  delayMicroseconds(2);
  digitalWrite(tri, HIGH);
  delayMicroseconds(10);
  digitalWrite(tri, LOW);
  if(nowmil-ltim>500)
  {
    ltim=nowmil;
    time=pulseIn(ech,HIGH,30000);
  }
  d=time/58;
  if(d>100)
  {
    mode=1;
    sta="SAFE";
  }
  else if(d>=50)
  {
    mode=2;
    sta="CAUTION";
  }
  else if(d>=20)
  {
    mode=3;
    sta="WARNING";
  }
  else
  {
    mode=4;
    sta="STOP";
  }
  //led
  if(mode==1) 
  {
    digitalWrite(gre, HIGH);
    lit=true;
  }
  else if(mode==2) 
  {
    digitalWrite(yel, HIGH);
    lit=true;
  }
  else if(mode==3&&nowmil-llit>=250)
  {
    lit=!lit;
    llit=nowmil;
  }
  else if(mode==4)
  {
    unsigned long wait=map(d,0,20,20,250);
    if(nowmil-llit>=wait)
    {
      lit=!lit;
      llit=nowmil;
    }
  }
  if(mode==3||mode==4)
  {
    digitalWrite(red,lit);
  }
  //but
  if(digitalRead(but)==LOW&&(mode==3||mode==4))
  {
    bep=!bep;
    delay(200);
  }
  //buz
  if(bep==true)
  {
    if(mode==1||mode==2) ibep=true;
    else if(mode==3)
    {
      if(nowmil-lbep>=500)
      {
        lbep=nowmil;
        ibep=!ibep;
      }
    }
    else if(mode==4)
    {
      if(nowmil-lbep>=125)
      {
        lbep=nowmil;
        ibep=!ibep;
      }
    }
    if(mode==3||mode==4)
    {
      digitalWrite(buz,ibep);
    }
  }
  //lcd
  lcd.setCursor(0,0);
  lcd.print(d);
  lcd.print(" cm        ");
  lcd.setCursor(0,1);
  lcd.print(sta);
  lcd.print(" ");
  if(bep==true) lcd.print("ALARM     ");
  else lcd.print("MUTE     ");
}
