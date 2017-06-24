# Arduino-wifi-
Home heating system connecting Arduino Wi-Fi shield  
My aim is to offer a simple, cheap, and effective way home heating automation system works

/*
  WiFi Web Server

 A simple web server that shows the value of the digital input pins.
 using a DHT11 sensor, WiFi shield and Arduino Uno.

 This example is written for a network using WPA encryption. For
 WEP or WPA, change the Wifi.begin() call accordingly.

 Circuit:
 * WiFi shield attached
 * Digital inputs attached to pins 2
Webpage: 
Home heating support system 

 Modifyed: 24 April, 2017 
 by DD. (Daniel)

 */

#include <SPI.h>
#include <WiFi.h>
#include <DHT.h>

char ssid[] = "xxxxxxxxxx";      // your network SSID (name)
char pass[] = "xxxxxxxxxx";   // your network password
int keyIndex = 0;                 // your network key Index number (needed only for WEP)

#define DHTPIN 2     // what digital pin we're connected to

// Uncomment whatever type you're using!
#define DHTTYPE DHT11   // DHT 11

DHT dht(DHTPIN, DHTTYPE);


int status = WL_IDLE_STATUS;

WiFiServer server(80);

int sensorValue; //New
float degree; //New

void setup() {
  //Initialize serial and wait for port to open:
  Serial.begin(9600);
  dht.begin();
  while (!Serial) {
    ; // wait for serial port to connect. Needed for native USB port only
  }

  // check for the presence of the shield:
  if (WiFi.status() == WL_NO_SHIELD) {
    Serial.println("WiFi shield not present");
    // don't continue:
    while (true);
  }

  String fv = WiFi.firmwareVersion();
  if (fv != "1.1.0") {
    Serial.println("Please upgrade the firmware");
  }

  // attempt to connect to Wifi network:
  while (status != WL_CONNECTED) {
    Serial.print("Attempting to connect to SSID: ");
    Serial.println(ssid);
    // Connect to WPA/WPA2 network. Change this line if using open or WEP network:
    status = WiFi.begin(ssid, pass);

    // wait 10 seconds for connection:
    delay(10000);
  }
  server.begin();
  // you're connected now, so print out the status:
  printWifiStatus();
}


