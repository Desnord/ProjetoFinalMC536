<div align="center"><h1> Projeto final da disciplina de Banco de Dados </h1></div>
<div align="center"><h3> Grupo: Glicemia</h3></div>
<div align="center"> Thomas Gomes Ferreira - 224919 </div>
<div align="center"> Pedro Jun Novais - 204878 </div>

## Resumo do Projeto
O projeto final da disciplina de banco de dados consiste em encontrar diversas fontes de dados, relacionados à um tema da saúde, com formatos diferentes e assim trabalhar para que possam ser interligados. E após isso, realizar algum tipo de análise. 
No nosso caso, trabalhamos com alguns dados que podem ser relacionados a gripe comum, e com isso realizamos a predição da quantidade de casos em um determinado estado e período. Além disso, as outras análises que foram feitas são visuais, exibindo dois gráficos que relacionam casos x período e voos x período, e os grafos de aeroportos e rotas. 

</br>
</br>
<div align="center"><h2> Stage02</h2></div>
Esse estágio serviu apenas para decidir o assunto a ser trabalhado, análises planejadas para serem feitas e algumas fontes de dados que poderiam ser utilizadas no projeto.
</br>
</br>
<div align="center"><h2> Stage03</h2></div>
Nessa etapa, escolhemos as duas fontes de dados que utilizamos em todo o projeto. De forma resumida, uma das fontes possuia os casos de gripe e a outra os dados de voos e cidades. Após escolher as fontes, iniciamos o processamento dos dados para interligá-los.

### Modelo Entidade Relacionamento
![ER1](https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/assets/entidadeRelacionamento1.png)

### Modelo Lógico
~~~
Estado(_UF_, Nome)

Cidade(_Nome_, _Estado_)
  Estado chave estrangeira -> Estado(UF)
  
Aeroporto(_Sigla_, Descricao, Cidade)
  Cidade chave estrangeira -> Cidade(Nome)
  
Voo(_VooID_, Partida, Chegada, Origem, Destino)
  Rota chave estrangeira -> Aeroporto(_Sigla_)
  Periodo chave estrangeira -> Aeroporto(_Sigla_)
  
Casos(_Estado_, _Semana_, _Ano_, NumCasos)
  Estado chave estrangeira -> Estado(Nome)
  Periodo chave estrangeira -> Periodo(Id)
~~~

### Processamento dos dados
Todos as fontes de dados estavam apresentadas através de arquivos csv. Baixamos os arquivos e processamos os dados através de scripts python e utilizando o Orange. Até aqui, não conseguimos implementar tudo que gostariamos, e tivemos alguns problemas com a interligação dos dados, que foi revisada e completada na etapa 4. O principal problema, estava com a tabela de voos, que possuia um volume muito grande de dados. 

### Queries realizadas
Algumas queries foram feitas nessa etapa, porém nenhuma análise foi feita com elas. Apenas para deixar como exemplo, uma das queries feitas foi a seguinte:

~~~sql
SELECT v.VooID, v.Origem, v.Destino, v.Partida, v.Chegada 
       FROM Voo v, Aeroporto aero, Cidade cid
       WHERE v.Destino = aero.Sigla and aero.Cidade = cid.Nome and cid.Estado = 'SP';	
~~~

### Considerações sobre essa etapa
De modo geral, conseguimos direcionar o projeto da forma que gostariamos, apesar dos problemas encontrados. Para a etapa 4, continuamos o que foi feito na etapa 3, complementando e adicionando o que fosse necessário. Assim, arrumamos o problema da tabela de voos, e realizamos as análises desejadas desde o começo do projeto, com sucesso. 
 
 </br>
 </br>
<div align="center"><h3> Stage04 e Final</h3></div>
Na etapa 4 revisamos e terminamos o processamento iniciado na etapa 3, obtendo as análises desejadas com o modelo relacional: gráficos e predição de casos. Além disso, implementamos um segundo modelo, o de grafos, atráves do neo4j/cypher. Com esse segundo modelo, realizamos uma análise visual de alguns dados.

## Processamento dos dados
Para processar os dados, continuamos do final da etapa 3 e utilizamos scripts python para processar os últimos csvs obtidos até então. Aqui, acrescentamos duas tabelas a fim de reduzir o volume de dados de voos, no caso, as tabelas periodo e rota. Com a tabela de periodo, ao invés de exibir os dados por dia e ano, agrupamos todos os voos por semana e separamos por rota. Assim, cada voo é uma rota que ocorre em um período, e possui uma quantidade de ocorrências.

### Modelo Entidade Relacionamento
O modelo entidade relacionamento foi revisado em conjunto com o modelo lógico, a fim de resolver os problemas encontrados com a tabela de voos:
![ER2](https://github.com/Desnord/ProjetoFinalMC536/blob/main/stage04/assets/entidadeRelacionamento2.png)

### Modelo Lógico
Esse foi o modelo lógico final obtido, em que foram adicionados duas tabelas, além de alterar as tabelas já existentes. Com esse modelo, foi possível reduzir a tabela de voos
que tinha ~10 milhões de linhas, para apenas ~3500 linhas, perdendo apenas um pouco de informação.

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

### Análises - Modelo Tabular (SQL)


### Análises - Modelo De Grafos (Neo4j/Cypher)


