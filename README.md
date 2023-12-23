#include <Servo.h>

//-----------------------------------------------------------------------------------------------------------------
//--------------------------------PROJETO: SISTEMA DE IRRIGAÇÃO UTILIZANDO ARDUINO --------------------------------
//-----------------------------------------------------------------------------------------------------------------
//------------------------------------ MICROCONTROLADORES - 2023.2-------------------------------------------------
//-----------------------------------------------------------------------------------------------------------------

const int obstaculoPin = 7;      // Pino digital para o sensor de obstáculo
const int switchPin = 8;        // Pino digital para o botão do switch
const int servoPin = 9;         // Pino digital para o servo motor
const int ledPin = 13;          // Pino digital para o LED

Servo myServo;                  // Objeto Servo para controlar o servo motor

int timerValue = 0;             // Valor inicial do timer (em segundos)
int obstacleThreshold = 500;    // Valor do sensor de obstáculo
bool switchState = false;        // Estado do botão do switch
unsigned long previousMillis = 0;  // Variável para armazenar o tempo anterior
const long interval = 1000;     // Intervalo para decrementar o timer (1 segundo)

void setup() {
  Serial.begin(9600);
  pinMode(obstaculoPin, INPUT);
  pinMode(switchPin, INPUT_PULLUP);
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW); // Inicialmente, o LED está apagado
  myServo.attach(servoPin);
  //instrução para o usuário digitar um tempo
  Serial.println("Digite o tempo do timer (em segundos) e dê enter e depois pressione o switch! ");
}

void loop() {
  // Leitura do sensor de obstáculo
  int obstaculoDetector = digitalRead(obstaculoPin);

  // Verifica se o botão do switch foi pressionado
  if (digitalRead(switchPin) == LOW && !switchState) {
    switchState = true;
    
    
    Serial.println("Começando circuito... ");
    while (Serial.available() == 0) {
      // Aguarda a entrada do usuário
    }
    timerValue = Serial.parseInt();
    Serial.println("Timer configurado para: " + String(timerValue) + " segundos");
    
  } else if (digitalRead(switchPin) == HIGH) {
    switchState = false;
  }

  // Atualiza o timer, decrementando 1 segundo por vez
  unsigned long currentMillis = millis();
  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;
    if (timerValue > 0) {
      Serial.print("Timer: ");
      Serial.println(timerValue);
      timerValue--;
    }
  }

  // Controla o servo motor e o LED
  if (timerValue > 0 && obstaculoDetector == HIGH) {
    myServo.write(180); // Move para 180 graus se o timer for maior que 0 e nenhum obstáculo for detectado
    digitalWrite(ledPin, HIGH); // Acende o LED enquanto o servo está funcionando
  } else if (timerValue > 0 && obstaculoDetector == LOW) {
    myServo.write(90); // Move para 90 graus se o timer for maior que 0 e obstáculo detectado
    // Piscar o LED enquanto há um obstáculo
    digitalWrite(ledPin, (millis() / 500) % 2 == 0 ? HIGH : LOW);
  } else {
    myServo.write(90); // Move para 90 graus se o timer atingir 0 ou obstáculo detectado
    digitalWrite(ledPin, LOW); // Apaga o LED se o servo não estiver funcionando
  }

  delay(100); // Aguarda um curto intervalo para evitar leituras rápidas
}