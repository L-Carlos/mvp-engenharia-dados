**MVP - Engenharia de dados - PUC RJ**  
**Nome:** Luis Carlos Firmino Pinheiro  
**Matrícula:** 4052026000838  
**Data:** 09/2026

![Joao Carlos Medau from Campinas, Brazil, CC BY 2.0 <https://creativecommons.org/licenses/by/2.0>, via Wikimedia Commons](imagens/header.jpg)

# Dados de Pontualidade de Viagens Aéreas no Brasil

---

## Sumário
- [1. Contexto de Negócios e Perguntas](#1-contexto-de-negócios-e-perguntas)
- [2. Carga dos Dados](#2-carga-dos-dados)
- [3. Modelagem e Catálogo de Dados](#3-modelagem-e-catálogo-de-dados)
- [4. Pipeline de Dados](#4-pipeline-de-dados)
- [5. Qualidade de Dados](#5-qualidade-de-dados)
- [6. Análise de Dados](#6-análise-de-dados)
- [7. Autoavaliação](#7-autoavaliação)

## 1. Contexto de Negócios e Perguntas

### 1.1 Objetivo:

O objetivo deste trabalho é utilizar dados disponíveis publicamente para entender o panorama de operações aéreas no território nacional, especificamente sobre a pontualidade dessas operações, limitando-se a rotas domésticas (sendo a companhia nacional ou não), e apenas de transporte de passageiros.  
O período será limitado de Janeiro de 2023 até Julho de 2026 (último mês dispónivel).

### 1.2 Perguntas de Negócio:

Para esse projeto teremos como objetivo principal algumas perguntas:

*OBS: Índice de pontualidade (I.P.): `Número de Voos pontuais na [etapa] / Número total de voos Realizados`*

1. **Qual o I.P. de partida e chegada, por companhia, em todo o período? Como cada uma se compara a média geral?** 
1. **Qual o I.P. de partida e chegada, por companhia, ano a ano (incluindo o ano incompleto de 2026)? Existe alguma piora ou melhora nesses índices?**
1. **Qual o I.P. de partida e chegada por Rota? Quais são as 10 melhores e 10 piores?**
1. **Qual o I.P de partida e chegada por Aeroporto? Quais são os 10 melhores e os 10 piores?**
1. **Como é o I.P de partida e chegada por mês? Existe algum ciclo ou sazonalidade?**
1. **Como é o I.P de partida e chegada por dia da semana? Existe alguma tendência?**
1. **Como é o I.P de partida e chegada por horário de partida do voo? Horários com mais partidas ou chegadas tem mais atrasos?**
1. **Qual é a média de tempo de atraso de partida e de chegada, por Companhia?**
1. **Qual é a média de tempo de atraso de partida e de chegada, por Rota?**
1. **Qual é a média de tempo de atraso de partida e de chegada, por Aeroporto?**

Algumas definições prévias:
- Só serão considerados voos realizados.
- O ranking (10 melhores, 10 piores) terá um piso mínimo de dados, a definir a partir da distribuição observada.
- A pontualidade é definida pela própria ANAC e é explicita em dois campos `Situação Partida` e `Situação Chegada`. Não será feito nenhum calculo arbitrário para definir por meio de tempo de atraso, usaremos apenas a fonte oficial.

### 1.3 Contexto dos dados
Usaremos como fonte de dados o [portal de dados abertos da ANAC](https://dados.gov.br/dados/organizacoes/visualizar/agencia-nacional-de-aviacao-civil-anac).

Abaixo a lista de conjuntos de dados utilizados:  

**Nome do Conjunto:**  Voos e operações aéreas - Voo Regular Ativo (VRA)  
**Link para o conjunto:** https://dados.gov.br/dados/conjuntos-dados/dadosabertos-areas-de-atuacao-voos-e-operacoes-aereas-voo-regular-ativo-vra  
**Descrição do Conjunto:** O Voo Regular Ativo – VRA é uma base de dados composta por informações de voos de empresas de transporte aéreo regular que apresenta alterações de voos (atrasos, antecipações e cancelamentos), horários em que os voos ocorreram e as justificativas apresentadas pelas empresas aéreas para tais alterações. Por meio desta base de dados, pode ser obtida os índices de pontualidade, regularidade e de desempenho operacional além dos percentuais de atrasos e cancelamentos. O mês publicado se refere às etapas cujas decolagens eram previstas para o mês em questão ou cujas decolagens, em caso de etapa não prevista, foram realizadas no mês em questão.  
**Licença:** Creative Commons Attribution (CC-BY)  
**Metadados Oficiais:** https://www.gov.br/anac/pt-br/acesso-a-informacao/dados-abertos/areas-de-atuacao/voos-e-operacoes-aereas/voo-regular-ativo-vra/62-voo-regular-ativo-vra

**Nome do Conjunto:**  Operador aéreo - Empresas aéreas   
**Link para o conjunto:** https://dados.gov.br/dados/conjuntos-dados/operador-aereo---empresas-aereas  
**Descrição do Conjunto:** Informações do Cadastro de empresas aéreas aptas a operar no Brasil.  
**Licença:** Creative Commons Attribution (CC-BY)  
**Metadados Oficiais:** https://www.gov.br/anac/pt-br/acesso-a-informacao/dados-abertos/areas-de-atuacao/operador-aereo/empresas-aereas/metadados-operador-aereo-empresas-aereas

**Nome do Conjunto:**   Aeródromos - Lista de Aeródromos Públicos V2     
**Link para o conjunto:** https://dados.gov.br/dados/conjuntos-dados/aerodromos---lista-de-aerodromos-publicos-v2    
**Descrição do Conjunto:** ​​Dados cadastrais de aeródromos privados, que são aqueles aeródromos abertos ao tráfego aéreo de uso privativo. [^1]   
**Licença:** Creative Commons Attribution (CC-BY)  
**Metadados Oficiais:** https://www.gov.br/anac/pt-br/acesso-a-informacao/dados-abertos/areas-de-atuacao/aerodromos/aerodromos-publicos/lista-de-aerodromos-publicos/metadados-do-conjunto-de-dados-lista-de-aerodromos-publicos-v2  

[^1] Nota: a descrição desse conjunto está inconsistente com o titulo, mas foi mantida conforme está disponível na fonte oficial.


### 1.4 Resumo da Estrutura de Dados Brutos

**VRA**  
Essa será nossa tabela fato, com os dados por viagem

| Campo | Descrição |
| --- | --- |
|Sigla ICAO Empresa Aérea | Sigla/Designador ICAO Empresa Aérea |
|Empresa Aérea | Nome da empresa aérea |
|Número Voo | Numeração do voo |
|Código DI | Caractere usado para identificar o Dígito Identificador (DI) para cada etapa de voo |
|Código Tipo Linha | Caractere usado para identificar o Tipo de Linha realizada para cada etapa de voo |
|Modelo Equipamento | Modelo do equipamento utilizado no voo no padrão ICAO |
|Número de Assentos | Quantidade de assentos da aeronave |
|Sigla ICAO Aeroporto Origem | Sigla/Designador ICAO Aeroporto de Origem |
|Descrição Aeroporto Origem | Nome do aeroporto, cidade, estado e país no qual é localizado |
|Sigla ICAO Aeroporto Destino | Sigla/Designador ICAO Aeroporto de Destino |
|Descrição Aeroporto Destino | Nome do aeroporto, cidade, estado e país no qual é localizado |
|Partida Prevista | Data e horário da partida prevista informada pela empresa aérea, em horário de Brasília |
|Partida Real | Data e horário da partida realizada informada pela empresa aérea, em horário de Brasília |
|Chegada Prevista | Data e horário da chegada prevista informada pela empresa aérea, em horário de Brasília |
|Chegada Real | Data e horário da chegada realizada, informada pela empresa aérea, em horário de Brasília |
|Situação do voo | Campo informando a situação do voo: realizado, cancelado ou não informado. |
|Justificativa | Este campo deixou de ser exigido a partir de abril de 2020, com a revogação da Instrução de Aviação Civil (IAC) 1504. |
|Situação Partida | Campo informando a situação do voo na partida: Antecipado, Pontual, Atraso 30-60, Atraso 60-120, Atraso 120-240, Atraso > 240. |
|Situação Chegada | Campo informando a situação do voo na chegada: Antecipado, Pontual, Atraso 30-60, Atraso 60-120, Atraso 120-240, Atraso > 240. |

**Empresas Aéreas**  
Tabela dimensão com as informações das companhias aéreas.

| Campo | Descrição |
| --- | --- |
| ICAO | Designador de 3 letras fornecidos pela Organização de Aviação Civil Internacional (OACI) às empresas aéreas. Ressalta-se que as regras para alocação e manutenção dos designadores são realizados pela OACI, no DOC 8585. Esta Gerência Técnica de Outorgas de Serviços Aéreos – GTOS disponibiliza os dados fornecidos pelas empresas aéreas.|
| id_empresa_aerea | Empresas, nacionais ou internacionais, que exploram serviços aéreos públicos mediante prévia concessão ou autorização, conforme disciplinado pelo art.180 da Lei nº 7565, de 19 de dezembro de 1986, que dispõe sobre o Código Brasileiro de Aeronáutica. |
| Razão Social| Nome de registro da empresa constante em seu ato constitutivo. | 
| CNPJ | Cadastro Nacional de Pessoa Jurídica mantido pela RFB. |
| Atividades Aéreas | Atividades desempenhadas por empresas aéreas mediante remuneração, que abrange o disposto no art. 175 da Lei nº 7565, de 19 de dezembro de 1986, que dispõe sobre o Código Brasileiro de Aeronáutica. São exemplos de atividades aéreas o transporte aéreo regular, transporte aéreo não-regular e serviços aéreos especializados.|
| Endereço Sede | Endereço da sede que consta no Contrato ou Estatuto Social da empresa aérea. |
| Cidade | Cidade em que a empresa aérea está sediada. |
| UF | UF em que a empresa aérea está sediada. |
| CEP | CEP em que a empresa aérea está sediada. |
| Telefone | Contato telefônico da empresa aérea. |
| E-Mail | Correio eletrônico da empresa aérea. |
| Decisão Operacional | Instrumento legal que autoriza, mediante autorização ou concessão, a exploração dos serviços aéreos públicos. |
| Data Decisão Operacional | Data em que a Decisão Operacional é publicada no Diário Oficial da União. |
| Validade Operacional | Data de expiração da decisão operacional |

**Aeródromos**  
Tabela dimensão com as informações dos aeroportos.

| Campo | Descrição |
| --- | --- |
| Código OACI | Código de 4 letras utilizado internacionalmente para identificação de um aeródromo, conforme regras da Organização de Aviação Civil Internacional (OACI). No Brasil, são utilizados os códigos que iniciam com as letras SB, SD, SI, SJ, SN, SS e SW. |
| CIAD | O Código de Identificação do Aeródromo se trata de um identificador único de aeródromos civis, e é formado pela junção de dois caracteres e quatro dígitos, sendo que os caracteres são referentes à unidade da federação onde o aeródromo se localiza e os dígitos são atribuídos de forma sequencial (formato UF0000). |
| Nome | Denominação do aeródromo. |
| Município | Município onde se localiza o aeródromo, considerando as coordenadas de referência do mesmo. |
| UF | Unidade da federação do município onde se localiza o aeródromo. |
| Município Servido | Município principal servido pelo aeródromo. |
| UF Servido | Unidade da federação do município servido. |
| LATGEOPOINT | Latitude do ponto de referência do aeródromo. A coordenada geográfica é fornecida em decimais [^2]|
| LONGEOPOINT | Longitude do ponto de referência do aeródromo. A coordenada geográfica é fornecida em decimais |
| Latitude | Latitude do ponto de referência do aeródromo. A coordenada geográfica é fornecida em graus, minutos e segundos, o sistema de referência é WGS-84. |
| Longitude | Longitude do ponto de referência do aeródromo. A coordenada geográfica é fornecida em graus, minutos e segundos, o sistema de referência é WGS-84. |
| Altitude | Altitude do ponto de referência mais elevado na área de pouso, em metros. |
| Operação Diurna | Tipo de operação do período diurno para o qual o aeródromo está habilitado, considerando as regras de voo (VFR – Visual, IFR – Instrumento). |
| Operação Noturna | Tipo de operação do período noturno para o qual o aeródromo está habilitado, considerando as regras de voo (VFR – Visual, IFR – Instrumento). |
| Situação | Indicação de eventual medida administrativa aplicada pela ANAC ao aeródromo, em geral constituindo-se em algum tipo de restrição operacional. |
| Validade do Registro | Data de validade do cadastro do aeródromo, que pode ser renovada mediante requerimento prévio, desde que estejam mantidas as condições técnicas para as quais o aeródromo foi aberto ao tráfego aéreo. |
| Portaria de Registro | Número e ano da última Portaria de cadastro do aeródromo dentro da validade, ou seja, o ato que dá regularidade cadastral à infraestrutura. A Portaria é indicada no formato PAxxxx-yyyy, onde PA representa Portaria ANAC, xxxx é o ano de publicação e yyyy é o número da Portaria. Assim, por exemplo a indicação PA2020-0001 representa uma Portaria ANAC número 0001 do ano 2020. |
| Link Portaria | Link para acesso direto ao arquivo .pdf da portaria de cadastro do aeródromo. |

[^2] Nota: LATGEOPOINT e LONGEOPOINT não sao mencionados na documentação oficial, mas estão presentes no arquivo csv, seu conteudo foi inferido a partir dos dados.


## 2. Carga dos Dados

## 3. Modelagem e Catálogo de Dados

## 4. Pipeline de Dados

## 5. Qualidade de Dados

## 6. Análise de Dados

## 7. Autoavaliação

