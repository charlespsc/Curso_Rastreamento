# 📊 Curso de Observabilidade — Tracing com OpenTelemetry e Jaeger

Projeto prático de **observabilidade de uma API REST**, utilizando **tracing distribuído, métricas e logs centralizados**.

O ambiente reúne uma aplicação Java com Spring Boot, PostgreSQL, Redis, Nginx, OpenTelemetry, Jaeger, Prometheus, Grafana e Loki, orquestrados com Docker Compose.

> Este projeto foi desenvolvido como laboratório de estudos em **Observabilidade, SRE, monitoramento e rastreamento distribuído**, com foco em entender como uma requisição percorre diferentes componentes de uma arquitetura e como seus sinais podem ser coletados e analisados.

---

# 🖥️ Ambiente

![Ambiente](https://github.com/charlespsc/Curso_Rastreamento/assets/81668900/26480a51-e24f-494c-a5c4-ba50d7656963)

---

## 🎯 Objetivo

O principal objetivo do projeto é construir um ambiente no qual seja possível observar uma aplicação distribuída utilizando os três principais sinais de observabilidade:

- 🔎 **Traces** — rastreamento das requisições;
- 📈 **Metrics** — métricas de infraestrutura e aplicação;
- 📝 **Logs** — registros estruturados da aplicação.

A ideia é acompanhar uma requisição desde a entrada pelo **Nginx**, passando pela API Spring Boot e seus componentes de infraestrutura, até a coleta e visualização dos dados de observabilidade.

---

# 🧩 Arquitetura

O ambiente é composto pelos seguintes serviços:

```text
                         ┌─────────────────────┐
                         │   Cliente / Gerador  │
                         │     de tráfego      │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │       Nginx         │
                         │   Reverse Proxy     │
                         │   + OpenTelemetry   │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │     Spring Boot     │
                         │      API Cursos     │
                         │       :8080         │
                         └──────┬───────┬──────┘
                                │       │
                    ┌───────────┘       └────────────┐
                    ▼                                ▼
             ┌──────────────┐                 ┌──────────────┐
             │  PostgreSQL  │                 │    Redis     │
             │    :5432     │                 │    :6379     │
             └──────────────┘                 └──────────────┘

                         Observabilidade
                                │
             ┌──────────────────┼──────────────────┐
             │                  │                  │
             ▼                  ▼                  ▼
      ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
      │ OpenTelemetry│    │ Prometheus  │    │    Loki     │
      │   Collector  │    │    :9090    │    │    :3100    │
      └──────┬──────┘    └──────┬──────┘    └─────────────┘
             │                   │
             ▼                   ▼
      ┌─────────────┐     ┌─────────────┐
      │    Jaeger   │     │   Grafana   │
      │   :16686    │     │    :3000    │
      └─────────────┘     └─────────────┘
```

---

# 🛠️ Tecnologias

## Backend

- ☕ Java 11
- 🌱 Spring Boot 2.6.7
- Spring Web
- Spring Data JPA
- Spring Validation
- Spring Actuator

## Banco e cache

- 🐘 PostgreSQL
- 🔴 Redis

## Observabilidade

- 🔭 OpenTelemetry
- 🔎 OpenTelemetry Collector
- 🧭 Jaeger
- 📈 Prometheus
- 📊 Grafana
- 📝 Loki
- Logback
- Micrometer

## Infraestrutura

- 🐳 Docker
- Docker Compose
- Nginx

---

# 🔎 Tracing distribuído

O projeto utiliza **OpenTelemetry** para instrumentação e coleta de traces.

A aplicação Spring Boot é executada com o **OpenTelemetry Java Agent**:

```text
-javaagent:opentelemetry-javaagent.jar
```

As configurações do ambiente definem o serviço como:

```text
OTEL_SERVICE_NAME=api-cursos
```

e enviam os traces para o OpenTelemetry Collector:

```text
http://collector-api-cursos:4318/
```

O Nginx também possui instrumentação OpenTelemetry, permitindo observar a entrada das requisições no proxy.

O Collector recebe os traces via OTLP e os encaminha para o Jaeger.

```text
Nginx
  │
  │ Trace
  ▼
OpenTelemetry Collector
  │
  ▼
Jaeger
```

---

# 🧭 Jaeger

O **Jaeger** é utilizado para visualizar os traces distribuídos.

A interface web é disponibilizada na porta:

```text
16686
```

Após iniciar o ambiente:

```text
http://localhost:16686
```

No Jaeger é possível analisar:

- serviços;
- operações;
- duração das requisições;
- spans;
- sequência de chamadas;
- relação entre componentes;
- tempo gasto em cada etapa.

---

# 📈 Prometheus

A aplicação Spring Boot utiliza o **Spring Boot Actuator** e o **Micrometer** para disponibilizar métricas.

O endpoint utilizado pelo Prometheus é:

```text
/actuator/prometheus
```

O Nginx expõe esse endpoint através de:

```text
/metrics
```

O Prometheus realiza a coleta periódica das métricas.

A interface web fica disponível em:

```text
http://localhost:9090
```

O projeto utiliza um intervalo de coleta de:

```text
15 segundos
```

---

# 📊 Grafana

O **Grafana** é utilizado como plataforma de visualização das métricas e dos dados de observabilidade.

A interface fica disponível em:

```text
http://localhost:3000
```

O ambiente também possui integração com Loki para consulta dos logs.

---

# 📝 Logs com Loki

A aplicação utiliza **Logback** para gerar logs estruturados.

Além da saída no console, os logs são enviados para o **Loki** utilizando o appender:

```text
loki4j
```

O Loki recebe os logs através de:

```text
http://loki-api-cursos:3100/loki/api/v1/push
```

Os logs carregam informações como:

- aplicação;
- hostname;
- thread;
- nível do log;
- classe;
- método;
- timestamp;
- exceções.

Fluxo simplificado:

```text
Spring Boot
     │
     ▼
  Logback
     │
     ▼
   Loki
     │
     ▼
  Grafana
```

---

# 🧪 API de Cursos

A aplicação principal é uma API REST para gerenciamento de cursos.

A API possui operações:

| Método | Endpoint | Operação |
|---|---|---|
| `GET` | `/cursos` | Lista cursos |
| `GET` | `/cursos/{id}` | Consulta curso |
| `POST` | `/cursos` | Cadastra curso |
| `PUT` | `/cursos/{id}` | Atualiza curso |
| `DELETE` | `/cursos/{id}` | Exclui curso |

A API também possui paginação através do `Pageable`.

Exemplo:

```http
GET /cursos?page=0&size=10&sort=dataInscricao,DESC
```

---

# 🗃️ Modelo de dados

A entidade principal é `CursoModel`.

```text
Curso
├── id                  UUID
├── numeroMatricula     String
├── numeroCurso         String
├── nomeCurso           String
├── categoriaCurso      String
├── preRequisito        String
├── nomeProfessor       String
├── periodoCurso        String
└── dataInscricao       LocalDateTime
```

O identificador utiliza UUID e os campos `numeroMatricula` e `numeroCurso` possuem restrição de unicidade.

---

# ⚡ Cache com Redis

A API utiliza Redis para armazenamento em cache.

São utilizados dois caches:

```text
listaDeCursos
nomeCurso
```

As consultas de listagem e busca por ID utilizam cache, enquanto operações de alteração removem as entradas relacionadas para evitar dados desatualizados.

Configuração utilizada:

```text
spring.cache.type=redis
```

O Redis é executado no container:

```text
cache-api-cursos
```

---

# 🌐 Nginx

O Nginx atua como **Reverse Proxy** entre os clientes e a API.

```text
Cliente
   │
   ▼
Nginx :80
   │
   ▼
Spring Boot :8080
```

Além do encaminhamento das requisições, o Nginx também possui instrumentação OpenTelemetry para geração de traces.

Endpoints expostos pelo proxy:

```text
/cursos
/info
/metrics
/health
```

---

# 🤖 Gerador de tráfego

O projeto possui um cliente baseado em shell script responsável por gerar requisições automaticamente.

O script executa diferentes operações sobre a API, incluindo:

- listagem de cursos;
- consultas paginadas;
- criação de cursos;
- consulta por UUID;
- atualização;
- exclusão.

O objetivo é gerar tráfego contínuo para que seja possível observar os sinais de telemetria no ambiente.

Fluxo:

```text
Gerador de tráfego
       │
       ▼
     Nginx
       │
       ▼
      API
       │
       ├── Traces ──► OpenTelemetry ──► Jaeger
       │
       ├── Metrics ─► Prometheus ─────► Grafana
       │
       └── Logs ────► Loki ───────────► Grafana
```

---

# 🐳 Executando o ambiente

## Pré-requisitos

É necessário possuir:

- Docker;
- Docker Compose.

Verifique:

```bash
docker --version
docker compose version
```

---

## 1. Clonar o repositório

```bash
git clone https://github.com/charlespsc/Curso_Rastreamento.git
```

Entrar no diretório:

```bash
cd Curso_Rastreamento
```

---

## 2. Subir todo o ambiente

```bash
docker compose up -d
```

Para acompanhar os containers:

```bash
docker compose ps
```

Para visualizar os logs:

```bash
docker compose logs -f
```

---

## 3. Verificar a API

A API fica atrás do Nginx na porta `80`.

Teste:

```bash
curl http://localhost/health
```

Também é possível consultar:

```bash
curl http://localhost/info
```

E:

```bash
curl http://localhost/metrics
```

---

# 🔗 Interfaces do ambiente

Depois que os containers estiverem em execução:

| Serviço | Endereço | Função |
|---|---|---|
| Nginx / API | `http://localhost` | Entrada da aplicação |
| Grafana | `http://localhost:3000` | Visualização |
| Prometheus | `http://localhost:9090` | Métricas |
| Jaeger | `http://localhost:16686` | Traces |

---

# 🔬 O que este laboratório permite observar

Este ambiente foi construído para permitir uma experiência prática com situações comuns de observabilidade.

### Tracing

É possível acompanhar uma requisição através de diferentes componentes:

```text
Cliente
   ↓
Nginx
   ↓
Spring Boot
   ↓
PostgreSQL / Redis
```

### Métricas

É possível coletar informações expostas pelo Spring Boot Actuator e Micrometer através do Prometheus.

### Logs

Os logs estruturados da aplicação são encaminhados para o Loki e podem ser consultados através das ferramentas de visualização.

---

# 🧠 Conceitos praticados

Este projeto permite praticar conceitos de:

- Observabilidade;
- SRE;
- APM;
- Distributed Tracing;
- OpenTelemetry;
- OpenTelemetry Collector;
- Jaeger;
- Prometheus;
- Grafana;
- Loki;
- Logs estruturados;
- Métricas;
- Spans;
- Instrumentação automática;
- Reverse Proxy;
- Docker;
- Docker Compose;
- Spring Boot Actuator;
- Micrometer;
- Redis Cache;
- PostgreSQL;
- APIs REST.

---

# 📚 Contexto de estudo

Este projeto foi utilizado como laboratório para estudar **rastreamento distribuído e observabilidade**, conectando desenvolvimento de aplicações com infraestrutura e monitoramento.

A proposta é sair de uma visão em que a aplicação é observada apenas pelos seus logs e evoluir para uma visão baseada em múltiplos sinais:

```text
             OBSERVABILIDADE
                    │
       ┌────────────┼────────────┐
       │            │            │
       ▼            ▼            ▼
    TRACES       METRICS       LOGS
       │            │            │
       ▼            ▼            ▼
    Jaeger      Prometheus      Loki
                    │
                    ▼
                 Grafana
```

---

# 🚧 Possíveis evoluções

Algumas melhorias que podem ser implementadas futuramente:

- [ ] Atualizar versões das imagens e dependências para versões atuais;
- [ ] Remover arquivos temporários e artefatos de build do repositório;
- [ ] Externalizar credenciais e configurações sensíveis para variáveis de ambiente;
- [ ] Criar `healthchecks` para todos os componentes necessários;
- [ ] Documentar os dashboards do Grafana;
- [ ] Criar dashboards específicos para RED Metrics;
- [ ] Adicionar alertas no Prometheus/Grafana;
- [ ] Melhorar a correlação entre logs e trace IDs;
- [ ] Adicionar testes automatizados mais abrangentes;
- [ ] Documentar cenários de troubleshooting;
- [ ] Automatizar o ambiente com CI/CD.

---

# ⚠️ Observação sobre o projeto

Este repositório é um **laboratório de estudos**, e não pretende representar uma arquitetura pronta para produção.

Alguns componentes utilizam versões fixadas ou imagens `latest`, existem artefatos gerados no diretório do projeto e algumas configurações podem ser aprimoradas para um ambiente produtivo.

Esses pontos fazem parte justamente das possibilidades de evolução do laboratório.

---

## 👨‍💻 Autor

**Charles Pereira**

Tecnologia • Desenvolvimento • Redes • Infraestrutura • Cloud • Observabilidade • IoT • Educação Tecnológica

---

⭐ Se este projeto foi útil para você, considere deixar uma estrela no repositório.
