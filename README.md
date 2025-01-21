# livechat-spring

Este projeto implementa uma aplicação de chat em tempo real usando WebSockets e STOMP (Simple/Streaming Text Oriented Messaging Protocol). O backend foi desenvolvido em Java com Spring Boot, enquanto o frontend utiliza JavaScript para interagir com o servidor via WebSocket.

---

## Funcionalidades
- Conexão e desconexão ao servidor de WebSocket.
- Envio e recebimento de mensagens em tempo real.
- Publicação de mensagens no tópico `/topics/livechat`.

---

## Estrutura do Projeto

### Backend (Spring Boot)
- **`WebSocketConfig`**: Configuração do WebSocket com STOMP, definindo o endpoint `/buildrun-livechat-websocket` e prefixos de aplicação e mensagem.
- **`LiveChatController`**: Controlador que gerencia mensagens enviadas para o tópico `/topics/livechat`.
  - Endereça mensagens recebidas em `/app/new-message`.
  - Encaminha mensagens para os assinantes do tópico usando `@SendTo`.

### Frontend (JavaScript e jQuery)
- Conecta ao servidor WebSocket utilizando a biblioteca `@stomp/stompjs`.
- Gera eventos para:
  - Conectar e desconectar do servidor.
  - Publicar mensagens para o servidor.
  - Atualizar a interface do usuário com mensagens recebidas em tempo real.

---

## Tecnologias Utilizadas

### Backend:
- **Java 17**
- **Spring Boot 3**
  - Spring Web
  - Spring Messaging

### Frontend:
- **HTML/CSS**
- **JavaScript (ES6+)**
- **jQuery**
- **STOMP.js**

---

## Configuração do Projeto

1. Clone este repositório:
```bash
git clone <https://github.com/pedroleonez/livechat-spring.git>
cd <livechat-spring>
```

2. Inicie o servidor Spring Boot:
```bash
./mvnw spring-boot:run
```

3. Acesse o frontend:
- Abra o arquivo HTML correspondente no navegador ou configure um servidor web local para servi-lo.

---

## Uso

1. Conecte ao servidor clicando no botão **Conectar**.
2. Digite um nome de usuário e mensagem no formulário.
3. Envie a mensagem clicando no botão **Enviar**.
4. Mensagens de outros participantes aparecerão no painel do chat em tempo real.
5. Para desconectar, clique no botão **Desconectar**.

---

## Estrutura do Código

### Backend
- **Configuração do WebSocket**
  ```java
  @Configuration
  @EnableWebSocketMessageBroker
  public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
      @Override
      public void configureMessageBroker(MessageBrokerRegistry registry) {
          registry.enableSimpleBroker("/topics");
          registry.setApplicationDestinationPrefixes("/app");
      }

      @Override
      public void registerStompEndpoints(StompEndpointRegistry registry) {
          registry.addEndpoint("/buildrun-livechat-websocket");
      }
  }
  ```

- **Controlador de Mensagens**
  ```java
  @Controller
  public class LiveChatController {
      @MessageMapping("/new-message")
      @SendTo("/topics/livechat")
      public ChatOutput newMessage(ChatInput input) {
          return new ChatOutput(HtmlUtils.htmlEscape(input.user() + ": " + input.message()));
      }
  }
  ```

### Frontend
- **Conexão com o WebSocket**
  ```javascript
  const stompClient = new StompJs.Client({
      brokerURL: 'ws://' + window.location.host + '/buildrun-livechat-websocket'
  });

  stompClient.onConnect = (frame) => {
      setConnected(true);
      stompClient.subscribe('/topics/livechat', (message) => {
          updateLiveChat(JSON.parse(message.body).content);
      });
  };

  stompClient.onWebSocketError = (error) => {
      console.error('Error with websocket', error);
  };

  stompClient.onStompError = (frame) => {
      console.error('Broker reported error: ' + frame.headers['message']);
  };
  ```

- **Interação com o Usuário**
  ```javascript
  $("#connect").click(() => connect());
  $("#disconnect").click(() => disconnect());
  $("#send").click(() => sendMessage());
  ```

---

