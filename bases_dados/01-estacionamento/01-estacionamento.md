<p align="center">
  <img src="../../icones/01-estacionamento.svg" width="88" height="88" alt="Icone do tema Estacionamento">
</p>

<h1 align="center">Tema 1 &mdash; Estacionamento</h1>

<p align="center">
  <strong>Rede ParkTotal</strong><br>
  Plano de modelagem da base de dados mockada
</p>

| Desafio de origem | Uso nos desafios | Situação | Gerador |
|:--|:--|:--|:--|
| Trabalho 1 (Python) | 1 · 2 · 3 · 4 · 5 | Plano validado contra o script; base gerada em CSV | `gerar_bases.py` (seed 42) |

**Nesta página:** [Contexto](#contexto-do-tema) · [Modelo de dados](#modelo-de-dados) · [Cardinalidade](#cardinalidade) · [Volumetria](#volumetria) · [Geração](#estratégia-de-geração-de-dados-mockados) · [Expansão](#expansão-sugerida-para-o-desafio-4) · [Pendências](#pendências)

---

## Contexto do tema

Uma rede de estacionamentos de médio porte (shoppings e hospitais), com **51 unidades**. Cada unidade tem vagas divididas por setor e tipo, e cada entrada e saída de veículo é registrada. O caso é simples o bastante para começar, mas já traz cálculo de tempo e de valor, o que o torna bom terreno para a Estatística e para as fórmulas de data do Excel.

---

## Modelo de dados

### Tabela 1: `vagas` (dimensão/cadastro)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `vaga_id` | inteiro | PK | Identificador único da vaga |
| `unidade_id` | inteiro | | Unidade da rede à qual a vaga pertence (1 a 51) |
| `cidade_unidade` | texto | | Cidade da unidade (uma cidade por unidade, nome gerado por Faker) |
| `setor` | texto | | Setor do estacionamento (`A` a `F`) |
| `andar` | inteiro | | Andar ou nível (0 = térreo, até 5) |
| `tipo_vaga` | texto | | `comum` · `idoso` · `pcd` · `moto` · `eletrico` |
| `tarifa_hora` | decimal | | Valor cobrado por hora, em reais (entre 3,00 e 15,00) |

### Tabela 2: `movimentacoes` (fato/evento)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `movimentacao_id` | inteiro | PK | Identificador único da movimentação |
| `vaga_id` | inteiro | FK → `vagas.vaga_id` | Vaga ocupada |
| `placa_veiculo` | texto | | Placa fictícia (padrão Mercosul, `LLLNLNN`) |
| `tipo_veiculo` | texto | | `carro` · `moto` · `utilitario` |
| `data_hora_entrada` | data/hora | | Entrada do veículo |
| `data_hora_saida` | data/hora | | Saída do veículo (vazio/nulo se ainda estacionado) |
| `valor_pago` | decimal | | Calculado a partir do tempo e da `tarifa_hora` (0 se ainda estacionado) |
| `forma_pagamento` | texto | | `dinheiro` · `cartao` · `pix` · `app` |

---

## Cardinalidade

```text
vagas 1 ────< N movimentacoes
```

Uma vaga tem muitas movimentações ao longo do tempo; cada movimentação pertence a exatamente uma vaga.

---

## Volumetria

Mínimo do projeto: 20.000 linhas somadas. O script gera pelo menos 10.000 linhas por tabela.

| Tabela | Linhas geradas |
|:--|--:|
| `vagas` | 10.200 |
| `movimentacoes` | 42.000 |
| **Total** | **52.200** |

Média de aproximadamente 4 movimentações por vaga, distribuídas entre 01/01/2025 e 01/08/2026.

---

## Estratégia de geração de dados mockados

**Distribuições definidas no script**

| Campo | Regra |
|:--|:--|
| `unidade_id` | Sorteio uniforme entre 51 unidades |
| `setor` | Uniforme entre A, B, C, D, E e F |
| `tipo_vaga` | `comum` 72% · `moto` 10% · `idoso` 8% · `pcd` 6% · `eletrico` 4% |
| `tarifa_hora` | Uniforme entre 3,00 e 15,00, fixa por vaga |
| `tipo_veiculo` | Depende do tipo de vaga: em vaga `moto`, sempre `moto`; nas demais, `carro` 94% · `utilitario` 6%. No total: `carro` ~85% · `moto` ~10% · `utilitario` ~5% |
| `forma_pagamento` | `cartao` 35% · `pix` 35% · `app` 20% · `dinheiro` 10% |

**Como as linhas são produzidas**

- `vagas`: sorteio aleatório (não determinístico) de unidade, setor, andar, tipo e tarifa para cada uma das 10.200 vagas.
- `movimentacoes`: para cada linha, sorteia-se a `vaga_id` e uma `data_hora_entrada` entre 01/01/2025 e 01/08/2026. A duração segue uma distribuição gama (média de 2 horas), limitada entre 10 minutos e 24 horas, o que dá uma maioria entre 30 min e 4 h e cauda longa. A saída é a entrada somada à duração.
- `valor_pago` = duração em horas × `tarifa_hora` da vaga, com 2 casas decimais. A cobrança é proporcional ao tempo, sem tolerância nem arredondamento para a hora seguinte.
- Cerca de 1,5% das movimentações simulam veículo ainda estacionado: `data_hora_saida` vazia e `valor_pago` igual a 0.
- `placa_veiculo`: formato Mercosul sorteado letra a letra, sem relação com placas reais.

**Pontos de atenção para o Desafio 4**

- `tipo_veiculo` é coerente com `tipo_vaga` (moto só em vaga de moto, e carro ou utilitário nas demais). Essa regra foi corrigida na revisão do script.
- `data_hora_saida` pode ultrapassar 01/08/2026, pois a duração é somada à entrada.
- `cidade_unidade` depende de `unidade_id` (dependência que a normalização resolve, ver expansão abaixo).
- A mesma placa pode aparecer em linhas diferentes por acaso; não há cadastro de veículos.

---

## Expansão sugerida para o Desafio 4

- Tabela `unidades` normalizando `unidade_id` e `cidade_unidade`, que hoje se repetem em `vagas` (cada unidade aparece em cerca de 200 vagas).
- Tabela `funcionarios` (operador do caixa) referenciada por `movimentacoes.funcionario_id`, que adiciona uma segunda dimensão e permite métricas por operador.
- Tabela `mensalistas` (vaga fixa associada a uma pessoa) com relação 1:1 opcional com `vagas`.

---

[Voltar ao índice](../../README.md)
