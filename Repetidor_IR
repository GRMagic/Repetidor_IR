/*
 *  Repetidor de Controle IR
 *  by Gustavo Rossi Müller
 *  gustavo.rm90@gmail.com
 */

#include <IRremote.h>
#include <EEPROM.h>
#include "Repetidor_IR.h"

IRrecv irrecv(PINO_SENSOR);
IRsend irsend;
int gravandoFase = 0;
int leituras = 0;

ComandoIR cmdEntrada;
ComandoIR cmdSaida;
decode_results results_ant;
decode_results results;

void setup() {
  
  ///Serial.begin(9600);
  
  pinMode(PINO_PILOTO, OUTPUT);
  pinMode(PINO_GRAVAR, INPUT_PULLUP);
  pinMode(PINO_BUZZER, OUTPUT);

  digitalWrite(PINO_PILOTO, HIGH);
  
  irrecv.enableIRIn(); // Inicia o sensor
  irrecv.blink13(true);

  ///Serial.println("Inicializado!");
  bipOK(); 
  
}

void loop() {
  int botao = tempoBotao();
  switch(botao){
    case 0: // Não pressionou
      break;
    case 1: // Pressionou
      // Grava um comando
      gravandoFase = 1;
      leituras = 0;
      bipInfo();
      ///Serial.println("Iniciou gravacao!");
      break;
    case 2: // Manteve Precionado
      if(tempoBotao() && tempoBotao() && tempoBotao() && tempoBotao() && tempoBotao() && tempoBotao() && tempoBotao() && tempoBotao() && tempoBotao()){
        // Manteve precionado por aproximadamente 10 Segundos
        // Inicia a limpeza da memória
        ///Serial.println("Manteve precionado por aproximadamente 10 segundos!");
        bipInfo();
        limparMemoria();
        bipOK();
      }else{
        // Manteve precionado por pouco tempo
        ///Serial.println("Manteve precionado por pouco tempo!");
      }
      break;
  }

  // Lê o controle remoto
  if (irrecv.decode(&results)) {
    ///Serial.print("Recebeu: ");
    ///Serial.print(results.decode_type);
    ///Serial.print(" - ");
    ///Serial.print(results.value, HEX);
    ///Serial.print(" - ");
    ///Serial.println(results.bits);
    if(results.decode_type != UNKNOWN && (results.decode_type != NEC || results.value != REPEAT)){
      ///Serial.println("Valido!");
      if(gravandoFase == 1){
        ///Serial.println("Fase 1 da gravacao!");
        if(results.decode_type != NEC && results.decode_type != SONY && results.decode_type != RC5 && results.decode_type != RC6){
          // Protocolo não suportado (na verdade, só não foram testado os outros protocolos, pode ser que funcionem!)
          ///Serial.println("Protocolo não suportado!");
          gravandoFase = 0;
          bipErro();
        }else{
          if(leituras == 0){
            results_ant = results;
            leituras++;
          }else if (leituras < 2){
            if(results_ant.decode_type == results.decode_type && results_ant.value == results.value){
              results_ant = results;
              leituras++;
            }else{
              gravandoFase = 0;
              bipErro();
            }
          }else if (leituras == 2){
            if(results_ant.decode_type == results.decode_type && results_ant.value == results.value){
              cmdEntrada.tipo = results.decode_type;
              cmdEntrada.valor = results.value;
              cmdEntrada.bits = results.bits;
              leituras = 0;
              gravandoFase = 2;
              bipInfo();
            }else{
              gravandoFase = 0;
              bipErro();
            }
          }
        }
      }else if (gravandoFase == 2){
        if(results.decode_type != NEC && results.decode_type != SONY && results.decode_type != RC5 && results.decode_type != RC6){
          // Protocolo não suportado (na verdade, só não foram testado os outros protocolos, pode ser que funcionem!)
          gravandoFase = 0;
          bipErro();
        }else{
          if(leituras == 0){
            results_ant = results;
            leituras++;
          }else if (leituras < 2){
            if(results_ant.decode_type == results.decode_type && results_ant.value == results.value){
              results_ant = results;
              leituras++;
            }else{
              gravandoFase = 0;
              bipErro();
            }
          }else if (leituras == 2){
            if(results_ant.decode_type == results.decode_type && results_ant.value == results.value){
              cmdSaida.tipo = results.decode_type;
              cmdSaida.valor = results.value;
              cmdSaida.bits = results.bits;
              gravarMemoria(cmdEntrada, cmdSaida);
              gravandoFase = 0;
              ///Serial.println("Fim da gravacao!");
              //bipOK(); a gravação, não está retornando nada, ela mesma já bipa para o usuário
            }else{
              gravandoFase = 0;
              bipErro();
            }
          }
        }
      }else{
        ///Serial.println("Reproducao!");
        cmdEntrada.tipo = results.decode_type;
        cmdEntrada.valor = results.value;
        cmdEntrada.bits = results.bits;
        ///Serial.println("Buscando...");
        int posicao = buscarMemoria(cmdEntrada);
        ///Serial.print("Resultado da busca: ");
        EEPROM.get(posicao + sizeof(ComandoIR), cmdSaida);
        if (posicao >= 0){
          ///Serial.println(posicao, HEX);
          ///Serial.print("Enviando: ");
          ///Serial.print(cmdSaida.tipo);
          ///Serial.print(" - ");
          ///Serial.print(cmdSaida.valor, HEX);
          ///Serial.print(" - ");
          ///Serial.println(cmdSaida.bits);
          sendIR(cmdSaida);
          irrecv.enableIRIn(); // Reinicia o sensor
        }else{
          ///Serial.println("Nao achou!");
        }
        ///Serial.println("Fim do envio!");
      }
    }
    irrecv.resume(); 
  }
  
}

