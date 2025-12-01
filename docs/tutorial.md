# Tutorial de Execução e Validação

## Pré‑requisitos
- Docker e Docker Compose instalados
- Portas livres: 8000, 8001, 8443, 6379, 8082, 8080, 9090, 3001

## Subir o ambiente
```
docker-compose up -d --build
```

## Interfaces e Acessos
- Kong Proxy: `http://localhost:8000`
- Kong Admin: `http://localhost:8001`
- Kafka UI: `http://localhost:8080`
- Redis Commander: `http://localhost:8082`
- Prometheus: `http://localhost:9090`
- Grafana: `http://localhost:3001`
- Konga (UI para Kong): `http://localhost:1337`

### Logins/Configurações
- Grafana: usuário `admin`, senha `admin` (mudança exigida no primeiro login)
- Konga: criar uma conta na primeira execução; depois, configurar a conexão para `http://kong:8001`
- Kafka UI, Prometheus, Redis Commander e Kong Admin: sem autenticação por padrão (uso controlado em ambiente local)

## Verificar serviços
- Health básico via métricas:
```
curl http://localhost:8000/users/metrics
curl http://localhost:8000/products/metrics
curl http://localhost:8000/orders/metrics
curl http://localhost:8000/payments/metrics
curl http://localhost:8000/notification/metrics
curl http://localhost:8001/metrics  # métricas do Kong
```

## Fluxo de testes (end‑to‑end)
1. Criar usuário:
```
curl -X POST http://localhost:8000/users/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Ana Silva","email":"ana.silva@example.com"}'
```
2. Criar produto:
```
curl -X POST http://localhost:8000/products/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Notebook Pro","description":"14","price":5999.90,"stock":25}'
```
3. Criar pedido (publica no Kafka `orders-topic`):
```
curl -X POST http://localhost:8000/orders/orders \
  -H "Content-Type: application/json" \
  -d '{"userId":1,"productId":1,"quantity":2}'
```
4. Validar que `payments` consumiu e criou pagamento (e publicou `notifications-topic`):
```
curl http://localhost:8000/payments/payments
```
5. Validar que `notification` consumiu e persiste notificação:
```
curl http://localhost:8000/notification/notifications
```
6. Conferir tópicos no Kafka UI: `http://localhost:8080` (ver `orders-topic` e `notifications-topic`)

## Observabilidade
- Prometheus já configurado com `prometheus.yml` para coletar `/metrics` de todos os serviços e do Kong.
- Em Grafana, adicionar Data Source:
  - Type: `Prometheus`
  - URL: `http://prometheus:9090`
- Importar/Configurar dashboards conforme necessidade (podemos criar painéis específicos sob demanda).

## Postman
- Importar `postman/environment.json` e `postman/collection.json`
- Selecionar ambiente "Ecommerce Local" e executar requisições
- Substituir `orderIdMongo` por um ID real retornado na criação do pedido

## K6 (opcional)
- Rodar testes de carga a partir da rede Docker:
```
docker run --rm --network=g2_app-net -v %cd%\k6-scripts:/scripts grafana/k6 run /scripts/order-load-test.js
```

## Notas de Produção
- Proteja `Kong Admin` (porta 8001) com ACLs, rede privada ou autenticação quando fora de ambiente local.
- Configurar dashboards e alertas no Grafana/Prometheus para métricas de latência, taxa de erro e throughput Kafka.
- Persistência: volumes configurados para Postgres, MongoDB, e Konga; SQLite do `notification` é local ao container (adequado para demos).