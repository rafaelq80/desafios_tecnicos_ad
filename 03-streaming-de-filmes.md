<p align="center">
  <img src="icones/03-streaming-de-filmes.svg" width="88" height="88" alt="Icone do tema Streaming de filmes">
</p>

<h1 align="center">Tema 3 &mdash; Streaming de Filmes</h1>

<p align="center">
  <strong>StreamPlay</strong><br>
  Plano de modelagem da base de dados mockada
</p>

| Desafio de origem | Uso nos desafios | Situação | Gerador |
|:--|:--|:--|:--|
| Trabalho 1 (Python) | 1 · 2 · 3 · 4 · 5 | Plano validado contra o script; base gerada em CSV | `gerar_bases.py` (seed 42) |

**Nesta página:** [Contexto](#contexto-do-tema) · [Modelo de dados](#modelo-de-dados) · [Cardinalidade](#cardinalidade) · [Volumetria](#volumetria) · [Geração](#estratégia-de-geração-de-dados-mockados) · [Expansão](#expansão-sugerida-para-o-desafio-4) · [Pendências](#pendências)

---

## Contexto do tema

Um catálogo de filmes com histórico de visualizações por usuário. É um cenário clássico de recomendação e consumo de conteúdo, com boa variedade de dimensões (gênero, ano, duração) para gráficos e tabelas dinâmicas.

---

## Modelo de dados

### Tabela 1: `filmes` (dimensão/catálogo)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `filme_id` | inteiro | PK | Identificador único do filme |
| `titulo` | texto | | Título fictício e genérico (não usar título de filme real protegido por direito autoral em material publicado) |
| `genero` | texto | | `ação` · `comédia` · `drama` · `terror` · `documentário` · `animação` · `ficção científica` · `romance` |
| `ano_lancamento` | inteiro | | Ano fictício de lançamento (entre 1985 e 2024) |
| `duracao_min` | inteiro | | Duração em minutos (entre 75 e 184) |
| `classificacao_indicativa` | texto | | `L` · `10` · `12` · `14` · `16` · `18` |
| `diretor_ficticio` | texto | | Nome fictício de diretor (gerado por Faker) |

### Tabela 2: `visualizacoes` (fato/evento)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `visualizacao_id` | inteiro | PK | Identificador único da visualização |
| `filme_id` | inteiro | FK → `filmes.filme_id` | Filme assistido |
| `usuario_id` | inteiro | | Identificador fictício de usuário (sem dado pessoal) |
| `data_hora_inicio` | data/hora | | Início da sessão de visualização |
| `tempo_assistido_min` | inteiro | | Minutos efetivamente assistidos |
| `dispositivo` | texto | | `smart_tv` · `celular` · `computador` · `tablet` · `console` |
| `concluido` | booleano | | Se o usuário assistiu até o fim |

---

## Cardinalidade

```text
filmes 1 ────< N visualizacoes
```

Um filme é assistido muitas vezes; cada visualização é de exatamente um filme.

---

## Volumetria

Mínimo do projeto: 20.000 linhas somadas. O script gera pelo menos 10.000 linhas por tabela.

| Tabela | Linhas geradas |
|:--|--:|
| `filmes` | 10.300 |
| `visualizacoes` | 55.000 |
| **Total** | **65.300** |

---

## Estratégia de geração de dados mockados

**Distribuições definidas no script**

| Campo | Regra |
|:--|:--|
| `genero` | `drama` 18% · `ação` 16% · `comédia` 15% · `ficção científica` 12% · `animação` 11% · `terror` 10% · `romance` 10% · `documentário` 8% |
| `classificacao_indicativa` | `L` 25% · `12` 20% · `14` 20% · `10` 15% · `16` 12% · `18` 8% |
| `dispositivo` | `smart_tv` 40% · `celular` 30% · `computador` 15% · `tablet` 10% · `console` 5% |
| `usuario_id` | Sorteio uniforme entre 1 e 59.999 (cerca de 36 mil usuários distintos aparecem) |
| `data_hora_inicio` | Entre 01/01/2025 e 01/08/2026 |

**Como as linhas são produzidas**

- `filmes`: o título combina um adjetivo e um substantivo de listas genéricas (ex.: "Sombra Horizonte"), o que evita qualquer título real. Como há poucas combinações possíveis, repetições recebem um número ao final, e todos os 10.300 títulos saem únicos.
- `ano_lancamento` é sempre anterior a 2025, ou seja, anterior à janela de visualizações.
- `visualizacoes`: o `filme_id` é sorteado com viés de popularidade (distribuição de Pareto, tipo long tail): os 10% de filmes mais assistidos concentram cerca de 45% das visualizações.
- `tempo_assistido_min` = duração do filme × fração assistida (distribuição Beta, concentrada em valores altos), com mínimo de 1 minuto. Nunca ultrapassa a duração do filme; em poucos casos é exatamente igual a ela.
- `concluido` é verdadeiro quando `tempo_assistido_min` é de pelo menos 90% de `duracao_min`, regra que o aluno pode conferir diretamente nos dados.

---

## Expansão sugerida para o Desafio 4

- Tabela `usuarios` normalizando `usuario_id` (hoje sem atributos) em uma dimensão com plano de assinatura e data de cadastro.
- Tabela `avaliacoes` (nota do usuário por filme) com relação N:N entre `usuarios` e `filmes`.

---

[Voltar ao índice](README.md)
