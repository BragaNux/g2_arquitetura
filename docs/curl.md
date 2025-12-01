# cURL de Teste — Endpoints via Kong

Base: `http://localhost:8000`

## Users

- Criar usuário
```
curl -X POST http://localhost:8000/users/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Ana Silva","email":"ana.silva@example.com"}'
```

- Listar usuários
```
curl http://localhost:8000/users/users
```

- Buscar usuário por id
```
curl http://localhost:8000/users/users/1
```

- Deletar usuário
```
curl -X DELETE http://localhost:8000/users/users/1
```

- Métricas (Prometheus)
```
curl http://localhost:8000/users/metrics
```

## Products

- Criar produto
```
curl -X POST http://localhost:8000/products/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Notebook Pro","description":"14","price":5999.90,"stock":25}'
```

- Listar produtos
```
curl http://localhost:8000/products/products
```

- Buscar produto por id
```
curl http://localhost:8000/products/products/1
```

- Atualizar estoque
```
curl -X PATCH http://localhost:8000/products/products/1/stock \
  -H "Content-Type: application/json" \
  -d '{"quantity":2}'
```

- Deletar produto
```
curl -X DELETE http://localhost:8000/products/products/1
```

- Métricas
```
curl http://localhost:8000/products/metrics
```

## Orders

- Criar pedido (publica no `orders-topic`)
```
curl -X POST http://localhost:8000/orders/orders \
  -H "Content-Type: application/json" \
  -d '{"userId":1,"productId":1,"quantity":2}'
```

- Listar pedidos
```
curl http://localhost:8000/orders/orders
```

- Buscar pedido por id
```
curl http://localhost:8000/orders/orders/<orderIdMongo>
```

- Métricas
```
curl http://localhost:8000/orders/metrics
```

## Payments

- Listar pagamentos
```
curl http://localhost:8000/payments/payments
```

- Buscar pagamento por id
```
curl http://localhost:8000/payments/payments/1
```

- Tipos de pagamento
```
curl http://localhost:8000/payments/payments/types
```

- Métricas
```
curl http://localhost:8000/payments/metrics
```

## Notification

- Criar notificação direta
```
curl -X POST http://localhost:8000/notification/notify \
  -H "Content-Type: application/json" \
  -d '{"type":"EMAIL","recipient":"ana.silva@example.com","subject":"Bem-vinda","message":"Cadastro realizado com sucesso."}'
```

- Listar notificações
```
curl http://localhost:8000/notification/notifications
```

- Métricas
```
curl http://localhost:8000/notification/metrics
```

## Observações
- Os caminhos usam o prefixo de cada serviço conforme rotas do Kong (`strip_path: false` por padrão; em `notification` o prefixo é removido para compatibilizar com `/notify` e `/notifications`).
- Substitua `<orderIdMongo>` por um ID válido do MongoDB retornado na criação do pedido.