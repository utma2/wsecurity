# wsecurity
aplicacion para evitar la inseguridad contra las mujeres con botón de pánico y llamadas falas para asegurar la vida y seguridad de la persona en riesgo  
codigo de arduino para boton de panico con modulo bluetooth




#include <SoftwareSerial.h>

// Configura los pines para la comunicación con el módulo Bluetooth
SoftwareSerial BTSerial(10, 11);  // RX, TX

// Define el pin del buzzer
int buzzerPin = 9;
int modulo = 12;
void setup() {
  // Inicia la comunicación serie
  Serial.begin(9600);        // Comunicación con el monitor serie
  BTSerial.begin(9600);      // Comunicación con el módulo Bluetooth
  
  // Configura el pin del buzzer como salida
  pinMode(buzzerPin, OUTPUT);
  
  // Apaga el buzzer al inicio
  digitalWrite(buzzerPin, LOW);
  
  // Mensaje de inicio
  Serial.println("Esperando comandos vía Bluetooth...");
}

void loop() {
  // Si se recibe un dato desde el módulo Bluetooth
  if (BTSerial.available()) {
    char command = BTSerial.read();  // Lee el comando enviado por Bluetooth
    
    // Imprime el comando en el monitor serie
    Serial.print("Comando recibido: ");
    Serial.println(command);
    
    // Si el comando es '1', enciende el buzzer
    if (command == '1') {
      digitalWrite(buzzerPin, HIGH);  // Enciende el buzzer
      Serial.println("Buzzer encendido");
    }
    // Si el comando es '0', apaga el buzzer
    else if (command == '0') {
      digitalWrite(buzzerPin, LOW);   // Apaga el buzzer
      Serial.println("Buzzer apagado");
    }
  }
}
