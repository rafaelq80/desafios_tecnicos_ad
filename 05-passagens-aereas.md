<p align="center">
  <img src="icones/05-passagens-aereas.svg" width="88" height="88" alt="Icone do tema Passagens aereas">
</p>

<h1 align="center">Tema 5 &mdash; Compra de Passagens Aéreas</h1>

<p align="center">
  <strong>Venda de passagens com companhias fictícias</strong><br>
  Plano de modelagem da base de dados mockada
</p>

| Desafio de origem | Uso nos desafios | Situação | Gerador |
|:--|:--|:--|:--|
| Trabalho 1 (Python) | 1 · 2 · 3 · 4 · 5 | Plano validado contra o script; base gerada em CSV | `gerar_bases.py` (seed 42) |

**Nesta página:** [Contexto](#contexto-do-tema) · [Modelo de dados](#modelo-de-dados) · [Cardinalidade](#cardinalidade) · [Volumetria](#volumetria) · [Geração](#estratégia-de-geração-de-dados-mockados) · [Expansão](#expansão-sugerida-para-o-desafio-4) · [Pendências](#pendências)

---

## Contexto do tema

Um serviço de venda de passagens com voos cadastrados e reservas feitas por passageiros. O cenário tem boa variação geográfica (origem e destino) e financeira (preço por classe), útil para mapas e métricas de receita.

---

## Modelo de dados

### Tabela 1: `voos` (dimensão/cadastro)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `voo_id` | inteiro | PK | Identificador único do voo |
| `origem` | texto | | Nome da cidade de origem (20 cidades brasileiras reais) |
| `destino` | texto | | Nome da cidade de destino (sempre diferente da origem) |
| `companhia_aerea` | texto | | Nome fictício de companhia (5 companhias) |
| `data_hora_partida` | data/hora | | Horário programado de partida |
| `data_hora_chegada` | data/hora | | Horário programado de chegada |
| `capacidade_assentos` | inteiro | | Total de assentos da aeronave |

### Tabela 2: `reservas` (fato/evento)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `reserva_id` | inteiro | PK | Identificador único da reserva |
| `voo_id` | inteiro | FK → `voos.voo_id` | Voo reservado |
| `passageiro_id` | inteiro | | Identificador fictício do passageiro |
| `data_compra` | data | | Data em que a passagem foi comprada |
| `classe` | texto | | `economica` · `premium` · `executiva` |
| `valor_pago` | decimal | | Valor pago pela passagem, em reais |
| `forma_pagamento` | texto | | `cartao_credito` · `cartao_debito` · `pix` · `boleto` |
| `status` | texto | | `confirmada` · `cancelada` · `reembolsada` |

---

## Cardinalidade

```text
voos 1 ────< N reservas
```

Um voo tem muitas reservas (sempre abaixo do limite de `capacidade_assentos`, ver abaixo); cada reserva pertence a um único voo.

---

## Volumetria

Mínimo do projeto: 20.000 linhas somadas. O script gera pelo menos 10.000 linhas por tabela.

| Tabela | Linhas geradas |
|:--|--:|
| `voos` | 10.600 |
| `reservas` | 55.000 |
| **Total** | **65.600** |

Média de aproximadamente 5 reservas por voo.

---

## Estratégia de geração de dados mockados

**Distribuições definidas no script**

| Campo | Regra |
|:--|:--|
| `origem` / `destino` | Uniforme entre 20 cidades (São Paulo, Rio de Janeiro, Brasília, Salvador, Belo Horizonte, Fortaleza, Recife, Porto Alegre, Curitiba, Manaus, Belém, Goiânia, Florianópolis, Natal, Vitória, Campo Grande, Cuiabá, João Pessoa, Maceió, Teresina) |
| `companhia_aerea` | Azul Céu 30% · Rota Sul 25% · TransBrasil Air 20% · Voe Fácil 15% · LatinWings 10% |
| `capacidade_assentos` | 150 (30%) · 120 (25%) · 180 (25%) · 220 (12%) · 300 (8%) |
| `data_hora_partida` | Entre 01/06/2025 e 31/12/2026 |
| `classe` | `economica` 70% · `premium` 20% · `executiva` 10% |
| `forma_pagamento` | `cartao_credito` 55% · `pix` 25% · `cartao_debito` 15% · `boleto` 5% |
| `status` | `confirmada` 88% · `cancelada` 8% · `reembolsada` 4% |

**Como as linhas são produzidas**

- `voos`: a cidade de destino é sempre diferente da de origem. A duração do voo é sorteada entre 45 e 299 minutos e somada à partida para obter a chegada. Cidades reais são usadas apenas como contexto geográfico (cidade não é dado pessoal), e as rotas têm a mesma frequência entre si.
- `reservas`: o `voo_id` é sorteado uniformemente, o que dá no máximo 17 reservas por voo, bem abaixo da menor capacidade (120 assentos). O script não impõe o limite de capacidade de forma explícita; ele é respeitado na prática pela volumetria.
- `data_compra` = data da partida menos uma antecedência (distribuição exponencial com média de 20 dias, entre 0 e 300 dias).
- `valor_pago` = preço-base sorteado por classe × fator de antecedência `1 + 1/(antecedência + 3)`. Faixas-base: econômica de 180 a 700, premium de 600 a 1.500, executiva de 1.400 a 4.500 reais. Compra em cima da hora encarece a passagem em até 33%, um efeito discreto (correlação de cerca de -0,14 com a antecedência dentro da classe econômica), mas suficiente para o gancho de correlação no Desafio 2.

**Pontos de atenção para o Desafio 4**

- `valor_pago` está preenchido também nas reservas `cancelada` e `reembolsada`, então somar receita exige filtrar por `status`.
- Há voos com partida em datas futuras (até 31/12/2026), e compras com `data_compra` correspondente.

---

## Expansão sugerida para o Desafio 4

- Tabela `passageiros` normalizando `passageiro_id` em dimensão com programa de fidelidade.
- Tabela `companhias` normalizando `companhia_aerea` (hoje texto categórico na tabela `voos`) em catálogo próprio.

---

## Pendências

- `[A VERIFICAR]` o script já usa nomes de cidades reais (contexto geográfico, não dado pessoal). Confirmar com o instrutor se isso é aceitável ou se ele prefere nomes 100% fictícios para reforçar que a base é mockada.

---

[Voltar ao índice](README.md)
