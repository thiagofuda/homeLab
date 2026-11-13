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
curl -fsS https://seu-dominio.example/health
```

Antes de atualizar, o workflow salva a imagem atual com a tag local `rollback`.
Para restaura-la manualmente:

```bash
cd /opt/networklab
IMAGE_TAG=rollback docker compose -f compose.prod.yaml up -d --no-deps --wait api
docker compose -f compose.prod.yaml restart nginx
curl -fsS https://seu-dominio.example/health
```

Substitua `SHA_DO_COMMIT` e `seu-dominio.example` pelos valores corretos. O
rollback cobre somente a imagem da API. Preserve os volumes `letsencrypt` e
`certbot_webroot`; nao use `docker compose down --volumes` em deploys ou rollback.