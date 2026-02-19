# Manifests Kubernetes para `oficina-operations-service`

Aplicação: serviço Spring Boot localizado em `src/main/java/br/com/grupo99/operacoes`.

Como usar (exemplo mínimo):

1. Ajuste `k8s/secret.yaml` com os valores reais (ou crie secret via kubectl):

```bash
# Exemplo: criar secret a partir de valores
kubectl create secret generic operations-service-secrets \
  --from-literal=database-url='jdbc:postgresql://postgres:5432/oficina_operations' \
  --from-literal=database-username='postgres' \
  --from-literal=database-password='postgres' \
  --from-literal=jwt-secret='super-secret'
```

2. Ajuste o `k8s/configmap.yaml` se necessário (bootstrap-servers do Kafka).

3. Aplique os manifests juntos (usando kustomize):

```bash
kubectl apply -k k8s/
```

Boas práticas incluídas:
- Separação de `ConfigMap` e `Secret` (config vs segredos).
- Probes de readiness/liveness usando `/actuator/health`.
- `resources.requests` e `limits` definidos.
- `kustomization.yaml` para compor os manifests e adicionar labels comuns.

AWS Free Tier - recomendações padrão:
- **Replica única:** `replicas: 1` para reduzir custo e uso de recursos.
- **Recursos reduzidos:** requests/límites baixos (`128Mi` / `256Mi`, `100m` / `300m`).
- **Evitar LoadBalancer automático:** o tipo `LoadBalancer` pode criar ELB/ALB com custo; usar `ClusterIP` ou um Ingress gerenciado quando necessário.
- **Imagem e registro:** publique a imagem no ECR e atualize `DOCKER_REGISTRY` antes de aplicar.
- **Segredos:** mantenha credenciais em AWS Secrets Manager ou crie `Secret` via `kubectl` (exemplo acima).
- **Alternativa para acesso externo:** considere usar um Ingress Controller (NGINX/ALB) com regras de entrada em vez de Service `LoadBalancer` direto.

Observação: valores aqui são conservadores para caber em instâncias t3.micro/t3a.micro. Ajuste conforme o tamanho do seu cluster EKS.
