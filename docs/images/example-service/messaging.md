# NATS integration best practices

A NATS server with JetStream is deployed in the cluster tests to demonstrate event-driven messaging patterns.
This section captures our current understanding of best NATS JetStream practices from an images perspective.

- Use JetStream for message delivery over core NATS when events must not be lost.
- Initialize `streams` before application starts to avoid race conditions and `consumers` polling
    - In Tests: we can achieve this by using `init-containers` before the main application starts.
    - In Production/Staging: We should install the `nats-controllers-for-kubernetes` (NACK) and use the `Stream` Custom Resource Definition.
        - [NATS Kubernetes Controller](https://docs.nats.io/running-a-nats-service/configuration/resource_management/configuration_mgmt/kubernetes_controller)
        - [NACK Install Guide](https://github.com/nats-io/nack#getting-started)
- Security:
    - We should also consider Mutual TLS and whether its nessassary to use the `Accounts` Custom Resource Definition
- Use durable consumers with explicit acknowledgment for reliable message processing
- Configure appropriate timeouts and limits:
    - retry limits `max_deliver` and
    - acknowledgment timeouts `ack_wait` based on processing requirements.
- Use subject filtering (e.g., `events.>`, `events.users.*`) for message types.
- Leverage consumer groups for horizontal scaling - JetStream automatically distributes work across replicas without duplication.
