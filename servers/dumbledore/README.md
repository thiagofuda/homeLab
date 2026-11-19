# Deploy do InfraLab

InfraLab e o nome publico da aplicacao. O codigo permanece no repositorio
tecnico `NetworkLab`, e os paths e nomes de imagem/servico existentes nao mudam.

## Operacao

A API, o Nginx e o Certbot rodam em containers Docker Compose. O Nginx publica
as portas 80 e 443; a API fica acessivel somente pela rede do Compose. Um timer
systemd no host verifica a renovacao do certificado duas vezes ao dia e recarrega
o Nginx quando ela tem sucesso.

O Compose espera os diretorios `NetworkLab` e `homeLab` lado a lado em
`/opt/networklab` e `/opt/homeLab`.

No bootstrap HTTP, somente o desafio ACME fica disponível. As demais rotas
retornam 503 até que a configuração HTTPS de produção seja ativada.

Em HTTPS, os endpoints Terraform AWS e OCI compartilham o limite por IP de
10 requisições por minuto (burst 3), com corpo de até 1 MiB. O OCI IAM com
Gemini tem limite próprio de 3 requisições por minuto por IP (burst 1), com
corpo de até 32 KiB. O Nginx retorna 413 ou 429 em JSON quando rejeita a
requisição. A API deve continuar acessível apenas pela rede do Compose e
o Nginx deve sobrescrever o IP encaminhado com o endereço que ele observou.

## HTTPS para infralab.fuda.com.br

O registro DNS do subdominio deve resolver para a VPS antes da emissao.
Execute os comandos em `/opt/networklab` com a API ativa e saudavel.
Guarde o commit anterior do homeLab e o SHA da imagem da API para recuperar
o ambiente se a troca falhar.

Para emitir o certificado inicial, recrie o Nginx em modo bootstrap. Nesse
modo, o caminho ACME responde em HTTP e a aplicacao retorna 503.

```bash
docker compose -f compose.prod.yaml -f compose.bootstrap.yaml \
  up -d --no-deps --force-recreate nginx
docker compose -f compose.prod.yaml --profile certbot run --rm certbot certonly \
  --webroot -w /var/www/certbot --cert-name infralab.fuda.com.br \
  -d infralab.fuda.com.br --non-interactive --agree-tos --email "$ACME_EMAIL"
```

Defina `ACME_EMAIL` com seu email de contato antes da emissao. Confirme com
`certbot certificates` pelo mesmo servico Compose que o certificado e a chave
estao em `/etc/letsencrypt/live/infralab.fuda.com.br/`. Valide a configuracao
HTTPS antes de recriar o proxy sem o override.

```bash
docker compose -f compose.prod.yaml run --rm --no-deps nginx nginx -t
docker compose -f compose.prod.yaml up -d --no-deps --force-recreate nginx
curl -fsS https://infralab.fuda.com.br/health
```

O timer `networklab-certbot-renew.timer` usa os registros do Certbot para renovar
e recarregar o Nginx. Verifique o desafio em HTTPS de producao com
`renew --dry-run --cert-name infralab.fuda.com.br` pelo servico Certbot. Depois
do smoke test e de confirmar que nenhum outro servico usa o certificado
anterior, retire o registro antigo com `certbot delete --cert-name NOME_ANTERIOR`
pelo mesmo servico. Isso evita tentativas de renovacao do hostname retirado.

Se a ativacao falhar, restaure a configuracao anterior do homeLab e recrie o
Nginx com o Compose de producao enquanto o certificado anterior ainda existir.
Preserve os volumes do Certbot durante toda a troca.

## GitHub Actions

O workflow do repositorio NetworkLab:

- Executa lint, testes, build da imagem e health check do container em pushes e
  pull requests.
- Publica no GHCR uma imagem identificada pelo commit em pushes para `sprint_4`
  e `main`.
- Em `main`, tambem publica a tag `latest` e faz o deploy da imagem daquele
  commit.

O pacote GHCR precisa ser publico para que o servidor baixe a imagem sem
credenciais do registry. O GitHub Actions usa o environment `production`, que
deve permitir deploy somente a partir de `main`, com estas configuracoes:

- Variables `DEPLOY_HOST`, `DEPLOY_USER` (valor `deploy`) e
  `DEPLOY_KNOWN_HOSTS`.
- Secret `DEPLOY_SSH_PRIVATE_KEY` com uma chave SSH dedicada ao deploy.

O usuario de deploy precisa de acesso ao Docker. Pertencer ao grupo `docker`
equivale, na pratica, a ter privilegios de root no host. Fixe em
`DEPLOY_KNOWN_HOSTS` a chave do servidor verificada por um canal confiavel; nao
use `ssh-keyscan` dinamico no workflow. Nunca armazene chaves privadas ou senhas
neste repositorio.

## Deploy manual e rollback

Use o SHA do commit como tag da imagem:

```bash
cd /opt/networklab
IMAGE_TAG=SHA_DO_COMMIT docker compose -f compose.prod.yaml pull api
IMAGE_TAG=SHA_DO_COMMIT docker compose -f compose.prod.yaml up -d --no-deps --wait api
docker compose -f compose.prod.yaml restart nginx
curl -fsS https://infralab.fuda.com.br/health
```

Antes de atualizar, o workflow salva a imagem atual com a tag local `rollback`.
Para restaura-la manualmente:

```bash
cd /opt/networklab
IMAGE_TAG=rollback docker compose -f compose.prod.yaml up -d --no-deps --wait api
docker compose -f compose.prod.yaml restart nginx
curl -fsS https://infralab.fuda.com.br/health
```

Substitua `SHA_DO_COMMIT` pelo SHA correto. O
rollback cobre somente a imagem da API. Preserve os volumes `letsencrypt` e
`certbot_webroot`; nao use `docker compose down --volumes` em deploys ou rollback.