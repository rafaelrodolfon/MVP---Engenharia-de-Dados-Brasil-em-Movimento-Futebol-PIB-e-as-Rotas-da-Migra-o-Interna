# MVP---Engenharia-de-Dados-Brasil-em-Movimento-Futebol-PIB-e-as-Rotas-da-Migra-o-Interna


 Objetivo: Este trabalho tem como função apresentar fatores socioeconômicos que podem servir como indicadores para a formulação de políticas públicas mais eficazes, com foco na promoção de maior equidade no desenvolvimento entre os estados brasileiros. A proposta é mitigar as desigualdades regionais.
 
 A partir de dados públicos entre os anos de 2012 e 2024 — como Produto Interno Bruto (PIB) estadual, número de nascimentos e óbitos, população residente e número de pessoas empregadas com 15 anos ou mais, o projeto busca compreender como esses fatores se relacionam com o crescimento e a movimentação interna da população brasileira, especialmente com foco na migração dos responsáveis familiares.

 Além dos dados tradicionais, o projeto propõe utilizar a presença e o desempenho de clubes de futebol das Séries A e B do Campeonato Brasileiro como um indicador alternativo de desenvolvimento regional. A hipótese é que o futebol, como fenômeno cultural e econômico de grande alcance no Brasil, reflete não apenas a paixão esportiva da população, mas também aspectos estruturais e socioeconômicos das regiões, como capacidade de investimento, infraestrutura urbana e organização institucional.


## 1. Visão Geral
---

Este documento descreve o pipeline de ingestão de dados do projeto, incluindo:

>Transformações aplicadas

>Estrutura de armazenamento no Databricks

>Controles de qualidade


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
