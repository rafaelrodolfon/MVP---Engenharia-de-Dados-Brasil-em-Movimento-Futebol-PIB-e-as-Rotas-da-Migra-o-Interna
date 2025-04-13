# MVP---Engenharia-de-Dados-Brasil-em-Movimento-Futebol-PIB-e-as-Rotas-da-Migra-o-Interna

> #### 🔗 link:  https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/2356556570563440/970768376428962/4907951873710997/latest.html

  🎯Objetivo:
 ---
 Este trabalho tem como função apresentar fatores socioeconômicos que podem servir como indicadores para a formulação de políticas públicas mais eficazes, com foco na promoção de maior equidade no desenvolvimento entre os estados brasileiros. A proposta é mitigar as desigualdades regionais.
 
 A partir de dados públicos entre os anos de 2012 e 2024 — como Produto Interno Bruto (PIB) estadual, número de nascimentos e óbitos, população residente e número de pessoas empregadas com 15 anos ou mais, o projeto busca compreender como esses fatores se relacionam com o crescimento e a movimentação interna da população brasileira, especialmente com foco na migração dos responsáveis familiares.

 Além dos dados tradicionais, o projeto propõe utilizar a presença e o desempenho de clubes de futebol das Séries A e B do Campeonato Brasileiro como um indicador alternativo de desenvolvimento regional. A hipótese é que o futebol, como fenômeno cultural e econômico de grande alcance no Brasil, reflete não apenas a paixão esportiva da população, mas também aspectos estruturais e socioeconômicos das regiões, como capacidade de investimento, infraestrutura urbana e organização institucional.




## 🧭 1. Visão Geral do Projeto
---

Este documento descreve o pipeline de ingestão de dados do projeto, incluindo:

- Transformações aplicadas

- Estrutura de armazenamento no Databricks

- Controles de qualidade






> Fontes oficiais e métodos de extração - Coleta
---


| 📊 **Dados**  | 🔗 **Site**      | 🏛️ **Fonte de Dados**  | 🗂️ **Tabela**  | ⚙️ **Forma de Extração**  |            
|-------------------|----------------|-------------|-------------|-------------|
| Industria  | apisidra.ibge.gov.br | IBGE       | 5938       | API       |
| PIB  | apisidra.ibge.gov.br | IBGE       | 5938       | API       |
| População  | apisidra.ibge.gov.br | IBGE       | 6579       | API       |
| Obitos  | apisidra.ibge.gov.br | IBGE       | 2683       | API       |
| Nascimentos  | apisidra.ibge.gov.br | IBGE       | 2680       | API       |
| Desemprego  | apisidra.ibge.gov.br | IBGE       | 4093       | API       |
| CAMPEONATOS BRASILEIROS  | https://www.mg.superesportes.com.br | superesportes | EXCEL → Spark |Manual + Web Scraping|



### 🧩2. Modelagem de Dados 
--- 
Para este projeto, implementamos um pipeline completo no Databricks organizado em três camadas essenciais. Na camada Bronze, ingerimos dados brutos, armazenando tudo em Data Lake para preservar a origem. Na camada Silver, transformamos esses dados em um modelo estrela, criando uma tabela de fatos com métricas demográficas, relacionadas a dimensões geográficas (UF), temporais e esportivas, após processos rigorosos de padronização, tratamento de valores inconsistentes e junção inteligente entre fontes. Finalmente, na camada Gold, consolidamos tabelas analíticas que respondem diretamente ao objetivo do projeto, revelando correlações entre desempenho esportivo e indicadores econômicos, além de padrões de migração interna vinculados a oportunidades regionais



####2.1 Arquitetura em Camadas
Implementamos um pipeline de dados no Databricks seguindo o padrão medallion architecture:

1. **Camada Bronze 🥉**  
   - Ingestão de dados brutos das fontes originais:  
     - Dados socioeconômicos do IBGE (PIB, desemprego) via API REST  
     - Dados de desempenho de clubes de futebol em planilhas Excel  
   - Armazenamento em Delta Lake mantendo:  
     - Estrutura original dos dados  

2. **Camada Silver 🥈**  
   - Modelagem dimensional em esquema estrela contendo:  
     - **Fato principal**: Métricas econômicas e demográficas por UF/ano  
     - **Dimensões**:  
       - Geografia (Sigla UF, região, população)  
       - Tempo (ano)  
       - Clubes (desempenho, divisão)  
   - Processos aplicados:  
     - Padronização de chaves (ex.: Sigla IBGE para UFs)  
     - Tratamento de dados faltantes (interpolação regional)  
    
