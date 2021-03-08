``` c linenums="1" hl_lines="107"
#include <Arduino.h>

#include <ESP8266WiFi.h>
#include <WebSocketsServer.h>
#include <ESP8266WebServer.h>
#include <ESP8266mDNS.h>
#include <Hash.h>
#include <Servo.h>   // Include the library


Servo servo1;        // Create the Servo and name it "servo1"
Servo servo2;        // Create the Servo and name it "servo2"
Servo servo3;        // Create the Servo and name it "servo3"
Servo servo4;        // Create the Servo and name it "servo4"

#define SERVO1    D0
#define SERVO2    D2
#define SERVO3    D3
#define SERVO4    D4

#define USE_SERIAL Serial

ESP8266WebServer server(80);
WebSocketsServer webSocket = WebSocketsServer(81);

String website = "<html><head><script>"
        "var connection = new WebSocket('ws://' + location.hostname + ':81/', ['arduino']);"
        "connection.onopen = function(){ connection.send('Connected on ' + new Date()); };"
        "connection.onerror = function (error){ console.log('WebSocket Error ', error); };"
        "connection.onmessage = function(e){console.log('Server: ', e.data); };"      
        "function sendValue(){"
        "var value = parseInt(document.getElementById('r').value).toString(16);"
        "if (value.length < 2){value = '0' + value;}"
            "color = document.getElementById('color').value;"
            "var command = '$' + color + value;"
            "console.log('Command: ' + command);"
            "connection.send(command);}</script></head>"
        "<body>LED Control:<br/><select id=\"color\">"
        "<option value=\"R\">Red</option><option value=\"G\">Green</option><option value=\"B\">Blue</option><option value=\"W\">White</option>"
        "</select><br/><br/>Intensity: <br/> <input id=\"r\" type=\"range\" min=\"0\" max=\"255\" step=\"1\" oninput=\"sendValue();\"/><br/></body></html>";

void webSocketEvent(uint8_t num, WStype_t type, uint8_t * payload, size_t length) {

  switch (type) {
    case WStype_DISCONNECTED:
      USE_SERIAL.printf("[%u] Disconnected!\n", num);
      break;
    case WStype_CONNECTED: {
        IPAddress ip = webSocket.remoteIP(num);
        USE_SERIAL.printf("[%u] Connected from %d.%d.%d.%d url: %s\n", num, ip[0], ip[1], ip[2], ip[3], payload);

        // send message to client
        webSocket.sendTXT(num, "Connected");
      }
      break;
    case WStype_TEXT:
      USE_SERIAL.printf("[%u] got Text: %s\n", num, payload);

      if (payload[0] == '$') {
        // decode rgb data
        uint8_t value = (uint8_t) strtol((const char *) &payload[2], NULL, 16);
        
        USE_SERIAL.println(value);
        
        if(payload[1] == 'R'){
          servo1.write(value);
        }else if(payload[1] == 'G'){
          servo2.write(value);
        }else if(payload[1] == 'B'){
          servo3.write(value);
        }else if(payload[1] == 'W'){
          servo4.write(value);
        }
      }

      break;
  }

}

void setup() {

  USE_SERIAL.begin(115200);

  //USE_SERIAL.setDebugOutput(true);

  USE_SERIAL.println();
  USE_SERIAL.println();
  USE_SERIAL.println();

  for (uint8_t t = 4; t > 0; t--) {
    USE_SERIAL.printf("[SETUP] BOOT WAIT %d...\n", t);
    USE_SERIAL.flush();
    delay(300);
  }

  servo1.attach(SERVO1);
  servo2.attach(SERVO2);
  servo3.attach(SERVO3);
  servo4.attach(SERVO4);

  servo1.write(90); 
  servo2.write(90); 
  servo3.write(90); 
  servo4.write(90); 

  WiFi.softAP("padra1za1");

  IPAddress myIP = WiFi.softAPIP();
  Serial.print("AP IP address: ");
  Serial.println(myIP);

  // start webSocket server
  webSocket.begin();
  webSocket.onEvent(webSocketEvent);

  if (MDNS.begin("esp8266")) {
    USE_SERIAL.println("MDNS responder started");
  }

  // handle index
  server.on("/", []() {
    // send index.html
    server.send(200, "text/html", website);
  });

  server.begin();

  // Add service to MDNS
  MDNS.addService("http", "tcp", 80);
  MDNS.addService("ws", "tcp", 81);

}

void loop() {
  webSocket.loop();
  server.handleClient();
}
```
