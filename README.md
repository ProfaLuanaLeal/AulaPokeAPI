# AulaPokeAPI
# O Agente POKE API de Automação de QA

---

> **Instrução:** Copie e cole o texto abaixo na sua ferramenta de IA.
> 

```
"Atue como um Engenheiro de QA Sênior especializado em Automação de APIs e Performance. Seu objetivo é criar um framework de testes completo para a PokéAPI (https://pokeapi.co/api/v2/).
```

`Siga rigorosamente estas diretrizes:`

1. **Exploração:** Identifique os 3 endpoints mais críticos em termos de complexidade de dados (aninhamento).
2. **Suite de Testes:** Gere um arquivo 'test_poke_automation.py' usando Pytest. Inclua testes funcionais (status 200), testes de contrato (validação de tipos de campos) e testes de segurança (SQL Injection no parâmetro de busca).
3. **Performance:** Crie um cenário de carga usando 'concurrent.futures' que simule 50 requisições simultâneas e calcule o percentual de falhas e a latência P95.
4. **Self-Healing:** Projete uma função que, caso um campo esperado mude de nome, utilize uma busca por similaridade de string para sugerir o novo campo correto.
5. **Relatório:** Ao final, gere um resumo executivo dos testes em formato Markdown."

---

## 📈 O que acontece após esse prompt?

Ao usar essa abordagem no seu **Codespaces**, você deixa de fazer "scripts soltos" e passa a ter um **Sistema de Qualidade**. Veja a diferença:

### 1. Testes de Contrato (Pydantic/IA)

A IA não vai apenas checar se a resposta veio. Ela vai criar um "contrato". Se a PokéAPI prometeu que o campo `id` é um inteiro, mas em uma atualização ele virar uma string, a IA detecta isso antes do seu app quebrar em produção.

### 2. Monitoramento de Latência P95

Em vez de olhar apenas para a média (que pode ser enganosa), a IA vai te mostrar o **P95** (o tempo de resposta dos 5% de usuários que tiveram a pior experiência). Isso é vital para entender gargalos em conexões instáveis.

### 3. O Diferencial: IA Judicativa

Você pode pedir para a IA externa:

> *"Analise o log de erros desta suite de testes. Existe algum padrão de horário ou de endpoint que sugira que a PokéAPI está limitando nossa taxa de acesso (Rate Limiting)?"*
> 

---

### 1. Criar o Arquivo de Automação

No terminal do seu Codespaces, crie o arquivo:

Bash

# 

```java
touch test_poke_automation.py
```

### 2. O Código Gerado (Padrão Engenharia 2026)

Cole o código abaixo no arquivo. Este script não apenas testa a API, mas usa o **P95** para medir a performance real, simulando um cenário de alta demanda.

Python

```java
import requests
import pytest
import time
import statistics
from concurrent.futures import ThreadPoolExecutor

# Configurações do Agente de Teste
BASE_URL = "https://pokeapi.co/api/v2/pokemon/"
TARGETS = ["pikachu", "charizard", "bulbasaur", "squirtle", "mewtwo"]

# 1. Teste Funcional e de Contrato
@pytest.mark.parametrize("poke_name", TARGETS)
def test_poke_api_contract(poke_name):
    response = requests.get(f"{BASE_URL}{poke_name}")
    assert response.status_code == 200
    data = response.json()
    
    # Validação de Contrato (Tipagem)
    assert isinstance(data['name'], str)
    assert isinstance(data['id'], int)
    assert "abilities" in data

# 2. Teste de Performance com Latência P95
def test_performance_p95():
    latencias = []
    
    def make_request():
        start = time.perf_counter()
        requests.get(f"{BASE_URL}pikachu")
        return time.perf_counter() - start

    # Simula 20 requisições simultâneas
    with ThreadPoolExecutor(max_workers=5) as executor:
        resultados = list(executor.map(lambda f: f(), [make_request] * 20))
    
    latencias = [r * 1000 for r in resultados] # Converte para ms
    p95 = statistics.quantiles(latencias, n=100)[94] # Calcula o percentil 95
    
    print(f"\n⏱️ Latência Média: {statistics.mean(latencias):.2f}ms")
    print(f"🚀 Latência P95 (Pior cenário): {p95:.2f}ms")
    
    # O teste falha se 95% das requisições demorarem mais de 800ms
    assert p95 < 800, f"Performance inaceitável: P95 de {p95}ms"

if __name__ == "__main__":
    pytest.main([__file__, "-s"])
```

---

### 3. Execução no Terminal

Para rodar a suíte completa e ver o relatório detalhado:

Bash

```java
pytest test_poke_automation.py -v -s
```

### 4. Por que isto é "IA Level"?

Este script aplica conceitos que a IA sugeriria para um ambiente de produção:

- **Parametrização:** Em vez de escrever 5 testes para 5 Pokémons, usamos `@pytest.mark.parametrize` para rodar o mesmo teste com dados diferentes (Data-Driven Testing).
- **Latência P95:** Ignoramos a "média" comum e focamos no usuário que está tendo a pior experiência (os 5% mais lentos). Em Manaus, onde a internet pode oscilar, o P95 é a métrica que realmente importa para saber se o sistema é resiliente.
- **Contrato:** Verificamos se o `id` é `int`. Se a API mudar para `string`, seu teste quebra na hora, avisando que o backend mudou sem avisar.

---

### 📚 Insight para sua Aula

Você pode mostrar aos seus alunos que o **"Sucesso"** de um software não é apenas "dar 200 OK". O sucesso é:

1. **Integridade:** O dado veio no formato certo?
2. **Velocidade:** O usuário mais lento foi atendido em tempo hábil?
3. **Escalabilidade:** O sistema aguentou as requisições simultâneas?

| **Camada de Teste** | **Ferramenta / Técnica** | **O que garante?** |
| --- | --- | --- |
| **Funcional** | `pytest` + `requests` | A API entrega o dado correto. |
| **Contrato** | `isinstance(data, type)` | O formato do dado não quebra o Frontend. |
| **Performance** | `ThreadPoolExecutor` | O sistema aguenta múltiplos usuários (Carga). |
| **Estatística** | Latência **P95** | Qualidade de experiência para o usuário mais lento. |
| **Ambiente** | **GitHub Codespaces** | Padronização e independência de hardware. |

Sensacional! Ver esses `PASSED` em verde no terminal é a prova de que seu ambiente de **QA Engineer** está configurado e funcional.

---

### 🔄 O Próximo Passo: Manutenção Preditiva com IA

Imagine que a PokéAPI mude amanhã. Em vez de você abrir o código e corrigir manualmente, você usaria um **Agente de IA** para realizar o *Self-Healing* (Autocorreção).

### Como simular isso no Codespaces:

1. **Identificação de Falhas:** Se o teste de contrato falhar, você copia o erro do terminal.
2. **Prompt de Refatoração:** Você cola no chat da IA (Copilot ou Gemini):
    
    > *"Meu teste de contrato falhou porque o campo 'moves' agora se chama 'movimentos'. Refatore o arquivo `test_poke_automation.py` para mapear o novo campo, mas mantenha a compatibilidade com a versão anterior."*
    > 
3. **Aplicação:** A IA gera o código corrigido, e você apenas valida.

---

### 🎓 Conclusão da sua Trilha de Automação

Ao longo desta jornada, você construiu um ecossistema completo de testes:

| **Camada de Teste** | **Ferramenta / Técnica** | **O que garante?** |
| --- | --- | --- |
| **Funcional** | `pytest` + `requests` | A API entrega o dado correto. |
| **Contrato** | `isinstance(data, type)` | O formato do dado não quebra o Frontend. |
| **Performance** | `ThreadPoolExecutor` | O sistema aguenta múltiplos usuários (Carga). |
| **Estatística** | Latência **P95** | Qualidade de experiência para o usuário mais lento. |
| **Ambiente** | **GitHub Codespaces** | Padronização e independência de hardware. |

---

---

### 1. Instalar o Gerador de Relatórios

No terminal do seu Codespaces, instale a extensão:

```java
pip install pytest-html
```

### 2. Rodar os Testes Gerando o HTML

Agora, em vez do comando comum, você vai adicionar um parâmetro que diz ao `pytest` para criar o arquivo da página:

Bash

```java
pytest test_poke_automation.py --html=relatorio_pokemon.html --self-contained-html
```

- **`-html=relatorio_pokemon.html`**: Cria o arquivo com este nome.
- **`-self-contained-html`**: Garante que o CSS e os ícones fiquem dentro do próprio arquivo (facilita para você enviar o arquivo por e-mail ou abrir em qualquer lugar).

---

### 3. Como Visualizar a Página no Codespaces

1. Após rodar o comando, o arquivo `relatorio_pokemon.html` aparecerá na lista de arquivos à esquerda.
2. **Clique com o botão direito** no arquivo.
3. Selecione **"Open Preview"** (Abrir Visualização) ou, se preferir baixar, clique com o botão direito e selecione **"Download"**.

---

### 4. O que terá na sua Página HTML?

A página será um dashboard profissional contendo:

- **Sumário Executivo:** Quantos testes passaram, falharam ou deram erro.
- **Tempo de Execução:** Quanto tempo cada Pokémon levou para ser validado.
- **Detalhes de Erro:** Se algum teste falhar (como o nosso teste de contrato de "movimentos"), o HTML mostrará o "Traceback" (o caminho do erro) destacado em vermelho.
- **Ambiente:** Informações sobre o sistema (Python version, plataforma, etc.).

---

### Desafio

**Prompt para a IA :**

> *"Crie um arquivo conftest.py para o pytest que altere o título do relatório HTML para 'Dashboard de Qualidade - PokéAPI' e adicione uma linha no sumário com seu nome"*
>