void sendIR(ComandoIR cmd){
  switch(cmd.tipo){
    case NEC:
      irsend.sendNEC(cmd.valor, cmd.bits);
      break;
    case SONY:
      irsend.sendSony(cmd.valor, cmd.bits);
      break;
    case RC5:
      irsend.sendRC5(cmd.valor, cmd.bits);
      break;
    case RC6:
      irsend.sendRC6(cmd.valor, cmd.bits);
      break;
  }
  delay(100);
}

// Testado
int tempoBotao() {
  /*
   * 0 Não pressionou
   * 1 Pressionou
   * 2 Manteve Precionado por aproximadamente 1 segundo
   */
  int tempo = 0;
  int status = digitalRead(PINO_GRAVAR);
  while (status == LOW && tempo < 10) {
    tempo++;
    delay(100);
    status = digitalRead(PINO_GRAVAR);
  }
  if(tempo == 0) return 0;
  if(tempo < 10) return 1;
  return 2;
}

// Testado
void gravarMemoria(ComandoIR entrada, ComandoIR saida) {
  int posicao = buscarMemoria(entrada);
  if (posicao < 0) {
    ComandoIR vazio;
    vazio.tipo = 0;
    vazio.valor = 0;
    vazio.bits = 0;
    posicao = buscarMemoria(vazio);
  }
  if (posicao < 0) {
    // Acabou a memória
    bipErro();
    return;
  }
  EEPROM.put(posicao, entrada);
  EEPROM.put(posicao + sizeof(ComandoIR), saida);
  bipOK();
}

// Testado
int buscarMemoria(ComandoIR cmd) {
  /*
   * Na memória deve estar guardado um comando de entrada seguido de um comando de saída
  */
  ComandoIR memorizado;
  for ( int i = 0 ; i <= EEPROM.length() - sizeof(ComandoIR) * 2 ; i += sizeof(ComandoIR) * 2 ) {
    EEPROM.get(i, memorizado);
    if (cmd.tipo == memorizado.tipo && cmd.valor == memorizado.valor && cmd.bits == memorizado.bits) {
      return i;
    }
    // Se achou um lugar zerado da memória e não é isso que está procurando, para de procurar
    if (0 == memorizado.tipo && 0 == memorizado.valor && 0 == memorizado.bits) {
      return -1;
    }
  }
  return -1;
}


// Testado
void limparMemoria() {
  for ( int i = 0 ; i < EEPROM.length() ; i++ )
    EEPROM.write(i, 0);
}

// Testado
void bipOK() {
  // Dois Bips Rápidos
  digitalWrite(PINO_BUZZER, HIGH);
  delay(50);
  digitalWrite(PINO_BUZZER, LOW);
  delay(50);
  digitalWrite(PINO_BUZZER, HIGH);
  delay(50);
  digitalWrite(PINO_BUZZER, LOW);
  delay(50);
}

// Testado
void bipErro() {
  // Um Bip Longo
  digitalWrite(PINO_BUZZER, HIGH);
  delay(500);
  digitalWrite(PINO_BUZZER, LOW);
  delay(50);
}


// Testado
void bipInfo() {
  // Um Bip
  digitalWrite(PINO_BUZZER, HIGH);
  delay(150);
  digitalWrite(PINO_BUZZER, LOW);
  delay(50);
}
