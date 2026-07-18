# Rollback

O **GitHub Actions não realiza rollback automaticamente**. Ele apenas executa os passos definidos no workflow. O rollback precisa ser implementado pela própria pipeline ou pela estratégia de implantação utilizada (Docker, Kubernetes, Azure, AWS, etc.).

Abaixo estão as formas mais comuns de realizar rollback.

---

# Estratégia 1 – Fazer rollback para a versão anterior

A abordagem mais comum é manter a versão anterior da aplicação disponível.

Fluxo:

```mermaid
flowchart TD
    A[Versão 1.0 em produção] --> B[Deploy da versão 2.0]
    B --> C{Deploy funcionou?}
    C -->|Sim| D[Manter versão 2.0]
    C -->|Não| E[Rollback para versão 1.0]
```

Exemplo:

```mermaid
flowchart TD
    A[Versão atual 1.0] --> B[Deploy da versão 2.0]
    B --> C[Erro]
    C --> D[Rollback para a versão 1.0]
```

---

# Estratégia 2 – Utilizando GitHub Releases

Uma prática comum é publicar cada versão como uma **Release**.

```
Release 1.0

Release 1.1

Release 1.2
```

Se a versão **1.2** apresentar problemas, basta fazer o deploy da **Release 1.1**.

---

# Estratégia 3 – Utilizando Tags

Antes de cada deploy:

```
v1.0

v1.1

v1.2
```

Caso seja necessário retornar:

```bash
git checkout v1.1
```

Depois execute novamente a pipeline utilizando essa tag.

---

# Estratégia 4 – Rollback utilizando Git

Caso o código publicado esteja incorreto, é possível desfazer o commit.

Exemplo:

```bash
git revert HEAD
git push
```

Fluxo:

```mermaid
flowchart TD
    A[Commit A] --> B[Commit B]
    B --> C[Commit C - Erro]
    C --> D[git revert]
    D --> E[Commit D - Desfaz Commit C]
    E --> F[Pipeline executa novamente]
```

O histórico permanece preservado.

---

# Estratégia 5 – Rollback utilizando Docker

Uma das estratégias mais utilizadas.

Imagine:

```
app:1.0

app:1.1

app:1.2
```

O deploy utiliza:

```
app:1.2
```

Caso haja falha:

```
docker stop app

docker run app:1.1
```

A versão anterior volta rapidamente.

---

# Estratégia 6 – Rollback no Kubernetes

O Kubernetes mantém o histórico das implantações.

Exemplo:

```bash
kubectl rollout undo deployment/api
```

Ou voltar para uma revisão específica:

```bash
kubectl rollout undo deployment/api --to-revision=2
```

Fluxo:

```mermaid
flowchart TD
    A[Revision 1] --> B[Revision 2]
    B --> C[Revision 3 - Erro]
    C --> D[Rollback]
    D --> E[Revision 2]
```

---

# Estratégia 7 – Rollback utilizando ambientes

Uma prática muito utilizada é separar os ambientes.

```mermaid
flowchart TD
    A[Development] --> B[Homologação]
    B --> C[Produção]
```

Se a produção apresentar falha:

```mermaid
flowchart TD
    A[Produção] --> B[Rollback]
    B --> C[Versão anterior]
```

---

# Estratégia 8 – Blue/Green Deployment

Mantêm-se dois ambientes idênticos.

```mermaid
flowchart LR
    A[Usuários] --> B[Blue Produção]

    C[Deploy nova versão] --> D[Green]

    D --> E{Funcionou?}

    E -->|Sim| F[Tráfego vai para Green]

    E -->|Não| G[Continuar usando Blue]
```

Se houver erro:

```mermaid
flowchart TD
    A[Blue continua atendendo] --> B[Green é descartado]
```

Quase não há indisponibilidade.

---

# Estratégia 9 – Canary Deployment

A nova versão é liberada para poucos usuários.

```
flowchart TD
    A[5% dos usuários] --> B[20% dos usuários]
    B --> C[50% dos usuários]
    C --> D[100% dos usuários]
```

Caso apareçam erros:

```mermaid
flowchart TD
    A[100% dos usuários] --> B[50% dos usuários]
    B --> C[20% dos usuários]
    C --> D[0% dos usuários]
```

A nova versão é retirada antes de afetar todos os usuários.

---

# Estratégia 10 – Rollback automático na pipeline

Uma pipeline pode detectar falhas após o deploy e executar automaticamente um rollback.

Exemplo:

```yaml
name: Deploy

jobs:

  deploy:

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: ./deploy.sh

      - name: Health Check
        run: ./healthcheck.sh

      - name: Rollback
        if: failure()
        run: ./rollback.sh
```

Nesse fluxo:

* `deploy.sh` publica a nova versão;
* `healthcheck.sh` verifica se a aplicação está saudável;
* se qualquer etapa falhar, `rollback.sh` é executado automaticamente graças à condição `if: failure()`.

---

# Fluxo completo

```mermaid
flowchart TD
    A[Build] --> B[Testes]
    B --> C[Deploy]
    C --> D[Health Check]

    D --> E{Aplicação saudável?}

    E -->|Sim| F[Deploy concluído]

    E -->|Não| G[Rollback]

    G --> H[Versão anterior restaurada]
```

---

# Resumo

| Estratégia                   | Vantagem                                | Cenário de uso                          |
| ---------------------------- | --------------------------------------- | --------------------------------------- |
| `git revert`                 | Simples e mantém o histórico            | Código incorreto enviado ao repositório |
| Releases/Tags                | Fácil de identificar versões            | Controle de versões da aplicação        |
| Docker                       | Retorno rápido para uma imagem anterior | Aplicações conteinerizadas              |
| Kubernetes Rollout           | Rollback nativo                         | Aplicações em Kubernetes                |
| Blue/Green                   | Baixíssima indisponibilidade            | Sistemas críticos                       |
| Canary                       | Reduz impacto de falhas                 | Grandes aplicações com muitos usuários  |
| Pipeline com `if: failure()` | Automatiza a recuperação                | Deploys com validação automática        |

Em resumo, o **GitHub Actions orquestra o processo**, mas o rollback depende da tecnologia utilizada para o deploy. Em ambientes modernos, é comum combinar GitHub Actions com Docker ou Kubernetes, pois essas plataformas oferecem mecanismos rápidos e confiáveis para retornar à versão anterior caso algo dê errado.
