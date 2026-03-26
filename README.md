# 🚚 Transpetro Rota Fácil

Simulador de tempo e distância rodoviária entre unidades operacionais, utilizando a API [OpenRouteService](https://openrouteservice.org/) (gratuita, sem necessidade de cartão de crédito).

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Streamlit](https://img.shields.io/badge/Streamlit-1.30%2B-red)
![License](https://img.shields.io/badge/License-MIT-green)

---

## Funcionalidades

- **Rota individual** a partir de uma base de centros pré-cadastrados ou coordenadas manuais
- **Processamento em lote** de milhares de rotas via upload de planilha (Excel/CSV)
- **Robustez para grandes volumes**: chunking, rate limit com backoff exponencial e persistência parcial
- Exportação de resultados em **Excel** e **JSON** (compatível com Power BI)
- Detecção automática de pares origem = destino
- Suporte a coordenadas com vírgula decimal (padrão brasileiro)

## Arquitetura

```
┌─────────────────────────────────────────────────┐
│                  Streamlit UI                    │
│  ┌───────────┐  ┌──────────┐  ┌──────────────┐  │
│  │Base Centros│  │  Manual  │  │ Upload Lote  │  │
│  └─────┬─────┘  └────┬─────┘  └──────┬───────┘  │
│        └──────────────┼───────────────┘          │
│                       ▼                          │
│              calcular_ors()                      │
│         retry + backoff exponencial              │
│                       │                          │
│                       ▼                          │
│           OpenRouteService API                   │
│          (Directions v2 / driving-car)           │
└─────────────────────────────────────────────────┘
```

## Pré-requisitos

- Python 3.10+
- Chave de API do [OpenRouteService](https://openrouteservice.org/dev/#/signup) (gratuita)

### Limites do plano gratuito ORS

| Recurso | Limite |
|---------|--------|
| Directions v2 | 2.000 requisições/dia |
| Rate limit | 40 requisições/minuto |

## Instalação

```bash
# Clonar o repositório
git clone https://github.com/SEU_USUARIO/transpetro-rota-facil.git
cd transpetro-rota-facil

# Criar ambiente virtual (recomendado)
python -m venv .venv
source .venv/bin/activate   # Linux/Mac
# .venv\Scripts\activate    # Windows

# Instalar dependências
pip install -r requirements.txt
```

## Configuração

### 1. Chave da API

Obtenha sua chave gratuita em [openrouteservice.org/dev/#/signup](https://openrouteservice.org/dev/#/signup).

A chave é informada diretamente na barra lateral do aplicativo (campo com máscara de senha). Não é necessário configurar variáveis de ambiente.

### 2. Bases de dados (opcional)

Para o modo "Base de Centros", coloque na raiz do projeto:

- `nomes e localização.csv` — cadastro de centros (separador `;`)
- `coordenadas_consolidadas.csv` — coordenadas geográficas

Ambos devem compartilhar a coluna `Centro` como chave de junção.

### 3. Mascote (opcional)

Coloque uma imagem `Mascote.png` na raiz do projeto para exibir na barra lateral.

## Uso

```bash
streamlit run app.py
```

### Modo 1: Base de Centros

Selecione origem e destino nos dropdowns e clique em **Calcular Rota**.

### Modo 2: Manual

Informe nome e coordenadas (latitude/longitude) de origem e destino.

### Modo 3: Processamento em Lote

1. Baixe o template Excel pelo botão na interface
2. Preencha com suas rotas (colunas obrigatórias abaixo)
3. Faça upload e clique em **Calcular Todas as Rotas**

#### Colunas obrigatórias do template

| Coluna | Tipo | Exemplo |
|--------|------|---------|
| `Nome_Origem` | texto | Terminal Duque de Caxias |
| `Latitude_Origem` | float | -22.4716 |
| `Longitude_Origem` | float | -43.3019 |
| `Nome_Destino` | texto | Terminal de Ilha d'Água |
| `Latitude_Destino` | float | -22.8584 |
| `Longitude_Destino` | float | -43.1365 |

### Configuração de parâmetros de lote

Na barra lateral, ajuste conforme a necessidade:

| Parâmetro | Default | Recomendado (plano gratuito) | Descrição |
|-----------|---------|------------------------------|-----------|
| Tamanho do bloco | 500 | 500 | Rotas por chunk (divisão lógica) |
| Pausa entre chamadas | 0.4 s | **1.6 s** | Respeita 40 req/min do ORS |
| Salvar parcial a cada | 50 | 50 | Frequência de checkpoint |

**Exemplo prático:** para 3.876 rotas com pausa de 1.6s:
- Dia 1: ~2.000 rotas em ~53 min
- Dia 2: ~1.876 rotas em ~50 min

## Estrutura do Projeto

```
transpetro-rota-facil/
├── app.py                          # Aplicação principal
├── requirements.txt                # Dependências Python
├── README.md                       # Este arquivo
├── LICENSE                         # Licença MIT
├── CHANGELOG.md                    # Histórico de versões
├── .gitignore                      # Arquivos ignorados pelo git
├── .streamlit/
│   └── config.toml                 # Configurações do Streamlit
├── Mascote.png                     # (opcional) Imagem da sidebar
├── nomes e localização.csv         # (opcional) Base de centros
└── coordenadas_consolidadas.csv    # (opcional) Coordenadas
```

## Mecanismos de Robustez

O processamento em lote implementa três estratégias para operar de forma estável dentro dos limites do plano gratuito:

### 1. Chunking
O DataFrame é dividido em blocos de tamanho configurável. O progresso é exibido por bloco, e em caso de falha, o reprocessamento é feito a partir do último checkpoint.

### 2. Rate Limit com Backoff Exponencial
Cada chamada respeita a pausa mínima configurada. Em caso de HTTP 429 (rate limit) ou erros 5xx, o sistema faz até 3 retentativas com espera exponencial (2s, 4s, 8s).

### 3. Persistência Parcial
A cada N rotas processadas, os resultados são salvos em `st.session_state`. Se a sessão cair, a retomada recomeça do último ponto salvo (dentro da mesma sessão do navegador).

## Saída de Dados

O DataFrame de resultados contém:

| Coluna | Descrição |
|--------|-----------|
| `Nome_Origem` | Nome do ponto de origem |
| `Latitude_Origem` | Coordenada |
| `Longitude_Origem` | Coordenada |
| `Nome_Destino` | Nome do ponto de destino |
| `Latitude_Destino` | Coordenada |
| `Longitude_Destino` | Coordenada |
| `Distancia_KM` | Distância rodoviária em km |
| `Distancia_MI` | Distância rodoviária em milhas |
| `Tempo_Minutos` | Tempo estimado de viagem |
| `Status` | `Calculado`, `Origem = Destino` ou `Erro: ...` |
| `Data_Calculo` | Timestamp do cálculo |

## Contribuição

1. Faça um fork do projeto
2. Crie uma branch para sua feature (`git checkout -b feature/minha-feature`)
3. Commit suas mudanças (`git commit -m 'feat: adiciona minha feature'`)
4. Push para a branch (`git push origin feature/minha-feature`)
5. Abra um Pull Request

## Licença

Este projeto é distribuído sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para detalhes.

## Créditos

- [OpenRouteService](https://openrouteservice.org/) — API de rotas (Heidelberg Institute for Geoinformation Technology)
- [Streamlit](https://streamlit.io/) — Framework de interface
