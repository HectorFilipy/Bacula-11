## Enviar mensagem do Bacula usando Telegram


### Criar um Bot no Telegram

Iremos adicionar o usuário @BotFather para a sua conta do telegram ou acessar o endereço https://telegram.me/BotFather
e seguir os passos abaixo:

- /newbot - criar um novo bot

- Digitar um nome para o bot. Exemplo: Bacula Bot

- Digitar um nome de usário para o bot. Precisar terminar com 'bot' Exemplo: (bacula_bot)

Exemplo: 

```txt
Anotar a chave da API (API KEY):
1234567890:AAFd2sDMplKGyoajsPWARnSOwa9EqHiy17U
```

* Substituir na url com a chave da API

```txt
https://api.telegram.org/bot${API_KEY}/getUpdates

https://api.telegram.org/bot1234567890:AAFd2sDMplKGyoajsPWARnSOwa9EqHiy17U/getUpdates
```

* Enviar uma mensagem para o BOT, abrir o browser e colar a URL obtida.

* Você vai receber uma saída no formato JSON, pegar o valor do 'id'.

```txt
{"ok":true,"result":[{"update_id":565543449,
"message":{"message_id":3,"from":{"id":123456789,"first_name":"Some Name","last_name":"Some Last Name",
"username":"someusername"},"chat":{"id":123456789,"first_name":"Some Name","last_name":"Some Last Name",
"username":"someuser","type":"private"},"date":1472165730,"text":"hello"}}]}
```

### Preencher os campos no script

* Iremos alterar os seguintes campos 

  - DBUSER - Para o usuário do seu bacula.
  - DBPASSWD - Para a senha que voce definiu no seu bacula.

```txt
DBHOST="localhost"
DBPORT="3306"
DBNAME="bacula"
DBUSER="bacula"
DBPASSWD="bacula"
```

* No PostgreSQL é possível criar um arquivo no diretório home como ".pgpass" com o seguinte conteúdo: (substituir com os valores corretos) 
**hostname:port:database:username:password**

* Alterar a permissão para somente o usuário ter acesso

```bash
chmod 600 ~/.pgpass
```

### Adicionar o recurso RunsScript na configuração do JobDefs

```bash
JobDefs {
   ...
   RunScript {
     Command = "/etc/bacula/scripts/_send_telegram.sh %i"
     RunsWhen = After
     RunsOnFailure = yes
     RunsOnClient = no
     RunsOnSuccess = yes # default, you can drop this line
  }
}
```

### Instalar dependências

```basg
apt-get install curl
```