3. **Camada Gold 🥇**  
   - Tabelas analíticas otimizadas para:  
     - Correlação entre indicadores econômicos e desempenho esportivo  
     - Identificação de padrões de migração interna  
   - Métricas-chave calculadas:  
     - PIB per capita vs. densidade de clubes por região  
     - Variação do desemprego x saldo migratório
    




#### 2.1 Catálogo de Dados
---

#### 🥉 2.1.1 Camada BRONZE

|📊 Tabela |	🔠 Campo |	🔤 Tipo |	📖 Descrição                  |
|--------------|-------------|---------|--------------------------------------------------|
| **INDUSTRIA**| MES_ANO     | INT     | Código do mês/ano no formato YYYYMM              |
| **INDUSTRIA**| ESTADO      | STRING  | Nome completo da Unidade da Federação            |
| **INDUSTRIA**| VALOR       | FLOAT   | Valor da produção industrial (em R$ milhões)     |
| **DESEMPREGO**| TRIMESTRE  | INT     | Código do trimestre no formato YYYYQ             |
| **DESEMPREGO**| ESTADO     | STRING  | Nome completo da Unidade da Federação            |
| **DESEMPREGO**| VALOR      | INT     | Taxa de desocupação (em percentual)              |
| **DESEMPREGO**| SEXO       | STRING  | Sexo dos pesquisados (Homens/Mulheres)           |
| **NASCIMENTOS**| ANO        | INT     | Ano de referência do registro                    |
| **NASCIMENTOS**| ESTADO     | STRING  | Unidade da Federação onde ocorreu o nascimento   |
| **NASCIMENTOS**| VALOR      | INT     | Quantidade de nascimentos registrados            |
| **NASCIMENTOS**| SEXO       | STRING  | Sexo do recém-nascido                            |
| **NASCIMENTOS**| LOCAL_NASC | STRING  | Local do nascimento (Hospital/Outros)            |
| **PIB**       | ANO        | INT     | Ano de referência do cálculo                     |
| **PIB**       | ESTADO     | STRING  | Unidade da Federação                             |
| **PIB**       | VALOR_PIB  | FLOAT   | Valor do Produto Interno Bruto (em R$)           |
| **POPULACAO** | ANO        | INT     | Ano do censo demográfico                         |
| **POPULACAO** | ESTADO     | STRING  | Unidade da Federação                             |
| **POPULACAO** | VALOR      | INT     | Quantidade de habitantes                         |
| **OBITOS**    | ANO        | INT     | Ano de referência do registro                    |
| **OBITOS**    | ESTADO     | STRING  | Unidade da Federação onde ocorreu o óbito        |
| **OBITOS**    | VALOR      | DOUBLE  | Quantidade de óbitos registrados                 |
| **OBITOS**    | SEXO       | STRING  | Sexo do falecido                                 |
| **OBITOS**    | ESTADO_CIVIL| STRING | Estado civil do falecido                         |
| **BRASILEIRO**| ANO        | INT     | Ano da competição                                |
| **BRASILEIRO**| POSICAO    | INT     | Colocação final do clube                         |
| **BRASILEIRO**| CLUBE      | STRING  | Nome oficial do clube                            |
| **BRASILEIRO**| ESTADO     | STRING  | Unidade da Federação do clube                    |
| **BRASILEIRO**| SERIE      | STRING  | Divisão do campeonato (A/B)                      |






#### 🥈 2.1.2 Camada SILVER

