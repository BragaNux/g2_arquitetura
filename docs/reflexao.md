# Reflexão de Arquitetura: Escalabilidade, Resiliência e Trade-offs

## Escalabilidade

- Kafka viabiliza expansão horizontal de consumidores por grupo, suportando picos sem acoplar produtores aos processadores.
- Microsserviços com bancos separados permitem evolução independente e particionamento por contexto.
- Redis cache reduz latência para leituras frequentes, diminuindo pressão nos bancos.

## Resiliência

- Consumidores com reconexão automática e commits de offsets evitam perda de eventos.
- Kong provê limites de taxa e tamanho, protegendo a borda contra abusos.
- Separação por tópicos isola falhas: se pagamentos estiverem indisponíveis, pedidos continuam sendo aceitos e enfileirados.

## Observabilidade

- Métricas padronizadas em `/metrics` viabilizam alertas (latência, erro, consumo de CPU/memória).
- Grafana sobre Prometheus facilita correlação entre serviços e análise de tendências.

## Trade-offs

- Remoção de RabbitMQ simplifica a topologia, mas elimina possibilidade de roteamento avançado por exchange; Kafka cobre casos de streaming/ordenação e throughput.
- Uso de múltiplos bancos aumenta a complexidade operacional (backup, migração), em troca de melhor adequação tecnológica por domínio.
- Kong em modo declarativo melhora previsibilidade, mas exige pipeline de entrega para manter configuração versionada.

## Próximos Passos

- Implementar autenticação e autorização robusta (JWT com escopos) no gateway.
- Adicionar DLQs e retries explícitos para cenários de falha em pagamentos.
- Provisionar dashboards Grafana para métricas de negócio (pedidos/hora, taxa de aprovação).