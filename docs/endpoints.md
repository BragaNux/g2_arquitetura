# Catálogo de Endpoints

Todos os endpoints abaixo são acessíveis pelo Kong (`http://localhost:8000`). Para acesso interno, use os hosts dos serviços.

## Users

- `POST /users/register`
  - Body: `{ "name": "string", "email": "string" }`
  - 201: `{ id, name, email, createdAt }`

- `GET /users`
  - 200: `[{ id, name, email, createdAt }]`

- `GET /users/:id`
  - 200: `{ id, name, email, createdAt }`
  - 404 se não encontrado

- `DELETE /users/:id`
  - 200: `{ message }`

- `GET /users/metrics`
  - `text/plain` Prometheus exposition

## Products

- `GET /products`
- `GET /products/:id`
- `POST /products`
  - Body: `{ name, description?, price, stock }`
  - 201: `Product`
- `PATCH /products/:id/stock`
  - Body: `{ quantity }` (int > 0)
- `DELETE /products/:id`
- `GET /products/metrics`

## Orders

- `POST /orders`
  - Body: `{ userId, productId, quantity }`
  - 201: `Order`
  - Side‑effect: publica no `orders-topic`

- `GET /orders`
- `GET /orders/:id`
- `GET /orders/metrics`

## Payments

- Consumer do `orders-topic`.
- `GET /payments`
- `GET /payments/:id`
- `GET /payments/types`
- `GET /payments/metrics`
- Side‑effect: publica no `notifications-topic`

## Notification

- Serviço interno (apenas na rede Docker), sem acesso via Kong.
- Consome `notifications-topic` e persiste eventos.
- Endpoints acessíveis somente internamente: `/notify`, `/notifications`, `/metrics` (não expostos pelo gateway).

## Gateway (Kong)

- Serviços e rotas declarados em `kong.yml`:
  - `/users`, `/products`, `/orders`, `/payments`, `/notification`
- Plugin Prometheus habilitado; métricas em `http://localhost:8001/metrics`.