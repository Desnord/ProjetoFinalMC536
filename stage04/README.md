# Etapa 04 - Análises com o Segundo Modelo Lógico

## Slides da Apresentação da Etapa
> https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/slides/slidesEtapa4.pdf

## Modelo Conceitual Atualizado

modelo antigo:
![ER1](https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/assets/entidadeRelacionamento1.png)

modelo final:
![ER2](https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/assets/entidadeRelacionamento2.png)

## Modelos Lógicos Atualizados

modelo lógico SQL:
~~~
Estado(_UF_, Nome)

Cidade(_Nome_, _Estado_)
  Estado chave estrangeira -> Estado(UF)
  
Aeroporto(_Sigla_, Descricao, Cidade)
  Cidade chave estrangeira -> Cidade(Nome)
 
Periodo(_Id_,_Semana_,_Ano_)

Rota(_Id_, Origem, Destino, VoosTotais)
  Origem chave estrangeira -> Aeroporto(Sigla)
  Destino chave estrangeira -> Aeroporto(Sigla)
 
Voo(_Rota_, _Periodo_, Quantidade)
  Rota chave estrangeira -> Rota(Id)
  Periodo chave estrangeira -> Periodo(Id)
  
Casos(_Estado_, _Periodo_, NumCasos)
  Estado chave estrangeira -> Estado(Nome)
  Periodo chave estrangeira -> Periodo(Id)
~~~

## Programa de extração e conversão de dados atualizado
Para a atualização das tabelas da etapa 3, decidimos continuar utilizando o python localmente para extrair e converter os dados, para otimizar o tempo. Mesmo que algumas partes pudessem ser feitas no jupyter, iria demandar tempo extra para um tarefa não essencial. Os scripts utilizados para gerar as tabelas se encontram na pasta src. </br>
https://github.com/Desnord/ProjetoFinalMC536/tree/main/stage04/src

## Conjunto de queries de dois modelos
As queries geradas para a revisão da etapa 3 se encontram também na pasta src. Porém, elas também estão disponíveis em um notebook.</br>
Além disso, esse notebook contém diversos selects e views, que são utilizadas para análise de dados, gerando um csv que é utilizado no notebook de análises. </br> 
pasta scr: https://github.com/Desnord/ProjetoFinalMC536/tree/main/stage04/src </br>
notebook de queries (jupyter/binder): https://github.com/Desnord/ProjetoFinalMC536/tree/main/stage04/notebooks/queries.ipynb </br>
notebook de análises (colab): https://github.com/Desnord/ProjetoFinalMC536/tree/main/stage04/notebooks/analise.ipynb </br>

Como resultado do notebook de análises, realizamos análises visuais, com dois gráficos interativos. Um deles apresenta as ocorrências de casos de gripe ao longo dos períodos, e o segundo apresenta as ocorrências de voos que chegam ao longo dos períodos. Através desses gráficos, é possível escolher quais estados mostrar as informações, exibindo por default todas as 27 curvas.
Após isso, com o modelo de regressão DecisionTreeRegressor do sklearn (ML), treinamos os dados de análise para realizar a predição dos casos em determinados período e estado. Obtemos uma confiabilidade de 92% para a predição com esse modelo, ou seja, o erro é bem pequeno ao prever casos. Assim, no notebook de análises, podemos dar como entrada um período (ano e semana), estado e quantidade de voo, e atráves do modelo treinado realizar a predição da quantidade de casos aproximados para a gripe comum. 

![G1](https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/assets/grafico1.png)
![G2](https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/assets/grafico2.png)
![PRED](https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/assets/predicao.png)

*para executar o notebook analises.ipynb, recomenda-se utilizar o google colab, por questões de compatibilidade de algumas bibliotecas utilizadas. 

### cypher 
Para a segunda parte das análises, escolhemos outro modelo lógico, o de grafos, pois este funciona melhor com a parte de rotas entre aeroportos. Cada aeroporto é um nó e as rotas são as arestas. Escolhemos usar o cypher, pois foi trabalhado em aulas e também em alguns labs, facilitando o processo.

cria nós, cada aeroporto é um nó que possui sigla e o nome da cidade em que está localizado
~~~ cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/Desnord/ProjetoFinalMC536/main/stage04/data/processed/aeroportoFINAL.csv' AS line
CREATE (:aeroporto {cidade: line.Cidade , sigla: line.Sigla})


MATCH (a:aeroporto)
RETURN a LIMIT 50
~~~

cria arestas, cada rota é uma aresta que liga dois aeroportos, e possui como atributo a quantidade de voos da rota
~~~ cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/Desnord/ProjetoFinalMC536/main/stage04/data/processed/rota.csv' AS line
MATCH (a1:aeroporto {sigla:line.Origem})
MATCH (a2:aeroporto {sigla:line.Destino})
CREATE (a1)-[r:rota {total: toInt(line.VoosTotais)}]->(a2)


MATCH p = ()-[r:rota]->() 
RETURN p LIMIT 5
~~~

imagem simplificada do grafo (mostrando apenas 25 nós e suas arestas) </br>
![AR1](https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/assets/aeroportosErotas.png)

"Pagerank" dos aeroportos com pesos (total de voos)

