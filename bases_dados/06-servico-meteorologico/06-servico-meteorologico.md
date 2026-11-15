<p align="center">
  <img src="../../icones/06-servico-meteorologico.svg" width="88" height="88" alt="Icone do tema Servico meteorologico">
</p>

<h1 align="center">Tema 6 &mdash; Serviço Meteorológico</h1>

<p align="center">
  <strong>Rede internacional de estações</strong><br>
  Plano de modelagem da base de dados mockada
</p>

| Desafio de origem | Uso nos desafios | Situação | Gerador |
|:--|:--|:--|:--|
| Trabalho 1 (Python) | 1 · 2 · 3 · 4 · 5 | Plano validado contra o script; base gerada em CSV | `gerar_bases.py` ou `gerar_bases_6_7.py` (seed 42; resultado idêntico) |

**Nesta página:** [Contexto](#contexto-do-tema) · [Modelo de dados](#modelo-de-dados) · [Cardinalidade](#cardinalidade) · [Volumetria](#volumetria) · [Geração](#estratégia-de-geração-de-dados-mockados) · [Expansão](#expansão-sugerida-para-o-desafio-4) · [Pendências](#pendências)

---

## Contexto do tema

Uma rede internacional de estações meteorológicas registrando leituras (condição do tempo, temperatura, umidade, chuva, vento e pressão). É um cenário de série temporal por natureza, ótimo para gráfico de linha, sazonalidade e correlação entre variáveis climáticas. Com a coluna categórica `condicao_tempo`, também permite comparar grupos (média por condição) e analisar frequência de eventos extremos.

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
| `leitura_id` | inteiro | PK | Identificador único da leitura (sequencial, em ordem cronológica) |
| `estacao_id` | inteiro | FK → `estacoes.estacao_id` | Estação que registrou a leitura |
| `data_hora` | data/hora | | Momento da leitura |
| `condicao_tempo` | texto | | Condição do tempo: `ensolarado` · `nublado` · `chuvoso` · `tempestade` |
| `temperatura_c` | decimal | | Temperatura em graus Celsius |
| `umidade_pct` | decimal | | Umidade relativa do ar, em % (entre 20 e 100) |
| `precipitacao_mm` | decimal | | Chuva acumulada, em milímetros (0 quando não chove) |
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

Período simulado: 01/01/2025 a 01/08/2026. Em média há cerca de 6 leituras por estação (de 1 a 16), em ordem cronológica.

> **Mudança importante.** As leituras **não são periódicas**. O script garante pelo menos uma leitura por estação e sorteia as demais, e o instante de cada uma, de forma independente, em vez de gerar uma leitura por hora (ou a cada 3 h) por estação, como o plano original supunha. A volumetria mínima de 10.000 estações torna inviável uma série horária completa. Para séries temporais, agregue por dia, mês, país ou faixa de latitude, e não por estação individual.

---

## Estratégia de geração de dados mockados

O tema 6 tem **gerador próprio** (numpy com semente `[42, 6]` e Faker com semente 48), independente dos outros temas. Por isso o resultado é o mesmo no script completo (`gerar_bases.py`) e no reduzido (`gerar_bases_6_7.py`), e dá para regerar só este tema sem alterar os demais.

**Distribuições definidas no script**

| Campo | Regra |
|:--|:--|
| `pais` | Brasil 30% · Estados Unidos 10% · Argentina, Canadá, Alemanha e França 6% cada · Portugal, Japão, Austrália e México 5% cada · África do Sul e Índia 4% cada · Reino Unido e Itália 3% cada · Chile 2% |
| `latitude` / `longitude` | Uniformes dentro de uma caixa aproximada de cada país |
| `altitude_m` | Uniforme entre 0 e 3.499 |
| `condicao_tempo` | `nublado` 50% · `ensolarado` 38% · `chuvoso` 9% · `tempestade` 3% |

**Como as linhas são produzidas**

A `condicao_tempo` é sorteada **primeiro** em cada leitura e "puxa" as demais variáveis, o que cria correlações realistas entre elas:

| Condição | Ajuste na temperatura | Umidade (média · desvio) | Vento (gama, média) | Ajuste na pressão | Chuva |
|:--|--:|:--|--:|--:|:--|
| `ensolarado` | +1,5 °C | 45 % · 12 | 10 km/h | +1 hPa | nenhuma |
| `nublado` | 0 °C | 65 % · 12 | 12 km/h | 0 hPa | nenhuma |
| `chuvoso` | −2,0 °C | 85 % · 7 | 14 km/h | −3 hPa | exponencial, média de 4 mm (mínimo 0,1 mm) |
| `tempestade` | −4,0 °C | 92 % · 5 | 28 km/h | −8 hPa | 5 mm + exponencial de média 7 mm (média de 12 mm, mínimo 5 mm) |

- `temperatura_c` = 25 − |latitude| ÷ 3 − 0,0065 × altitude (queda de cerca de 6,5 °C por km) + variação sazonal senoidal (amplitude 6 °C, **invertida entre hemisférios**) + variação diária senoidal (amplitude 5 °C, máxima por volta das 15 h, mínima de madrugada) + ajuste da condição (tabela acima) + ruído normal (desvio 2 °C). A temperatura depende de **latitude, altitude, época do ano, hora e condição do tempo**.
- `umidade_pct`: distribuição normal com média e desvio da condição, limitada entre 20 e 100.
- `velocidade_vento_kmh`: distribuição gama (forma 2) com escala 5, 6, 7 ou 14 conforme a condição (média geral de cerca de 12 km/h).
- `pressao_hpa` = normal com média 1013 e desvio 8 − 0,10 hPa por metro de altitude + ajuste da condição (tabela acima).
- `precipitacao_mm`: **só existe em leituras `chuvoso` e `tempestade`** (cerca de 12% das leituras, média de 6 mm entre as que têm chuva); nas demais é exatamente 0. Toda leitura `chuvoso` ou `tempestade` tem chuva maior que zero. Pico esparso e não uniforme, como no plano.

> **Mudança em relação à versão anterior.** Antes, umidade, vento, pressão e chuva eram sorteados de forma independente (umidade uniforme entre 20 e 100, chuva em 12% das leituras sem relação com as outras variáveis). Agora todas dependem de `condicao_tempo`, então há correlação real: tempestade tem pressão menor, umidade e vento maiores.

**Pontos de atenção para o Desafio 4**

- `cidade`, `pais`, `latitude` e `longitude` são coerentes entre si (por exemplo, estações do Japão têm cidades japonesas e coordenadas dentro do país). A caixa geográfica é aproximada, então algumas coordenadas podem cair no mar. Essas correções foram feitas na revisão do script.
- Estações em grande altitude têm pressão bem menor (a pressão média geral fica em torno de 840 hPa, porque a altitude é sorteada uniformemente até 3.499 m) e temperaturas mais baixas.
- A temperatura segue a estação do ano de cada hemisfério: janeiro é mais frio que julho no hemisfério norte, e o contrário no sul.
- Médias por condição (`GROUP BY condicao_tempo`) mostram o efeito esperado: `tempestade` tem a menor pressão e a maior umidade e velocidade de vento; `ensolarado` tem a menor umidade e a maior temperatura. Como a altitude domina a pressão, compare condições dentro da mesma faixa de altitude (ou da mesma estação) para enxergar o efeito de forma nítida.
- `condicao_tempo` e `precipitacao_mm` são redundantes em parte (chuva > 0 equivale a `chuvoso` ou `tempestade`). Vale lembrar disso ao escolher filtros e ao interpretar correlações.

**Validações automáticas do script**

O script confere, ao final, as regras deste tema: toda estação tem pelo menos uma leitura; leituras em ordem cronológica; latitude do Brasil entre −33 e 5; umidade entre 0 e 100; `condicao_tempo` só assume os 4 valores válidos; chuva só em `chuvoso`/`tempestade` (e toda leitura dessas condições tem chuva); e, na média, `tempestade` tem pressão menor e umidade e vento maiores que `ensolarado`.

---

## Expansão sugerida para o Desafio 4

- Tabela `alertas` (evento derivado: quando `precipitacao_mm` ou `velocidade_vento_kmh` ultrapassa um limiar, ou quando `condicao_tempo` é `tempestade`) relacionada a `leituras`, o que introduz uma segunda tabela de fato.
- Tabela `paises` (ou `regioes`) normalizando o agrupamento de `estacoes` por país, continente ou região.
- Tabela `condicoes_tempo` normalizando o campo `condicao_tempo` (hoje texto categórico em `leituras`) em catálogo com descrição e nível de severidade.

---

[Voltar ao índice](../../README.md)
