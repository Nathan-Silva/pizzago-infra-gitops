# PizzaGo Infrastructore GitOps

![GitHub repo size](https://img.shields.io/github/repo-size/Nathan-Silva/pizzago-infra-gitops?style=for-the-badge)
![GitHub language count](https://img.shields.io/github/languages/count/Nathan-Silva/pizzago-infra-gitops?style=for-the-badge)
![GitHub forks](https://img.shields.io/github/forks/Nathan-Silva/pizzago-infra-gitops?style=for-the-badge)
![Bitbucket open issues](https://img.shields.io/bitbucket/issues/Nathan-Silva/pizzago-infra-gitops?style=for-the-badge)
![Bitbucket open pull requests](https://img.shields.io/bitbucket/pr-raw/Nathan-Silva/pizzago-infra-gitops?style=for-the-badge)

<img width="1440" alt="image" src="https://github.com/user-attachments/assets/e602d14b-324d-47ae-8de3-749273fb1dfb" />


## 💻 Pré-requisitos

Antes de começar, verifique se você atendeu aos seguintes requisitos:

- Você possui a versão mais recente das ferramentas e dependências necessárias (ex: kubectl, Helm, ArgoCD, etc.).
- Está utilizando uma máquina com sistema operacional Linux ou MacOS (Windows não é recomendado para esse setup local).
- Leu e entendeu a [documentação oficial do ArgoCD](https://argo-cd.readthedocs.io/en/stable/) e qualquer outro material relacionado à infraestrutura do projeto.

## 🧰 Instalando o Argo CD

Antes de aplicar os manifests do `pizzago-infra-gitops`, é necessário instalar o **Argo CD** no seu cluster Kubernetes.

### 🔹 1. Crie o namespace do Argo CD

```bash
kubectl create namespace argocd
```

### 🔹 2. Instale o Argo CD via manifests oficiais

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

> 💡 Isso instalará todos os componentes do Argo CD no namespace `argocd`.

### 🔹 3. Acesse a interface web do Argo CD

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Acesse no navegador: [https://localhost:8080](https://localhost:8080)

---

### 🔐 4. Obtendo a senha do usuário admin

Por padrão, o Argo CD gera uma senha para o usuário `admin`, que é armazenada em um Secret:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d && echo
```

---

### 🧪 5. (Opcional) Instale a CLI do Argo CD

#### Para macOS (via Homebrew)

```bash
brew install argocd
```

#### Para Linux (última versão)

```bash
VERSION=$(curl --silent "https://api.github.com/repos/argoproj/argo-cd/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*//')
curl -sSL -o argocd "https://github.com/argoproj/argo-cd/releases/download/$VERSION/argocd-linux-amd64"
chmod +x argocd && sudo mv argocd /usr/local/bin/
```

---

### ✅ 6. Faça login na CLI (opcional)

```bash
argocd login localhost:8080 --username admin --password <SENHA_AQUI> --insecure
```

---

Após isso, o Argo CD estará pronto para gerenciar suas aplicações GitOps! 🎉

---

## 🚀 Subindo o projeto pizzago-infra-gitops com Argo CD

Após instalar o ArgoCD, você pode configurar o repositório Git com os manifests do projeto `pizzago-infra-gitops` criando um `Application`. Essa aplicação será gerenciada automaticamente via GitOps.

### 🔧 1. Crie o Application apontando para seu repositório

Você pode aplicar o manifesto abaixo adaptando o repositório e caminho do seu projeto:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: pizzago-infra
  namespace: argocd
spec:
  destination:
    name: ''
    namespace: default
    server: 'https://kubernetes.default.svc'
  project: default
  source:
    repoURL: 'https://github.com/seu-usuario/pizzago-infra-gitops.git'
    targetRevision: HEAD
    path: overlays/qa
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

> 💡 **Importante:** substitua o campo `repoURL` pelo caminho correto do seu repositório, e o `path` pela pasta que contém seus manifests no estilo Kustomize ou Helm.

---

### 📥 2. Aplique o manifesto via `kubectl`

```bash
kubectl apply -f application.yaml
```

Ou crie diretamente via CLI do Argo CD:

```bash
argocd app create pizzago-infra \
  --repo https://github.com/seu-usuario/pizzago-infra-gitops.git \
  --path overlays/qa \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated
```

---

### ✅ 3. Sincronize a aplicação (caso não tenha sync automático)

```bash
argocd app sync pizzago-infra
```

---

Agora o Argo CD está monitorando seu repositório e mantendo o ambiente sincronizado com os manifests declarativos do `pizzago-infra-gitops`. 🚀