gera grafo do pagerank
~~~ cypher
CALL gds.graph.create('prRotas','aeroporto','rota',{relationshipProperties: 'total'})
~~~
calcula e exibe pontuação para cada aeroporto
~~~ cypher
CALL gds.pageRank.stream('prRotas',{relationshipWeightProperty: 'total'})
YIELD nodeId, score 
RETURN gds.util.asNode(nodeId).sigla AS sigla, gds.util.asNode(nodeId).cidade AS cidade, score AS pontuacao
ORDER BY pontuacao DESC
~~~
calcula e armazena pontuacao em cada aeroporto
~~~ cypher
CALL gds.pageRank.stream('prRotas',{relationshipWeightProperty: 'total'})
YIELD nodeId, score
MATCH (a:aeroporto {sigla: gds.util.asNode(nodeId).sigla})
SET a.prscore = score
~~~

comunidades dos aeroportos

gera grafo das comunidades
~~~ cypher
CALL gds.graph.create('comunidade','aeroporto','rota')
~~~

obtem e exibe comunidades
~~~ cypher
CALL gds.louvain.stream('comunidade')
YIELD nodeId, communityId
RETURN gds.util.asNode(nodeId).sigla AS sigla, communityId
ORDER BY sigla DESC
~~~

obtem comunidades e armazenas ids nos aeroportos
~~~ cypher
CALL gds.louvain.stream('comunidade')
YIELD nodeId, communityId
MATCH (a:aeroporto {sigla: gds.util.asNode(nodeId).sigla})
SET a.comunidade = communityId
~~~

Imagens com o grafo (uma geral e outra simplificada): 
</br> representando comunidades (pelas cores) 
</br> pagerank (tamanho dos nós)
</br> pesos das rotas (espessura das arestas)
![PCT1](https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/assets/pagerankcommunity.png)
![PCT2](https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/assets/pagerankcommunity2.png)

Essas imagens foram geradas com o neovis.js, utilizando o exemplo deles como base, fazendo apenas algumas pequenas modificações para exibir o nosso grafo. O arquivo html se encontra na pasta src: https://github.com/Desnord/ProjetoFinalMC536/tree/main/stage04/src/visualgraphs.html </br>

## Bases de Dados
título da base | link | breve descrição
----- | ----- | -----
`voos no brasil` | https://www.anac.gov.br/assuntos/dados-e-estatisticas/historico-de-voos | `datasets com voos no brasil, por ano e mês`
`infogripe*` | http://info.gripe.fiocruz.br | `sistema de casos de gripe no brasil`


*Para estes dados, seria necessário baixar centenas de csvs pelo sistema do infogripe, pois o sistema não tem uma api de acesso direta aos dados.
Entretanto, encontramos um projeto de um grupo que já fez essa parte de juntar os dados de cada semana, para todos os anos e estados. Assim, utilizamos
o csv do repositório deles, que pode ser encontrado no link: https://github.com/belisards/srag_brasil/blob/master/data/casos_uf.csv ou também na pasta
external. </br>

## Arquivos de Dados
nome do arquivo | link | breve descrição
----- | ----- | -----
`aeroporto.csv` | https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/data/interim/aeroporto.csv | `Arquivo CSV de aeroportos obtido na etapa 3.`
`cidade.csv` | https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/data/interim/cidade.csv | `Arquivo CSV de cidades obtido na etapa 3.`
`01voosANO.csv` | https://drive.google.com/drive/folders/1W9NUGk94Ys2_5HG5TKvImnto6k3IUcuc?usp=sharing | `Drive com todos os CSVs de voos obtidos ao final da etapa 3, e que foram utilizados como base na etapa 4.`
`casos_uf.csv` | https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/data/external/casos_uf.csv | `Arquivo CSV de casos, obtido a partir da fonte original, encontrado em outro projeto no github.`
`casos.csv` | https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/data/interim/casos.csv | `Arquivo CSV de casos, obtido a partir do anterior, apos ser processado na etapa 3.`

> Arquivos Finais

nome do arquivo | link | breve descrição
----- | ----- | -----
`aeroportoFINAL.csv` | https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/data/processed/aeroportoFINAL.csv | `csv final de aeroporto`
`cidadeFINAL.csv` | https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/data/processed/cidadeFINAL.csv | `csv final de cidade`
`casosFINAL.csv` | https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/data/processed/casosFINAL.csv | `csv final de casos`
`periodo.csv` | https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/data/processed/periodo.csv | `csv de periodo`
`estado.csv` | https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/data/processed/estado.csv | `csv de estado`
`rotaFINAL.csv` | https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/data/processed/rota.csv | `csv final de rotas`
`vooFINAL.csv` | https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/data/processed/voo.csv | `csv final de voos`
`analise.csv` | https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/data/processed/analise.csv | `csv obtido em um select feito no notebook de queries`

nome do arquivo | link | breve descrição
----- | ----- | -----
`pagerankAeroportos.csv` | https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/data/processed/pagerankAeroportos.csv | `csv de pagerank dos aeroportos`
`comunidadesAeroportos.csv` | https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/data/processed/comunidadesAeroportos.csv | `csv de comunidades de aeroportos`
