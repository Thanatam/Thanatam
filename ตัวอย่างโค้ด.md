
#include <LiquidCrystal_I2C.h>//เลือกใช้plug in จาก folder library ใน folder program Arduino โดยต้องโหลดLiquidCrystal_I2C ลงใน library ก่อน
LiquidCrystal_I2C lcd(0x27, 16, 2);//set ค่าตัวอักษรโดยบางจอled ใช้ 0x3F หรือ 0x27

int pin2 = 6;// ตั้งค่าการนำเข้าออกข้อมูล ใช้คำสั่งนี้ก็ต่อเมื่อมีการเชื่อมต่อผ่านช่อง digital(D0,D1,D2,D3,D4,D5,D6, ...เป็นต้น) บนบอร์ด Arduino 
int pin1 = 4;
int relay = 2;
unsigned long duration1;//รับค่า ตัวเลขจำนวนเต็มที่มีความยาว 4 ไบต์ ใส่ค่าได้ตั้งแต่ 0 ถึง 4,294,967,295 (2^32 - 1)
unsigned long duration2;

unsigned long starttime;
unsigned long sampletime_ms = 3000;//sampe 1s ;
unsigned long lowpulseoccupancy1 = 0;
unsigned long lowpulseoccupancy2 = 0;
float ratio1 = 0;// รับตัวเลขที่มีทศนิยม
float ratio2 = 0;
float concentration1 = 0;
float concentration2 = 0;
float AQI = 0;
float AQI2 = 0;

void setup() {
  Serial.begin(9600);
  pinMode(4,INPUT);
  pinMode(6,INPUT);
  pinMode(2,OUTPUT);
  starttime = millis();//get the current time;
  lcd.begin();
}

void loop() {
  duration1 = pulseIn(pin1, LOW);
  duration2 = pulseIn(pin2, LOW);
  lowpulseoccupancy1 = lowpulseoccupancy1+duration1;
  lowpulseoccupancy2 = lowpulseoccupancy2+duration2;


  if ((millis()-starttime) > sampletime_ms)//if the sampel time == 30s
  {
    ratio1 = lowpulseoccupancy1/(sampletime_ms*10.0);  // Integer percentage 0=>100
    concentration1 = 1.1*pow(ratio1,3)-3.8*pow(ratio1,2)+520*ratio1+0.62; // using spec sheet curve
    AQI = concentration1/(10*10*10);
    
    ratio2 = lowpulseoccupancy2/(sampletime_ms*10.0);  // Integer percentage 0=>100
    concentration2 = 1.1*pow(ratio2,3)-3.8*pow(ratio2,2)+520*ratio2+0.62; // 
    AQI2 = concentration2/(10*10*10);
    
    lcd.setCursor(0, 0);
    lcd.print("PM10 "); // แสดงข้อความ pm 10
    lcd.setCursor(6, 0);
    lcd.print(AQI,5); // แสดงปริมาณ pm 10
    lcd.setCursor(0, 1);
    lcd.print("PM2.5 "); // แสดงข้อความ pm 2.5
    lcd.setCursor(6, 1);
    lcd.print(AQI2,5); // แสดงปริมาณ pm 2.5
    
    Serial.print("AQI = ");// แสดงข้อความ pm 10 คำสั่งที่มี Serial. นำหน้าข้อมูลนั้นจะถูกแสดงผลบน Serialmornitor มุมบนขวาใน program Arduino
    Serial.print(AQI);// แสดงปริมาณ pm 10
    Serial.print(" (mg/1000)/m*m*m  -  "); //แสดงหน่วย

    Serial.print("AQI2 = ");// แสดงข้อความ pm 2.5
    Serial.print(AQI2);// แสดงปริมาณ pm 2.5
    Serial.print(" (mg/1000)/m*m*m  -  ");//แสดงหน่วย

    
    if (concentration1 < 1000) {
       
     digitalWrite(relay, LOW);
    
  }
    if (concentration1 > 1000 && concentration1 < 10000) {
     
     digitalWrite(relay, HIGH);
     
    }
    
    if (concentration1 > 10000 && concentration1 < 20000) {
 
    digitalWrite(relay, HIGH);
    }
      if (concentration1 > 20000 && concentration1 < 50000) {
    
     
     digitalWrite(relay, HIGH);
  }

    if (concentration1 > 50000 ) {

      
     
     digitalWrite(relay, HIGH);
    } 
      
    lowpulseoccupancy1 = 0;
    lowpulseoccupancy2 = 0;
    starttime = millis();
  }
}

