
# Controle de LEDs com ESP32 e App Mobile 🚀

Este projeto permite controlar LEDs com uma placa ESP32, utilizando um aplicativo mobile desenvolvido em React Native. Este guia foi feito para te ajudar a configurar e executar ambos os códigos de forma eficiente e sem dificuldades.

## ✅ Requisitos
Antes de iniciar, certifique-se de ter os seguintes itens prontos:

1. 🛠 Placa ESP32.
2. 🛠 LEDs para conectar à ESP32.
3. 🛠 Cabo USB para conectar a ESP32 ao computador.
4. 🛠 Smartphone ou emulador para executar o aplicativo React Native.
5. 🛠 Acesso a uma rede Wi-Fi.

## 1. Configurando o Hardware ⚙️

1. **Conecte os LEDs** — Ligue os LEDs às portas especificadas na ESP32. No código da ESP32, os LEDs estão conectados aos pinos **2, 4, 5, 18, 19 e 21**.
2. **Conecte a ESP32** — Utilize um cabo USB para conectar a ESP32 ao computador e comece a programar.

## 2. Configurando a ESP32 🛠

1. **Bibliotecas Necessárias** — Certifique-se de ter a biblioteca `WiFi.h` e `WebServer.h` no Arduino IDE.

2. **Conecte ao Wi-Fi** — No código, altere as variáveis `ssid` e `password` para corresponder ao nome e senha da sua rede Wi-Fi.

3. **Faça o Upload do Código** — Carregue o seguinte código na sua ESP32 usando o Arduino IDE:

    ```cpp
    #include <WiFi.h>
    #include <WebServer.h>

    // Defina o SSID e a senha da sua rede Wi-Fi
    const char* ssid = "LAB-SI"; 
    const char* password = "LAB@SI1010";

    // Crie uma instância do servidor web na porta 80
    WebServer server(80);

    // Definição dos pinos dos LEDs (ajuste conforme necessário)
    const int ledPins[] = {2, 4, 5, 18, 19, 21};

    void setup() {
      Serial.begin(115200);

      // Configuração dos pinos dos LEDs como saída
      for (int i = 0; i < 10; i++) {
        pinMode(ledPins[i], OUTPUT);
        digitalWrite(ledPins[i], LOW); // Inicializa os LEDs como desligados
      }

      // Conexão à rede Wi-Fi
      WiFi.begin(ssid, password);
      Serial.print("Conectando-se ao Wi-Fi...");
      while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
      }
      Serial.println("
    Conectado ao Wi-Fi!");
      Serial.print("Endereço IP: ");
      Serial.println(WiFi.localIP());

      // Configuração das rotas
      server.on("/led", handleLED);

      // Inicia o servidor
      server.begin();
      Serial.println("Servidor HTTP iniciado");
    }

    void loop() {
      server.handleClient();
    }

    // Função para manipular as requisições para "/led"
    void handleLED() {
      if (server.hasArg("led") && server.hasArg("state")) {
        int led = server.arg("led").toInt();
        int state = server.arg("state").toInt();

        if (led >= 0 && led < 6 && (state == 0 || state == 1)) {
          digitalWrite(ledPins[led], state);
          server.send(200, "text/plain", "LED atualizado com sucesso");
          Serial.printf("LED %d %s
    ", led + 1, state ? "ligado" : "desligado");
        } else {
          server.send(400, "text/plain", "Parâmetros inválidos");
        }
      } else {
        server.send(400, "text/plain", "Parâmetros 'led' e 'state' são necessários");
      }
    }
    ```

4. **Monitor Serial** — Abra o monitor serial para encontrar o endereço IP da ESP32 ao conectar-se ao Wi-Fi.

## 3. Configurando o App Mobile 📱

1. **Clone o Repositório** — Clone ou baixe o código do aplicativo React Native em seu computador.