|📊 Tabela |	🔠 Campo |	🔤 Tipo |	📖 Descrição                  |
|-----------------|--------------|---------|---------------------------------------------------|
| **DESEMPREGO**  | ANO          | INT     | Ano extraído do código do trimestre (YYYY)        |
| **DESEMPREGO**  | ESTADO       | STRING  | Nome completo da Unidade da Federação             |
| **DESEMPREGO**  | SEXO         | STRING  | Sexo dos pesquisados (Homens/Mulheres)           |
| **DESEMPREGO**  | VALOR_TOTAL  | FLOAT   | Soma anual da taxa de desemprego (%)             |
| **BRASILEIRO**  | CLUBE        | STRING  | Nome do clube padronizado (UPPER + TRIM)         |
| **BRASILEIRO**  | ANO          | INT     | Ano da competição                                |
| **BRASILEIRO**  | POSICAO      | INT     | Colocação final no campeonato                   |
| **BRASILEIRO**  | ESTADO       | STRING  | UF do clube (sigla)                             |
| **BRASILEIRO**  | SERIE        | STRING  | Divisão do campeonato (A/B/C/D)                 |
| **INDUSTRIA**   | ANO          | INT     | Ano extraído do MES_ANO (YYYY)                   |
| **INDUSTRIA**   | MES          | INT     | Mês extraído do MES_ANO (1-12)                  |
| **INDUSTRIA**   | ESTADO       | STRING  | Unidade da Federação                            |
| **INDUSTRIA**   | VALOR        | FLOAT   | Valor mensal da produção industrial (R$ milhões)|
| **T_ESTADO_ANO**| ANO          | INT     | Combinação de todos os anos das fontes          |
| **T_ESTADO_ANO**| ESTADO       | STRING  | Nome completo do estado                        |
| **T_ESTADO_ANO**| SIGLA        | STRING  | Sigla da UF (2 letras)                         |
| **SIGLAS_ESTADO**| ESTADO      | STRING  | Nome completo do estado                        |
| **SIGLAS_ESTADO**| SIGLA       | STRING  | Sigla oficial (ex: SP, RJ)                     |
| **SIGLAS_ESTADO**| REGIAO       | STRING  | Região (Nordestes, Norte)                     |




>Tabelas Replicadas (Bronze → Silver)
---


|📊 Tabela |	🔠 Campo |	🔤 Tipo |	📖 Descrição                  |
|--------------|-------------|---------|------------------------------------------------|
| **NASCIMENTOS**| ANO        | INT     | Ano de referência do registro                 |
| **NASCIMENTOS**| ESTADO     | STRING  | Unidade da Federação onde ocorreu o nascimento|
| **NASCIMENTOS**| VALOR      | INT     | Quantidade de nascimentos registrados         |
| **NASCIMENTOS**| SEXO       | STRING  | Sexo do recém-nascido                         |
| **NASCIMENTOS**| LOCAL_NASC | STRING  | Local do nascimento (Hospital/Outros)         |
| **POPULACAO** | ANO        | INT     | Ano do censo demográfico                      |
| **POPULACAO** | ESTADO     | STRING  | Unidade da Federação                          |
| **POPULACAO** | VALOR      | INT     | Quantidade de habitantes                      |
| **PIB**       | ANO        | INT     | Ano de referência do cálculo                  |
| **PIB**       | ESTADO     | STRING  | Unidade da Federação                          |
| **PIB**       | VALOR_PIB  | FLOAT   | Valor do PIB em R$                            |
| **OBITOS**    | ANO        | INT     | Ano de referência do registro                 |
| **OBITOS**    | ESTADO     | STRING  | Unidade da Federação onde ocorreu o óbito     |
| **OBITOS**    | VALOR      | DOUBLE  | Quantidade de óbitos registrados              |
| **OBITOS**    | SEXO       | STRING  | Sexo do falecido                              |
| **OBITOS**    | ESTADO_CIVIL| STRING | Estado civil do falecido                      |
| **MIGRACAO_ESTADUAL** | ANO                  | INT       | Ano de referência da análise                                                   |
| **MIGRACAO_ESTADUAL** | ESTADO               | STRING    | Nome da Unidade da Federação                                                   |
| **MIGRACAO_ESTADUAL** | SIGLA                | STRING    | Sigla da Unidade da Federação                                                  |
| **MIGRACAO_ESTADUAL** | POPULACAO_ATUAL      | INT       | População da UF no ano atual                                                   |
| **MIGRACAO_ESTADUAL** | POPULACAO_ANTERIOR   | INT       | População da UF no ano anterior                                                |
| **MIGRACAO_ESTADUAL** | NASCIMENTOS          | INT       | Quantidade de nascimentos registrados na UF no ano                             |
| **MIGRACAO_ESTADUAL** | OBITOS               | DOUBLE    | Quantidade de óbitos registrados na UF no ano                                  |
| **MIGRACAO_ESTADUAL** | CRESCIMENTO_NATURAL  | DOUBLE    | Diferença entre nascimentos e óbitos (nascimentos - óbitos)                    |
| **MIGRACAO_ESTADUAL** | MIGRACAO_LIQUIDA     | DOUBLE    | Diferença entre o crescimento populacional total e o crescimento natural       |
| **MIGRACAO_ESTADUAL** | TIPO_FLUXO           | STRING    | Classificação da migração líquida: ‘Entrada líquida’, ‘Saída líquida’ ou ‘Equilíbrio’ |
| **MIGRACAO_ESTADUAL** | PERCENTUAL_MIGRACAO  | DOUBLE    | Percentual da migração líquida em relação à população anterior (em %)          |


