
# 📝 Tradutor de Artigos Técnicos com AzureAI - Passo a Passo

Este guia detalha o processo de configuração e execução do Tradutor de Artigos Técnicos com AzureAI.

## 📦 Pré-requisitos
- **Python** 3.10 ou superior
- **Conta Azure** com acesso ao Azure OpenAI
- **Docker** (opcional)

## Azure
- Crie um Grupo de Recursos
- Crie uma instância do serviço Azure OpenAI
- No Azure OpenAI Studio, implante o modelo GPT-4 Mini

## 1. Importar as Bibliotecas Necessárias

Certifique-se de instalar as bibliotecas necessárias:

```python
from bs4 import BeautifulSoup
import requests, uuid, json
import os
from dotenv import load_dotenv
```

## 2. Configurar Variáveis de Ambiente
Crie um arquivo .env na raiz do projeto com as seguintes credenciais do Azure:
```bash
AZURE_OPENAI_KEY=sua_chave_openai
AZURE_ENDPOINT=seu_endpoint_openai
```

No script, carregue as variáveis de ambiente para usar as credenciais:
```bash
# Carregar variáveis de ambiente do arquivo .env
load_dotenv()
azureai_endpoint = os.getenv("AZURE_ENDPOINT")
API_KEY = os.getenv("AZURE_OPENAI_KEY")
```

### 3. Criar a Função de Extração de Texto
A função extract_text() extrai o conteúdo de uma URL. Ela remove tags irrelevantes e formata o texto para ser utilizado na tradução.
```python
def extract_text(url):
    response = requests.get(url)
    if response.status_code == 200:
        soup = BeautifulSoup(response.text, 'html.parser')
        for script in soup(["script", "style"]):
            script.decompose()
        text = soup.get_text(" ", strip=True)
        return text
    else:
        print("Falha ao buscar a URL. Código de status:", response.status_code)
        return None

# Exemplo de uso:
url = "https://dev.to/eric_dequ/steve-jobs-the-visionary-who-blended-spirituality-and-technology-3ppi"
extract_text(url)
```

## 4. Criar a Função de Tradução de Texto
A função translate_article() envia o texto extraído para o endpoint do Azure OpenAI, configurando a tradução para o idioma desejado.
```python
def translate_article(text, lang):
    headers = {
        "Content-Type": "application/json",
        "api-key": API_KEY,
    }
    payload = {
      "messages": [
        {
          "role": "system",
          "content": [
            {"type": "text", "text": "Você atua como tradutor de textos"}
          ]
        },
        {
          "role": "user",
          "content": [
            {"type": "text", "text": f"traduza: {text} para o idioma {lang} e responda apenas com a tradução no formato markdown"}
          ]
        }
      ],
      "temperature": 0.7,
      "top_p": 0.95,
      "max_tokens": 900
    }
    
    try:
        response = requests.post(azureai_endpoint, headers=headers, json=payload)
        response.raise_for_status()
        return response.json()['choices'][0]['message']['content']
    except requests.RequestException as e:
        raise SystemExit(f"Failed to make the request. Error: {e}")
```
