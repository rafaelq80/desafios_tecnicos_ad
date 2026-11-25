# Guia do Desafio 03 · Analista de Dados

<div align="center"><img src="../icones/excel.svg" width="20%"/></div>

**Parte 1 de 3 — Preparação e conteúdo da sessão**

<table width="100%">
  <tr>
    <td align="left" width="33%"><strong>Início</strong></td>
    <td align="center" width="34%"><a href="../README.md">README</a></td>
    <td align="right" width="33%"><a href="02.md">Avançar → Parte 2</a></td>
  </tr>
</table>

---

## 1. Como usar este guia

Este guia conduz você da base de dados do seu tema até um **relatório dinâmico no Excel**: tabela dinâmica, gráficos e um conjunto de fórmulas obrigatórias, terminando com uma **recomendação de decisão** apoiada em números.

Todo o trabalho é feito no **Excel para a Web**, direto no navegador. O relatório é a **própria planilha** (`.xlsx`): fórmulas, tabela dinâmica, gráficos e textos de leitura ficam nela. A planilha final é enviada ao **GitHub**, e o link vai no **Canvas**. **Tudo é entregue no mesmo dia da sessão.**

| **Período**                     | **O que é**                                                                                                                                                                                                                                                 |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SESSÃO SÍNCRONA**            | - Abrir os arquivos CSV do tema no Excel<br />- Organizar as duas tabelas e conferir a base antes de calcular<br />- Montar a tabela dinâmica básica<br />- Aplicar PROCV com SEERRO, SOMASE, CONT.SE e uma formatação condicional                               |
| **SESSÃO ASSÍNCRONA** | - Completar as fórmulas obrigatórias (SOMASES, CONT.SES, MÉDIA e variações, SE, ARRUMAR, ÉCEL.VAZIA, datas)<br />- Aplicar validação de dados e auditar a tabela dinâmica<br />- Gráficos, recomendação de decisão e registro de IA<br />- Enviar a planilha ao GitHub e entregar o link no Canvas |

<br />

> [!IMPORTANT]
>
> ### Base de dados
>
> A base utilizada neste desafio é a do **seu tema**, nos **arquivos CSV disponíveis no repositório do GitHub** do Desafio. O tema escolhido no Desafio 1 permanece o mesmo: **não troque de tema**.

<br />

> [!IMPORTANT]
>
> ### Sem Power Query
>
> O Excel para a Web **não possui o Power Query ativo em todos os planos**. Por isso, neste desafio você **abre os CSV direto no Excel** e traz os dados para a sua planilha por copiar e colar (Passo 2). A junção entre as duas tabelas é feita com **PROCV**.
>
> Se você estiver usando o Excel de computador, siga o mesmo caminho, **sem Power Query**: assim todos usam os mesmos passos e as mesmas fórmulas.

<br />

> [!NOTE]
>
> ### Fórmulas em português
>
> As fórmulas deste guia usam os nomes em português e o **ponto e vírgula (`;`)** como separador, como no Excel em português do Brasil. Se o seu Excel estiver em inglês, use a tabela de nomes do **Anexo I** e troque `;` por `,`.

<br />

> [!NOTE]
>
> ### O exemplo de referência: Biblioteca
>
> Os exemplos usam o tema **Biblioteca Pública**, que **não faz parte dos 8 temas disponíveis no desafio**. Ele serve apenas para mostrar o **formato** das fórmulas.
>
> | **Tabela**           | **Arquivo**       | **O que guarda**                                                                                  |
> | -------------------- | ----------------- | ------------------------------------------------------------------------------------------------- |
> | Dimensão (`livros`)  | `livros.csv`      | Acervo: título, autor, gênero, ano de publicação, número de páginas, idioma, filial e conservação |
> | Fato (`emprestimos`) | `emprestimos.csv` | Cada empréstimo: livro, leitor, datas, renovações, dias de atraso, multa, canal e status          |
>
> A chave estrangeira é `livro_id`: cada empréstimo aponta para um livro do acervo.
>
> **O guia não traz respostas**, nem do tema Biblioteca nem dos 8 temas: a pergunta de negócio, as colunas, as leituras e as decisões saem dos dados do **seu** tema. O guia indica **o que usar** em cada passo.

