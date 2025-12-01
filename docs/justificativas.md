# Justificativas Técnicas e Decisões de Arquitetura

## Por que Microsserviços
- Permite evolução independente por domínio (users, products, orders, payments).
- Escala horizontal por serviço conforme demanda específica.
- Isola falhas: indisponibilidade em `payments` não impede criação de pedidos.

## Por que Kafka (vs. RabbitMQ)
- Throughput alto e partições garantem ordenação por partição e paralelismo controlado.
- Grupos de consumo simplificam escala de consumidores sem duplicar processamento.
- Log distribuído persistente facilita reprocessamento e auditoria de eventos.
- Remoção de RabbitMQ reduz complexidade operacional mantendo pilares solicitados.

## Por que Kong como API Gateway
- Roteamento declarativo (`kong.yml`) e plugins nativos (rate limiting, request-size, Prometheus).
- Ponto único de entrada para segurança, observabilidade e governança.
- `strip_path: true` remove o prefixo do serviço ao encaminhar, compatível com design dos endpoints internos.

## Por que Redis e Caching TTL
- Cache em leitura para endpoints idempotentes reduz latência e carga nos bancos.
- TTLs foram definidos conforme requisitos:
  - `users/:id` 1 dia
  - `products` 4 horas
  - `orders/:id` 30 dias
  - `payments/types` infinito
- Implementação com `ioredis` e middleware simples (`cache.js`) aplicado por rota.

## Por que Prometheus/Grafana
- Métricas padronizadas via `/metrics` com `prom-client` para todos os serviços.
- Observabilidade unificada; dashboards e alertas orientados a SLO.
- Plugin Prometheus no Kong permite visibilidade do gateway.

## Por que K6
- Testes de carga reprodutíveis com estágios; integração nativa com Grafana/InfluxDB (opcional).
- Scripts simples para validar latência e taxa de erro das rotas principais.

## Postgres vs MongoDB
- Postgres com Prisma para `users`, `products`, `payments`: consistência ACID, esquemas bem definidos.
- MongoDB com Mongoose para `orders`: flexibilidade de documento e evolução de payload de eventos.

## Notificação interna
- Requisito explícito: serviço `notification` é interno.
- Não exposto no Kong; consome `notifications-topic` e persiste notificações.
- Evita superfície pública desnecessária e potenciais abusos.

## Proteções no gateway
- Rate limiting de 10 req/min global e limite de 200KB por request aplicados no Kong.
- Centralização em um boundary único facilita governança e auditoria.

## Seeds e Inicialização
- `prisma db push` e `seed.js` nos serviços SQL; `orders/seed.js` para Mongo.
- `sleep` nas `command` do Docker para garantir que bancos/Kafka estejam prontos.
- Objetivo: fazer tudo funcionar com um único `docker-compose up -d --build`.

## Operação e Deploy
- `docker-compose.yml` orquestra bancos, serviços, gateway e UIs auxiliares.
- `DB-less` no Kong via arquivo declarativo (`kong.yml`) para previsibilidade e versionamento.

## Padrões de Código
- Express e JSON simples para APIs.
- Prisma e Mongoose para persistência.
- `prom-client` para instrumentação automática.
- `kafkajs` para produtores/consumidores.

## Perguntas Prováveis e Respostas
- "Por que consolidar em Kafka?"
  - Simplifica topologia, melhora throughput e facilita reprocessamento; atende ao pilar de mensageria pedido.
- "Como garantem que as rotas são cacheadas corretamente?"
  - TTL aplicado por rota via middleware `cache(seconds)` com Redis; chaves geradas por `originalUrl`.
- "Como protegem contra DDoS e payloads grandes?"
  - Plugins globais no Kong: rate-limiting 10/min e request-size-limiting 200KB.
- "Como validam observabilidade?"
  - `/metrics` em todos os serviços e plugin Prometheus; Prometheus coleta; Grafana visualiza.
- "Como escalar consumidores?"
  - Aumentando réplicas de `payments`/`notification` e mantendo `groupId` adequados no Kafka.
- "Por que `strip_path: true`?"
  - Para alinhar caminhos externos com internos e simplificar mapeamento de rotas.
- "O que acontece se `users` estiver fora?"
  - `orders` publica no Kafka; `payments` marca pagamento `PENDING` e segue fluxo quando `users` voltar.

## Trade-offs Considerados
- Remover RabbitMQ reduz features de roteamento por exchange, mas Kafka cobre ordenação e throughput.
- Múltiplos bancos aumentam complexidade operacional; ganhos por adequação ao domínio.
- Kong em modo declarativo exige disciplina de versionamento; ganhos em previsibilidade e reprodutibilidade.