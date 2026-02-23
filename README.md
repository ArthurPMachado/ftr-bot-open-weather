# Bot de Clima para Telegram (n8n)

Este projeto contém um workflow do n8n que recebe mensagens no Telegram com cidade/estado, consulta o clima atual na OpenWeatherMap e responde no próprio Telegram com uma mensagem curta e objetiva

## Como o bot funciona

1. O usuário envia uma mensagem no Telegram com cidade e UF (ex.: `São Paulo,SP`).
2. O workflow normaliza o texto e adiciona a mensagem em uma fila no Redis.
3. O fluxo espera e consolida mensagens recentes (janela curta de tempo).
4. Um agente com Gemini formata a entrada para o padrão `CIDADE,UF,PAIS`, fazendo consulta para saber o país da cidade.
5. O nó HTTP Request consulta a API da OpenWeatherMap.
6. O workflow valida a resposta e extrai temperatura/cidade.
7. Outro agente com Gemini gera a resposta final.
8. O bot envia a resposta de volta no Telegram.

## Pré-requisitos

- n8n em execução
- Bot do Telegram criado no BotFather (com token)
- Conta/instância Redis acessível pelo n8n
- Chave de API do Google Gemini (para os nós de IA)
- Chave de API da OpenWeatherMap

## Como importar no n8n

1. Abra o n8n.
2. Clique em **Workflows**.
3. Clique em **Import from File**.
4. Selecione o arquivo `workflow-chatbot-telegram.json`.
5. Salve o workflow importado.
6. Configure as credenciais dos nós antes de ativar.

## Configuração das credenciais

### 1) Telegram (token do bot)

No workflow, os nós que usam Telegram são:

- `Telegram Trigger`
- `Send a text message`

Passos:

1. No Telegram, converse com `@BotFather`.
2. Execute `/newbot` e siga as instruções para criar o bot.
3. Copie o token gerado (formato parecido com `123456:ABC...`).
4. No n8n, abra qualquer um dos nós Telegram acima.
5. Em **Credentials**, crie/edite uma credencial `Telegram API`.
6. Cole o token no campo do token da API.
7. Salve e vincule a mesma credencial aos dois nós.

### 2) Redis

Nós que usam Redis:

- `Add To Queue`
- `Get Messages`
- `Erase Queue`

Passos:

1. No n8n, abra um dos nós Redis.
2. Em **Credentials**, crie/edite uma credencial `Redis`.
3. Preencha host, porta, usuário/senha (se necessário) e TLS (se aplicável).
4. Teste a conexão e salve.
5. Reutilize a mesma credencial nos três nós.

### 3) Google Gemini

Nós que usam Gemini:

- `Google Gemini Chat Model`
- `Google Gemini Chat Model1`

Passos:

1. Gere sua API key no Google AI Studio (ou serviço equivalente da sua conta).
2. No n8n, abra um dos nós Gemini.
3. Em **Credentials**, crie/edite `Google Gemini(PaLM) Api account`.
4. Cole sua API key e salve.
5. Vincule a mesma credencial aos dois nós de modelo.

### 4) OpenWeatherMap no HTTP Request

Nó:

- `Get Weather1`

Passos para preencher a chave (`appid`):

1. Crie conta em https://openweathermap.org/ e gere uma API key.
2. No n8n, abra o nó `Get Weather1` (HTTP Request).
3. Vá em **Query Parameters**.
4. Localize o parâmetro `appid` (já existe no workflow).
5. Preencha o valor do `appid` com sua API key da OpenWeatherMap.
6. Salve o nó.

Observação: os demais parâmetros já estão definidos no workflow (`q`, `units=metric`, `lang=pt_br`).

## Exemplo de uso

Mensagem enviada no Telegram:

`Campinas,SP`

Exemplo de resposta esperada:

`☀️ Campinas,SP: 27.3 ºC`

## Ativação e teste

1. Ative o workflow no n8n.
2. Envie uma cidade para seu bot no Telegram no formato `Cidade,UF`.
3. Verifique se o bot responde com temperatura em Celsius.
4. Se retornar cidade não encontrada, tente um formato mais específico (`Cidade,UF`).