<br />

---

## 2. O que você será capaz de fazer

- Construir uma **tabela dinâmica** com as métricas mais relevantes do seu tema.
- Aplicar **PROCX, SE e SEERRO** para buscar e validar dados entre as tabelas dimensão e fato.
- Aplicar **SOMASE, SOMASES, CONT.SE, CONT.SES, MÉDIA, MÉDIASE e MÉDIASES** para calcular métricas por critério.
- Tratar texto e data com **ARRUMAR, ÉCEL.VAZIA** e funções de data antes de calcular.
- Aplicar **formatação condicional** e **validação de dados** para deixar o relatório confiável e fácil de ler.
- Montar **gráficos** com título e eixos identificados.
- Construir um **Dashboard** é um diferencial

---

## 3. Mapa dos passos

| **Passo** | **O que você faz**                                           | **Quando**        |
| --------- | ------------------------------------------------------------ | ----------------- |
| 1         | Definir a pergunta de negócio e a métrica                    | Sessão Síncrona   |
| 2         | Obter os dados: abrir os 2 CSV e montar a pasta de trabalho  | Sessão Síncrona   |
| 3         | Transformar os dados em tabelas e conhecê-las                | Sessão Síncrona   |
| 4         | Checar a base antes de calcular                              | Sessão Síncrona   |
| 5         | Montar a tabela dinâmica com a métrica principal             | Sessão Síncrona   |
| 6         | PROCX: trazer o cadastro para a tabela fato       | Sessão Síncrona   |
| 7         | Somar e contar por critério: SOMASE e CONT.SE                | Sessão Síncrona   |
| 8         | Formatação condicional e primeiras leituras                  | Sessão Síncrona   |
| 9         | Completar as fórmulas por critério: SOMASES, CONT.SES, médias e SE | Sessão Assíncrona |
| 10        | Tratar texto, vazios e datas                                 | Sessão Assíncrona |
| 11        | Validação de dados e auditoria da tabela dinâmica            | Sessão Assíncrona |
| 12        | Gráficos                                                     | Sessão Assíncrona |
| 13        | Redigir a recomendação de decisão (2 medidas ou mais)        | Sessão Assíncrona |
| 14        | Diferencial: Dashboard                                       | Sessão Assíncrona |
| 15        | Registrar o uso de IA, enviar ao GitHub e entregar no Canvas | Sessão Assíncrona |

> [!IMPORTANT]
>
> **Ordem de prioridade:** primeiro tudo o que é **obrigatório** (Passos 1 a 13 e 15). O diferencial (Passo 14) só entra depois.

---

## 4. Excel para a Web

Você trabalha em **uma única pasta de trabalho**, com **6 abas** (e uma sétima, opcional, para o diferencial). Os dados ficam nas abas `dimensao` e `fato`; o relatório fica nas outras.

| Nº   | **Guia**      | **Para que serve**                                           | **Passos** |
| ---- | ------------- | ------------------------------------------------------------ | ---------- |
| 1    | `Leitura`     | Textos: pergunta de negócio, leituras, recomendação e registro de IA. É a primeira guia a ser lida | 1 a 15     |
| 2    | `Relatorio`   | Fórmulas por critério, indicadores, seletor, formatação condicional e gráficos | 7 a 12     |
| 3    | `Dinamica`    | A tabela dinâmica                                            | 5, 11      |
| 4    | `Conferencia` | Verificações da base: tamanho, tipos, ausentes, chaves       | 3 e 4      |
| 5    | `Fato`        | Dados do CSV da tabela fato, como **Tabela** `tbl_fato`, mais colunas calculadas por você | 2 a 10     |
| 6    | `Dimensao`    | Dados do CSV da tabela dimensão, como **Tabela** `tbl_dimensao` | 2, 3, 6    |
| 7    | `Dashboard`   | OPCIONAL - Construção do Dashboard (as tabelas dinâmicas e as segmentações do Dashboard devem ser criadas nesta guia) |            |