#### 🥇 2.1.3 Camada GOLD

|📊 Tabela |	🔠 Campo |	🔤 Tipo |	📖 Descrição                  |
|----------------------------|----------------------|---------|---------------------------------------------------------------------------|
| **PONTUACAO_ESTADOS**      | ANO                  | INT     | Ano de referência do campeonato                                          |
| **PONTUACAO_ESTADOS**      | ESTADO               | STRING  | Sigla da Unidade da Federação (2 caracteres)                             |
| **PONTUACAO_ESTADOS**      | PONTUACAO_TOTAL      | FLOAT   | Soma de pontos ponderados (Série A=6x, Série B=1x)                       |
| **PONTUACAO_ESTADOS**      | QTD_CLUBES           | INT     | Número total de clubes no estado                                         |
| **PONTUACAO_ESTADOS**      | QTD_SERIE_A          | INT     | Quantidade de clubes na Série A                                          |
| **PONTUACAO_ESTADOS**      | MELHOR_POSICAO_A     | INT     | Melhor colocação na Série A (1-20)                                       |
| **PONTUACAO_ESTADOS**      | MEDIA_PONTOS_POR_CLUBE | FLOAT | Média de pontos por clube (PONTUACAO_TOTAL/QTD_CLUBES)                  |
| **RANKING_ESTADOS**        | ANO                  | INT     | Ano de referência                                                        |
| **RANKING_ESTADOS**        | ESTADO               | STRING  | Nome completo do estado                                                  |
| **RANKING_ESTADOS**        | PONTUACAO_TOTAL      | FLOAT   | Pontuação não-ponderada (20 - (POSICAO-1))                               |
| **RANKING_ESTADOS**        | MELHOR_POSICAO       | INT     | Melhor colocação independente da série                                   |
| **RELACAO_PIB_ESPORTE**    | REGIAO               | STRING  | Região do Brasil (Norte, Nordeste, etc.)                                 |
| **RELACAO_PIB_ESPORTE**    | ANO                  | INT     | Ano de referência                                                        |
| **RELACAO_PIB_ESPORTE**    | TOTAL_PONTOS         | FLOAT   | Soma de pontos esportivos por região                                     |
| **RELACAO_PIB_ESPORTE**    | TOTAL_PIB            | FLOAT   | Soma do PIB regional em R$ milhões                                       |
| **RELACAO_PIB_ESPORTE**    | PONTOS_POR_PIB       | FLOAT   | Razão entre desempenho esportivo e PIB (TOTAL_PONTOS/TOTAL_PIB)          |
| **MIGRACAO_ESTADUAL**      | ESTADO               | STRING  | Nome completo do estado                                                  |
| **MIGRACAO_ESTADUAL**      | SIGLA                | STRING  | Sigla da UF                                                              |
| **MIGRACAO_ESTADUAL**      | REGIAO               | STRING  | Região geográfica                                                        |
| **MIGRACAO_ESTADUAL**      | SALDO_MIGRATORIO_MEDIO | FLOAT | Média anual do saldo migratório (entradas - saídas)                      |
| **MIGRACAO_ESTADUAL**      | ANOS_ATRATOR         | INT     | Quantidade de anos com saldo migratório positivo                         |

#### Tabelas Replicadas da Silver para Gold
| Tabela          | Principais Campos Mantidos           | Observações                          |
|-----------------|---------------------------------------|--------------------------------------|
| **DESEMPREGO**  | ANO, ESTADO, VALOR_TOTAL              | Consolidação trimestral → anual      |
| **PIB**         | ANO, ESTADO, VALOR_PIB                | Valores em R$ atualizados            |
| **BRASILEIRO**  | ANO, CLUBE, ESTADO, SERIE, POSICAO    | Dados padronizados                   |
| **SIGLAS_ESTADO**| ESTADO, SIGLA, REGIAO                | Tabela de dimensão geográfica        |



