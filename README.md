# Plataforma de E-commerce — Arquitetura de Microsserviços

Este projeto implementa uma plataforma de e-commerce baseada em microsserviços com gateway de API, mensageria assíncrona e observabilidade pronta para produção. A solução foi redesenhada integralmente para apresentar escrita original, padronização técnica e boas práticas modernas.

## Pilares Técnicos

- Microsserviços: `users`, `products`, `orders`, `payments`, `notification`
- Mensageria: `Kafka` (tópicos `orders-topic` e `notifications-topic`)
- API Gateway: `Kong` (declaração em `kong.yml`)
- Observabilidade: `Prometheus` + `Grafana`
- Testes de carga: `K6`
- Infraestrutura: `Docker Compose`
- Bancos: `Postgres` (users, products, payments) + `MongoDB` (orders)

## Como executar

1. Pré‑requisitos: `Docker` e `Docker Compose` instalados.
2. Subir a stack:
   - `docker-compose up -d --build`
3. Acessos úteis:
   - `Kong Proxy`: `http://localhost:8000`
   - `Kong Admin`: `http://localhost:8001`
   - `Kafka UI`: `http://localhost:8080`
   - `Prometheus`: `http://localhost:9090`
   - `Grafana`: `http://localhost:3001`

Os serviços são expostos internamente pelo `Kong`. As rotas estão declaradas em `kong.yml`.

## Serviços e Endpoints

Resumo dos principais endpoints. A lista completa com exemplos está em `docs/endpoints.md`.

- `users` (Postgres):
  - `POST /register` — cria um usuário
  - `GET /users` — lista usuários
  - `GET /users/:id` — busca usuário por id
  - `DELETE /users/:id` — remove usuário
  - `GET /metrics` — métricas Prometheus

- `products` (Postgres):
  - `GET /products` — lista produtos
  - `GET /products/:id` — detalhe
  - `POST /products` — cria produto
  - `PATCH /products/:id/stock` — baixa estoque
  - `DELETE /products/:id` — remove
  - `GET /metrics` — métricas Prometheus

- `orders` (MongoDB + Kafka):
  - `POST /orders` — cria pedido e publica no `orders-topic`
  - `GET /orders` — lista pedidos
  - `GET /orders/:id` — detalhe
  - `GET /metrics` — métricas Prometheus

- `payments` (Postgres + Kafka):
  - Consumer do `orders-topic` e `producer` do `notifications-topic`
  - `GET /payments` — lista pagamentos
  - `GET /payments/:id` — detalhe
  - `GET /payments/types` — tipos suportados
  - `GET /metrics` — métricas Prometheus

- `notification` (SQLite + Kafka):
  - Consumer do `notifications-topic`
  - `POST /notify` — cria notificação direta
  - `GET /notifications` — lista notificações
  - `GET /metrics` — métricas Prometheus

## Observabilidade

- `prometheus.yml` inclui `scrape_configs` para todos os serviços e para o endpoint `/metrics` do Kong (plugin Prometheus habilitado).
- `Grafana` pode se conectar ao `Prometheus` (URL `http://prometheus:9090`).

## Testes de carga (K6)

- Scripts em `k6-scripts/`:
  - `order-load-test.js` — leitura de pedidos
  - `payment-load-test.js` — leitura de pagamentos

Execute dentro da rede Docker, por exemplo usando a imagem oficial do K6:

```
docker run --rm --network=g2_app-net -v $PWD/k6-scripts:/scripts grafana/k6 run /scripts/order-load-test.js
```

## Segurança e Boas Práticas

- Segredos via variáveis de ambiente; nunca versionados.
- Limites de taxa e de tamanho de requisição habilitados no Kong.
- Cache controlado via Redis para respostas idempotentes.
- Logs claros para integração e diagnóstico em mensageria.
- Métricas expostas via `/metrics` em todos os serviços.

## Arquitetura

O documento `docs/architecture.md` detalha fluxos, tópicos Kafka e o papel de cada serviço, incluindo diagramas atualizados.

## Licença

Projeto para fins acadêmicos e de demonstração de arquitetura distribuída.
