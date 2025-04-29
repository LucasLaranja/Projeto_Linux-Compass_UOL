# 👨‍💻 Projeto Linux - Compass UOL - DevSecOps Abril 2025

## Monitorando servidor nginx com Webhook

### Etapas:

* Configuração do ambiente;

* Como instalar e configurar o servidor web;

* Como funciona o script de monitoramento;

* Teste e validação da solução.

* Conclusão

---
## Etapa 1 - Configuração do ambiente:
    Nesse projeto decide utilizar uma máquina virtual da Oracle VirtualBox, utilizando o sistema Ubuntu como sugerido, na versão 25.04. Habilitando o modo bridge e configurando nos seus requisitos mínimos.

    ![Print 1](/Prints/1.png)

    ![Print 2](/Prints/2.png)
--

Foi necessário criar um servidor para receber a mensagem via webhook, foram dadas as opções de utilizar Discord, Telegram ou Slack. No caso desse projeto foi utilizado o Discord.

![Print 3](/Prints/3.png)

![Print 4](/Prints/4.png)

![Print 5](/Prints/5.png)

---

## Etapa 2 - Como instalar e configurar o servidor web:
    Após configurar o ambiente, faremos a instalação de algumas aplicações:
    >`apt-get install nginx` - Servidor Web

    >`apt-get install curl` - Para verificar a conectividade da URL

    Logo após fui até até a pasta www através do comando:
    >`cd /var/www`

    E criei uma pasta nova dedicada ao projeto e inseri o HTML na pasta. Sendo o nome da pasta devseclaranja.local. Após isso comecei a personalizar o HTML com css, até um nível que considerei satisfatório.

    ![Print 6](/Prints/6.png)

    Adicionei a ele algumas informações do projeto e o finalizei.

---

## Etapa 3 - Como funciona o script de monitoramento:
    Em seguida fui para a parte de criação do script, no meu caso criei uma pasta chamada scripts na pasta nginx localizada em:
    >`cd /etc/nginx`

    E nela criei um script chamado verifica_siteprojeto.sh, foram dadas outras opções como utilizar python para o script, mas optei por usar bash, assim criei o código.
    ```
#!/bin/bash

SITE_URL="http://devseclaranja.local"
WEBHOOK="https://discord.com/api/webhooks/..."
LOG="/var/log/devseclaranja.log"

while true; do
    DATA_HORA=$(date '+%d/%m/%Y %H:%M:%S')
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$SITE_URL")

    if [ "$STATUS" != "200" ]; then
        echo "$DATA_HORA - [$SITE_URL] está offline! Status: $STATUS" >> "$LOG"

        curl -H "Content-Type: application/json" \
             -X POST \
             -d "{\"content\": \"O site [$SITE_URL] está apresentando erro! Código HTTP: $STATUS - $DATA_HORA\"}" \
             "$WEBHOOK"
    else
        echo "$DATA_HORA - [$SITE_URL] Está online! - Status: $STATUS" >> "$LOG"
    fi

    sleep 60

````
### Explicação do Código:

>`#!/bin/bash` - Indica que o script será executado usando o interpretador de comandos do Bash. É obrigatório em scripts Bash. Chama-se "shebang".

>`SITE_URL="http://devseclaranja.local"` - Variável chamada SITE_URL que armazena o endereço do site que vamos monitorar. No meu caso eu personalizei, mas pode ser chamado localhost ou até o ip da máquina.

>`WEBHOOK="https://discord.com/api/webhooks/..."` - Outra variável, chamada WEBHOOK, guarda o link do webhook do Discord. Esse link é onde as mensagens vão ser enviadas quando o site estiver offline. No caso desse está incompleto por motivos de segurança.

>`LOG="/var/log/devseclaranja.log"` - Essa variável define o caminho do arquivo de log onde os status do site (online ou offline) serão salvos.

>`while true; do` - Esse comando inicia um loop infinito, ou seja, o script vai rodar para sempre, verificando o site continuamente.

>`DATA_HORA=$(date '+%d/%m/%Y %H:%M:%S')` - Essa linha salva a data e a hora atual numa variável chamada DATA_HORA. O comando date formata a data nesse padrão: dia/mês/ano hora:minuto:segundo.

>`STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$SITE_URL")` - Aqui usamos o curl para checar se o site está online:
-s: modo silencioso (não mostra nada no terminal).
-o /dev/null: ignora o conteúdo da resposta.
-w "%{http_code}": mostra somente o código HTTP (como 200, 404, 500, etc).
Tudo isso é armazenado na variável STATUS.
Se o site estiver ok, STATUS será 200.

>`if [ "$STATUS" != "200" ]; then` - Se o código de status não for 200, isso significa que o site está com problema.

>`echo "$DATA_HORA - [$SITE_URL] está offline! Status: $STATUS" >> "$LOG"` - Essa linha grava uma mensagem no log, dizendo que o site está offline. O >> serve para adicionar a mensagem ao final do arquivo, sem apagar o conteúdo anterior.

>`curl -H "Content-Type: application/json" \
-X POST \
-d "{\"content\": \"O site [$SITE_URL] está apresentando erro! Código HTTP: $STATUS - $DATA_HORA\"}" \
"$WEBHOOK"` - Aqui, usamos novamente o curl, mas dessa vez para enviar uma mensagem para o Discord: -H: define o tipo de conteúdo (JSON). -X POST: envia os dados como um POST (envio de formulário). -d: os dados a serem enviados. No caso, um texto formatado com a variável de status e a hora. "$DISCORD_WEBHOOK": é o destino da mensagem.

>`else
echo "$DATA_HORA - [$SITE_URL] Está online! - Status: $STATUS" >> "$LOG_FILE"` - Se o status for 200, o site está online, e isso é salvo no arquivo de log também.

>`fi` - Fecha o bloco do if.

>`sleep 60` - Faz o script esperar 60 segundos antes de repetir tudo de novo. Assim, ele faz a checagem uma vez por minuto.

>`done` - Fecha o while, e o loop começa de novo.

---

## Etapa 4 - Teste e validação da solução:
    Após modificar o index.html e criar o script, chegou a hora de testar. abro a máquina virtual, abro o terminal, entro no modo root através do:
    >`sudo su`

    E para testar o webhoook, iremos parar o nginx:
    >`systemctl stop nginx`

    Agora vamos testar com o comando:
    >`bash verifica_siteprojeto.sh`

    E as mensagens começaram a chegar.
    ![print 7](/Prints/7.png)

    Caso queira voltar e só utilizar o comando:
    >`systemctl restart nginx`

    Após ver que as mensagens estão chegando, vamos conferir o log:
    ![print 8](/Prints/8.png)

    Assim temos as informações, data e horário.

---
## Etapa Final - Conclusão
    Após testar, alcançamos as metas do projeto e verificamos seu funcionamento. Vou deixar algumas referências utilizadas para concluir esse projeto.

    * Primeiros Passos no Linux - Conceitos e Principais Comandos - Udemy

    * Descubra como configurar o NGINX: AULA COMPLETA com dicas de segurança e performance - Youtube

    * Programação Shell Script - Automatizando Rotinas no Linux - Udemy