void loop() {
  // listen for incoming clients
  WiFiClient client = server.available();
  if (client) {
    Serial.println("new client");
    // an http request ends with a blank line
    boolean currentLineIsBlank = true;
    while (client.connected()) {
      if (client.available()) {
        char c = client.read();
        Serial.write(c);
        // if you've gotten to the end of the line (received a newline
        // character) and the line is blank, the http request has ended,
        // so you can send a reply
        if (c == '\n' && currentLineIsBlank) {
          // send a standard http response header
          client.println("HTTP/1.1 200 OK");
          client.println("Content-Type: text/html");
          client.println();  // the connection will be closed after completion of the response
          client.println();  // refresh the page automatically every 5 sec
         client.println();
         client.println("<font Refresh: 5</form>");
          client.print("<font color='Red'>Network Studio 2017</font>");
          client.print("</form>");
          client.println("<font size='15'>Home-Heating-Support</font>");
          client.print("<c>");
          client.print("<c>");

          // the content of the HTTP response follows header:
            client.print("<form action='/Temp-ON' method='GET'>");
            client.print("<button type='submit'>Temp-ON</button>");
            client.print("</form>");
            client.print("<form action='/Temp-Off' method='GET'>");
            client.print("<button type='submit'>Temp-Off</button>");
            client.print("</form>");

            //To press button header for Home Heatting Support
            client.print("<font color='blue'>Themostart Controller</font>");
             client.print("</form>");
            client.print("Press AUTO for Home Heatting Support Automatically ");
            client.print("<form action='/AUTO' method='GET'>");
            client.print("<button type='submit'>AUTO</button>");
            client.print("</form>"); //website

            client.print("</form>"); //

            client.print("Room temperature is ! ");
            
            float hi = dht.readHumidity();
            float t = dht.readTemperature();
            float f = dht.readTemperature(true);
          
          //  if (isnan(h) || isnan(t) || isnan(f)) 
            {
            //  Serial.println("Failed to read from DHT sensor!");
         //     return;
            }
            float hif = dht.computeHeatIndex(f, hi);
            float hic = dht.computeHeatIndex(t, hi, false);
            
            client.print("Humidity: ");
             client.print(hi);
           client.print(" %\t");
            client.print("Temperature: ");
            client.print(t);
            client.print(" *C ");
             client.print(f);
             client.print(" *F\t");
            // client.print("Heat index: ");
            client.print(hic);
             client.print(" *C ");
             client.print(hif);
           client.println(" *Input Values");
            
           // client.print("Network Studio 2017");

            
            // HTML Document type
          client.println();
//          client.println("<!DOCTYPE HTML>");
//          client.println("<html>");
          // output the value of each analog input pin
          for (int digitalChannel = 2; digitalChannel < 6; digitalChannel++) {
            int sensorReading = digitalRead(digitalChannel);
            //New lines to caclulate 
            sensorValue = digitalRead(2); //To get Digital reading
            degree=sensorValue*(0.5/1023.0)/0.0009; //New
            t = ((t-85)/10) * ((87-t)/5); //temperature is between 80 and 87 degrees F, 
            hi = 0.5 * (t + 61.0 + ((t-68.0)*1.2) + (hi*0.094)); //temperature and humidity warrant a heat index value 
            //about 80 degrees F. In those cases, a simpler formula is applied to calculate values consistently.  
            client.print(degree); //New
            
            Serial.print("TempReader input ");
           Serial.print(digitalChannel);
            Serial.print(" is ");
           Serial.print(sensorReading);
            client.println("<c />");
            //client.println("<meta http-equiv=refresh content=1;URL='//192.168.1.10/AUTO'"); // Refresh main website
                        
          }
          //HTTP response ends with a blank line
          client.println();
          //break from while loop lines
          break;
        }
        if (c == '\n') {
          // you're starting a new line
          currentLineIsBlank = true;
        } else if (c != '\r') {
          // you've gotten a character on the current line
          currentLineIsBlank = false;
        }
      }
    }
    // give the web browser time to receive the data
    delay(1);

    // close the connection:
    client.stop();
    Serial.println("client disonnected");
  }
}

void dhtSensor(){
  // Wait a few seconds between measurements.
  delay(2000);

  // Reading temperature or humidity takes about 250 milliseconds!
  // Sensor readings may also be up to 2 seconds 'old' (its a very slow sensor)
  float h = dht.readHumidity();
  // Read temperature as Celsius (the default)
  float t = dht.readTemperature();
  // Read temperature as Fahrenheit (isFahrenheit = true)
  float f = dht.readTemperature(true);

  // Check if any reads failed and exit early (to try again).
  if (isnan(h) || isnan(t) || isnan(f)) 
  {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  // Compute heat index in Fahrenheit (the default)
  float hif = dht.computeHeatIndex(f, h);
  // Compute heat index in Celsius (isFahreheit = false)
  float hic = dht.computeHeatIndex(t, h, false);

  Serial.print("Humidity: ");
  Serial.print(h);
  Serial.print(" %\t");
  Serial.print("Temperature: ");
  Serial.print(t);
  Serial.print(" *C ");
  Serial.print(f);
  Serial.print(" *F\t");
  Serial.print("Heat index: ");
  Serial.print(hic);
  Serial.print(" *C ");
  Serial.print(hif);
  //Serial.println(" *F");

}


void printWifiStatus() {
  // print the SSID of the network you're attached to:
  Serial.print("SSID: ");
  Serial.println(WiFi.SSID());

  // print your WiFi shield's IP address:
  IPAddress ip = WiFi.localIP();
  Serial.print("IP Address: ");
  Serial.println(ip);

  // print the received signal strength:
  long rssi = WiFi.RSSI();
  Serial.print("signal strength (RSSI):");
  Serial.print(rssi);
  Serial.println(" dBm");
}
