# Exploração e análise de dados de crédito com SQL

Este notebook faz parte do segundo projeto desenvolvido no curso **Profissão: Analista de Dados** da **Escola Britânica de Artes Criativas & Tecnologia**. O projeto foi realizado a partir das aulas sobre SQL que foram ministradas por `Mariane Neiva`

Para a realização da presente análise, utilizamos o conjunto de dados disponibilizado no [**Github**](https://github.com/andre-marcos-perez/ebac-course-utils/tree/main/dataset) de `André Perez`. Como a tabela possuia mais de 10.000 linhas e nós utilizamos os serviços da **AWS** para criação da tabela, serviço este que tem um limite para utilização sem que seja necessário realizar algum pagamento, usamos o método `sample()` do pacote Pandas para selecionar uma amostra aleatória da base de dados maior.

Para realizar esta tarefa, o seguinte código foi utilizado:

```
# Download da base de dados completa
!wget -q "https://raw.githubusercontent.com/andre-marcos-perez/ebac-course-utils/main/dataset/credito.csv" -O credito.csv

# Import dos pacotes necessários

import pandas as pd
import numpy as np

# Carrega a base de dados em um DataFrame

df = pd.read_csv('credito.csv', sep=',')

# Seleciona as colunas de interesse

df = df[['idade', 'sexo', 'dependentes', 'escolaridade',
         'salario_anual', 'tipo_cartao',
         'qtd_produtos', 'iteracoes_12m',
         'meses_inativo_12m', 'limite_credito', 'valor_transacoes_12m',
         'qtd_transacoes_12m']]

# Seleciona amostra aleatória (25%) da base de dados maior

df = df.sample(frac=0.25)

# Redefine o índice do DataFrame e salva em um arquivo csv

df.reset_index(inplace=True, drop=True)

df.to_csv('sampleCredito.csv', sep=',', index=False)
```

Em seguida, foi criada uma tabela no **AWS Athena**, em conjunto com o **S3 Bucket**, com essa nova versão reduzida da base de dados. Já as visualizações dos dados foram criadas utilizando o **Power BI Desktop**.

# Estrutura dos dados:

Os dados representam informações de clientes de um banco e contam com as seguintes variáveis:

* idade = idade do cliente
* sexo = sexo do cliente (F ou M)
* dependentes = número de dependentes do cliente
* escolaridade = nível de escolaridade do clientes
* estado_civil = se o cliente é casado, solteiro ou divorciado
* salario_anual = faixa salarial do cliente
* tipo_cartao = tipo de cartao do cliente
* qtd_produtos = quantidade de produtos comprados nos últimos 12 meses
* iteracoes_12m = quantidade de iterações/transacoes nos ultimos 12 meses
* meses_inativo_12m = quantidade de meses que o cliente ficou inativo
* limite_credito = limite de credito do cliente
* valor_transacoes_12m = valor das transações dos ultimos 12 meses
* qtd_transacoes_12m = quantidade de transacoes dos ultimos 12 meses

**Comando para a criação da tabela no Athena:**

```
CREATE EXTERNAL TABLE IF NOT EXISTS default.sampleCredit (
  `idade` int,
  `sexo` string,
  `dependentes` int,
  `escolaridade` string,
  `estado_civil` string,
  `salario_anual` string,
  `tipo_cartao` string,
  `qtd_produtos` bigint,
  `iteracoes_12m` int,
  `meses_inativo_12m` int,
  `limite_credito` float,
  `valor_transacoes_12m` float,
  `qtd_transacoes_12m` int
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
  'serialization.format' = ',',
  'field.delim' = ','
) LOCATION 's3://credito-projetosql-carlos/'
TBLPROPERTIES ('has_encrypted_data'='false');
```

# Exploração de dados

Nesta primeira etapa, buscamos entender qual a nossa matéria prima, ou seja, qual o tamanho da nossa base de dados, quais tipos de dados nós vamos lidar, se o dataset contém valores nulos ou não etc.

**Qual a extensão da nossa base de dados?**

**Query1:** SELECT count( * ) FROM samplecredit;

> Resposta: 2532

Vale dizer que quanto maior a base de dados, mais confiável será a análise. No entanto, como já mencionado, utilizaremos essa base de dados reduzida por conta de limites financeiros e computacionais


**Vejamos agora como são esses dados:**

**Query2:** SELECT * FROM samplecredit LIMIT 10; 


![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery2.png?raw=true)


> A tabela apresenta alguns valores nulos (na) na coluna 'escolaridade', 'estado_civil' e 'salario_anual'. Vamos olhar os tipos de dados e os valores de cada coluna


**Quais os tipos de cada dado?**

**Query3:** DESCRIBE samplecredit;

![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery3.png?raw=true)


Vamos verificar as variáveis do tipo string. Esta parte é importante para ver se há dados ausentes nessas colunas.

**Quais são os tipos de escolaridade disponiveis na base de dados?**

**Query4:** SELECT DISTINCT escolaridade FROM samplecredit;

![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery4.png?raw=true)

> Os dados apresentam vários níveis de escolaridade: graduação, doutorado, ensino médio, sem educação formal e mestrado. Dentre os valores, encontramos alguns valores nulos nesta coluna. Nesse caso, quando formos utilizá-la, é importante que tratemos estes valores.


**E o estado civil?**

​
**Query5:** SELECT DISTINCT estado_civil FROM samplecredit;

​
![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery5.png?raw=true)

​
> Também encontramos valores nulos nesta coluna.


**Quais são as faixas salariais disponíveis?**

**Query6:** SELECT DISTINCT salario_anual FROM samplecredit;

![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery6.png?raw=true)

> Nesta coluna também temos dados faltantes. Em relação aos valores, vemos que os salários dos clientes estão disponíveis de acordo com faixas, não apresentando quanto cada cliente ganha exatamente. 


**Quais os tipos de cartão disponíveis?**

**Query7:** SELECT DISTINCT tipo_cartao FROM samplecredit;

![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery7.png?raw=true)


> Neste caso em específico não há necessidade de tratamento dos dados faltantes


# Análise de dados

Após essa fase inicial, em que já entendemos mais ou menos com qual conjunto de dados estamos lidando, podemos começar a análisar quais informações podemos extrair para entender o que está acontecendo no banco de dados. Assim, para entender quais os perfis dos clientes, vamos fazer algumas perguntas ao dataset. Essa parte é importante, pois, a partir das informações aqui extraidas, é possível traçar estratégias que vão subsidiar, por exemplo, decisões de campanhas de marketing. 


**Quantos clientes temos em cada faixa salarial?**

**Query8:** SELECT sexo, salario_anual, count( * ) as quantidade_clientes From samplecredit GROUP BY sexo, salario_anual ORDER BY quantidade_clientes DESC;

![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery8.png?raw=true)
![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Graficos/query8.png?raw=true)

> Quase 40% dos clientes dessa base de dados possuem uma renda menor do que 40 mil reais, enquanto a minoria dos clientes possuem uma renda de mais de 120 mil reais por ano. Além disso, 288 pessoas não informaram sua renda ou não consta a faixa salarial. Como o número de clientes que tem uma renda menor que 40k anual é bem maior que os das demais faixas, talvez seja interessante focar nesse público de renda mais baixa. Além disso, é interessante notar que essa base de dados não possui mulheres com salário acima de 60 mil por ano.  


**Quantos clientes são homens e quantos são mulheres?**

**Query9:** SELECT sexo, count ( * ) as quantidade_clientes FROM samplecredit GROUP BY sexo;

![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery9.png?raw=true)

![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Graficos/query9.png?raw=true)


> Aqui, vemos que a maioria dos clientes são mulheres.


**Quantos clientes são casados?**


**Query10:** SELECT estado_civil, count( * ) as quantidade_clientes FROM samplecredit WHERE estado_civil != 'na' GROUP BY estado_civil ORDER BY quantidade_clientes desc;


![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery10.png?raw=true)

> A maioria dos clientes deste dataset são casados. Os número de solteiros é um pouco menor.


**Qual é a idade dos clientes?**

​
**Query11:** SELECT sexo, min(idade) as idade_min, max(idade) as idade_max, round(avg(idade), 2) as media_idade FROM samplecredit GROUP BY sexo;
​

![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery11.png?raw=true)
​

> A média de idade dos clientes do sexo femino e masculino são bem parecidas, 46 anos.


**Qual a maior e menor transação dos clientes?**
​

**Query12:** SELECT min(valor_transacoes_12m) as transacao_minima, max(valor_transacoes_12m) as transacao_maxima FROM samplecredit;
​

![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery12.png?raw=true)
​

> Neste banco, o valor da soma das transações em 12 meses variam de 594.41 a 17.634,03


**Qual tipo de cartão lidera em quantidade de transações e valor de transações?**


**Query13:** SELECT tipo_cartao, sum(qtd_transacoes_12m) as total_qtd_transacoes, round(sum(valor_transacoes_12m), 2) as total_valor_transacoes, COUNT( * ) as quantidade_clientes FROM samplecredit GROUP BY tipo_cartao ORDER BY total_valor_transacoes desc;


![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery13.png?raw=true)


![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Graficos/query12.png?raw=true)


> Aqui, vemos que o tipo de cartão com maior número de transações no ano foi o blue, que também lidera quando olhamos a soma dos valores das transações realizadas. É interessante notar que o volume e o valor total de transações do tipo de cartão platinum está bem abaixo dos demais, embora cartões platinum geralmente sejam destinados a clientes com uma renda mais alta. Isso se deve ao fato de que o número de clientes com cartão platinum é bem menor que os demais. 


**Os clientes que possuem maior limite de crédito são os que mais gastam?**


**Quais são os maiores e menores limites de crédito?**

**Query14:** SELECT MIN(limite_credito) AS "Menor Limite de Crédito", MAX(limite_credito) AS "Maior Limite de Crédito" FROM samplecredit;

![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery-14.png?raw=true)


**Os clientes que possuem maior limite de crédito são os que possuem um gasto maior anual?**


**Query15:** SELECT sexo, tipo_cartao, valor_transacoes_12m, limite_credito FROM samplecredit WHERE limite_credito >= 30000 ORDER BY valor_transacoes_12m DESC limit 10; 

![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery-15.png?raw=true)


**Query16:** SELECT sexo, tipo_cartao, valor_transacoes_12m, limite_credito FROM samplecredit WHERE limite_credito <= 10000 ORDER BY valor_transacoes_12m DESC limit 10;


![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery-16.png?raw=true)


> As pessos com o limite máximo de crédito possuem o cartão do tipo silver e não foram as que tiveram um valor anual de gastos maior (como vimos anteriormente, o maior valor de transação foi de 17.634,03). A pessoas com um limite de crédito menor que 10 mil também parecem gastar um montante semelhante às pessoas com um limite de crédito maior.


**Query17:** SELECT sexo, tipo_cartao, round(avg(limite_credito), 2) as media_limite_credito, round(avg(valor_transacoes_12m), 2) as media_valor_transacoes, COUNT( * ) as quantidade_clientes FROM samplecredit GROUP BY sexo, tipo_cartao ORDER BY media_valor_transacoes Desc;


![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery16.png?raw=true)
![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Graficos/query16.png?raw=true)


> Entretanto, quando olhamos para a média do limite e do valor de transação por tipo de cartao e por sexo, percebemos que a maioria dos clientes possuem os valores de limite de crédito e de transação mais baixos e possuem o cartão blue. Uma minoria, que possui o cartão platinum, lidera a média de gastos e de limite de crédito.


**Qual a faixa salarial dos clientes que mais gastam?**


**Query18:** SELECT salario_anual, round(sum(valor_transacoes_12m), 2) as total_valor_transacoes, COUNT( * ) as quantidade_clientes FROM samplecredit WHERE salario_anual != 'na' GROUP BY salario_anual ORDER BY total_valor_transacoes desc;


![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery17.png?raw=true)


![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Graficos/query17.png?raw=true)


> Aqui, vemos que a faixa salarial que tem um total de gastos maior são das pessoas que ganham menos de 40 mil por ano. Como essa faixa possui um publico bem maior, ela acaba liderando no valor total de transações. Esse ranking muda quando olhamos para a média, ao invés da soma do valor de transações. 



**O salário impacta no limite e no valor das tansações?**


**Query19:** SELECT salario_anual, round(avg(limite_credito), 2) as media_limite, round(avg(valor_transacoes_12m), 2) as media_valor_transacoes FROM samplecredit WHERE salario_anual != 'na' GROUP BY salario_anual ORDER BY media_limite DESC;


![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery18.png?raw=true)

![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Graficos/query18.png?raw=true)


> Aqui vemos que a faixa salarial parece impactar diretamente no limite de crédito. Quando se trata do valor de transação, é que vemos as pessoas que ganham entre 80 mil e 120 mil dolares por ano com o menor média de valor de trasação e as pessoas que estão na faixa salarial de 60 mil a 80 mil dolares por ano ocupando a terceira posição. 


**Quais as características dos clientes que possuem os maiores créditos?**


**Query20:** SELECT escolaridade, tipo_cartao, sexo, max(limite_credito) AS limite_credito FROM sampleCredit WHERE escolaridade != 'na' AND tipo_cartao != 'na' GROUP BY escolaridade, tipo_cartao, sexo ORDER BY limite_credito desc limit 10;


![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery19.png?raw=true)


> Aqui, vemos que não parece haver um impacto da escolaridade no limite, já que o limite mais alto é oferecido a um homem sem educação formal. Como já tinhamos inferido, o tipo de cartão também não parece estar relacionado a um limite mais alto. Entre os maiores limites, há um predomínio de homens



**E de quem possui os menores créditos?**


**Query21:** SELECT escolaridade, tipo_cartao, sexo, max(limite_credito) AS limite_credito FROM sampleCredit WHERE escolaridade != 'na' AND tipo_cartao != 'na' GROUP BY escolaridade, tipo_cartao, sexo ORDER BY limite_credito ASC limit 10;


![](https://github.com/cmpbj/Analise-Exploratoria-de-Dados-de-Credito-com-SQL/blob/main/Tabelas/imgQuery20.png?raw=true)


> Agora vemos um equilíbrio entre homens e mulheres. Todos os cartões são do tipo blue. E mais uma vez a escolaridade não parece influenciar no limite de crédito.


# Conclusão

**Alguns insights:**

* Escolaridade não parece influenciar no limite de crédito
* Os clientes com maiores limites são do sexo masculino
* Quanto maior o limite, maior a chance de ter um cartão platinum
* A faixa salarial impacta no valor do limite, mas o valor do limite não impacta no valor das transações
* Não existem clientes do sexo femino com salario anual acima de 60 mil



**Perfil dos clientes**

* Maioria dos clientes são mulheres
* Maioria dos clientes são casados
* Quase 40% dos clientes possuem uma renda anual menor do que 40 mil reais por ano
* A idade média dos clientes é de 46 anos
* A maioria dos clientes possuem um cartão do tipo blue, sendo este tipo de cartão o que lidera em termos de quantidade de transação e de valor de transação por ano
* A maioria dos clientes possuem uma média de limite de crédito e de valor de trasações baixo. Aqueles que possuem o cartão platinum e uma renda mais alta tem uma média de gastos maior e um limite maior, mas são minoria
* Quem tem os maiores salários são aqueles que possuem os maiores limites

Disso, concluimos que um perfil interessante para se trabalhar numa campanha de marketing poderia ser o seguinte:
* Mulher, 46 anos, casada, possui uma renda menor que 40 mil reais anuais. Seu limite de cartão é menor do que 4200 reais e seu cartão é do tipo blue. 
