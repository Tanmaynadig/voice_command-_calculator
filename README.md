# voice_command-_calculator
#voice command calculator using esp32 wifi module and ic2 dispaly
#include <WiFi.h>
#include <Wire.h>
#include <WebServer.h>
#include <LiquidCrystal_I2C.h>

const char* ssid = "Vivo5G2";
const char* password = "Thanu@123";

WebServer server(80);

LiquidCrystal_I2C lcd(0x27, 16, 2); // Set the I2C address and LCD dimensions (16x2)

void setup() {
  Serial.begin(115200);

  // Initialize the LCD
  lcd.init();
  lcd.backlight();

  // Connect to WiFi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");
  Serial.println("IP address:");
  Serial.println(WiFi.localIP());


  // Set up routes
  server.on("/", HTTP_GET, handleRoot);
  server.on("/calculate", HTTP_POST, handleCalculate);
  server.begin();
  Serial.println("HTTP server started");
}

void loop() {
  server.handleClient();
}

void handleRoot() {
  // Send HTTP response with the calculator webpage
  String webpage = "<!DOCTYPE html><html><head><title>Voice Command Calculator</title></head><body>";
  webpage += "<h1>Voice Command Calculator</h1>";
  webpage += "<form action='/calculate' method='POST'>";
  webpage += "Voice Command: <input type='text' name='command'><br>";
  webpage += "<input type='submit' value='Calculate'>";
  webpage += "</form>";
  webpage += "</body></html>";

  server.send(200, "text/html", webpage);
}

void handleCalculate() {
  String command = server.arg("command");
  Serial.print("Command received: ");
  Serial.println(command);

  float result = calculate(command);
  float num1, num2;
  // Display result on the LCD
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print(command);
   lcd.setCursor(0, 1);
  lcd.print("Result:");
  if ((num1/num2) == 0)
  {
    lcd.setCursor(8, 1);
  lcd.print("error");  
  }
   else
   {
  lcd.setCursor(8, 1);
  lcd.print(result);
   }
  // Formulate the response
  String response = "Result: " + String(result);

  // Append the result to the original webpage
  String webpage = "<!DOCTYPE html><html><head><title>Voice Command Calculator</title></head><body>";
  webpage += "<h1>Voice Command Calculator</h1>";
  webpage += "<form action='/calculate' method='POST'>";
  webpage += "Voice Command: <input type='text' name='command' value='" + command + "'><br>";
  webpage += "<input type='submit' value='Calculate'>";
  webpage += "</form>";
  webpage += "<p>" + response + "</p>";
  webpage += "</body></html>";

  server.send(200, "text/html", webpage);
}

float calculate(String command) {
  // Parse the command and perform calculations
  // For simplicity, let's assume the command is in the format "number1 operator number2"
  // For example: "5 plus 3", "10 minus 2", "4 multiply 7", "20 divide 5"
  
   float result = calculate(command);
  float num1, num2;
  char op[10]; // Operator can be up to 10 characters long
  float result = calculate(command);

  sscanf(command.c_str(), "%f %s %f", &num1, op, &num2);
    
  if (strcmp(op, "+") == 0) {
    return num1+num2;
     lcd.setCursor(8, 1);
  lcd.print(result);
  } else if (strcmp(op, "-") == 0) {
    return num1-num2;
     lcd.setCursor(8, 1);
  lcd.print(result);
  } else if (strcmp(op, "*") == 0) {
    return num1*num2;
     lcd.setCursor(8, 1);
  lcd.print(result);
  } else if (strcmp(op, "/") == 0) {
    if (num2 != 0) {
      return num1/num2;
       lcd.setCursor(8, 1);
  lcd.print(result);
    } else {
      Serial.println("Error: Division by zero");
       lcd.setCursor(8, 1);
  lcd.print("error");
      
      // You might want to handle this error case differently based on your application's requirements
      return 0;
    }
  } else {
    Serial.println("Error: Invalid operator");
    // You might want to handle this error case differently based on your application's requirements
    return 0;
  
  }
}
