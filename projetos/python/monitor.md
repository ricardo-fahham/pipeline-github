# Monitorando uma api na Pipeline

```yaml
name: Monitoramento da API ServeRest

on:
  workflow_dispatch:

jobs:

  monitoramento:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout do projeto
        uses: actions/checkout@v4

      - name: Instalar Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.x'

      - name: Instalar dependências
        run: |
          python -m pip install --upgrade pip
          pip install requests

      - name: Executar monitoramento
        run: python monitor.py
```

## O Código

Crie o arquivo `monitor.py` 
```python
import requests

    url = "https://serverest.dev/usuarios"

    response = requests.get(url)

    if response.status_code == 200:
        print("✅ API respondendo com Status Code 200 OK")
    else:
        print(f"❌ API respondeu com Status Code {response.status_code}")
```