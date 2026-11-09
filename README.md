# TP Final — Docker Compose

**Arquitetura de Serviços em Nuvem** · Sistemas de Informação · Unisanta

| Aluno | RA |
| --- | --- |
| Gabriel Vilela Carvalho de Souza | 240209 |
| Bianca Mendes | 240286 |

## Objetivo

Criar um ambiente com dois containers Docker orquestrados pelo Docker Compose:

| Serviço | Imagem | Porta no PC | Porta no container | Descrição |
| --- | --- | --- | --- | --- |
| `site` | `casamento-nginx:1.0` (customizada) | **8080** | 80 | Site estático servido pelo nginx: a página de entrada do nosso casamento, com contagem regressiva |
| `n8n` | `n8nio/n8n` (oficial) | **8081** | 5678 | Plataforma de automação de workflows, com dados persistidos em volume |

## Estrutura

```
tp-final-docker/
├── site/
│   ├── Dockerfile        # receita da imagem customizada
│   └── index.html        # página que substitui o "Welcome to nginx"
├── docker-compose.yml    # descreve o sistema com os 2 serviços
└── README.md
```

## Dockerfile

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

Parte da imagem oficial do nginx (versão Alpine, mais leve) e substitui a página padrão pelo nosso `index.html`. Assim, o site fica embutido na imagem.

## Como executar

Pré-requisito: Docker Desktop rodando.

```bash
# subir tudo (builda a imagem do site e baixa o n8n)
docker compose up -d

# acessar
#   http://localhost:8080  -> site
#   http://localhost:8081  -> n8n

# ver status, logs e entrar no container
docker compose ps
docker compose logs site
docker compose logs -f n8n
docker exec -it site-casamento sh

# após alterar o index.html, rebuildar
docker compose up -d --build

# derrubar o ambiente (o volume do n8n é mantido)
docker compose down
```

## Conceitos aplicados

- **Imagem x container:** a imagem é a receita congelada; o container é a imagem em execução.
- **Dockerfile x Compose:** o Dockerfile descreve *uma imagem*; o Compose descreve *o sistema*, ou seja, quais containers rodam juntos, com quais portas, variáveis, volumes e rede.
- **Mapeamento de portas (`PC:container`):** os dois serviços usam portas internas diferentes (80 e 5678) e são expostos no PC em 8080 e 8081.
- **Rede:** o Compose cria a rede `tp-final-docker_default`, na qual os containers se enxergam pelo nome do serviço.
- **Volume:** `n8n_data` guarda os dados do n8n fora do container, então eles sobrevivem ao `docker compose down`.
- **Restart policy:** `unless-stopped` religa os containers automaticamente se caírem.
