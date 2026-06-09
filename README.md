# Nexus Verde

## Descrição da Solução

O Nexus Verde é uma solução de monitoramento ambiental que utiliza Inteligência Artificial e minissatélites para identificar possíveis desmatamentos e queimadas em áreas florestais.

A solução realiza o monitoramento contínuo de regiões estratégicas através de imagens capturadas por satélites. As imagens são analisadas automaticamente por algoritmos de IA, capazes de identificar alterações na vegetação e gerar alertas para órgãos responsáveis pela preservação ambiental.

O sistema foi desenvolvido utilizando Java Spring Boot, banco de dados H2 e conteinerização com Docker, sendo implantado em uma Máquina Virtual Linux na Microsoft Azure.

---

# Benefícios para o Negócio

* Monitoramento contínuo de áreas florestais.
* Identificação rápida de queimadas e desmatamentos.
* Redução do tempo de resposta para eventos ambientais.
* Centralização dos alertas em uma única plataforma.
* Escalabilidade através de containers Docker.
* Facilidade de implantação em ambientes de nuvem.

---

# Arquitetura Macro

Azure Subscription
→ Resource Group (rg-nexusverde)
→ Virtual Network (vnet-nexusverde)
→ Network Security Group (Portas 22, 8080 e 1521)
→ Azure Linux VM (Ubuntu Server 22.04 LTS)

Dentro da VM:

Docker Engine
├── Container Java Spring Boot (nexusverde-566022)
└── Container H2 Database (h2-566022)

Fluxo:

Usuário/Postman
→ API Spring Boot (porta 8080)
→ Banco H2 (porta 1521)

---

# Endpoints da API

## Monitoramentos

GET /api/monitoramentos

GET /api/monitoramentos/{id}

POST /api/monitoramentos

PUT /api/monitoramentos/{id}

DELETE /api/monitoramentos/{id}

## Alertas

GET /api/alertas

GET /api/alertas/{id}

POST /api/alertas

PUT /api/alertas/{id}

DELETE /api/alertas/{id}

---

# How To - Instalação Completa

## 1. Clonar os repositórios

```bash
cd ~

git clone https://github.com/carlosguedesneto/docker-entrypoint-initdb.d.git

git clone https://github.com/carlosguedesneto/java-devops.git
```

## 2. Criar rede Docker

```bash
docker network create nexusverde-network
```

## 3. Criar volume persistente

```bash
docker volume create h2-566022-data
```

## 4. Construir imagem do Banco H2

```bash
cd ~/docker-entrypoint-initdb.d

docker build -f Dockerfile.h2 -t h2-nexusverde .
```

## 5. Executar Container H2

```bash
docker run --name h2-566022 -d \
--network nexusverde-network \
-p 1521:1521 \
-v h2-566022-data:/opt/h2-data \
h2-nexusverde
```

## 6. Construir imagem da Aplicação Java

```bash
cd ~/java-devops

docker build --no-cache -f Dockerfile.api -t nexusverde-api .
```

## 7. Executar Container da Aplicação

```bash
docker run --name nexusverde-566022 -d \
--network nexusverde-network \
-p 8080:8080 \
nexusverde-api
```

## 8. Verificar Containers

```bash
docker ps
```

## 9. Visualizar Logs

### Banco H2

```bash
docker logs h2-566022
```

### Aplicação Java

```bash
docker logs nexusverde-566022
```

## 10. Verificar usuário e diretórios

### Banco

```bash
docker exec -it h2-566022 pwd

docker exec -it h2-566022 ls -l

docker exec -it h2-566022 whoami
```

### Aplicação

```bash
docker exec -it nexusverde-566022 pwd

docker exec -it nexusverde-566022 ls -l

docker exec -it nexusverde-566022 whoami
```

## 11. Testar API Externamente

### Listar Monitoramentos

```http
GET http://IP_PUBLICO_DA_VM:8080/api/monitoramentos
```

### Listar Alertas

```http
GET http://IP_PUBLICO_DA_VM:8080/api/alertas
```

## 12. Conectar ao Banco pelo DBeaver

Driver:

```text
H2 Server
```

URL JDBC:

```text
jdbc:h2:tcp://IP_PUBLICO_DA_VM:1521/nexusverde
```

Usuário:

```text
carlos
```

Senha:

```text
fiapcloud
```

## 13. Executar Consultas SQL

```sql
SELECT * FROM MONITORAMENTOS;
```

```sql
SELECT * FROM ALERTAS;
```

---

# Tecnologias Utilizadas

* Java 17
* Spring Boot 3
* Spring Data JPA
* Maven
* H2 Database
* Docker
* Microsoft Azure
* DBeaver
* Postman

