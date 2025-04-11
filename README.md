# MVP---Engenharia-de-Dados-Brasil-em-Movimento-Futebol-PIB-e-as-Rotas-da-Migra-o-Interna


** Objetivo:**
 ---
 Este trabalho tem como função apresentar fatores socioeconômicos que podem servir como indicadores para a formulação de políticas públicas mais eficazes, com foco na promoção de maior equidade no desenvolvimento entre os estados brasileiros. A proposta é mitigar as desigualdades regionais.
 
 A partir de dados públicos entre os anos de 2012 e 2024 — como Produto Interno Bruto (PIB) estadual, número de nascimentos e óbitos, população residente e número de pessoas empregadas com 15 anos ou mais, o projeto busca compreender como esses fatores se relacionam com o crescimento e a movimentação interna da população brasileira, especialmente com foco na migração dos responsáveis familiares.

 Além dos dados tradicionais, o projeto propõe utilizar a presença e o desempenho de clubes de futebol das Séries A e B do Campeonato Brasileiro como um indicador alternativo de desenvolvimento regional. A hipótese é que o futebol, como fenômeno cultural e econômico de grande alcance no Brasil, reflete não apenas a paixão esportiva da população, mas também aspectos estruturais e socioeconômicos das regiões, como capacidade de investimento, infraestrutura urbana e organização institucional.


## 1. Visão Geral
---

Este documento descreve o pipeline de ingestão de dados do projeto, incluindo:

- Transformações aplicadas

- Estrutura de armazenamento no Databricks

- Controles de qualidade






> Fontes oficiais e métodos de extração - Coleta
---


| Dados       | Site     | Fonte de Dados     | Tabela |  Forma de Extração       |
|-------------------|----------------|-------------|-------------|-------------|
| Industria  | apisidra.ibge.gov.br | IBGE       | 5938       | API       |
| PIB  | apisidra.ibge.gov.br | IBGE       | 5938       | API       |
| População  | apisidra.ibge.gov.br | IBGE       | 6579       | API       |
| Obitos  | apisidra.ibge.gov.br | IBGE       | 2683       | API       |
| Nascimentos  | apisidra.ibge.gov.br | IBGE       | 2680       | API       |
| Desemprego  | apisidra.ibge.gov.br | IBGE       | 4093       | API       |
| CAMPEONATOS BRASILEIROS  | https://www.mg.superesportes.com.br | superesportes | EXCEL → Spark |Manual + Web Scraping|



### 2. Modelagem
--- 
Para este projeto, implementamos um pipeline completo no Databricks organizado em três camadas essenciais. Na camada Bronze, ingerimos dados brutos, armazenando tudo em Data Lake para preservar a origem. Na camada Silver, transformamos esses dados em um modelo estrela, criando uma tabela de fatos com métricas demográficas, relacionadas a dimensões geográficas (UF), temporais e esportivas, após processos rigorosos de padronização, tratamento de valores inconsistentes e junção inteligente entre fontes. Finalmente, na camada Gold, consolidamos tabelas analíticas que respondem diretamente ao objetivo do projeto, revelando correlações entre desempenho esportivo e indicadores econômicos, além de padrões de migração interna vinculados a oportunidades regionais



####2.1 Arquitetura em Camadas
Implementamos um pipeline de dados no Databricks seguindo o padrão medallion architecture:

1. **Camada Bronze**  
   - Ingestão de dados brutos das fontes originais:  
     - Dados socioeconômicos do IBGE (PIB, desemprego) via API REST  
     - Dados de desempenho de clubes de futebol em planilhas Excel  
   - Armazenamento em Delta Lake mantendo:  
     - Estrutura original dos dados  

2. **Camada Silver**  
   - Modelagem dimensional em esquema estrela contendo:  
     - **Fato principal**: Métricas econômicas e demográficas por UF/ano  
     - **Dimensões**:  
       - Geografia (Sigla UF, região, população)  
       - Tempo (ano)  
       - Clubes (desempenho, divisão)  
   - Processos aplicados:  
     - Padronização de chaves (ex.: Sigla IBGE para UFs)  
     - Tratamento de dados faltantes (interpolação regional)  
    
3. **Camada Gold**  
   - Tabelas analíticas otimizadas para:  
     - Correlação entre indicadores econômicos e desempenho esportivo  
     - Identificação de padrões de migração interna  
   - Métricas-chave calculadas:  
     - PIB per capita vs. densidade de clubes por região  
     - Variação do desemprego x saldo migratório
    




#### 2.1 Catálogo de Dados
---

##### 2.1.1 Camada BRONZE

| Tabela       | Campo       | Tipo    | Descrição                                        |
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






#### 2.1.2 Camada SILVER

| Tabela          | Campo        | Tipo    | Descrição                                         |
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





>Tabelas Replicadas (Bronze → Silver)
---


| Tabela       | Campo       | Tipo    | Descrição                                      |
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






