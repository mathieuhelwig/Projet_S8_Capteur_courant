---
layout: default
title: Programmation arduino
parent: Réalisation et programmation de l'objet connecté
grand_parent: Conception et prototypage
nav_order: 2
---

## 1.2 Programmation

### 1.2.1 Les différentes bibliothèques à intégrer


### 1.2.2 Calcul de la puissance


### 1.2.3 Récupération du timestamp


### 1.2.4 Transmission des données via MQTT


### 1.2.5 Programme définitif tournant sur l'Arduino Uno équipé du shield W5100

``` arduino
// Ces deux bibliothèques sont indispensables pour le shield ethjernet W5100
#include <SPI.h>
#include <Ethernet.h>
// Pour gérer la connexion UDP indispensable pour récupérer le timestamp
#include <EthernetUdp.h>
// Pour gérer la connexion au broker MQTT
#include <ArduinoMqttClient.h>
// Pour la gestion du bus I2C
#include <Wire.h>
// Pour la gestion des ADS1115
#include <Adafruit_ADS1X15.h>

// L'adresse MAC du shield
byte mac[] = { 0x90, 0xA2, 0xDA, 0x0E, 0xA5, 0x7E };//au hasard, je ne l'ai pas trouvée

unsigned int localPortUDP = 8888;  // local port to listen for UDP packets

const char timeServer[] = "time.nist.gov";  // time.nist.gov NTP server

const int NTP_PACKET_SIZE = 48;  // NTP time stamp is in the first 48 bytes of the message

byte packetBuffer[NTP_PACKET_SIZE];  //buffer to hold incoming and outgoing packets

// A UDP instance to let us send and receive packets over UDP
EthernetUDP Udp;
// L'adresse IP que prendra le shield
//IPAddress ip(10, 100, 0, 150);//adresse IP qui fonctionne à ESIEE réseau RIOC
IPAddress ip(192, 168, 2, 4);//adresse IP qui fonctionne à ESIEE sortie réseau RIOC
// L'objet qui nous servira à la communication
EthernetClient client;
//objet MQTT
MqttClient mqttClient(client);
//DAC ADS1115
Adafruit_ADS1115 ads1, ads2, ads3;
// Le serveur à interroger
//const char broker[] = "test.mosquitto.org";  //broker test modquitto address
//IPAddress ip_broker(10,19,4,42);//adresse IP broker serveur sur réseau ESIEE
IPAddress ip_broker(213,151,173,126);//adresse IP broker serveur sortie réseau ESIEE
//int port = 1883;                             //broker default port
int port = 25283;                             //broker port hors réseau ESIEE
const char* username_client_broker = "admin";//username client broker sur serveur ESIEE
const char* password_client_broker = "grp2pass";          //password client broker sur serveur ESIEE
//donnees à envoyer
const char topic_i1[] = "Projet_S8_Ieff1_mh1977";
const char topic_i2[] = "Projet_S8_Ieff2_mh1977";
const char topic_i3[] = "Projet_S8_Ieff3_mh1977";
const char topic_p1[] = "Projet_S8_Puiss1_mh1977";
const char topic_p2[] = "Projet_S8_Puiss2_mh1977";
const char topic_p3[] = "Projet_S8_Puiss3_mh1977";
const char topic_tmstmp[] = "Projet_S8_TmStmp_mh1977";
//set interval for sending messages (milliseconds)
const long interval = 1000;
unsigned long previousMillis = 0;
unsigned long beginMillis = 0;
unsigned long endMillis = 0;
unsigned long epoch = 0;

const float FACTOR = 50;
const float multiplier = 0.0625F;
int time_counter = 0;

//prototypes des fonctions
void PrintValue(String prefix, float value, String postfix);
void PrintConnectError(int error);
float get_I_Eff(int input);
void sendNTPpacket(const char* address);
unsigned long GetUnixTimestamp();
/*void SendMQTT_float_message(char top[], float val);
void SendMQTT_uint_message(char top[], unsigned long val);*/
void SendMQTT_message(char top[], float val);
void SendMQTT_message(char top[], unsigned long val);

//char serveur[] = "perdu.com";

void setup() {
  bool flag_ads1_init;
  bool flag_ads2_init;
  bool flag_ads3_init;
  // On démarre la voie série pour déboguer
  Serial.begin(9600);
  Wire.begin();
  while (!Serial) {
    ;  // wait for serial port to connect. Needed for native USB port only
  }

  ads1.setGain(GAIN_TWO);  // +/- 2.048V  0.0625mV
  ads2.setGain(GAIN_TWO);  // +/- 2.048V  0.0625mV
  ads3.setGain(GAIN_TWO);  // +/- 2.048V  0.0625mV
  ads1.setDataRate(860);
  ads2.setDataRate(860);
  ads3.setDataRate(860);
  /*if (!ads1.begin(0x48) || !ads2.begin(0x49) || !ads3.begin(0x4B)) {
    Serial.println("Failed to initialize one of the 3 ADS.");
    while (1)
      ;
  }*/
  while ( flag_ads1_init = !ads1.begin(0x48) ){
    // do nothing or
    Serial.println("Failed to initialize ADS1.");
    if(flag_ads1_init == false){
      Serial.println("Success to initialize ADS1.");
    }
    //while (1);
  };
  while ( flag_ads2_init = !ads2.begin(0x49) ){
    // do nothing or
    Serial.println("Failed to initialize ADS2.");
    if(flag_ads2_init == false){
      Serial.println("Success to initialize ADS2.");
    }
    //while (1);
  };
  while ( flag_ads3_init = !ads3.begin(0x4B) ){
    // do nothing or
    Serial.println("Failed to initialize ADS3.");
    if(flag_ads3_init == false){
      Serial.println("Success to initialize ADS3.");
    }
    //while (1);
  };

  char erreur = 0;
  // On démarre le shield Ethernet SANS adresse IP (donc donnée via DHCP)
  Serial.println("Parametrage avec IP via DHCP...");
  erreur = Ethernet.begin(mac);
  //Serial.println("Init avec IP via DHCP...");
  // Donne une seconde au shield pour s'initialiser
  //delay(1000);

  if (erreur == 0) {
    //Serial.println("Le parametrage de l'IP via DHCP a echoué !");
    Serial.println("Parametrage avec IP fixe...");
    // si une erreur a eu lieu cela signifie que l'attribution DHCP
    // ne fonctionne pas. On initialise donc en forçant une IP
    Ethernet.begin(mac, ip);
    //Serial.println("Init avec IP fixe...");
    // Donne une seconde au shield pour s'initialiser
    //delay(1000);
  }
  Serial.println("Init...");
  delay(1000);
  // Donne une seconde au shield pour s'initialiser
  Serial.print("Pret ! IP: ");
  Serial.println(ip);

  // On démarre la connexion UDP sur le port 8888 (defaut pour NTP) pour recuperer le timestamp
  Udp.begin(localPortUDP);

  //connexion au MQTTbroker
  Serial.print("Attempting to connect to the MQTT broker: ");
  //Serial.println(broker);//broker test mosquitto
  Serial.println(ip_broker);
  //if (!mqttClient.connect(broker, port)) {//broker address au format char[]
  mqttClient.setUsernamePassword(username_client_broker, password_client_broker);
  if (!mqttClient.connect(ip_broker, port)) {
    Serial.print("MQTT connection failed! Error code = ");
    Serial.println(mqttClient.connectError());
    PrintConnectError(mqttClient.connectError());
    while (1)
      ;
  }

  Serial.println("You're connected to the MQTT broker!");
  Serial.println();

  /*if (erreur == 1) {
    // Pas d'erreur ? on continue !
    Serial.println("Connexion OK, envoi en cours...");

    // On construit l'en-tête de la requête
    client.println("GET / HTTP/1.1");
    client.println("Host: perdu.com");
    client.println("Connection: close");
    client.println();
    } else {
    // La connexion a échoué :(
    Serial.println("Echec de la connexion");
    switch (erreur) {
      case (-1):
        Serial.println("Time out");
        break;
      case (-2):
        Serial.println("Serveur invalide");
        break;
      case (-3):
        Serial.println("Tronque");
        break;
      case (-4):
        Serial.println("Reponse invalide");
        break;
    }
    while (1); // On bloque la suite
    }*/
    epoch = GetUnixTimestamp();
    Serial.println(epoch);
}

void loop(void) {
  //beginMillis = millis();
  //Serial.println(epoch);
  //on met à jour le timestamp que toutes les minutes
  if (time_counter > 60) {
    epoch = GetUnixTimestamp();  //prend t>1s. pour recuperer le timestamp
    time_counter = 1;
  } else {
    epoch = epoch + (int)((endMillis - beginMillis)/1000);//durée d'une loop et mise à jour du timestamp
  }
  beginMillis = millis();

  //Serial.println("Mesure entre 0_1");
  float I_eff_1 = get_I_Eff(1);                //prend t>1s. pour recuperer I_eff
  //float Puissance_1 = 230.0 * I_eff_1 * 0.77;  //P = U_eff*I_eff*cos_phi (cos_phi = 0,77)
  int Puissance_1 = 230 * I_eff_1;
  //PrintValue("I_eff_1: ", I_eff_1, "A ,");
  //PrintValue("Puissance_1: ", Puissance_1, "W");
  float I_eff_2 = get_I_Eff(2);                //prend t>1s. pour recuperer I_eff
  //float Puissance_2 = 230.0 * I_eff_2 * 0.77;  //P = U_eff*I_eff*cos_phi (cos_phi = 0,77)
  int Puissance_2 = 230 * I_eff_2;
  //PrintValue("I_eff_2: ", I_eff_2, "A ,");
  //PrintValue("Puissance_2: ", Puissance_2, "W");
  float I_eff_3 = get_I_Eff(3);                //prend t>1s. pour recuperer I_eff
  //float Puissance_3 = 230.0 * I_eff_3 * 0.77;  //P = U_eff*I_eff*cos_phi (cos_phi = 0,77)
  int Puissance_3 = 230 * I_eff_3;
  Serial.println(Puissance_3);
  //PrintValue("I_eff_3: ", I_eff_3, "A ,");
  //PrintValue("Puissance_3: ", Puissance_3, "W");

  unsigned long currentMillis = millis();
  if (currentMillis - previousMillis >= interval) {  //n'envoie que si interval > 1s.
    // save the last time a message was sent
    previousMillis = currentMillis;

    //record random value from A0, A1 and A2
    /*int Rvalue = analogRead(A0);
      int Rvalue2 = analogRead(A1);
      int Rvalue3 = analogRead(A2);*/
    //float value1 = I_eff_1;
    //float value2 = Puissance_1;
    unsigned long value3 = epoch;

    /*Serial.print("Sending message to topic: ");
    Serial.println(topic);
    Serial.println(Rvalue);

    Serial.print("Sending message to topic: ");
    Serial.println(topic2);
    Serial.println(Rvalue2);

    Serial.print("Sending message to topic: ");
    Serial.println(topic3);
    Serial.println(Rvalue3);*/

    // send message, the Print interface can be used to set the message contents
    //SendMQTT_message(topic_i1, I_eff_1);
    SendMQTT_message(topic_p1, Puissance_1);
    //SendMQTT_message(topic_i2, I_eff_2);
    SendMQTT_message(topic_p2, Puissance_2);
    //SendMQTT_message(topic_i3, I_eff_3);
    SendMQTT_message(topic_p3, Puissance_3);
    
    SendMQTT_message(topic_tmstmp, epoch);

    /*mqttClient.beginMessage(topic);
    mqttClient.print(Rvalue);
    mqttClient.endMessage();

    mqttClient.beginMessage(topic2);
    mqttClient.print(Rvalue2);
    mqttClient.endMessage();

    mqttClient.beginMessage(topic3);
    mqttClient.print(Rvalue3);
    mqttClient.endMessage();*/

    //Serial.println();
  }

  // call poll() regularly to allow the library to send MQTT keep alive which
  // avoids being disconnected by the broker
  mqttClient.poll();

  /*char carlu = 0;
    // on lit les caractères s'il y en a de disponibles
    if (client.available()) {
    carlu = client.read();
    Serial.print(carlu);
    }

    // Si on est bien déconnecté.
    if (!client.connected()) {
    Serial.println();
    Serial.println("Deconnexion !");
    // On ferme le client
    client.stop();
    while (1); // On ne fait plus rien
    }*/
  endMillis = millis();
  time_counter++;
  Serial.println(epoch);
}

void PrintConnectError(int error) {
  // La connexion a échoué :(
  Serial.println("Echec de la connexion");
  switch (error) {
    case (-1):
      Serial.println("Time out");
      break;
    case (-2):
      Serial.println("Serveur invalide");
      break;
    case (-3):
      Serial.println("Tronque");
      break;
    case (-4):
      Serial.println("Reponse invalide");
      break;
  }
}

/*void PrintValue(String prefix, float value, String postfix) {
  Serial.print(prefix);
  Serial.print(value, 3);
  Serial.println(postfix);
}*/

float get_I_Eff(int input) {//note: changer le nom de la variable input en num_ADS
  int16_t results;
  float voltage;
  float current;
  float i_eff;
  float sum = 0;
  int counter = 0;
  long t = millis();
  while (millis() - t < 1000)  //on effectue des mesures sur 1s.
  {
    if (input == 1) {
      results = ads1.readADC_Differential_0_1();
      //Serial.print("Differential_0_1 sur ADS_1: "); Serial.println(results);
    }
    if (input == 2) {
      results = ads2.readADC_Differential_0_1();
      //Serial.print("Differential_0_1 sur ADS_2: "); Serial.println(results);
    }
    if (input == 3) {
      results = ads3.readADC_Differential_0_1();
      //Serial.print("Differential_0_1 sur ADS_3: "); Serial.println(results);
    }
    voltage = results * multiplier;
    //PrintValue("Tension de mesure 0_1: ", voltage, " mV");
    //PrintValue("Tension de mesure 0_3: ", voltage, " mV");
    current = voltage * FACTOR;
    current /= 1000.0;
    //PrintValue("Intensité mesurée 0_1: ", current, " A");
    //PrintValue("Intensité mesurée 0_3: ", current, " A");
    sum += sq(current);  //on somme i² pendant 1s.
    counter = counter + 1;
  }
  //Serial.print("sum = "); Serial.println(sum, 3);
  //Serial.print("counter = "); Serial.println(counter);

  i_eff = sqrt(sum / counter);
  return (i_eff);
}

// send an NTP request to the time server at the given address
void sendNTPpacket(const char* address) {
  // set all bytes in the buffer to 0
  memset(packetBuffer, 0, NTP_PACKET_SIZE);
  // Initialize values needed to form NTP request
  // (see URL above for details on the packets)
  packetBuffer[0] = 0b11100011;  // LI, Version, Mode
  packetBuffer[1] = 0;           // Stratum, or type of clock
  packetBuffer[2] = 6;           // Polling Interval
  packetBuffer[3] = 0xEC;        // Peer Clock Precision
  // 8 bytes of zero for Root Delay & Root Dispersion
  packetBuffer[12] = 49;
  packetBuffer[13] = 0x4E;
  packetBuffer[14] = 49;
  packetBuffer[15] = 52;

  // all NTP fields have been given values, now
  // you can send a packet requesting a timestamp:
  Udp.beginPacket(address, 123);  // NTP requests are to port 123
  Udp.write(packetBuffer, NTP_PACKET_SIZE);
  Udp.endPacket();
}

unsigned long GetUnixTimestamp() {
  unsigned long epoch;
  sendNTPpacket(timeServer);  // send an NTP packet to a time server

  // wait to see if a reply is available
  delay(1000);
  if (Udp.parsePacket()) {
    // We've received a packet, read the data from it
    Udp.read(packetBuffer, NTP_PACKET_SIZE);  // read the packet into the buffer

    // the timestamp starts at byte 40 of the received packet and is four bytes,
    // or two words, long. First, extract the two words:

    unsigned long highWord = word(packetBuffer[40], packetBuffer[41]);
    unsigned long lowWord = word(packetBuffer[42], packetBuffer[43]);
    // combine the four bytes (two words) into a long integer
    // this is NTP time (seconds since Jan 1 1900):
    unsigned long secsSince1900 = highWord << 16 | lowWord;
    /*Serial.print("Seconds since Jan 1 1900 = ");
    Serial.println(secsSince1900);*/

    // now convert NTP time into everyday time:
    // Unix time starts on Jan 1 1970. In seconds, that's 2208988800:
    const unsigned long seventyYears = 2208988800UL;
    // subtract seventy years:
    epoch = secsSince1900 - seventyYears;
    // print Unix time:
    //Serial.print("Unix time = ");
    //Serial.println(epoch);

    // print the hour, minute and second:
    /*Serial.print("The UTC time is ");       // UTC is the time at Greenwich Meridian (GMT)
    Serial.print((epoch  % 86400L) / 3600); // print the hour (86400 equals secs per day)
    Serial.print(':');
    if (((epoch % 3600) / 60) < 10) {
      // In the first 10 minutes of each hour, we'll want a leading '0'
      Serial.print('0');
    }
    Serial.print((epoch  % 3600) / 60); // print the minute (3600 equals secs per minute)
    Serial.print(':');
    if ((epoch % 60) < 10) {
      // In the first 10 seconds of each minute, we'll want a leading '0'
      Serial.print('0');
    }
    Serial.println(epoch % 60); // print the second*/
  }
  //Serial.println(epoch);
  return epoch;
}

void SendMQTT_message(char top[], int val) {
  mqttClient.beginMessage(top);
  mqttClient.print(val);
  mqttClient.endMessage();
  //Serial.print("ok: "); Serial.println(val);
}

void SendMQTT_message(char top[], float val) {
  mqttClient.beginMessage(top);
  mqttClient.print(val);
  mqttClient.endMessage();
  //Serial.print("ok: "); Serial.println(val);
}

void SendMQTT_message(char top[], unsigned long val) {
  mqttClient.beginMessage(top);
  mqttClient.print(val);
  mqttClient.endMessage();
  //Serial.print("ok: "); Serial.println(val);
}
```

## 1.3 Quelques images de l'objet connecté

<img
    style="display: block; 
           margin-left: auto;
           margin-right: auto;
           width: 70%;"
src="../images/capteur_courant_3_ads.jpg"
alt = "capteur de courants avec ses 3 ADS1115">
<p style="text-align: center;"><em>Le capteur de courants avec ses 3 ADS1115 et le shield ethernet monté sur la carte Arduino Uno</em></p>
