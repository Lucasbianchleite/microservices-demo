# 🔐 Conectando o ArgoCD a Repositórios Git Privados

Em um ambiente de produção, manter o código-fonte e os manifestos de configuração em **repositórios privados** é uma prática de segurança fundamental.  
Para que uma ferramenta de **GitOps** como o **ArgoCD** possa automatizar o deploy, ela precisa de um meio seguro para acessar esses repositórios.

Este guia detalha **dois métodos robustos e amplamente utilizados** para estabelecer essa conexão:

1. Usando um **Personal Access Token (PAT)** sobre HTTPS.  
2. Configurando o acesso via **chaves SSH**.

---

## 🚀 Método 1: Conexão via Personal Access Token (HTTPS)

Esta abordagem é ideal para uma configuração rápida e quando a política de rede favorece o tráfego HTTPS.  
Um **PAT** funciona como uma senha com escopo limitado, permitindo um controle granular sobre as permissões.

---

### 🧩 Passo a Passo

#### 1️⃣ Gerar o Personal Access Token (PAT)

O primeiro passo é criar um token no seu provedor Git (GitHub, GitLab, Bitbucket, etc.).

**No GitHub:**

1. Vá para:  
   `Settings > Developer settings > Personal access tokens > Tokens (classic)`
2. Clique em **Generate new token**.
3. Dê um nome descritivo, como `argocd-read-only-token`.
4. **Expiration:** Defina uma data de expiração — **nunca use tokens permanentes**.
5. **Scopes:** Selecione apenas o escopo `repo`.
6. Clique em **Generate token** e **copie o token imediatamente** — ele **não será exibido novamente**.

---

#### 2️⃣ Montar a URL do Repositório

Com o token em mãos, construa a URL de clone no seguinte formato:

```bash
https://<SEU_TOKEN>@github.com/<NOME_DE_USUARIO>/<NOME_DO_REPOSITORIO>.git
```

> Substitua `<SEU_TOKEN>`, `<NOME_DE_USUARIO>` e `<NOME_DO_REPOSITORIO>` pelos valores reais.

---

#### 3️⃣ Adicionar o Repositório no ArgoCD

Ao criar uma nova aplicação no ArgoCD (pela UI ou via manifesto YAML), use a URL acima em:

```yaml
spec:
  source:
    repoURL: "https://<SEU_TOKEN>@github.com/<NOME_DE_USUARIO>/<NOME_DO_REPOSITORIO>.git"
```

O ArgoCD usará o token embutido na URL para se autenticar e acessar o repositório.

> ⚠️ **Alerta de Segurança:**  
> Trate seu PAT como uma senha. Nunca o exponha em logs ou arquivos públicos.  
> Use escopos limitados e datas de expiração para reduzir riscos.

---

## 🔑 Método 2: Conexão via Chaves SSH

O acesso via SSH é o método **preferido para automação e comunicação segura entre sistemas**.  
Ele utiliza um par de chaves criptográficas (pública e privada) para autenticação, eliminando a necessidade de senhas ou tokens.

---

### 🧩 Passo a Passo

#### 1️⃣ Gerar um Par de Chaves SSH

É uma boa prática gerar um par de chaves dedicado exclusivamente para o ArgoCD.

Execute o comando abaixo no terminal para gerar uma chave moderna e segura (ed25519):

```bash
ssh-keygen -t ed25519 -C "argocd-key" -f ./argocd_id_ed25519
```

> Quando o prompt solicitar uma **passphrase**, pressione **Enter** duas vezes para deixá-la em branco.  
> O ArgoCD precisa da chave **sem senha** para operar automaticamente.

Isso criará dois arquivos:

- `argocd_id_ed25519` → 🔒 chave **privada**
- `argocd_id_ed25519.pub` → 🔑 chave **pública**

---

#### 2️⃣ Adicionar a Chave Pública ao Repositório (Deploy Key)

A chave pública autoriza o ArgoCD a acessar o repositório.

1. Copie o conteúdo da chave pública:
   ```bash
   cat ./argocd_id_ed25519.pub
   ```
2. No GitHub, vá para:  
   `Settings > Deploy Keys > Add deploy key`
3. **Title:** `ArgoCD`
4. **Key:** cole o conteúdo copiado.
5. **Allow write access:** deixe **desmarcado** (somente leitura).
6. Clique em **Add key**.

---

#### 3️⃣ Adicionar a Chave Privada ao ArgoCD

Agora, forneça a chave privada ao ArgoCD para autenticação.

1. Acesse a **interface web** do ArgoCD.  
2. Vá para **Settings > Repositories**.  
3. Clique em **+ CONNECT REPO**.  
4. Preencha os campos:

| Campo | Valor |
|-------|-------|
| **CONNECTION METHOD** | SSH |
| **NAME** | `meu-repo-privado-ssh` |
| **PROJECT** | O projeto do ArgoCD que usará o repositório |
| **REPOSITORY URL** | `git@github.com:<NOME_DE_USUARIO>/<NOME_DO_REPOSITORIO>.git` |
| **SSH PRIVATE KEY** | Conteúdo da sua `argocd_id_ed25519` |

Obtenha o conteúdo da chave privada com:

```bash
cat ./argocd_id_ed25519
```

Após preencher, clique em **CONNECT**.

Se tudo estiver correto, o status aparecerá como ✅ **Successful**.

---

## 🧠 Conclusão

Com isso, o **ArgoCD** está pronto para **monitorar e sincronizar aplicações** a partir de um **repositório privado** de forma **segura, automatizada e auditável**.

> 💡 **Dica:** Prefira o método SSH em ambientes produtivos, pois ele elimina a exposição de tokens sensíveis em configurações YAML.

---

📘 **Referências:**
- [ArgoCD Official Docs — Repositories](https://argo-cd.readthedocs.io/en/stable/user-guide/private-repositories/)
- [GitHub Docs — Deploy Keys](https://docs.github.com/en/developers/overview/managing-deploy-keys)
- [GitHub Docs — PAT Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
