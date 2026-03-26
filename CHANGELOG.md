# Changelog

Todas as mudanças relevantes deste projeto serão documentadas neste arquivo.

O formato é baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/),
e este projeto adere ao [Versionamento Semântico](https://semver.org/lang/pt-BR/).

## [2.0.0] - 2026-03-26

### Adicionado
- Chunking real no processamento em lote (divisão em blocos configuráveis)
- Retry com backoff exponencial para HTTP 429 e erros 5xx (até 3 tentativas)
- Persistência parcial de resultados via `session_state` com retomada automática
- Parâmetros de lote configuráveis na sidebar (chunk size, pausa, frequência de salvamento)
- Estimativa de tempo exibida antes do processamento
- Botão para limpar resultados parciais
- Função `carregar_base()` com `@st.cache_data` para evitar recarregamento de CSVs
- Validação e fallback quando arquivos de base não existem
- Suporte a coordenadas com vírgula decimal (padrão brasileiro)
- Documentação completa para publicação no GitHub

### Corrigido
- Chamada solta `calcular_ors(...)` sem argumentos na função `processar_lote`
- `resultados.append()` duplicado causando registros repetidos
- `time.sleep()` posicionado antes da chamada à API (fora do fluxo correto)
- Tratamento de erro inconsistente entre os três modos de entrada

### Alterado
- Refatoração completa: funções extraídas e desacopladas (`processar_rota_individual`, `_build_row`, `_parse_coords`, `calcular_rota_individual`, `resultado_para_excel`)
- Eliminação de código duplicado entre os modos de entrada
- User-Agent atualizado para `Transpetro-ORS/2.0`

## [1.0.0] - 2026-03-26

### Adicionado
- Versão inicial do aplicativo Streamlit
- Três modos de entrada: base de centros, manual e upload em lote
- Integração com API OpenRouteService (Directions v2, driving-car)
- Cálculo de distância (km/mi) e tempo estimado
- Download de resultados em Excel e JSON
- Fórmula de Haversine para detecção de origem = destino
- Template Excel para processamento em lote