2. **Instalar Dependências** — Execute o comando abaixo na pasta do projeto para instalar todas as dependências necessárias:

   ```bash
   npm install
   ```

3. **Atualize o Endereço IP** — No arquivo `App.tsx`, altere a constante `esp32IP` para o endereço IP obtido no monitor serial da ESP32.

   ```tsx
   const esp32IP: string = 'http://SEU_ENDEREÇO_IP:80';
   ```

4. **Execute o Aplicativo** — Para rodar o app, utilize o comando:

   ```bash
   npx expo start
   ```

5. **Emulador ou Smartphone** — Execute o app em um emulador ou em um smartphone real. Certifique-se de que o smartphone esteja conectado à mesma rede Wi-Fi que a ESP32.

6. **Código do App** — O código do aplicativo React Native é o seguinte:

    ```tsx
    import React, { useState, useEffect } from 'react';
    import { Button, View, Text, StyleSheet, Modal, Pressable } from 'react-native';
    import axios from 'axios';

    const App: React.FC = () => {
      const esp32IP: string = 'http://10.0.3.50:80'; // Insira o IP que consta na Serial 115200 coloque o IP ds ESP32 mais porta :80
      const [modalLedsVisible, setModalLedsVisible] = useState(false);
      const [isBlinking, setIsBlinking] = useState(false);
      let blinkInterval: NodeJS.Timeout | null = null;

      // Função para controlar os LEDs
      const controlLED = (led: number, state: number): void => {
        axios.get(`${esp32IP}/led`, { params: { led, state } })
          .then(response => {
            console.log(`LED ${led} atualizado:`, response.data);
          })
          .catch(error => {
            console.error(`Erro ao controlar LED ${led}:`, error);
          });
      };

      // Função para alternar o piscar das luzes
      const toggleBlinking = () => {
        setIsBlinking(!isBlinking);
      };

      useEffect(() => {
        if (isBlinking) {
          blinkInterval = setInterval(() => {
            // Liga todos os LEDs
            for (let i = 0; i < 10; i++) {
              controlLED(i, 1);
            }

            setTimeout(() => {
              // Desliga todos os LEDs
              for (let i = 0; i < 10; i++) {
                controlLED(i, 0);
              }
            }, 500);
          }, 1000);
        } else {
          if (blinkInterval) {
            clearInterval(blinkInterval);
            blinkInterval = null;
          }
        }

        // Limpeza na desmontagem do componente
        return () => {
          if (blinkInterval) {
            clearInterval(blinkInterval);
            blinkInterval = null;
          }
        };
      }, [isBlinking]);

      return (
        <View style={styles.container}>
          <Text style={styles.header}>Controle de LEDs</Text>
          <Pressable style={styles.openButton} onPress={() => setModalLedsVisible(true)}>
            <Text style={styles.buttonText}>Abrir Controle dos LEDs</Text>
          </Pressable>

          {/* Modal para Controle dos LEDs */}
          <Modal
            animationType="slide"
            transparent={true}
            visible={modalLedsVisible}
            onRequestClose={() => {
              setModalLedsVisible(!modalLedsVisible);
            }}>
            <View style={styles.modalContainer}>
              <View style={styles.modalView}>
                {/* Controles dos LEDs */}
                <Text style={styles.sectionHeader}>Controle dos LEDs</Text>

                {/* Botão para piscar todas as luzes */}
                <Pressable
                  style={[styles.roundButton, styles.blinkButton]}
                  onPress={toggleBlinking}>
                  <Text style={styles.buttonText}>
                    {isBlinking ? 'Parar Piscar Todas as Luzes' : 'Piscar Todas as Luzes'}
                  </Text>
                </Pressable>

                {/* Controles individuais dos LEDs */}
                {Array.from({ length: 6 }, (_, i) => (
                  <View key={i} style={styles.ledControl}>
                    <Pressable
                      style={styles.roundButton}
                      onPress={() => controlLED(i, 1)}>
                      <Text style={styles.buttonText}>Ligar LED {i + 1}</Text>
                    </Pressable>
                    <Pressable
                      style={styles.roundButton}
                      onPress={() => controlLED(i, 0)}>
                      <Text style={styles.buttonText}>Desligar LED {i + 1}</Text>
                    </Pressable>
                  </View>
                ))}

                <Pressable
                  style={[styles.roundButton, styles.closeButton]}
                  onPress={() => setModalLedsVisible(!modalLedsVisible)}>
                  <Text style={styles.buttonText}>Fechar Controle dos LEDs</Text>
                </Pressable>
              </View>
            </View>
          </Modal>
        </View>
      );
    };

    const styles = StyleSheet.create({
      container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: '#f5f5f5',
        padding: 16,
      },
      header: {
        fontSize: 24,
        fontWeight: 'bold',
        marginBottom: 24,
      },
      openButton: {
        backgroundColor: '#2196F3',
        borderRadius: 20,
        padding: 10,
        elevation: 2,
        marginVertical: 10,
      },
      modalContainer: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: 'rgba(0, 0, 0, 0.5)',
      },
      modalView: {
        margin: 20,
        backgroundColor: 'white',
        borderRadius: 20,
        padding: 35,
        alignItems: 'center',
        shadowColor: '#000',
        shadowOffset: {
          width: 0,
          height: 2,
        },
        shadowOpacity: 0.25,
        shadowRadius: 4,
        elevation: 5,
      },
      sectionHeader: {
        fontSize: 20,
        fontWeight: 'bold',
        marginVertical: 12,
      },
      ledControl: {
        flexDirection: 'row',
        justifyContent: 'space-between',
        marginVertical: 6,
      },
      roundButton: {
        backgroundColor: '#2196F3',
        borderRadius: 30,
        padding: 10,
        margin: 5,
        elevation: 5,
        shadowColor: '#000',
        shadowOffset: {
          width: 0,
          height: 2,
        },
        shadowOpacity: 0.25,
        shadowRadius: 3.84,
      },
      blinkButton: {
        backgroundColor: '#FFC107',
      },
      closeButton: {
        backgroundColor: '#f44336',
      },
      buttonText: {
        color: 'white',
        fontWeight: 'bold',
        textAlign: 'center',
      },
    });

    export default App;
    ```