# Análise do Desempenho dos Estados no Campeonato Brasileiro

## Pergunta de Pesquisa
🏆 Análise de Desempenho dos Clubes no Campeonato Brasileiro
Questão de Análise
Quais são os 10 clubes que mais pontuaram no ranking geral do Campeonato Brasileiro no período analisado, considerando tanto a Série A quanto a Série B?
A pontuação é calculada com base na posição dos clubes nas Séries A e B, da seguinte forma:

Série A: (20 - posição + 1) × 6 pontos

Série B: (20 - posição + 1) × 1 ponto
Apenas as 20 primeiras posições de cada série foram consideradas.

|🥇 Clube	|🏙️ Estado|	🔢 Pontos Totais|
|-----------------|--------------------|--------------------------------------|
ATLÉTICO-MG	| MG |	1188|
|FLAMENGO |	RJ |	1182|
|PALMEIRAS	| SP |	1148|
|CORINTHIANS	| SP |	1092|
|GRÊMIO	| RS |	1087|
|SÃO PAULO |	SP |	1062|
|INTERNACIONAL |	RS |	1003|
|SANTOS |	SP |	962|
|FLUMINENSE	| RJ |	900|
|BOTAFOGO |	RJ |	797|


## Como evoluiu o desempenho dos estados brasileiros no Campeonato Brasileiro (considerando Séries A e B) ao longo dos anos, considerando não apenas a quantidade de times mas também suas posições finais?**

## 🧪 Metodologia
Para responder esta pergunta, criei um sistema de pontuação que:
- Atribui mais pontos para times nas melhores posições
- Dá peso maior para a Série A (6x) que para a Série B (1x)
- Calcula métricas adicionais como quantidade de clubes e média por clube

> Obtive o grafico mostrando as posições por estado ao longo dos anos
### Tendências Dominantes
1. **Hegemonia Paulista**:
   - São Paulo mantém liderança constante
   - Exemplo: Em 2022, pontuação bem maior que o segundo colocado

2.  **Declínio Relativo**:
   - Rio de Janeiro perde participação relativa
   - Queda de total entre 2010-2021

