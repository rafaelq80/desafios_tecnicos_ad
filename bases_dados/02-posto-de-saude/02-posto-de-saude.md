<p align="center">
  <img src="../../icones/02-posto-de-saude.svg" width="88" height="88" alt="Icone do tema Posto de saude">
</p>

<h1 align="center">Tema 2 &mdash; Posto de Saúde (Triagem)</h1>

<p align="center">
  <strong>Rede VidaMais Saúde</strong><br>
  Plano de modelagem da base de dados mockada
</p>

| Desafio de origem | Uso nos desafios | Situação | Gerador |
|:--|:--|:--|:--|
| Trabalho 1 (Python) | 1 · 2 · 3 · 4 · 5 | Plano validado contra o script; base gerada em CSV | `gerar_bases.py` (seed 42) |

**Nesta página:** [Aviso](#aviso-de-dado-sensível) · [Contexto](#contexto-do-tema) · [Modelo de dados](#modelo-de-dados) · [Cardinalidade](#cardinalidade) · [Volumetria](#volumetria) · [Geração](#estratégia-de-geração-de-dados-mockados) · [Expansão](#expansão-sugerida-para-o-desafio-4) · [Pendências](#pendências)

---

## Aviso de dado sensível

> **Atenção.** Este tema simula dado de saúde, categoria sensível mesmo quando fictício. Todo `paciente_id`, nome e sintoma deve ser **gerado**, nunca copiado de caso real, prontuário ou paciente conhecido. Nenhum aluno deve popular este tema com dado de gente de verdade (nem parente, nem colega). Regra do projeto: `.claude/rules/etica-privacidade-e-agentes.md`.

---

## Contexto do tema

Uma rede de unidades básicas de saúde (postos) que recebe pacientes e aplica triagem por prioridade antes do atendimento. É um cenário clássico de fila com prioridade, bom para ensinar classificação e tempo de espera.

**Padrão de referência:** Protocolo de Manchester (Sistema de Triagem de Manchester), com 5 níveis de prioridade por cor, usado amplamente em unidades de saúde no Brasil e em outros países. `[A VERIFICAR]` os tempos-alvo de espera por cor variam por instituição; os valores abaixo são de referência didática, não um SLA oficial de rede específica.

| Cor | Prioridade | Tempo-alvo de espera (referência) |
|:--|:--|:--|
| Vermelho | Emergência | Imediato |
| Laranja | Muito urgente | até 10 min |
| Amarelo | Urgente | até 60 min |
| Verde | Pouco urgente | até 120 min |
| Azul | Não urgente | até 240 min |

---

## Modelo de dados

### Tabela 1: `pacientes` (dimensão/cadastro)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `paciente_id` | inteiro | PK | Identificador único (nunca CPF real) |
| `nome_ficticio` | texto | | Nome gerado por Faker, nunca de pessoa real |
| `data_nascimento` | data | | Usada para calcular idade |
| `sexo` | texto | | `M` · `F` · `outro` |
| `bairro` | texto | | Bairro fictício de residência (gerado por Faker) |
| `tipo_convenio` | texto | | `SUS` · `particular` · `convenio` |

### Tabela 2: `atendimentos` (fato/evento)

| Coluna | Tipo | Chave | Descrição |
|:--|:--|:--|:--|
| `atendimento_id` | inteiro | PK | Identificador único do atendimento |
| `paciente_id` | inteiro | FK → `pacientes.paciente_id` | Paciente atendido |
| `data_hora_chegada` | data/hora | | Chegada à unidade |
| `sintoma_principal` | texto | | Queixa relatada (lista fechada de 10 sintomas comuns) |
| `nivel_triagem` | texto | | `vermelho` · `laranja` · `amarelo` · `verde` · `azul` |
| `tempo_espera_min` | inteiro | | Minutos entre chegada e início do atendimento |
| `profissional_responsavel` | texto | | Cargo fictício numerado (ex.: "Enfermeiro 03"), nunca nome real |
| `desfecho` | texto | | `alta` · `encaminhado` · `internado` · `nao_compareceu` |

---

## Cardinalidade

```text
pacientes 1 ────< N atendimentos
```

Um paciente pode ter vários atendimentos ao longo do tempo simulado; cada atendimento pertence a um único paciente.

---

## Volumetria

Mínimo do projeto: 20.000 linhas somadas. O script gera pelo menos 10.000 linhas por tabela.

| Tabela | Linhas geradas |
|:--|--:|
| `pacientes` | 10.500 |
| `atendimentos` | 42.000 |
| **Total** | **52.500** |

Média de aproximadamente 4 atendimentos por paciente, entre 01/01/2025 e 01/08/2026.

---

## Estratégia de geração de dados mockados

**Distribuições definidas no script**

| Campo | Regra |
|:--|:--|
| Faixa etária | Criança (0 a 11 anos) 22% · adulto (12 a 59) 53% · idoso (60 a 94) 25% |
| `sexo` | `F` 51% · `M` 47% · `outro` 2% |
| `tipo_convenio` | `SUS` 68% · `convenio` 20% · `particular` 12% |
| `nivel_triagem` | vermelho 2% · laranja 8% · amarelo 35% · verde 40% · azul 15% |
| `desfecho` | Aproximadamente `alta` 72% · `encaminhado` 16% · `nao_compareceu` 6% · `internado` 5%. Casos vermelho e laranja nunca são `nao_compareceu` |
| `sintoma_principal` | Uniforme entre 10 sintomas (dor no peito, febre, corte/ferimento, falta de ar, dor abdominal, tontura, dor de cabeça, vômito/náusea, alergia, check-up de rotina) |
| `profissional_responsavel` | Enfermeiro, Enfermeira ou Tecnico Enfermagem, numerados de 01 a 24 |

**Como as linhas são produzidas**

- `pacientes`: nome e bairro via `Faker('pt_BR')`. A idade é calculada em relação a 01/07/2026, anterior ao fim da janela de atendimentos. Os adultos são maioria, mas crianças e idosos ficam sobrerrepresentados em relação à população geral, o que é típico de um posto de saúde.
- `atendimentos`: `tempo_espera_min` segue uma distribuição normal em torno de uma média por nível (vermelho 0, laranja 8, amarelo 35, verde 70, azul 150 minutos), com desvio de 35% da média mais 5 minutos, truncada em 0.

**Pontos de atenção para o Desafio 4**

- O tempo-alvo é estourado em todos os níveis, por construção do ruído: aproximadamente 42% dos vermelhos esperam mais que "imediato", 35% dos laranjas passam de 10 min, e cerca de 4% a 6% dos amarelos, verdes e azuis passam do alvo. Isso gera discussão sobre SLA.
- `data_hora_chegada` é sorteada a partir da data de nascimento do paciente, então nenhum atendimento ocorre antes do nascimento. Essa regra foi corrigida na revisão do script.
- `nao_compareceu` só ocorre nos níveis amarelo, verde e azul. O `desfecho` continua independente do sintoma.
- `profissional_responsavel` mistura variações de gênero e cargo ("Enfermeiro 03" e "Enfermeira 03" aparecem como valores distintos), um bom caso de limpeza e normalização.

---

## Expansão sugerida para o Desafio 4

- Tabela `profissionais` normalizando `profissional_responsavel` (hoje texto categórico numerado) em uma dimensão própria com especialidade e turno.
- Tabela `sintomas` como catálogo, para o aluno trocar `sintoma_principal` (hoje texto de lista fechada) por uma FK, o que introduz a discussão de normalização.

---

[Voltar ao índice](../../README.md)