Os nomes das guias e das tabelas são **iguais para todos os temas**. Assim as fórmulas deste guia funcionam do mesmo jeito em qualquer tema: o que muda são os nomes das **colunas**.

Além das fórmulas e dos gráficos, a guia `Leitura` terá **9 blocos de texto escritos por você**. Eles apresentam suas análises e interpretações:

| **Passo** | **Bloco de texto na guia `Leitura`**                         |
| --------- | ------------------------------------------------------------ |
| 1         | Pergunta de negócio, métrica e decisão que o resultado apoiaria |
| 4         | Leitura da conferência                                       |
| 8         | Leitura da tabela dinâmica e das fórmulas por critério       |
| 9         | Leitura das médias                                           |
| 10        | Registro do tratamento de texto, vazios e datas              |
| 11        | Leitura da auditoria                                         |
| 12        | Frase de leitura dos gráficos                                |
| 13        | Recomendação                                                 |
| 15        | Registro do uso de IA                                        |

> [!TIP]
>
> O Excel para a Web **salva sozinho** (veja o aviso de salvamento no topo da tela). Mesmo assim, não feche a guia do navegador no meio de uma operação, como uma colagem grande.

---

## 5. O que você precisa ter em mãos

- Os **2 arquivos CSV do seu tema**, baixados do repositório do GitHub do Desafio.
- A **documentação do seu tema**, com o esquema de colunas, para conferir o tipo de cada coluna e identificar a chave estrangeira.
- Uma **conta Microsoft**, para usar o **Excel para a Web** e o **OneDrive** (o OneDrive é por onde o CSV é aberto).
- Uma **conta no GitHub**, com acesso ao repositório do Desafio 1 (**projeto_desafios_tecnicos_ad**).

---

# Sessão — Resumo

## Bloco 1 — Base de dados

### As duas tabelas da base

A base de cada tema tem **duas tabelas ligadas entre si**. Entender o papel de cada uma é o que permite cruzá-las com PROCV e comparar grupos.

| **Conceito**               | **O que é**                                                                                                                                         | **Na Biblioteca**                                                |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| **Tabela dimensão**        | O **cadastro**: descreve as "coisas" do tema. Cada linha é um item único e as colunas são as características dele                                   | `livros`: cada linha é um livro do acervo                        |
| **Tabela fato**            | O **registro de eventos**: cada linha é uma ocorrência e traz datas, quantidades e valores medidos. É onde ficam as colunas numéricas que se analisam | `emprestimos`: cada linha é um empréstimo                        |
| **Chave primária (PK)**    | A coluna que **identifica cada linha de forma única**: não se repete e não fica vazia. Toda tabela tem a sua                                        | `livro_id` em `livros` e `emprestimo_id` em `emprestimos`        |
| **Chave estrangeira (FK)** | A coluna da tabela fato que **guarda a chave primária de uma linha da dimensão**. É ela que liga cada evento ao seu cadastro                        | `livro_id` em `emprestimos` aponta para `livro_id` em `livros`   |

As colunas das duas tabelas da Biblioteca:

| **Tabela**           | **Colunas**                                                                                                                                                                                                           |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `livros` (dimensão)  | `livro_id` (PK), `titulo`, `autor`, `genero`, `ano_publicacao`, `numero_paginas`, `idioma`, `filial`, `estado_conservacao`                                                                                            |
| `emprestimos` (fato) | `emprestimo_id` (PK), `livro_id` (FK), `leitor_id`, `tipo_leitor`, `data_emprestimo`, `modalidade`, `renovacoes`, `data_devolucao_prevista`, `data_devolucao`, `dias_atraso`, `multa_reais`, `canal_retirada`, `status` |

