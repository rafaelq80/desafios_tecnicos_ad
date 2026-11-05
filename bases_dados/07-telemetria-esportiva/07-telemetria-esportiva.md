<p align="center">
  <img src="../../icones/07-telemetria-esportiva.svg" width="88" height="88" alt="Icone do tema Telemetria esportiva">
</p>

<h1 align="center">Tema 7 &mdash; Telemetria Esportiva (Corridas)</h1>

<p align="center">
  <strong>Simulador de corrida online</strong><br>
  Plano de modelagem da base de dados mockada
</p>

| Desafio de origem | Uso nos desafios | Situação | Gerador |
|:--|:--|:--|:--|
| Trabalho 1 (Python) | 1 · 2 · 3 · 4 · 5 | Plano validado contra o script; base gerada em CSV | `gerar_bases.py` (seed 42) |

**Nesta página:** [Contexto](#contexto-do-tema) · [Modelo de dados](#modelo-de-dados) · [Cardinalidade](#cardinalidade) · [Volumetria](#volumetria) · [Geração](#estratégia-de-geração-de-dados-mockados) · [Expansão](#expansão-sugerida-para-o-desafio-4) · [Pendências](#pendências)

---

## Contexto do tema

Dados de telemetria de um simulador de corrida online: pilotos cadastrados e o desempenho volta a volta em cada corrida. É um cenário rico para análise de performance (tempo de volta, consistência, consumo de combustível), tema já familiar a quem gosta de esportes a motor.

---

## Modelo de dados

### Tabela 1: `pilotos` (dimensão/cadastro)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `piloto_id` | inteiro | PK | Identificador único do piloto |
| `nome_ficticio` | texto | | Gamertag fictícia (ex.: `FastPro123`) |
| `equipe` | texto | | Nome fictício da equipe (formato "Equipe X Racing") |
| `categoria` | texto | | `kart` · `formula_sim` · `stock_sim` · `rally_sim` |
| `pais` | texto | | País de origem (real, sem dado pessoal associado) |

### Tabela 2: `voltas` (fato/evento)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `volta_id` | inteiro | PK | Identificador único do registro de volta |
| `piloto_id` | inteiro | FK → `pilotos.piloto_id` | Piloto que completou a volta |
| `corrida_id` | inteiro | | Identificador da corrida/evento (de 1 a 1.100) |
| `numero_volta` | inteiro | | Número sequencial da volta dentro da corrida (de 1 a 5) |
| `tempo_volta_seg` | decimal | | Tempo da volta, em segundos |
| `velocidade_media_kmh` | decimal | | Velocidade média na volta |
| `combustivel_restante_pct` | decimal | | Combustível restante ao final da volta, em % |
| `posicao_na_volta` | inteiro | | Posição do piloto ao cruzar a linha naquela volta (de 1 a 10) |

---

## Cardinalidade

```text
pilotos 1 ────< N voltas
```

Um piloto participa de muitas voltas; cada registro de volta pertence a um único piloto.

---

## Volumetria

Mínimo do projeto: 20.000 linhas somadas. O script gera pelo menos 10.000 linhas por tabela.

| Tabela | Linhas geradas |
|:--|--:|
| `pilotos` | 10.800 |
| `voltas` | 55.000 |
| **Total** | **65.800** |

As 55.000 voltas formam 1.100 corridas, cada uma com 10 pilotos e 5 voltas por piloto (1.100 × 10 × 5). Cerca de 6.900 dos 10.800 pilotos participam de pelo menos uma corrida.

---

## Estratégia de geração de dados mockados

**Distribuições definidas no script**

| Campo | Regra |
|:--|:--|
| `categoria` | `kart` 30% · `formula_sim` 30% · `stock_sim` 25% · `rally_sim` 15% |
| `pais` | Brasil 35% · Estados Unidos 10% · Alemanha 9% · Argentina e Itália 8% cada · Japão e França 7% cada · Portugal 6% · México e Espanha 5% cada |
| `equipe` | Sorteio entre até 400 nomes gerados (podem se repetir por acaso) |

**Como as linhas são produzidas**

- Cada corrida sorteia uma **categoria** e 10 pilotos **dessa categoria**, sem repetição. Todos os pilotos de uma corrida correm a mesma categoria.
- `tempo_volta_seg` = tempo-base da pista (kart 55 s, formula_sim 85 s, stock_sim 95 s, rally_sim 110 s, com variação de 4 s por corrida) + habilidade do piloto naquela corrida (desvio 1,5 s) + ruído por volta (desvio 1,2 s), limitado entre 40 e 160 s.
- `velocidade_media_kmh` = velocidade de referência da categoria (kart 70, formula_sim 180, stock_sim 150, rally_sim 100 km/h) × tempo-base da pista ÷ `tempo_volta_seg`. Quanto menor o tempo, maior a velocidade.
- `combustivel_restante_pct` = 100 − `numero_volta` × consumo por volta do piloto (sorteado entre 6 e 9 por corrida). Sempre decresce volta a volta dentro de piloto + corrida, terminando entre 55% e 70% na volta 5.
- `posicao_na_volta` é o ranking do tempo acumulado dos pilotos na corrida até aquela volta, então não há posições repetidas na mesma volta.

> **Ajuste em relação ao plano original.** O plano previa 120 pilotos e cerca de 50 corridas de 50 voltas. Para manter pelo menos 10.000 pilotos cadastrados e 55.000 voltas, o script gera 1.100 corridas curtas de 5 voltas, com 10 pilotos cada. A propriedade central do plano (combustível decrescente e monotônico dentro da corrida) é garantida pelo script revisado.

**Pontos de atenção para o Desafio 4**

- Quase 4.000 pilotos cadastrados não têm nenhuma volta registrada, e a maioria dos que têm participa de poucas corridas (em média 8 voltas, ou cerca de 1,6 corrida). Um `LEFT JOIN` a partir de `pilotos` mostra esse efeito.
- `corrida_id` ainda não tem pista, data ou categoria própria; essas informações só aparecem de forma indireta, pela categoria dos pilotos. A normalização em `corridas` (abaixo) resolve isso.

---

## Expansão sugerida para o Desafio 4

- Tabela `corridas` normalizando `corrida_id` (hoje solto em `voltas`) em dimensão própria com pista, data e categoria. Hoje é um atributo, e vira entidade.
- Tabela `equipes` normalizando o campo `equipe` (hoje texto categórico em `pilotos`) em catálogo com país-sede e ano de fundação fictício.

---

[Voltar ao índice](../../README.md)
