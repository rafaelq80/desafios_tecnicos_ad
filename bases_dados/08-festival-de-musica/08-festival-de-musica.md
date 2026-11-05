<p align="center">
  <img src="../../icones/08-festival-de-musica.svg" width="88" height="88" alt="Icone do tema Festival de musica">
</p>

<h1 align="center">Tema 8 &mdash; Festival de Música</h1>

<p align="center">
  <strong>Produtora de eventos</strong><br>
  Plano de modelagem da base de dados mockada
</p>

| Desafio de origem | Uso nos desafios | Situação | Gerador |
|:--|:--|:--|:--|
| Trabalho 1 (Python) | 1 · 2 · 3 · 4 · 5 | Plano validado contra o script; base gerada em CSV | `gerar_bases.py` (seed 42) |

**Nesta página:** [Contexto](#contexto-do-tema) · [Modelo de dados](#modelo-de-dados) · [Cardinalidade](#cardinalidade) · [Volumetria](#volumetria) · [Geração](#estratégia-de-geração-de-dados-mockados) · [Expansão](#expansão-sugerida-para-o-desafio-4) · [Pendências](#pendências)

---

## Contexto do tema

Uma produtora de eventos com festivais de música de 3 dias, atrações em diferentes palcos e venda de ingressos por tipo. É um cenário clássico de "público × receita", direto ao ponto do enunciado original (comparar audiência e faturamento por atração, gênero e dia).

---

## Modelo de dados

### Tabela 1: `atracoes` (dimensão/cadastro)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `atracao_id` | inteiro | PK | Identificador único da atração |
| `nome_artista_ficticio` | texto | | Nome fictício da banda/artista |
| `genero_musical` | texto | | `rock` · `pop` · `mpb` · `eletronica` · `funk` · `sertanejo` · `rap` · `forró` · `reggae` |
| `edicao_festival` | texto | | Edição do festival em que a atração se apresenta (ex.: "Festival X 2024") |
| `dia_apresentacao` | data | | Dia do festival em que se apresenta (dentro da janela de 3 dias da edição) |
| `palco` | texto | | `principal` · `secundario` · `eletronico` · `acustico` |
| `horario_inicio` | texto (`HH:MM`) | | Horário programado de início |

### Tabela 2: `ingressos_vendidos` (fato/evento)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `venda_id` | inteiro | PK | Identificador único da venda |
| `atracao_id` | inteiro | FK → `atracoes.atracao_id` | Atração associada ao ingresso (dia/lote vinculado à atração de referência) |
| `tipo_ingresso` | texto | | `pista` · `pista_premium` · `vip` · `camarote` |
| `canal_venda` | texto | | `site` · `app` · `bilheteria_fisica` · `revenda_parceira` |
| `data_hora_venda` | data/hora | | Momento da venda |
| `valor` | decimal | | Valor pago pelo ingresso, em reais |
| `forma_pagamento` | texto | | `cartao_credito` · `cartao_debito` · `pix` · `boleto` |

---

## Cardinalidade

```text
atracoes 1 ────< N ingressos_vendidos
```

Cada atração (usada aqui como proxy do "dia/lote" do festival) está associada a muitos ingressos vendidos; cada venda referencia uma atração.

> **Nota de modelagem.** Num festival real, o ingresso costuma dar acesso ao dia inteiro, não a uma atração específica. Para fins didáticos, a FK simplifica associando a venda a uma atração do dia, ponto que vale discutir em sala como simplificação proposital (ver `docs/specs/qualidade.md`, seção de simplificação).

---

## Volumetria

Mínimo do projeto: 20.000 linhas somadas. O script gera pelo menos 10.000 linhas por tabela.

| Tabela | Linhas geradas |
|:--|--:|
| `atracoes` | 10.100 |
| `ingressos_vendidos` | 55.000 |
| **Total** | **65.100** |

---

## Estratégia de geração de dados mockados

**Distribuições definidas no script**

| Campo | Regra |
|:--|:--|
| `genero_musical` | `pop` 16% · `rock`, `eletronica` e `sertanejo` 14% cada · `funk` 12% · `mpb` e `rap` 10% cada · `forró` e `reggae` 5% cada |
| `palco` | `secundario` 35% · `principal` 30% · `eletronico` 20% · `acustico` 15% |
| `tipo_ingresso` | `pista` 55% · `pista_premium` 20% · `vip` 18% · `camarote` 7% |
| `canal_venda` | `site` 40% · `app` 35% · `bilheteria_fisica` 15% · `revenda_parceira` 10% |
| `forma_pagamento` | `cartao_credito` 50% · `pix` 30% · `cartao_debito` 15% · `boleto` 5% |

**Como as linhas são produzidas**

- Existem **48 edições** do festival (6 por ano, de 2019 a 2026), com nome `Festival <palavra> <ano>`. Cada edição tem uma data de início e uma janela de 3 dias. A data de apresentação de cada atração é sorteada **dentro da janela da própria edição**, para que o nome da edição e a data sempre batam.
- `atracoes`: as 10.100 atrações se distribuem entre as 48 edições (cerca de 210 por edição). O nome do artista combina um prefixo (Os, As, DJ, Banda, Coletivo ou nenhum) com um sobrenome gerado por Faker. `horario_inicio` fica entre 12:00 e 23:45, em intervalos de 15 minutos, e **não se repete** dentro da mesma edição, dia e palco.
- `ingressos_vendidos`: a `atracao_id` é sorteada com viés de popularidade (distribuição de Pareto), de modo que poucas atrações concentram muitas vendas. Esse viés é multiplicado por um fator de palco (principal ×3, secundário ×1,5, eletrônico ×1, acústico ×0,7), então o palco principal vende mais: cerca de 52% das vendas, contra 30% do secundário, 12% do eletrônico e 5% do acústico.
- `valor` parte de um preço-base sorteado de forma uniforme por tipo (pista de 80 a 250, pista_premium de 200 a 450, vip de 400 a 900 e camarote de 800 a 2.000 reais) e é multiplicado por `1 + 0,5 × e^(−antecedência ÷ 30)`. Os lotes mais baratos saem primeiro e o preço sobe até 50% perto do dia do show. A correlação entre preço e antecedência é negativa (cerca de -0,3 em `pista`), o gancho estatístico do Desafio 2.
- `data_hora_venda` = dia da atração menos uma antecedência (distribuição exponencial com média de 25 dias, entre 0 e 200 dias), com horário sorteado entre 06:00 e 23:59.

> **Ajuste em relação ao plano original.** O plano previa 40 atrações em 4 dias. Para manter pelo menos 10.000 atrações cadastradas, o script gera 48 edições de 3 dias, com cerca de 210 atrações cada. As demais regras do plano (palco principal vende mais, preço sobe perto da data, horários sem repetição por palco) são garantidas pelo script revisado.

**Pontos de atenção para o Desafio 4**

- Como cada edição tem cerca de 210 atrações em 3 dias e 4 palcos, há até 38 atrações no mesmo palco e dia, com horários de início distintos, de 15 em 15 minutos. A duração da apresentação não é modelada, então "sem sobreposição" vale só para o horário de início.
- `horario_inicio` é texto no formato `HH:MM` e precisa ser convertido para hora nas análises.

---

## Expansão sugerida para o Desafio 4

- Tabela `dias_festival` normalizando `dia_apresentacao` em entidade própria, corrigindo a simplificação citada acima: o ingresso passa a se relacionar com o **dia**, e a atração vira uma tabela associativa (agenda), aproximando-se do cenário real.
- Tabela `edicoes` normalizando `edicao_festival` (hoje texto repetido em muitas linhas de `atracoes`) em dimensão própria com ano e data de início.
- Tabela `artistas` separando o artista (entidade estável) da atração (evento pontual em um festival específico), útil porque o mesmo artista pode aparecer em mais de uma edição.

---

[Voltar ao índice](../../README.md)