> [!NOTE]
>
> `leitor_id` é apenas um número: **não existe uma tabela de leitores**, então ele não é chave estrangeira. Só é chave estrangeira a coluna que aponta para a chave primária de outra tabela da base.

<br />

✅**Exemplo:**

**Tabela dimensão `livros`**

| `livro_id` (PK) | `titulo` |
| --------------- | -------- |
| 1               | Livro A  |
| 2               | Livro B  |
| 3               | Livro C  |

**Tabela fato `emprestimos`**

| `emprestimo_id` (PK) | `livro_id` (FK) | `data_emprestimo` |
| -------------------- | --------------- | ----------------- |
| 1                    | 2               | 2025-03-10        |
| 2                    | 2               | 2025-03-12        |
| 3                    | 1               | 2025-03-12        |

O `Livro B` (`livro_id` 2) aparece em **dois empréstimos**: uma linha da dimensão pode estar ligada a **várias linhas da fato**. Já cada empréstimo aponta para **um único livro**.

<br />

**Por que isso importa para o relatório**

- A tabela **fato** tem as colunas numéricas que você vai somar, contar e tirar a média.
- A tabela **dimensão** traz as categorias para **comparar grupos**. Para usá-las na tabela dinâmica, você **traz a coluna da dimensão para a fato com PROCV** (Passo 6).
- Antes de cruzar, é preciso confirmar que **toda chave estrangeira aponta para uma chave primária que existe** na dimensão (Passo 4).

> [!TIP]
>
> ### Faça no seu tema
>
> Na documentação do seu tema, localize:
>
> 1. Qual arquivo é a **tabela dimensão** e qual é a **tabela fato**.
> 2. A **chave primária** de cada tabela.
> 3. A **chave estrangeira** da tabela fato e a chave primária da dimensão a que ela aponta. Você vai usar as duas no Passo 4 e no Passo 6.

<br />

### Abrir e conferir a base

Antes de calcular qualquer métrica, é preciso trazer os arquivos CSV para o Excel e confirmar que estão limpos: coluna com o tipo certo, sem chave repetida e sem valor ausente inesperado.


> [!WARNING]
>
> Pular essa etapa é a causa mais comum de relatório errado. Uma soma com número guardado como texto, ou uma média com valor absurdo, fica distorcida e o Excel **não acusa nenhum erro**.

<br />

> [!NOTE]
>
> ### Valor ausente nem sempre é erro
>
> Imagine uma tabela de pedidos com a coluna `nota_avaliacao`, vazia para os clientes que ainda não avaliaram. Não é defeito. A pergunta certa é: **por que está vazio?** Só depois você decide se ignora, preenche ou sinaliza.

<br />

---

## Bloco 2 — As funções do relatório

### 2.1 Buscar e validar: PROCV, SE e SEERRO

O **PROCX** busca um valor na tabela e traz outra informação da mesma linha ou coluna. É a base de qualquer relatório que cruza duas tabelas, como a dimensão e a fato do seu tema.


### 2.2 Somar e contar por critério: a base da tabela dinâmica

As funções por critério são a **versão manual do que a tabela dinâmica faz automaticamente**. Entender a fórmula ajuda a entender por que a tabela dinâmica mostra o número que mostra.

| **Quero**                          | **Um critério** | **Vários critérios** |
| ---------------------------------- | --------------- | -------------------- |
| Somar                              | `SOMASE`        | `SOMASES`            |
| Contar                             | `CONT.SE`       | `CONT.SES`           |
| Média                              | `MÉDIASE`       | `MÉDIASES`           |
| Média simples, sem critério        | `MÉDIA`         | —                    |
| Contar só as células com número    | `CONT.NÚM`      | —                    |

