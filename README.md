# Laboratório Docker — Da aplicação C# ao deploy versionado

Felipe Batalha Neves (RA 22509392) · Integração DevOps · Prof. Daniel Lim-Apo · CEUB

API ASP.NET Core (.NET 10) empacotada em imagem Docker multi-stage, executada em staging e production, versionada (1.0.0, 1.1.0 e 1.2.0), com falha controlada, rollback e Docker Compose.

## Construção
```bash
docker build -t devops-webapi:1.0.0 .
```

## Execução
```bash
docker run -d --name devops-api -p 8080:8080 devops-webapi:1.0.0
curl -f http://localhost:8080/health
```

## Staging
```bash
docker run -d --name devops-api-staging -p 8081:8080 -e APP_ENVIRONMENT=staging -e APP_VERSION=1.0.0 devops-webapi:1.0.0
```

## Production
```bash
docker run -d --name devops-api-prod -p 8082:8080 -e APP_ENVIRONMENT=production -e APP_VERSION=1.0.0 devops-webapi:1.0.0
```

## Health Check
```bash
curl -f http://localhost:8081/health
curl -f http://localhost:8082/health
```

## Nova versão
```bash
docker build -t devops-webapi:1.1.0 .   # adiciona o endpoint /version
docker rm -f devops-api-staging
docker run -d --name devops-api-staging -p 8081:8080 -e APP_ENVIRONMENT=staging -e APP_VERSION=1.1.0 devops-webapi:1.1.0
```

## Falha controlada
`devops-webapi:1.2.0`: o `/health` passa a responder HTTP 500. Em staging, `curl -f` falha e a versão **não deve ser promovida**. Para simular um incidente, ela foi colocada em production e o health check também falhou.

## Rollback
```bash
docker rm -f devops-api-prod
docker run -d --name devops-api-prod -p 8082:8080 -e APP_ENVIRONMENT=production -e APP_VERSION=1.1.0 devops-webapi:1.1.0
curl -f http://localhost:8082/health   # healthy, versão 1.1.0
```

## Docker Compose
```bash
docker compose up -d
docker compose ps
docker compose down
```

O `compose.yaml` também tem um **healthcheck** (desafio avançado). A imagem `aspnet:10.0` não traz `curl`, então o teste usa o `bash` da imagem, abrindo `/dev/tcp/127.0.0.1/8080` e conferindo se `/health` responde `200 OK`. Os dois serviços aparecem como `healthy` no `docker compose ps`.
