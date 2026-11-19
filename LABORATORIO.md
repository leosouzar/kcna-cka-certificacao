# Laboratório local | Windows → WSL 2 → Docker → kind

O cluster deste roteiro chama-se **`kcna-cka-estudo`**. O arquivo [cluster.yaml](infra/kind/cluster.yaml) define um control plane e dois workers. Cada nó é um contêiner Linux no Docker. O kubeconfig fica no diretório pessoal do Ubuntu.

## Começar uma sessão nesta máquina (Teste)

O Ubuntu 22.04 foi convertido para WSL 2. O laboratório usa o Docker Engine 29.9.3 já instalado, kind v0.33.0 e Kubernetes v1.37.0. Um cliente kubectl v1.37.0 próprio do laboratório fica em `~/.local/share/kcna-cka-estudo/bin`. O usuário `leosouzar` recebeu acesso ao grupo `docker`.

1. No **PowerShell**, entre na pasta do projeto com `Set-Location -LiteralPath 'C:\Users\v8\OneDrive - V8.TECH\Documentos\Leo\kcna-cka-certificacao'` e abra o Ubuntu com `wsl -d Ubuntu-22.04`. O Ubuntu herda a pasta atual.
2. No **Ubuntu**, execute `source infra/kind/ativar.sh`. Isso seleciona o cliente e o kubeconfig de estudo nesse terminal.
3. Execute `kubectl get nodes`. Se o WSL estava parado, mantenha o terminal aberto enquanto o Docker e a API inicializam e repita a consulta. Com os três nós `Ready`, comece o [exercício 01](README.md).

As seções seguintes documentam como reproduzir a instalação.

**Validação realizada.** Os três nós chegaram a `Ready`, os Pods de `kube-system` ficaram prontos e um Pod BusyBox resolveu `kubernetes.default.svc.cluster.local.`. Durante uma inicialização a frio, o primeiro teste DNS recebeu conexão recusada. Depois que o CoreDNS ficou pronto, o teste passou. Para conferir antes dos exercícios de rede, execute `kubectl -n kube-system wait --for=condition=Ready pod -l k8s-app=kube-dns --timeout=180s`.

## 1. Conferir a distribuição no PowerShell

```powershell
wsl --list --verbose
```

Leia a coluna `VERSION` da linha `Ubuntu-22.04`. O valor esperado é **2**. `wsl --version` mostra a versão do software WSL instalado no Windows. Se a distribuição já estiver em 2, avance para o Ubuntu. Se aparecer `Converting`, aguarde o processo em curso.

Para converter uma distribuição que ainda aparece em 1, encerre as sessões de trabalho nela. Se houver arquivos importantes sem cópia, faça uma exportação para um diretório privado com espaço disponível antes de prosseguir. O exemplo de exportação abaixo usa o perfil local do Windows e um nome com data e hora.

```powershell
wsl --export Ubuntu-22.04 "$env:USERPROFILE\Ubuntu-22.04-$(Get-Date -Format yyyyMMdd-HHmmss).tar"
```

```powershell
wsl --set-version Ubuntu-22.04 2
wsl --list --verbose
```