- Toda métrica da tabela dinâmica pode ser **conferida por uma fórmula equivalente**: é um bom hábito de auditoria.
- Quem sabe montar a fórmula equivalente confia mais na tabela dinâmica e sabe corrigir quando o número parece estranho.
- **Atenção à ordem dos argumentos:** em `SOMASE` e `MÉDIASE` o critério vem **antes** da coluna numérica; em `SOMASES` e `MÉDIASES` a coluna numérica vem **primeiro**.

### 2.3 Texto, data e apresentação: deixando o relatório confiável

Um relatório com número certo, mas com texto sujo ou difícil de ler, passa confiança errada.

| **Recurso**            | **O que faz**                                                | **No Excel**                            |
| ---------------------- | ------------------------------------------------------------ | --------------------------------------- |
| VAZIO                  | Verifica se uma célula está em branco, antes de calcular em cima dela | `=ÉCEL.VAZIA([@coluna])`                |
| ARRUMAR                | Remove espaços extras do texto (problema clássico em dado importado ou digitado) | `=ARRUMAR([@coluna])`                   |
| Funções de data        | Extraem mês e ano, calculam diferença de dias e idade        | `MÊS`, `ANO`, `DIAS`, `DATADIF`, `HOJE` |
| Formatação condicional | Destaca visualmente o que importa: prazo estourado, nota baixa, valor fora do padrão | Página Inicial → Formatação Condicional |
| Validação de dados     | Limita o que pode ser digitado na célula, direto na origem do problema | Dados → Validação de Dados              |

> [!NOTE]
>
> No Excel em português, a função que o roteiro chama de **VAZIO** se chama **`ÉCEL.VAZIA`**.

### Diferenciais (só depois do obrigatório)

- **FILTRO** extrai uma lista dinâmica dos registros que atendem a um critério, sem fórmula matricial complexa.
- **Imagem** inserida na planilha enriquece o cabeçalho do relatório.

Nenhum dos dois é obrigatório.

<br />

---

## Bloco 3 — Do número ao relatório: apoiando uma decisão

O relatório só tem valor se alguém conseguir **decidir algo** a partir dele. Um número sem recomendação é dado; um número com recomendação é **insight**. E o relatório mistura **tabela dinâmica, gráfico e fórmula**: não é uma lista de números soltos.

Todo insight tem três partes, e a primeira é sempre o número:

| **Parte**   | **Pergunta que responde**         | **Como escrever**                                                    |
| ----------- | --------------------------------- | -------------------------------------------------------------------- |
| **Medida**  | Qual número sustenta a conclusão? | Cite a medida, a coluna e o valor                                    |
| **Leitura** | O que esse número significa?      | Compare com outra medida (por exemplo, total e quantidade, ou dois grupos) |
| **Decisão** | O que fazer a respeito?           | Diga uma ação possível, em linguagem que alguém de fora entenderia   |

Formato de um insight (troque o que está entre colchetes):

> **Medida:** o total de [coluna] do grupo [A] é [valor], e a quantidade de registros é [valor].
> **Leitura:** [o grupo A concentra muito mais do que os outros / tem poucos registros e pede cautela].
> **Decisão:** [ação possível, em uma frase].

- Todo insight cita a medida que o sustenta: **não existe insight sem número por trás**.
- Escreva a recomendação em **uma frase** que alguém de fora da área entenderia.
- Todo gráfico precisa de **título** e **eixos identificados**, e de **uma frase** dizendo o que ele mostra.
- Um relatório certo também precisa ser **fácil de ler**: formatação condicional, poucas cores e números formatados.

> [!IMPORTANT]
>
> Cada função obrigatória precisa aparecer **pelo menos uma vez, num uso que faça sentido para o seu tema**. SEERRO e validação de dados existem para o relatório **não quebrar** quando o dado vier incompleto.

<br />

---

<table width="100%">
  <tr>
    <td align="left" width="33%"><strong>Início</strong></td>
    <td align="center" width="34%"><a href="../README.md">README</a></td>
    <td align="right" width="33%"><a href="02.md">Avançar → Parte 2</a></td>
  </tr>
</table>
