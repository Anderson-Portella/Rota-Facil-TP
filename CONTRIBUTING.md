# Guia de Contribuição

Obrigado pelo interesse em contribuir com o **Transpetro Rota Fácil**!

## Como contribuir

### Reportar bugs

Abra uma [Issue](../../issues) com:
- Descrição clara do problema
- Passos para reproduzir
- Comportamento esperado vs. observado
- Screenshots (se aplicável)
- Versão do Python e do Streamlit

### Sugerir melhorias

Abra uma Issue com a tag `enhancement` descrevendo:
- O problema que a melhoria resolve
- Proposta de solução
- Alternativas consideradas

### Enviar código

1. Faça um fork do repositório
2. Crie uma branch descritiva:
   ```bash
   git checkout -b feature/descricao-curta
   # ou
   git checkout -b fix/descricao-do-bug
   ```
3. Faça suas alterações seguindo as convenções abaixo
4. Teste localmente com `streamlit run app.py`
5. Commit com mensagens no padrão [Conventional Commits](https://www.conventionalcommits.org/):
   ```bash
   git commit -m "feat: adiciona exportação em CSV"
   git commit -m "fix: corrige parsing de coordenadas negativas"
   ```
6. Push e abra um Pull Request

## Convenções de código

- **Python**: PEP 8, type hints quando possível
- **Docstrings**: formato Google
- **Idioma do código**: variáveis e funções em inglês ou português (manter consistência com o existente)
- **Idioma da UI**: português (pt-BR)
- **Commits**: Conventional Commits em português ou inglês

## Ambiente de desenvolvimento

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
streamlit run app.py
```
