# Stirling PDF no Render

Este repositorio sobe o [Stirling PDF](https://docs.stirlingpdf.com/) como um servico Docker no Render, pronto para uso via API.

Exemplo de rota:

`https://SEU-SERVICO.onrender.com/api/v1/convert/pdf/img`

## O que foi preparado

- `Dockerfile` usando a imagem oficial `stirlingtools/stirling-pdf:latest-ultra-lite`
- `render.yaml` com deploy via Docker e healthcheck para tentar rodar no plano free do Render
- `compose.yaml` para testar localmente
- Configuracao sem login (`SECURITY_ENABLELOGIN=false`) para facilitar uso via API
- Modo reduzido com `DISABLE_ADDITIONAL_FEATURES=true` e remocao do grupo `LibreOffice`

## Teste local

Suba localmente:

```bash
docker compose up --build
```

Acesse:

- UI: `http://localhost:8080`
- Swagger: `http://localhost:8080/swagger-ui.html`
- Healthcheck: `http://localhost:8080/api/v1/health`

## Subir no GitHub

Se esta pasta ainda nao estiver conectada ao repo:

```bash
git init
git branch -M main
git remote add origin https://github.com/enggavina/stirling_pdf.git
git add .
git commit -m "chore: setup stirling pdf for render"
git push -u origin main
```

Se o repositorio remoto ja existir com conteudo, faca antes um pull/rebase para nao sobrescrever historico.

## Subir no Render

### Opcao 1: Blueprint com `render.yaml`

1. No Render, clique em `New +`.
2. Escolha `Blueprint`.
3. Conecte o repositorio `enggavina/stirling_pdf`.
4. Confirme a criacao do servico.

O arquivo ja define:

- runtime Docker
- healthcheck em `/api/v1/health`
- `PORT` e `SERVER_PORT` em `10000`
- plano `free`
- modo ultra-lite

### Opcao 2: Web Service manual

Se preferir criar manualmente:

1. `New +` > `Web Service`
2. Selecione o repositorio
3. Em `Environment`, escolha `Docker`
4. Garanta que o `Dockerfile Path` seja `./Dockerfile`
5. Configure as variaveis:

```text
PORT=10000
SERVER_PORT=10000
SERVER_ADDRESS=0.0.0.0
SECURITY_ENABLELOGIN=false
LANGS=pt_BR
```

6. Configure `Health Check Path` como:

```text
/api/v1/health
```

## Observacoes importantes

- No Render Free, nao existe persistent disk. Toda configuracao local e efemera e pode ser perdida a cada restart ou redeploy.
- O Stirling expoe a documentacao local da API em `/swagger-ui.html`.
- Esta configuracao tenta reduzir uso de memoria com a imagem `ultra-lite` e removendo funcionalidades dependentes de LibreOffice.
- O endpoint `/api/v1/convert/pdf/img` recebe upload `multipart/form-data`. Voce pode validar o formato exato no Swagger depois do deploy.

## Exemplo de chamada

Exemplo com `curl` apos o deploy:

```bash
curl -X POST "https://SEU-SERVICO.onrender.com/api/v1/convert/pdf/img" \
  -F "fileInput=@arquivo.pdf" \
  -o saida.zip
```

Se quiser, o proximo passo pode ser eu tambem deixar um script de teste da API aqui no repositorio.
