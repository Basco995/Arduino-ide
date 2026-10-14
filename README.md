/*
  Arduino Fire-Fighting Robot
  Board: Arduino Uno
  Sensors: 3-pin digital flame sensors
  Motor driver: L298N
  Servo signal: A4
  Pump control: A5 (through a suitable driver)
*/

#define enA 10
#define in1 9
#define in2 8
#define in3 7
#define in4 6
#define enB 5

// Flame sensor digital outputs
#define ir_R A0
#define ir_F A1
#define ir_L A2

// Servo and pump control
#define servoPin A4
#define pump A5

// Most flame sensor modules are active LOW.
// Change to HIGH if your module outputs HIGH on detection.
const int FLAME_ACTIVE = LOW;

const int Speed = 160;  // Motor speed: 0 to 255

int s1, s2, s3;

// ---------- SERVO CONTROL ----------
// Software-generated servo pulses; no Servo library needed.

void servoPulseOnce(int angle) {
  angle = constrain(angle, 0, 180);

  int pulseWidth = 500 + (angle * 2000L / 180);

  digitalWrite(servoPin, HIGH);
  delayMicroseconds(pulseWidth);
  digitalWrite(servoPin, LOW);

  delay(20);
}

void setServoAngle(int angle) {
  // Repeat pulses to let the servo reach the target.
  for (int i = 0; i < 15; i++) {
    servoPulseOnce(angle);
  }
}

void sweepServo() {
  for (int angle = 90; angle <= 140; angle += 3) {
    servoPulseOnce(angle);
  }

  for (int angle = 140; angle >= 90; angle -= 3) {
    servoPulseOnce(angle);
  }

  setServoAngle(90);
}

// ---------- MOTOR CONTROL ----------

void forward() {
  analogWrite(enA, Speed);
  analogWrite(enB, Speed);

  digitalWrite(in1, HIGH);
  digitalWrite(in2, LOW);
  digitalWrite(in3, LOW);
  digitalWrite(in4, HIGH);
}

void backward() {
  analogWrite(enA, Speed);
  analogWrite(enB, Speed);

  digitalWrite(in1, LOW);
  digitalWrite(in2, HIGH);
  digitalWrite(in3, HIGH);
  digitalWrite(in4, LOW);
}

void turnRight() {
  analogWrite(enA, Speed);
  analogWrite(enB, Speed);

  digitalWrite(in1, LOW);
  digitalWrite(in2, HIGH);
  digitalWrite(in3, LOW);
  digitalWrite(in4, HIGH);
}

void turnLeft() {
  analogWrite(enA, Speed);
  analogWrite(enB, Speed);

  digitalWrite(in1, HIGH);
  digitalWrite(in2, LOW);
  digitalWrite(in3, HIGH);
  digitalWrite(in4, LOW);
}

void Stop() {
  digitalWrite(in1, LOW);
  digitalWrite(in2, LOW);
  digitalWrite(in3, LOW);
  digitalWrite(in4, LOW);

  analogWrite(enA, 0);
  analogWrite(enB, 0);
}

// ---------- SETUP ----------

void setup() {
  Serial.begin(9600);

  pinMode(ir_R, INPUT);
  pinMode(ir_F, INPUT);
  pinMode(ir_L, INPUT);

  pinMode(enA, OUTPUT);
  pinMode(in1, OUTPUT);
  pinMode(in2, OUTPUT);
  pinMode(in3, OUTPUT);
  pinMode(in4, OUTPUT);
  pinMode(enB, OUTPUT);

  pinMode(servoPin, OUTPUT);
  pinMode(pump, OUTPUT);

  digitalWrite(pump, LOW);

  Stop();
  setServoAngle(90);

  Serial.println("Fire-fighting robot ready");
}

// ---------- MAIN LOOP ----------

void loop() {
  // Read digital sensor outputs
  s1 = digitalRead(ir_R);
  s2 = digitalRead(ir_F);
  s3 = digitalRead(ir_L);

  bool rightFire = (s1 == FLAME_ACTIVE);
  bool centerFire = (s2 == FLAME_ACTIVE);
  bool leftFire = (s3 == FLAME_ACTIVE);

  Serial.print("Right: ");
  Serial.print(s1);
  Serial.print(" | Center: ");
  Serial.print(s2);
  Serial.print(" | Left: ");
  Serial.println(s3);

  // Center sensor detects flame: stop and spray
  if (centerFire) {
    Stop();

    digitalWrite(pump, HIGH);
    sweepServo();
    digitalWrite(pump, LOW);

    Stop();
    delay(200);
  }

  // Flame detected on the left
  else if (leftFire) {
    digitalWrite(pump, LOW);
    turnLeft();
    delay(200);
    Stop();
  }

  // Flame detected on the right
  else if (rightFire) {
    digitalWrite(pump, LOW);
    turnRight();
    delay(200);
    Stop();
  }

  // No flame detected
  else {
    digitalWrite(pump, LOW);
    Stop();
  }

  delay(50);
}
