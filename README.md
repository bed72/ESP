## Problema

O código está escrito em **C++** e utiliza a biblioteca `<thread>` para lidar com _threads_, o que é uma boa abordagem para ativar ambas as bombas simultaneamente. Entretanto, ele não será compatível diretamente com o **ESP8266**, pois:

- **ESP8266** utiliza _Arduino_ **C++** com sua própria **API**.
- A biblioteca `<thread>` e funções como `std::this_thread::sleep_for` não são suportadas no ambiente **Arduino/ESP8266**.

## Problemas no Contexto ESP8266

- O **ESP8266** é um _microcontrolador_ e não tem suporte ao `<thread>` por ser uma biblioteca do **C++** padrão, que depende de um sistema operacional subjacente.
- Funções bloqueantes como `std::this_thread::sleep_for` não são ideais em um ambiente de _microcontroladores_. O `delay` da biblioteca _Arduino_ seria mais adequado.
- O uso de `std::cin` e `std::cout` também não é apropriado, já que a interação geralmente ocorre via _Serial_ no **ESP8266**.

## Sugestões de Ajustes

Aqui está uma versão adaptada para o **ESP8266** usando a **IDE Arduino**:

    #define BOMBA1_PIN D1 // Substitua pelos pinos conectados às bombas
    #define BOMBA2_PIN D2

    void ativarBomba(int tempo_ms, int pin) {
        digitalWrite(pin, HIGH); // Liga a bomba
        delay(tempo_ms);         // Mantém a bomba ligada pelo tempo especificado
        digitalWrite(pin, LOW);  // Desliga a bomba
    }

    void setup() {
        Serial.begin(115200);

        pinMode(BOMBA1_PIN, OUTPUT);
        pinMode(BOMBA2_PIN, OUTPUT);

        int tempoBomba1, tempoBomba2;

        Serial.println("Digite o tempo de ativação da Bomba 1 (em milissegundos):");
        while (Serial.available() == 0); // Espera o input
        tempoBomba1 = Serial.parseInt();

        Serial.println("Digite o tempo de ativação da Bomba 2 (em milissegundos):");
        while (Serial.available() == 0);
        tempoBomba2 = Serial.parseInt();

        Serial.println("Ativando as bombas...");
        ativarBomba(tempoBomba1, BOMBA1_PIN);
        ativarBomba(tempoBomba2, BOMBA2_PIN);

        Serial.println("Mistura completa!");
    }

    void loop() {
        // O código principal geralmente fica vazio se não houver uma funcionalidade contínua
    }

## Código compilado

```
// Definição dos pinos dos relés
#define RELE1 5  // Pino D5
#define RELE2 18 // Pino D18
#define RELE3 19 // Pino D19

// Função para ligar uma bomba por um determinado tempo
void acionarBomba(int pino, int tempo) {
  digitalWrite(pino, HIGH); // Liga o relé
  delay(tempo);             // Aguarda o tempo especificado
  digitalWrite(pino, LOW);  // Desliga o relé
}

void setup() {
  // Configuração dos pinos como saída
  pinMode(RELE1, OUTPUT);
  pinMode(RELE2, OUTPUT);
  pinMode(RELE3, OUTPUT);

  // Inicializa os relés desligados
  digitalWrite(RELE1, LOW);
  digitalWrite(RELE2, LOW);
  digitalWrite(RELE3, LOW);
}

void loop() {
  // Liga cada bomba por um tempo específico (em milissegundos)
  acionarBomba(RELE1, 3000); // Bomba 1 ligada por 3 segundos
  delay(1000);               // Intervalo de 1 segundo
  acionarBomba(RELE2, 2000); // Bomba 2 ligada por 2 segundos
  delay(1000);               // Intervalo de 1 segundo
  acionarBomba(RELE3, 4000); // Bomba 3 ligada por 4 segundos
  delay(1000);               // Intervalo de 1 segundo
}
```

## Ajustes Importantes

- **Pinos**: Configure os pinos corretos de acordo com o esquema de conexão.
- **Interface Serial**: A comunicação usa **Serial** para interação com o computador.
- **Sincronização**: As bombas não funcionam em paralelo aqui; ambas são ativadas sequencialmente. Caso deseje paralelismo, você precisará implementar lógica de temporização utilizando funções como `millis()`.
