# Overlay AWS Free Tier

Este overlay aplica os manifests base com pequenas alterações para uso em EKS/AWS Free Tier:

- Altera `Service` para `LoadBalancer` com anotação para NLB.
- Adiciona `ServiceAccount` `oficina-sa` que referencia um `imagePullSecret` chamado `regcred`.

Passos recomendados:

1. Criar o `imagePullSecret` apontando para seu registry (ECR, DockerHub, etc.). Exemplo para DockerHub:

```bash
kubectl create secret docker-registry regcred \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=YOUR_USER \
  --docker-password=YOUR_PASS \
  --docker-email=you@example.com \
  -n oficina
```

Exemplo ECR (recomenda usar login via AWS CLI):

```bash
aws ecr get-login-password --region <region> | \
  kubectl create secret docker-registry regcred \
    --docker-server=<aws_account_id>.dkr.ecr.<region>.amazonaws.com \
    --docker-username=AWS \
    --docker-password-stdin -n oficina
```

2. Ajuste `k8s/secret.yaml` e `k8s/configmap.yaml` se necessário.

3. Aplicar o overlay no cluster EKS:

```bash
kubectl apply -k k8s/overlays/aws-free-tier
```

Notas:
- O overlay mantém `replicas: 1` do base para ser econômico.
- Para produção, considere usar Ingress com ALB e RBAC/IRSA para acesso a ECR.
