# Servidor Dumbledore

## NetworkLab

O Nginx e o Certbot rodam em containers Docker. O Nginx recebe as conexoes
publicas nas portas 80 e 443 e encaminha as requisicoes para a API pela rede
interna do Compose. A API nao publica portas no host.

Os arquivos em `nginx/` sao a fonte das configuracoes do virtual host. Um timer
systemd do host executa o Certbot container duas vezes ao dia; o Nginx so e
recarregado depois que o comando de renovacao termina com sucesso. O Certbot nao
recebe acesso ao socket do Docker.

### Instalacao inicial

1. Mantenha `NetworkLab` e `homeLab` como diretorios irmaos no servidor:

   ```bash
   /opt/networklab
   /opt/homeLab
   ```

2. Com Caddy parado e os arquivos atualizados, suba a API e o Nginx com a
   configuracao temporaria HTTP:

   ```bash
   cd /opt/networklab
   docker compose -f compose.prod.yaml -f compose.bootstrap.yaml up -d --build --remove-orphans
   ```

3. Com o DNS apontando para a VM e as portas 80 e 443 liberadas, emita o
    certificado. Nesta instalacao, a conta foi registrada sem e-mail:

   ```bash
   docker compose --profile certbot -f compose.prod.yaml run --rm certbot \
     certonly --webroot -w /var/www/certbot -d xdb.com.br \
       --agree-tos --register-unsafely-without-email --non-interactive
   ```

    Sem e-mail, a renovacao automatica continua ativa, mas nao ha avisos de
    expiracao enviados pelo Let's Encrypt.

4. Recrie o Nginx com a configuracao HTTPS definitiva:

   ```bash
   docker compose -f compose.prod.yaml up -d --force-recreate nginx
   ```

5. Instale e habilite o timer de renovacao no host:

   ```bash
   sudo install -m 644 /opt/homeLab/servers/dumbledore/systemd/networklab-certbot-renew.service /etc/systemd/system/
   sudo install -m 644 /opt/homeLab/servers/dumbledore/systemd/networklab-certbot-renew.timer /etc/systemd/system/
   sudo systemctl daemon-reload
   sudo systemctl enable --now networklab-certbot-renew.timer
   cd /opt/networklab
   docker compose --profile certbot -f compose.prod.yaml run --rm certbot renew --dry-run
   ```

O timer verifica duas vezes por dia. O Certbot renova somente quando necessario;
depois, o systemd recarrega o Nginx container para usar o certificado atualizado.