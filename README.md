# terranvalot

/*
terran valaistusta 
*/

#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <FastLED.h>

//led alustus
//WS2812B Config
//
#define DATA_PIN     4
#define COLOR_ORDER GRB
#define CHIPSET     WS2812B
#define NUM_LEDS    15

#define BRIGHTNESS  50
#define FRAMES_PER_SECOND 60

CRGB leds[NUM_LEDS];
static uint8_t heatIndex = 0; // start out at 0
static const uint8_t sunriseLength = 1;
static const uint8_t interval = (sunriseLength * 60) / 256;


// Update these with values suitable for your network.

int Aamua;

WiFiClient espClient;
PubSubClient client(espClient);
long lastMsg = 0;
char msg[50];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  delay(500);
  fill_solid( leds, NUM_LEDS, CRGB::Black);
  FastLED.show();
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
    
  }
  Serial.println();
  

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
     // Move a single white led 
   for(int whiteLed = 0; whiteLed < NUM_LEDS; whiteLed = whiteLed + 1) {
      // Turn our current led on to white, then show the leds
      leds[whiteLed] = CRGB::White;
      // Show the leds (only one of which is set to white, from above)
      FastLED.show();
      // Wait a little bit
      Serial.print("Ei tapahdu mitään");
      delay(100);
      // Turn our current led back to black for the next loop around
      leds[whiteLed] = CRGB::Black;
   }
      //after loop turn all leds black
      leds[0] = CRGB::Black;
      FastLED.show();
  }
if ((char)payload[0] == '0') {
     // Turn all leds off  
    fill_solid( leds, NUM_LEDS, CRGB::Black);
    FastLED.show();   
     Serial.print("Tuli pimeetä");
      delay(100);
   }

if ((char)payload[0] == '8') {
     // Turn all leds off  
     fill_solid( leds, NUM_LEDS, ClearBlueSky);
     FastLED.show();   
     Serial.print("Tuli yö");
      delay(100);
}

if ((char)payload[0] == '4') {
     // Turn all leds off  
     fill_solid( leds, NUM_LEDS, CRGB::Green);
     FastLED.show();   
     Serial.print("Vihree");
      delay(100);
}
if ((char)payload[0] == '3') {
     // Turn all leds off  
     fill_solid( leds, NUM_LEDS, CRGB::Red);
     FastLED.show();   
     Serial.print("Pun");
      delay(100);
}
if ((char)payload[0] == '2') {
     // Turn all leds off  
     fill_solid( leds, NUM_LEDS, CRGB::Blue);
     FastLED.show();   
     Serial.print("Sininen");
      delay(100);
}


if ((char)payload[0] == '7') {
  do {
  sunrise();
  FastLED.show();
  Serial.println("aamunousee");
  Serial.println(heatIndex);
  heatIndex++;
  delay(1000);
} while (heatIndex < 240);
}



if ((char)payload[0] == '9') {
  fill_solid( leds, NUM_LEDS, OvercastSky);
  FastLED.show();
}
  
  
  
  else {
    digitalWrite(BUILTIN_LED, HIGH);  // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  FastLED.addLeds<CHIPSET, DATA_PIN, COLOR_ORDER>(leds, NUM_LEDS);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);



}

void loop() {

  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    ++value;
    snprintf (msg, 75, "hello world #%ld", value);
    Serial.print("Publish message: ");
    Serial.println(msg);
    client.publish("outTopic", msg);
  }
}

void sunrise() {
  
  // total sunrise length, in minutes
  
  // current gradient palette color index
 // static uint8_t heatIndex = 0; // start out at 0

  // HeatColors_p is a gradient palette built in to FastLED
  // that fades from black to red, orange, yellow, white
  // feel free to use another palette or define your own custom one
  CRGB color = ColorFromPalette(HeatColors_p, heatIndex);

  // fill the entire strip with the current color
  fill_solid(leds, NUM_LEDS, color);
  
  // slowly increase the heat
  EVERY_N_SECONDS(interval) {
    // stop incrementing at 255, we don't want to overflow back to 0
    if(heatIndex < 240) {
      heatIndex++;
      }
  }
}
