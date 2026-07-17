# Estrutura de um workflow

```yaml
name: Nome da Pipeline

on:
  evento:

jobs:

  nome-do-job:

    runs-on: ambiente

    steps:

      - name: Etapa 1
        comando ou action

      - name: Etapa 2
        comando ou action
```

## Fluxo de execução deste workflow

flowchart TD
    A[Push na branch main] --> D[Job build]
    B[Pull Request para main] --> D
    C[Schedule 02:00 UTC diariamente] --> D
    D --> E[Checkout do código]
    E --> F[Executa echo hello-world]