Se o Windows informar que falta a Plataforma de Máquina Virtual, habilite o componente em PowerShell **como administrador** com `Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -All -NoRestart`. Se for solicitada reinicialização, salve o trabalho, reinicie e então retome a conversão. [Instalação e conversão do WSL](https://learn.microsoft.com/pt-br/windows/wsl/install)

Para abrir o Ubuntu já na pasta que contém este guia, execute no PowerShell do projeto:

```powershell
wsl -d Ubuntu-22.04
```

## 2. Validar o Docker dentro do Ubuntu

```bash
uname -r
command -v docker
docker version
```

A saída de `docker version` deve incluir **Client e Server**. Se os dois aparecerem, pule a instalação. Se você já usa Docker Desktop, ative a integração para `Ubuntu-22.04` em **Settings → Resources → WSL Integration** e repita a consulta. [Integração do Docker Desktop](https://docs.docker.com/desktop/features/wsl/)

Para um Ubuntu sem Docker, uma opção é usar o pacote da própria distribuição. Em uma sessão Ubuntu:

```bash
sudo apt-get update
sudo apt-get install -y docker.io ca-certificates curl
sudo service docker start
sudo docker version
```

Para usar o daemon local com seu usuário sem `sudo`, o grupo `docker` dá privilégios equivalentes aos de root nesse ambiente. Se escolher esse modo, execute `sudo usermod -aG docker "$(id -un)"`, abra outra sessão Ubuntu com o comando do passo 1 e confirme `docker info`. Se preferir usar pacotes do fornecedor, siga a instalação oficial em um Ubuntu sem pacotes Docker conflitantes. [Docker Engine para Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

## 3. Preparar kind e kubectl

No Ubuntu, confira `kind version` e `kubectl version --client`. Se faltarem, o bloco abaixo instala as ferramentas em `~/.local/bin` para o seu usuário. Ele usa a versão estável publicada pelo projeto kind e a versão estável de kubectl. Depois de criar o cluster, confira se a versão minor do cliente está a até uma versão da do API server. [Instalação do kind](https://kind.sigs.k8s.io/docs/user/quick-start/) · [Instalação do kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

```bash
export PATH="$HOME/.local/bin:$PATH"
(
  set -eu
  arch=$(dpkg --print-architecture)
  case "$arch" in amd64|arm64) ;; *) echo "Arquitetura sem binário neste guia: $arch"; exit 1 ;; esac
  mkdir -p "$HOME/.local/bin"
  tmp=$(mktemp -d)
  cd "$tmp"
  if ! command -v kind >/dev/null; then
    release=https://github.com/kubernetes-sigs/kind/releases/latest/download
    curl -fLsS "$release/kind-linux-$arch" -o "kind-linux-$arch"
    curl -fLsS "$release/kind-linux-$arch.sha256sum" -o kind.sha256sum
    sha256sum -c kind.sha256sum
    install -m 0755 "kind-linux-$arch" "$HOME/.local/bin/kind"
  fi
  if ! command -v kubectl >/dev/null; then
    version=$(curl -fsSL https://dl.k8s.io/release/stable.txt)
    base="https://dl.k8s.io/release/$version/bin/linux/$arch"
    curl -fsSLO "$base/kubectl"
    curl -fsSLO "$base/kubectl.sha256"
    echo "$(cat kubectl.sha256)  kubectl" | sha256sum -c -
    install -m 0755 kubectl "$HOME/.local/bin/kubectl"
  fi
)
kind version
kubectl version --client
```

## 4. Criar o cluster de estudo

A partir da pasta que contém este guia, rode no Ubuntu:

```bash
export PATH="$HOME/.local/bin:$PATH"
export KUBECONFIG="$HOME/.kube/kcna-cka-estudo.yaml"
mkdir -p "$HOME/.kube" modules/01-fundamentos
kind get clusters
```

Se `kcna-cka-estudo` ainda não aparecer, crie-o:

```bash
kind create cluster --name kcna-cka-estudo --config infra/kind/cluster.yaml --kubeconfig "$KUBECONFIG" --wait 5m
```

Se ele já existir, reconecte o cliente ao mesmo cluster:

```bash
kind export kubeconfig --name kcna-cka-estudo --kubeconfig "$KUBECONFIG"
```

## 5. Validar e começar o exercício 01

```bash
kubectl config current-context
kubectl wait --for=condition=Ready nodes --all --timeout=180s
kubectl get nodes -o wide
kubectl -n kube-system rollout status deployment/coredns --timeout=180s
kubectl get pods -A
kubectl version
```

**Critério de aceite.** O contexto é `kind-kcna-cka-estudo`, aparecem três nós `Ready` e o rollout do CoreDNS termina com sucesso. O worker pode exibir `<none>` na coluna ROLES. Para conferir a topologia pelo kind, execute `kind get nodes --name kcna-cka-estudo`.

Em cada novo terminal Ubuntu, carregue `source infra/kind/ativar.sh` a partir do projeto. Registre os nomes e as versões observadas no [exercício 01 do roteiro](README.md). Se um nó demorar, consulte `docker ps -a`, `kubectl describe node NOME_DO_NO` e `kind export logs --name kcna-cka-estudo ~/kcna-cka-logs` antes de tentar recriar o ambiente.

