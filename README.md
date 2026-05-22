# Pet CRUD - Java + H2 + Docker + Azure

## Descrição do Projeto

Este projeto consiste em uma API REST desenvolvida em Java com Spring Boot para gerenciamento de pets.  
A aplicação realiza operações CRUD utilizando banco de dados H2, containerização com Docker e execução em máquina virtual Azure Linux.

A solução foi desenvolvida utilizando:

- Java 17
- Spring Boot
- Spring Data JPA
- H2 Database
- Docker
- Azure Virtual Machine
- Postman
- H2 Console

---

# Benefícios para o Negócio

- Facilidade de implantação utilizando containers Docker
- Ambiente padronizado e portátil
- Redução de custos utilizando H2 Database
- Facilidade de escalabilidade em nuvem Azure
- Simplicidade para testes e desenvolvimento
- API REST pronta para integração com sistemas externos

---

# Desenho Macro da Arquitetura

```text
                +------------------+
                |     Postman      |
                +------------------+
                         |
                         v
                +------------------+
                |   Spring Boot    |
                |   java-h2-app    |
                +------------------+
                         |
                         v
                +------------------+
                |    H2 Database   |
                |     H2Sprint     |
                +------------------+
                         |
                         v
                +------------------+
                | Docker Network   |
                |  app-network     |
                +------------------+
                         |
                         v
                +------------------+
                | Azure Linux VM   |
                +------------------+
```
---

# Rotas de Api

---

# GET /api/pets

---

# GET /api/pets/{id}

---

# POST /api/pets

---

# PUT /api/pets/{id}

---

# DELETE /api/pets/{id}

---

# Instalação da Solução - How To

```bash
# Clonar os repositórios
cd ~

git clone https://github.com/carlosguedesneto/docker-entrypoint-initdb.d.git
git clone https://github.com/carlosguedesneto/java-devops.git

# =========================
# CONFIGURAÇÃO DO H2
# =========================

cd ~/docker-entrypoint-initdb.d

# Criar Dockerfile do H2
nano Dockerfile.h2
```

```dockerfile
FROM oscarfonts/h2

ENV H2_DATABASE=test
ENV H2_USER=carlos
ENV H2_PASSWORD=fiapcloud

EXPOSE 1521

COPY init.sql /docker-entrypoint-initdb.d/init.sql
```

```bash
# Build da imagem H2
docker build -f Dockerfile.h2 -t h2-sprint .

# Criar volume e rede Docker
docker volume create h2-data
docker network create app-network

# Executar container H2
docker run --name H2Sprint -d \
--network app-network \
-p 1521:1521 \
-v h2-data:/opt/h2-data \
h2-sprint

# Verificar logs do H2
docker logs H2Sprint

# =========================
# CONFIGURAÇÃO DA API JAVA
# =========================

cd ~/java-devops

# Copiar init.sql para resources
cp ~/docker-entrypoint-initdb.d/init.sql \
src/main/resources/data.sql

# Editar application.properties
nano src/main/resources/application.properties
```

```properties
server.port=8080

spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=carlos
spring.datasource.password=fiapcloud
spring.datasource.driver-class-name=org.h2.Driver

spring.jpa.hibernate.ddl-auto=create
spring.jpa.show-sql=true
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.properties.hibernate.format_sql=true

spring.sql.init.mode=always
spring.jpa.defer-datasource-initialization=true

spring.h2.console.enabled=true
spring.h2.console.settings.web-allow-others=true
spring.h2.console.path=/h2-console

logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

```bash
# Criar Dockerfile da API
nano Dockerfile.api
```

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS builder

WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre

RUN addgroup --system appuser && adduser --system --ingroup appuser appuser

WORKDIR /app

COPY --from=builder /app/target/*.jar app.jar

RUN chown appuser:appuser app.jar

USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

```bash
# Build da API Java
docker build --no-cache -f Dockerfile.api -t java-h2-app .

# Executar container da API
docker run --name java-h2-app -d \
--network app-network \
-p 8080:8080 \
-e SPRING_DATASOURCE_URL=jdbc:h2:mem:testdb \
-e SPRING_DATASOURCE_USERNAME=carlos \
-e SPRING_DATASOURCE_PASSWORD=fiapcloud \
java-h2-app

# =========================
# TESTES
# =========================

# Verificar containers
docker ps

# Verificar logs
docker logs java-h2-app

# Confirmar usuário sem root
docker exec java-h2-app whoami

# Resultado esperado:
# appuser

# Testar API localmente
curl http://localhost:8080/api/pets
```

```http
# POSTMAN

GET http://IP_DA_VM:8080/api/pets
```

```http
# H2 CONSOLE

http://IP_DA_VM:8080/h2-console
```

```text
JDBC URL: jdbc:h2:mem:testdb
User Name: carlos
Password: fiapcloud
```

```sql
SELECT * FROM PETS;
```
