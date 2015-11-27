// Nokia 5110 LCD-Display (84x48 Bildpunkte)

#include <idDHT11.h>
#include <Adafruit_GFX.h>
#include <Adafruit_PCD8544.h>
#include <math.h>

int idDHT11pin = 2; //Digital pin for comunications
int idDHT11intNumber = 0; //interrupt number (must be the one that use the previus defined pin (see table above)

//declaration
void dht11_wrapper(); // must be declared before the lib initialization

// Lib instantiate
idDHT11 DHT11(idDHT11pin, idDHT11intNumber, dht11_wrapper);

// D7 - Serial clock out (CLK oder SCLK)
// D6 - Serial data out (DIN)
// D5 - Data/Command select (DC oder D/C)
// D4 - LCD chip select (CE oder CS)
// D3 - LCD reset (RST)
Adafruit_PCD8544 display = Adafruit_PCD8544(7, 6, 5, 4, 3);

unsigned int telemetry_counter = 0;
unsigned int timer = 590;

double Thermistor(int RawADC) {
  double Temp;
  //Temp = log(10000.0*((1024.0/RawADC-1)));
  Temp    = log(10000.0 / (1024.0 / RawADC - 1)); // for pull-up configuration
  Temp = 1 / (0.001129148 + (0.000234125 + (0.0000000876741 * Temp * Temp )) * Temp );
  Temp = Temp - 273.15;            // Convert Kelvin to Celcius
  //Temp = (Temp * 9.0)/ 5.0 + 32.0; // Convert Celcius to Fahrenheit
  return Temp;
}

void setup()   {

  Serial.begin(9600);      // open the serial port at 9600 bps:

  // Display initialisieren
  display.begin();

  // Kontrast setzen
  display.setContrast(60);
  display.clearDisplay();   // clears the screen and buffer
}

// This wrapper is in charge of calling
// mus be defined like this for the lib work
void dht11_wrapper() {
  DHT11.isrCallback();
}

void loop() {

  display.clearDisplay();      // Display wieder löschen
  int i = 1, m = 0;
  char buffer[16];
  float stemp, temp, hum, dew;

  //nTNC Initial
  Serial.println("CONFIG=ON");
  delay(100);
  Serial.println("MYCALL=HS5TQA-5");
  delay(100);
  Serial.println("TNC=ON");
  delay(100);
  Serial.println("DIGI=OFF");
  delay(100);
  Serial.println("TRACKER=OFF");
  delay(100);
  Serial.println("KISS=OFF");
  delay(100);
  Serial.println("PATH=WIDE1-1");
  delay(100);
  Serial.println("LOG=OFF");
  delay(100);
  Serial.println("RTS=OFF");
  delay(100);
  Serial.println("FREQ=144.3900");
  delay(100);
  Serial.println("SQL=1");
  delay(100);
  Serial.println("RFPO=0");
  delay(100);
  Serial.println("BTEXT==1343.66N\\10026.11E<");
  delay(100);
  Serial.println("BEACON=30");
  delay(100);
  Serial.println("CONFIG=OFF");
  delay(100);
  Serial.println("RESTART");
  delay(2000);

  //Send Telemetry PARM
  Serial.println(":HS5TQA-5 :PARM.Moisture,Soil Temp.,Temperature,Humidity,DewPoint,I1,I2,I3,I4,I5,I6,I7,I8");
  delay(2000);
  //Send Telemetry UNIT
  Serial.println(":HS5TQA-5 :UNIT.%,C,C,%RH,C,I1,I2,I3,I4,I5,I6,I7,I8");
  delay(2000);
  //Send Telemetry EQNS
  Serial.println(":HS5TQA-5 :EQNS.0,1,0,0,1,0,0,1,0,0,1,0,0,1,0");
  delay(2000);
  //Send Telemetry BITS
  Serial.println(":HS5TQA-5 :BITS.11111101,GPIO");
  delay(2000);
  while (1) {
    display.setTextSize(1);
    display.setTextColor(WHITE, BLACK); // 'inverted' text
    display.setCursor(0, 0);
    display.print("  APRS LOGER  ");
    display.setTextColor(BLACK);
    display.setTextSize(1);
    display.setCursor(0, 8);
    sprintf(buffer, "TC:%d  TM:%d", telemetry_counter, timer);
    display.print(buffer);

    m = i / 10;
    if (m > 100) m = 100;
    display.setCursor(0, 16);
    display.print(temp, 1);
    display.print("C");
    display.setCursor(45, 16);
    display.print(hum, 0);
    display.print("%RH");

    display.setCursor(0, 24);
    display.print("Dew: ");
    display.print(dew, 1);
    display.print("C");

    display.setCursor(0, 32);
    display.print("S.Temp: ");
    display.print(stemp, 1);
    display.print("C");

    display.setCursor(0, 40);
    display.print("Moisture: ");
    display.print(m);
    display.print("%");

    display.display();
    delay(100);
    i = analogRead(0);
    stemp = Thermistor(analogRead(1));
    display.clearDisplay();

    if ((timer % 50) == 0) {
      int result = DHT11.acquireAndWait();
      if (result == IDDHTLIB_OK) {
        temp = DHT11.getCelsius();
        hum = DHT11.getHumidity();
        dew = DHT11.getDewPointSlow();
      }
    }

    //Timer 60Sec
    if (timer++ > 600) {
      timer = 0;
      //Send Telemetry Data
      Serial.print("T#");
      if (telemetry_counter < 10) {
        Serial.print("00");
      } else if (telemetry_counter < 99) {
        Serial.print("0");
      }
      Serial.print(telemetry_counter++);
      Serial.print(",");
      Serial.print(m);
      Serial.print(",");
      Serial.print(stemp, 1);
      Serial.print(",");
      Serial.print(temp, 1);
      Serial.print(",");
      Serial.print(hum, 0);
      Serial.print(",");
      Serial.print(dew, 1);
      Serial.print(",");
      Serial.println("00000000"); //Fix data for test
      if (telemetry_counter > 999) telemetry_counter = 0;
    }
  }
  display.clearDisplay();      // Display wieder löschen
}

void set_text(int x, int y, String text, int color) {
  display.setTextColor(color); // Textfarbe setzen, also Schwarz oder Weiss
  display.setCursor(x, y);     // Startpunkt-Position des Textes
  display.println(text);       // Textzeile ausgeben
  display.display();           // Display aktualisieren
}
