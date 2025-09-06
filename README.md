# infra — Helm-чарт и деплой в Kubernetes

Содержит Helm-чарт `myapp` (Deployment, Service, ConfigMap, Secret, HPA) и GitLab CI для `helm lint` и деплоя в namespace `dev`.

## Предпосылки

1. Доступ из CI к кластеру Kubernetes:
   - В проекте **infra** в GitLab добавлена переменная **`KUBECONFIG_B64`** (Masked, Protected) со значением:
     ```bash
     base64 -w0 ~/.kube/config
     ```
2. Приватный реестр разрешён на нодах кластера (CRI-O, insecure registry):
   - На каждой ноде:
     ```bash
     sudo mkdir -p /etc/containers/registries.conf.d
     sudo tee /etc/containers/registries.conf.d/gitlab-insecure.conf >/dev/null <<'EOF'
     [[registry]]
     location = "192.168.0.10:5050"
     insecure = true
     EOF
     sudo systemctl restart crio || sudo systemctl restart cri-o
     ```
3. Секрет для pull образов в `dev`:
   ```bash
   kubectl create namespace dev --dry-run=client -o yaml | kubectl apply -f -
   kubectl -n dev create secret docker-registry regcred \
     --docker-server=192.168.0.10:5050 \
     --docker-username='<user>' \
     --docker-password='<token>' \
     --docker-email='ci@example.com'