## 4. Usando o Sistema 💡

1. **Controle dos LEDs** — Acesse a opção de controle dos LEDs no aplicativo. Você pode ligar ou desligar cada LED individualmente ou fazer todos piscarem simultaneamente.

## 5. Resolução de Problemas ⚠️
- **Problema de Conexão ao Wi-Fi**: Verifique o SSID e senha. Certifique-se de que o roteador esteja funcionando corretamente.
- **Aplicativo não Controla a ESP32**: Verifique se o endereço IP da ESP32 está correto e se a ESP32 está conectada à mesma rede do smartphone.
- **Erro ao Ligar LEDs**: Certifique-se de que os LEDs estão conectados às portas corretas e funcionando adequadamente.

## 📜 Referências e Ferramentas Utilizadas
- **React Native** — Para o desenvolvimento do app mobile.
- **Arduino IDE** — Para programar a ESP32.
- **Node.js e Expo** — Para executar e testar o aplicativo.

Esperamos que este guia tenha ajudado você a configurar e executar o projeto. Caso tenha dúvidas, sinta-se à vontade para perguntar!

## 🛡️ Sugestões de Melhorias Futuras
- **Sensores de Proximidade** — Adicionar sensores para detectar se os LEDs estão funcionando corretamente.
- **Integração com Assistente de Voz** — Adicionar comandos de voz para controlar os LEDs.

💡 **Dica**: Sempre documente qualquer alteração feita no código para facilitar a manutenção do projeto!

Boa sorte e divirta-se criando! 🚀😉
