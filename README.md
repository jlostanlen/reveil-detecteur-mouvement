# reveil-detecteur-mouvement
Construction un réveil muni d'un détecteur de mouvement avec un carte Arduino et le langage C++. Pour l'éteindre, il faut rester en mouvement face au détecteur pendant au moins 15 secondes.

![L'atelier...]([https://framasoft.org/nav/img/logo.png](https://github.com/jlostanlen/reveil-detecteur-mouvement/blob/main/IMG_0380%202%20(1)%20Medium.jpeg))


## Matériel utilisé : 
* Carte Arduino Uno
* Écran LCD
* Détecteur de mouvement HC-SR501
* 2 boutons poussoir
* Buzzer
* Module horloge RTC I2C

## Code : 
```
#include <LiquidCrystal.h>

#include <DS3231.h>

#include <string.h>

DS3231 rtc(SDA,SCL);

const int BUZZER = 9;
#define OUT_PIN 2
#define BUTTON_HEURES_PIN 3
#define BUTTON_MINUTES_PIN 4
uint8_t output_value = 0;
bool motion_detected = false;
const int DELAI_BOUTTON = 3000;

LiquidCrystal lcd(5, 6, 10, 11, 12, 13);


void setup() {
    lcd.begin(16,2);
    Serial.begin(9600);
    rtc.begin();
    pinMode(BUZZER, OUTPUT);
    pinMode(OUT_PIN, INPUT);
    pinMode(BUTTON_MINUTES_PIN, INPUT);
    pinMode(BUTTON_HEURES_PIN, INPUT);
    delay(1000);
    // Définir l'heure : 
    // rtc.setTime(01, 32, 00);
}

int minutes = 0;
int heures = 0;
String alarme_heure = "07:15:00";
String heures_string = "00";
String minutes_string = "00";
bool aSonne = false;
void loop() {
  lcd.clear();
  static unsigned long derniereTransition = 0;
  unsigned long _micros;
  uint8_t button_heures_state = digitalRead(BUTTON_HEURES_PIN);
  uint8_t button_minutes_state = digitalRead(BUTTON_MINUTES_PIN);
  // Paramétrage de l'horaire de la sonnerie
  // Heures
  if(button_heures_state == HIGH) { // Si le bouton a été pressé
    _micros = micros();
    if ((_micros - derniereTransition) >= DELAI_BOUTTON){ // Éviter les rebonds
      heures++;
      if (heures < 10){
        String tmp = String(heures);// Conversion de heures en String
        heures_string = "0"+tmp; 
        alarme_heure = heures_string+":"+minutes_string+":00";
      }
      else{
        String tmp = String(heures);
        heures_string = tmp; 
        alarme_heure = heures_string+":"+minutes_string+":00";
      }
      if (heures > 23){
        heures = 0;
        alarme_heure = "00:"+minutes_string+":00";
      }
      Serial.println("Alarme : ");
      Serial.println(alarme_heure);
    }
    derniereTransition = _micros;
  }
  
  // Minutes
  else if(button_minutes_state == HIGH) { // Si le bouton a été pressé
    _micros = micros();
    if ((_micros - derniereTransition) >= DELAI_BOUTTON){ // Éviter les rebonds
      minutes++;
      if (minutes < 10){
        String tmp = String(minutes);// Conversion de minutes en String
        minutes_string = "0"+tmp; 
        alarme_heure = heures_string+":"+minutes_string+":00";
      }
      else{
        String tmp = String(minutes);
        minutes_string = tmp; 
        alarme_heure = heures_string+":"+minutes_string+":00";
      }
      if (minutes > 59){
        minutes = 0;
        alarme_heure = heures_string+":00:00";
      }
      Serial.println("Alarme : ");
      Serial.println(alarme_heure);
    }
    derniereTransition = _micros;
  }

  // Alarme
  String real_time = rtc.getTimeStr();
  Serial.println(real_time);
  if (real_time.compareTo(alarme_heure) == 0){
    tone(BUZZER, 1000);
    aSonne = true;
  }
  if (aSonne){
    int attente = 0;
    output_value = digitalRead(OUT_PIN);
    while ((output_value == 1) && (attente < 15)){
      delay(1000);
      attente++;
      Serial.println("Mouvement !");
      output_value = digitalRead(OUT_PIN);
    }
    if (attente == 15){
      Serial.println("Attendu suffisamment");
      aSonne = false;
      noTone(BUZZER);
    }
  }


  // Affichage LCD
  lcd.setCursor(0,1);
  lcd.print("Heure : ");
  lcd.print(real_time);

  lcd.setCursor(0,0);
  lcd.print("Alarme: ");
  lcd.print(alarme_heure);

  delay(300);
}
```
