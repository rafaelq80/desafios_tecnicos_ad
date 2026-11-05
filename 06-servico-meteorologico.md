<p align="center">
  <img src="icones/06-servico-meteorologico.svg" width="88" height="88" alt="Icone do tema Servico meteorologico">
</p>

<h1 align="center">Tema 6 &mdash; Serviço Meteorológico</h1>

<p align="center">
  <strong>Rede internacional de estações</strong><br>
  Plano de modelagem da base de dados mockada
</p>

| Desafio de origem | Uso nos desafios | Situação | Gerador |
|:--|:--|:--|:--|
| Trabalho 1 (Python) | 1 · 2 · 3 · 4 · 5 | Plano validado contra o script; base gerada em CSV | `gerar_bases.py` (seed 42) |

**Nesta página:** [Contexto](#contexto-do-tema) · [Modelo de dados](#modelo-de-dados) · [Cardinalidade](#cardinalidade) · [Volumetria](#volumetria) · [Geração](#estratégia-de-geração-de-dados-mockados) · [Expansão](#expansão-sugerida-para-o-desafio-4) · [Pendências](#pendências)

---

## Contexto do tema

Uma rede internacional de estações meteorológicas registrando leituras (temperatura, umidade, chuva, vento e pressão). É um cenário de série temporal por natureza, ótimo para gráfico de linha, sazonalidade e correlação entre variáveis climáticas.

> **Ajuste em relação ao plano original.** O plano previa estações distribuídas por UF e regiões do Brasil. O script gera uma rede **internacional**, com a coluna `pais` no lugar de `uf` (ver tabela abaixo).

---

## Modelo de dados

### Tabela 1: `estacoes` (dimensão/cadastro)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `estacao_id` | inteiro | PK | Identificador único da estação |
| `nome_estacao` | texto | | Código da estação (formato `EST-00001`) |
| `cidade` | texto | | Cidade da estação, coerente com o país (Brasil: nomes gerados por Faker; demais países: lista de 5 cidades por país) |
| `pais` | texto | | País da estação (15 países) |
| `latitude` | decimal | | Coordenada geográfica, dentro de uma caixa aproximada do país |
| `longitude` | decimal | | Coordenada geográfica, dentro de uma caixa aproximada do país |
| `altitude_m` | inteiro | | Altitude em metros (entre 0 e 3.499) |

### Tabela 2: `leituras` (fato/evento)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `leitura_id` | inteiro | PK | Identificador único da leitura |
| `estacao_id` | inteiro | FK → `estacoes.estacao_id` | Estação que registrou a leitura |
| `data_hora` | data/hora | | Momento da leitura |
| `temperatura_c` | decimal | | Temperatura em graus Celsius |
| `umidade_pct` | decimal | | Umidade relativa do ar, em % |
| `precipitacao_mm` | decimal | | Chuva acumulada, em milímetros |
| `velocidade_vento_kmh` | decimal | | Velocidade do vento, em km/h |
| `pressao_hpa` | decimal | | Pressão atmosférica, em hPa |

---

## Cardinalidade

```text
estacoes 1 ────< N leituras
```

Uma estação gera várias leituras ao longo do tempo; cada leitura pertence a uma única estação.

---

## Volumetria

Mínimo do projeto: 20.000 linhas somadas. O script gera pelo menos 10.000 linhas por tabela.

| Tabela | Linhas geradas |
|:--|--:|
| `estacoes` | 10.050 |
| `leituras` | 62.000 |
| **Total** | **72.050** |

Período simulado: 01/01/2025 a 01/08/2026. Em média há cerca de 6 leituras por estação (de 1 a cerca de 16), em ordem cronológica.

> **Mudança importante.** As leituras **não são periódicas**. O script garante pelo menos uma leitura por estação e sorteia as demais, e o instante de cada uma, de forma independente, em vez de gerar uma leitura por hora (ou a cada 3 h) por estação, como o plano original supunha. A volumetria mínima de 10.000 estações torna inviável uma série horária completa. Para séries temporais, agregue por dia, mês, país ou faixa de latitude, e não por estação individual.

---

## Estratégia de geração de dados mockados

**Distribuições definidas no script**

| Campo | Regra |
|:--|:--|
| `pais` | Brasil 30% · Estados Unidos 10% · Argentina, Canadá, Alemanha e França 6% cada · Portugal, Japão, Austrália e México 5% cada · África do Sul e Índia 4% cada · Reino Unido e Itália 3% cada · Chile 2% |
| `latitude` / `longitude` | Uniformes dentro de uma caixa aproximada de cada país |
| `altitude_m` | Uniforme entre 0 e 3.499 |
| `umidade_pct` | Uniforme entre 20 e 100 |
| `velocidade_vento_kmh` | Distribuição gama (média de 12 km/h) |
| `pressao_hpa` | Normal com média 1013 e desvio 8, reduzida em 0,10 hPa por metro de altitude |

**Como as linhas são produzidas**

- `temperatura_c` = 25 − |latitude| ÷ 3 − 0,0065 × altitude (queda de cerca de 6,5 °C por km) + variação sazonal senoidal (amplitude 6 °C, **invertida entre hemisférios**) + variação diária senoidal (amplitude 5 °C, máxima por volta das 15 h, mínima de madrugada) + ruído normal (desvio 2 °C). A temperatura depende de **latitude, altitude, época do ano e hora**.
- `precipitacao_mm`: em cerca de 12% das leituras há chuva (valor exponencial, média de 6 mm); nas demais é zero. Pico esparso e não uniforme, como no plano.

**Pontos de atenção para o Desafio 4**

- `cidade`, `pais`, `latitude` e `longitude` são coerentes entre si (por exemplo, estações do Japão têm cidades japonesas e coordenadas dentro do país). A caixa geográfica é aproximada, então algumas coordenadas podem cair no mar. Essas correções foram feitas na revisão do script.
- Estações em grande altitude têm pressão bem menor (a pressão média geral fica em torno de 840 hPa, porque a altitude é sorteada uniformemente até 3.499 m) e temperaturas mais baixas.
- A temperatura segue a estação do ano de cada hemisfério: janeiro é mais frio que julho no hemisfério norte, e o contrário no sul.

---

## Expansão sugerida para o Desafio 4

- Tabela `alertas` (evento derivado: quando `precipitacao_mm` ou `velocidade_vento_kmh` ultrapassa um limiar) relacionada a `leituras`, o que introduz uma segunda tabela de fato.
- Tabela `paises` (ou `regioes`) normalizando o agrupamento de `estacoes` por país, continente ou região.

---

[Voltar ao índice](README.md)
