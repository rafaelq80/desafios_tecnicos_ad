<p align="center">
  <img src="icones/04-atendimento-ao-cliente.svg" width="88" height="88" alt="Icone do tema Atendimento ao cliente">
</p>

<h1 align="center">Tema 4 &mdash; Atendimento ao Cliente</h1>

<p align="center">
  <strong>Central de atendimento (SAC)</strong><br>
  Plano de modelagem da base de dados mockada
</p>

| Desafio de origem | Uso nos desafios | Situação | Gerador |
|:--|:--|:--|:--|
| Trabalho 1 (Python) | 1 · 2 · 3 · 4 · 5 | Plano validado contra o script; base gerada em CSV | `gerar_bases.py` (seed 42) |

**Nesta página:** [Contexto](#contexto-do-tema) · [Modelo de dados](#modelo-de-dados) · [Cardinalidade](#cardinalidade) · [Volumetria](#volumetria) · [Geração](#estratégia-de-geração-de-dados-mockados) · [Expansão](#expansão-sugerida-para-o-desafio-4) · [Pendências](#pendências)

---

## Contexto do tema

Uma central de atendimento (SAC) que registra o histórico de chamados por cliente e a nota de satisfação ao final. É um cenário direto para SLA, tempo de resolução e NPS/CSAT, temas já centrais na trilha de Suporte de TI e reaproveitáveis em Estatística.

---

## Modelo de dados

### Tabela 1: `clientes` (dimensão/cadastro)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `cliente_id` | inteiro | PK | Identificador único do cliente |
| `segmento` | texto | | `pessoa_fisica` · `pequena_empresa` · `grande_empresa` |
| `canal_preferido` | texto | | `telefone` · `chat` · `email` · `presencial` |
| `data_cadastro` | data | | Data de início do relacionamento |

### Tabela 2: `chamados` (fato/evento)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `chamado_id` | inteiro | PK | Identificador único do chamado |
| `cliente_id` | inteiro | FK → `clientes.cliente_id` | Cliente que abriu o chamado |
| `data_hora_abertura` | data/hora | | Abertura do chamado |
| `canal` | texto | | `telefone` · `chat` · `email` · `presencial` |
| `motivo` | texto | | `duvida` · `reclamacao` · `elogio` · `solicitacao_tecnica` · `cancelamento` |
| `tempo_atendimento_min` | inteiro | | Duração do atendimento em minutos (entre 1 e 180) |
| `nota_satisfacao` | inteiro | | Escala de 1 a 5 (pesquisa "como foi seu atendimento") |
| `resolvido_primeiro_contato` | booleano | | Indicador de FCR (First Call Resolution) |

---

## Cardinalidade

```text
clientes 1 ────< N chamados
```

Um cliente pode abrir vários chamados; cada chamado pertence a um único cliente.

---

## Volumetria

Mínimo do projeto: 20.000 linhas somadas. O script gera pelo menos 10.000 linhas por tabela.

| Tabela | Linhas geradas |
|:--|--:|
| `clientes` | 10.400 |
| `chamados` | 42.000 |
| **Total** | **52.400** |

Média de aproximadamente 4 chamados por cliente, entre 01/01/2025 e 01/08/2026.

---

## Estratégia de geração de dados mockados

**Distribuições definidas no script**

| Campo | Regra |
|:--|:--|
| `segmento` | `pessoa_fisica` 75% · `pequena_empresa` 20% · `grande_empresa` 5% |
| `canal_preferido` e `canal` | `chat` 35% · `telefone` 30% · `email` 25% · `presencial` 10% |
| `motivo` | `duvida` 35% · `reclamacao` 25% · `solicitacao_tecnica` 25% · `cancelamento` 10% · `elogio` 5% |
| `data_cadastro` | Entre 01/01/2019 e 01/07/2026 |

**Como as linhas são produzidas**

- `tempo_atendimento_min`: distribuição gama (média de 16 minutos), limitada entre 1 e 180.
- `resolvido_primeiro_contato`: verdadeiro quando o tempo de atendimento é menor que 1,3 vez a mediana **e** um sorteio de 85% é favorável. Resulta em cerca de 53% de chamados resolvidos no primeiro contato.
- `nota_satisfacao`: distribuição normal arredondada e limitada entre 1 e 5, com média 4,3 para chamados resolvidos e 2,6 para não resolvidos (desvio 0,8). A nota se relaciona com o tempo de forma indireta, via resolução: correlação de cerca de +0,72 com `resolvido_primeiro_contato` e de cerca de -0,47 com `tempo_atendimento_min`. Isso dá material real para análise de correlação no Desafio 2.

**Pontos de atenção para o Desafio 4**

- `canal` é sorteado de forma independente de `canal_preferido` (coincidem em cerca de 29% dos chamados), então a coluna do cliente não prevê o canal usado.
- `data_hora_abertura` é sorteada a partir de `data_cadastro`, então nenhum chamado é anterior ao cadastro do cliente. Essa regra foi corrigida na revisão do script.

---

## Expansão sugerida para o Desafio 4

- Tabela `atendentes` referenciada por `chamados.atendente_id`, que permite métricas de desempenho por atendente.
- Tabela `motivos` normalizando o campo `motivo` (hoje texto categórico de lista fechada) em catálogo com categoria macro (técnico/comercial/financeiro).

---

[Voltar ao índice](README.md)