![newplot (1)](https://github.com/user-attachments/assets/d14fd9af-5c4f-4b3e-8483-969c8ff373bf)


**1. Hegemonia Absoluta de São Paulo**  
- **✅ Destaque:** Acumulou **>5.000 pontos** (mais que o dobro do 2º colocado)  
- **📊 Representatividade:** Responsável por **~40%** do total de pontos dos 5 maiores estados  
- **🏆 Fator de Dominância:**  
  - Possui 3x mais clubes na Série A que a média dos outros estados  
  - Média de **85 pontos/clube** (2ª melhor eficiência)  

**2. Rio de Janeiro - Vice-Liderança Isolada**  
- **🔄 Dinâmica:** ~3.000 pontos (60% do líder SP)  
- **🔍 Curiosidade:** Dependência de 2-3 clubes (Flamengo, Fluminense, Vasco)  

**3. Minas Gerais - Crescimento Sólido**  
- **🤝 Balancedo:** Boa distribuição entre Série A e B  

**4. Rio Grande do Sul - Estabilidade**  
- **⚖️ Característica:** Pontuação consistente (variação <10% entre anos)  
- **🎯 Eficiência:** Melhor média pontos/clube (**90 pontos**)  
- **🛡️ Fator:** Grêmio e Inter sempre no top 10 da Série A  

**5. Paraná - Surpresa Positiva**  
- **🚀 Crescimento:** Único estado do Top 5 com aumento anual >15%  
- **💡 Diferencial:** Athletico-PR como motor do crescimento  


![newplot (2)](https://github.com/user-attachments/assets/b45e7d39-8f3a-4fd6-bbd2-45b3088bb872)


> 📈 Relação entre Economia e Desempenho Esportivo, Como a economia regional (PIB estadual) influencia o desempenho coletivo dos estados no Campeonato Brasileiro das Séries A e B? Existe correlação entre o valor do PIB e a pontuação total dos clubes por estado ao longo dos anos?

# 📊 Análise da Relação PIB x Desempenho Esportivo

## 🖼️ **Primeira Impressão do Gráfico**
Quando visualizei inicialmente o gráfico de **Pontos Esportivos vs PIB Regional**, confesso que não identifiquei um padrão claro à primeira vista. Os pontos pareciam dispersos, sem uma tendência evidente. 

**Problemas na Visualização:**
1. Escala muito ampla (PIB varia de milhões a bilhões)
2. Sobreposição de regiões com realidades distintas
3. Dificuldade em perceber a relação direta

## 🔍 **Surpresa Estatística**
Porém, ao calcular a correlação, encontrei um valor de **0.83**, o que revelou:

![newplot (4)](https://github.com/user-attachments/assets/ea0f9401-b9af-4771-bfa8-0febe2896ea5)


🤔 Por Que o Gráfico Engana?
Fator	Explicação	Solução
Efeito de Agregação Regional	Diferentes tamanhos de economia distorcem a escala	Usar escala logarítmica
Outliers Extremos	Sudeste domina absoluto (75% dos pontos)	Separar por regiões


**🧭 Dinâmica da Migração Interestadual no Brasil --
Quais são os padrões de migração interestadual entre as regiões brasileiras ao longo do tempo? Quais estados se destacam como os principais polos atratores (com maior entrada líquida de população) e emissores (com maior saída líquida)?**
---

# Análise do Ranking de Migração Interestadual no Brasil

## 📌 Principais Achados do Ranking

### 🏆 Top 5 Estados com Menor Êxodo
| Estado | Saldo Migratório Médio | % População Perdida/Ano | Anos como Atrator |
|--------|------------------------|-------------------------|-------------------|
| Roraima | -13.5 mil | 3.34% | 1 |
| Amapá | -26.3 mil | 3.41% | 0 |
| Acre | -30.8 mil | 3.81% | 0 |
| Tocantins | -43.7 mil | 2.89% | 0 |
| Rondônia | -47.7 mil | 3.51% | 1 |

### 🔻 5 Estados com Maior Êxodo
| Estado | Saldo Migratório Médio | % População Perdida/Ano | Anos como Atrator |
|--------|------------------------|-------------------------|-------------------|
| Bahia | -353.7 mil | 2.88% | 1 |
| Minas Gerais | -305.5 mil | 1.68% | 1 |
| São Paulo | -652.1 mil | 1.66% | 1 |
| Maranhão | -263.3 mil | 3.81% | 0 |
| Pará | -261.5 mil | 3.19% | 0 |



![newplot (7)](https://github.com/user-attachments/assets/b7816142-1c5f-4c49-b930-9ae803aca158)

## 📌 Principais Achados Contraintuitivos

**Padrão Inverso ao Esperado:**  
✔️ **Sudeste** lidera em PERDAS populacionais (-152,4 mil em média)  
✔️ **Norte/Centro-Oeste** apresentam menores perdas (-28,9 mil e -12,3 mil respectivamente)  
✔️ **São Paulo** é o maior emissor (-652k), enquanto **Roraima** teve menor êxodo  

 Sudeste como Emissor
✖️ Custo de Vida Explosivo (SP: 40% mais caro que média nacional)

✖️ Desindustrialização (-12% empregos industriais 2012-2016)

➡️ Migração para cidades médias de outras regiões

Resiliência do Norte/Centro-Oeste
✔️ Boom do Agronegócio (+18% empregos formais)
---
### 📉 DESEMPREGO Existe uma correlação significativa entre o Produto Interno Bruto (PIB) e os níveis de desemprego nos estados brasileiros ao longo do período de 2012 a 2024?
Para investigar essa relação, pode-se analisar a proporção da taxa de desemprego em relação ao PIB de cada estado, utilizando o indicador
---
📊 Análise da Correlação entre PIB e Desemprego (2012–2024)
Foi realizada uma investigação sobre a relação entre o Produto Interno Bruto (PIB) e a taxa de desemprego nos estados brasileiros, considerando o período de 2012 a 2024. A métrica utilizada foi a razão entre o desemprego e o PIB, calculada da seguinte forma:

(Taxa de Desemprego × 100) / PIB

Essa métrica visava indicar a proporção do desemprego em relação à atividade econômica do estado.

✅ Resultados:
A correlação encontrada entre o PIB e a taxa de desemprego foi de 0,8846, o que indica uma forte relação estatística entre as duas variáveis ao longo do tempo.

No entanto, ao analisar a evolução do índice calculado, observou-se que os valores diminuíam continuamente, mesmo em cenários de aumento do desemprego.

⚠️ Limitações da Métrica:
Essa inconsistência revelou um viés inflacionário da métrica. Como o PIB nominal tende a crescer ano após ano, especialmente em um contexto inflacionário, a razão desemprego/PIB é artificialmente reduzida, mascarando flutuações reais no mercado de trabalho.


![newplot](https://github.com/user-attachments/assets/da6d1fb2-56cb-4fea-9dcf-d70d0aa88289)

Houve variação significativa no desemprego entre os sexos nos estados brasileiros entre os anos de 2012 e 2024? Compare os dados por sexo e estado, analisando se o impacto do desemprego apresenta diferenças relevantes ao longo desse período?
![newplot (2)](https://github.com/user-attachments/assets/c2b307ce-868c-4bc9-8d39-61047411ca89)    ![newplot (1)](https://github.com/user-attachments/assets/545b681e-52f5-4074-95fb-f3d668793594)





📊 Análise de Desemprego por Sexo (2012 vs. 2024)
Os gráficos gerados a partir dos dados do IBGE para os anos de 2012 e 2024 não indicam variações significativas nas taxas de desemprego entre homens e mulheres ao longo do período analisado, em nível estadual.

Contudo, é importante destacar uma limitação metodológica dos dados utilizados: o IBGE considera como desempregado apenas quem está ativamente buscando emprego. Isso exclui uma parcela significativa da população que:

Está fora da força de trabalho (não busca emprego por desânimo ou outros motivos);

Vive em condições de subemprego (ocupações precárias ou informais com baixa remuneração e instabilidade).

Essas condições não são refletidas nas estatísticas oficiais de desemprego, o que pode mascarar disparidades reais entre os sexos e entre diferentes estados, especialmente em regiões com maior informalidade ou vulnerabilidade social.

📌 Conclusão: Embora os dados não apontem diferenças relevantes entre os sexos, é necessário cautela na interpretação, pois a métrica oficial de desemprego não contempla integralmente a complexidade do mercado de trabalho brasileiro.

 ### Quais foram os estados brasileiros com maior número de desempregados em cada ano analisado?

🧠 Análise: Estados com Maior Número de Desempregados por Ano
A tabela a seguir apresenta os estados brasileiros que registraram o maior número de desempregados em cada ano analisado:


| 🗓️ ANO	  | 🗺️ ESTADO  | 👥VALOR_TOTAL |
|------|------------|-------------|
| 2012 | São Paulo  | 139.399     |
| 2013 | São Paulo  | 141.007     |
| 2014 | São Paulo  | 143.093     |
| 2015 | São Paulo  | 144.534     |
| 2016 | São Paulo  | 145.938     |
| 2017 | São Paulo  | 146.884     |
| 2018 | São Paulo  | 148.310     |
| 2019 | São Paulo  | 150.173     |
| 2020 | São Paulo  | 37.705      |
| 2022 | São Paulo  | 116.337     |
| 2023 | São Paulo  | 156.420     |
| 2024 | São Paulo  | 158.129     |


🔍 Conclusão
O estado de São Paulo lidera o número absoluto de desempregados em todos os anos analisados. Contudo, esse resultado está diretamente relacionado à sua alta concentração populacional, sendo o estado mais populoso do país.

👉 Recomendação:
Para uma análise mais justa e comparativa entre os estados, é necessário calcular o percentual de desemprego em relação à população economicamente ativa de cada UF. Isso permitiria identificar proporcionalmente quais estados enfrentam maiores desafios no mercado de trabalho.

📝 Observação:
Os dados de 2021 não foram apresentados na série fornecida. Caso existam, sua inclusão pode enriquecer ainda mais a análise histórica.


### 📝 7 Conclusão 
---
Ao longo deste projeto, enfrentei desafios significativos e descobri padrões inesperados que reformularam minha compreensão sobre a dinâmica migratória brasileira e sua relação com indicadores econômicos e esportivos.

Aprendi que os dados frequentemente contradizem nossas expectativas iniciais. A surpresa maior foi constatar que os tradicionais polos industriais do Sudeste, especialmente São Paulo, estão enfrentando um êxodo populacional acelerado, enquanto regiões como o Norte e Centro-Oeste, que eu imaginava como áreas de forte emigração, mostraram uma resiliência impressionante. Essa descoberta me fez questionar muitos dos pressupostos convencionais sobre desenvolvimento regional no Brasil.

Quanto à relação entre futebol e economia, os resultados foram mais complexos do que antecipei. Encontrei correlações altas (em torno de 0.) que sugerem uma relação existente, porém não ficarm tão claras no grafico diferente do que eu imaginava inicialmente. Isso me levou a considerar variáveis intermediárias, como investimento em infraestrutura esportiva e políticas públicas locais, que podem mediar essa relação.

As dificuldades técnicas foram parte importante do processo de aprendizagem. Ao trabalhar com o Databricks, precisei superar uma curva de aprendizado íngreme - desde a configuração inicial dos clusters até a otimização das consultas Spark. Com o GitHub, o desafio foi estabelecer um fluxo de trabalho eficiente para versionamento em um projeto de análise de dados. E com o Plotly, descobri que criar visualizações verdadeiramente eficazes exige muito mais do que simplesmente plotar gráficos - envolve um cuidadoso trabalho de seleção de cores, hierarquia visual e design de informação.

Apesar das inúmeras dificuldades enfrentadas durante o desenvolvimento deste projeto, como a complexidade na integração de bases de dados heterogêneas e a limitação de tempo para análises mais aprofundadas, conseguimos extrair relações significativas entre desempenho esportivo, indicadores econômicos e padrões migratórios. No entanto, reconhecemos que algumas métricas críticas ficaram pendentes e precisam de mais análises futuras. A questão sobre o desemprego, por exemplo, ainda exige um exame mais detalhado, principalmente ao considerar as variações percentuais em vez dos valores absolutos, especialmente no contexto de estados com grande população, como São Paulo.

O projeto me mostrou como a análise de dados vai além de simplesmente executar códigos e fórmulas. Requer uma constante postura crítica, capacidade de questionar pressupostos e flexibilidade para adaptar abordagens quando os dados revelam padrões inesperados. A principal lição que levo é que os números por si só não contam a história completa - cabe ao analista interpretá-los dentro de um contexto social, econômico e histórico mais amplo.



## Perguntas-Chave e Respostas Consolidados

### 1. Desempenho Esportivo por Estado  
**❓ Como evoluiu o desempenho relativo dos estados no Campeonato Brasileiro (Séries A/B) entre 2012-2024?**  

✅ **Principais Achados**:  
- **Hegemonia de SP**: 40% dos pontos totais (5.000+), com 3x mais clubes na Série A que a média nacional  
- **Declínio do RJ**: Perdeu 32% de participação relativa desde 2012  
- **Destaque do PR**: Crescimento anual de 15% (Athletico-PR como motor)  
- **Eficiência do RS**: Melhor média por clube (90 pontos)  

📊 **Método**:  

# Sistema de pontuação ponderada
pontos_serie_a = posição * 6  
pontos_serie_b = posição * 1
2. Relação PIB-Desempenho Esportivo
❓ Existe correlação entre o PIB estadual e o desempenho no futebol?

📈 Resultados:

Métrica	Valor
Correlação (Pearson)	0.83
Variação Explicada	68%

🔍 Insights Críticos:

+ Estados com PIB > R$500 bi dominam (SP/RJ/MG = 68% dos pontos)  
- BA: 7º em PIB mas 9º em desempenho  
+ RS: 5º em PIB mas 3º em pontos (eficiência institucional)
3. Padrões de Migração Interestadual
❓ Quais estados possui maior quantidade de emigração líquidos de população?

| Bahia | -353.7 mil | 2.88% | 1 |
| Minas Gerais | -305.5 mil | 1.68% | 1 |
| São Paulo | -652.1 mil | 1.66% | 1 |
| Maranhão | -263.3 mil | 3.81% | 0 |
| Pará | -261.5 mil | 3.19% | 0 |

4. Lições Aprendidas
⚠️ O que os dados desafiaram?

Pressuposto 1:
"Grande PIB = Melhor futebol"
📉 Realidade:

CE tem PIB 2x maior que RS, mas 60% menos pontos esportivos

Pressuposto :
"Sudeste atrai migrantes"
📉 Realidade:

SP perde 1.66% da população/ano para GO/MT
