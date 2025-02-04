# O desafio do "One Billion Row Challenge"
[Link para o desafio](https://github.com/gunnarmorling/1brc)

### Objetivo
O objetivo deste projeto é demonstrar como processar eficientemente um arquivo de dados massivo contendo 1 bilhão de linhas (~14GB), especificamente para calcular estatísticas (Incluindo agregação e ordenação que são operações pesadas) utilizando Python.

### Exemplo de como os dados estão estruturados
O arquivo de dados consiste em medições de temperatura de várias estações meteorológicas. Cada registro segue um padrão:

| id | Nome da estação | Medição |
|----|-----------------|---------|
| 1  | Hamburg         | 12.0    |
| 2  | Bulawayo        | 8.9     |
| 3  | Palembang       | 38.8    |
| 4  | St. Johns       | 15.2    |
| 5  | Cracow          | 12.6    |
| ... | ...            | ...     |
| 1000000000 | Hamburg | 17.0    |


### Objetivo do desafio
1. Ler os Arquivos -> A primeira etapa do desafio é ler os dados do arquivo contendo as medições meteorológicas.
2. Cálculo das Estatísticas (Mínimo, méida e máximo) -> Calcular as estatísticas de temperatura para cada estação meteorológica.
3. Printar as Estatísticas -> Após o cálculo das estatísticas, os resultados devem ser armazenados no formato parquet, que é altamente eficiente para análise de dados em larga escala.

![alt text](./images/pipeline.png)


### Primeiro passo
Para realizar o desafio, é necessário ter o arquivo de amostra com 44k linhas de dados e a partir dele, vamos rodar o código python 'criar_dataset_csv.py' para gerar o arquivo com 1 bilhão de linhas. Lembrando que devemos armazenar a amostra no package 'data'.

##### Observação
- Após a execução do código 'criar_dataset_csv.py', poderá levar cerca de 15 a 20 minutos para gerar o arquivo com 1 bilhão de linhas.
- Tenha no mínimo 30GB de espaço livre no disco para armazenar o processamento e armaazenamento dos dados, pois são ~15 GB de dados e o resto é utilizado para processamento.

### DuckDB
O DuckDB é um banco de dado analítico colunar projeto para consultas rápidas em grande volumes de dados. Ele é altamente eficiente para processamento de dados locais e pode ser usando direteamente com Python em arquivo CSV, Parquet, pandas dataframe sem a necessidade de um servidor. Ele é altamente eficiente em processamento de grande volumes de dados e é uma ótima alternativa para quem não quer utilizar o Pandas.

- Todos os comandos do PostgreSQL são suportados pelo DuckDB, o que significa que você pode usar o DuckDB como um banco de dados SQL local sem a necessidade de um servidor de banco de dados.

##### Principais uso do DuckDB
1. Análise de dados e BI
    - Perfeito para ETL e exploração de dados locais sem precisar de um banco de dados tradicional.
    - Pode ser usaod como alternativa para o pandas para analises grandes e rápidas 
    - Funciona bem em jupyter notebooks, explorando arquivos CSV, Parquet com consulta.
2. Aprender SQl 
    - Ele tem a API do postgres, que consegue rodar os mesmo comandos sql sem precisar subir um servidor de banco de dados
3. Data Warehousing local 
    - Pode ser usado como um mini data warehouse para analises locais sem precisar de um servidor de banco de dados.
    Aceita arquivos grandes e faz consultas eficientes diretamente sobre eeles
    processamento de colunas orimizado, tornando-o uma alternativa para BigQuery ou Clickhouse, mas sem a necessidade de infraestrutura pesada.

##### Instalação no Windows
1. Faça o download do .zip no seguinte link: [DuckDB-Download](https://github.com/duckdb/duckdb/releases/download/v1.1.3/duckdb_cli-windows-amd64.zip)
2. Extraia o arquivo .zip e coloque o executável 'duckdb_cli.exe' em uma pasta no c driv, ex: 'C:\duckdb'
3. Execute o comando 'duckdb.exe' no terminal para abrir o DuckDB
4. Ou execute o seguinte comando no terminal:
```bash
C:\duckdb
```
ou
```bash
duckdb
```


##### Alguns testes para fazer no DuckDB
1. Criar um banco de dados:
```sql	
ATTACH 'meudb.duckdb' AS meudb;
```

2. Criar uma tabela:
```sql
CREATE TABLE meudb.usuarios (
    id INTEGER PRIMARY KEY,
    nome TEXT,
    email TEXT UNIQUE,
    idade INTEGER,
    criado_em TIMESTAMP DEFAULT now()
);
```
📝 Observações:
✅ DuckDB usa TEXT em vez de VARCHAR.
✅ TIMESTAMP DEFAULT now() adiciona a data e hora automaticamente.

3. Inserir os dados
```sql
INSERT INTO meudb.usuarios (id, nome, email, idade) VALUES
    (1, 'Alice', 'alice@email.com', 25),
    (2, 'Bob', 'bob@email.com', 30),
    (3, 'Carol', 'carol@email.com', 22);
```

4. Consultar os dados 
```sql
SELECT * FROM meudb.usuarios;   
```

- Filtrar por idade
```sql
SELECT * FROM meudb.usuarios WHERE idade > 25;
```

- Ordernar por idade 
```sql
SELECT * FROM meudb.usuarios ORDER BY idade DESC;
```

- Contar os registros 
```sql
SELECT COUNT(*) as quantidade FROM meudb.usuarios;
```

5. Atualiza e deleta os dados
```sql
UPDATE meudb.usuarios SET idade = 35 WHERE nome = 'Bob';

DELETE FROM meudb.usuarios WHERE nome = 'Carol';
```

6. Trabalhando com agregação

- Média de idade dos usuários
```sql
SELECT AVG(idade) as media_idade FROM meudb.usuarios;
```

- Quantidade de usuários por idade:
```sql
SELECT idade, COUNT(*) FROM meudb.usuarios GROUP BY idade;
```

- Maior e menor idade:
```sql
SELECT MAX(idade) as maior_idade, MIN(idade) as menor_idade FROM meudb.usuarios;
```

7. Trabalhando com datas

- Ver usuários cadastrados nos últimos 7 dias
```sql
SELECT * FROM meudb.usuarios WHERE criado_em > now() - INTERVAL '7 days';
```

- Formatar data
```sql
SELECT nome, STRFTIME('%Y-%m-%d', criado_em) AS data_criacao FROM meudb.usuarios;
```

8. Exportar e Importar Dados

- Salvar dados em CSV
```sql
COPY meudb.usuarios TO 'usuarios.csv' WITH (HEADER, DELIMITER ',');
```

- Carregar CSV para o DuckDB:
```sql
CREATE TABLE meudb.novos_usuarios AS SELECT * FROM read_csv_auto('usuarios.csv');
```

- Salvando arquivos parquet diretamente:
```sql
COPY meudb.usuarios TO 'usuarios.parquet' (FORMAT 'parquet');

CREATE TABLE meudb.novos_usuarios AS SELECT * FROM read_parquet('usuarios.parquet');
```

- Salvando arquivos JSON diretamente
```sql
COPY meudb.usuarios TO 'usuarios.json' (FORMAT 'json');

CREATE TABLE meudb.novos_usuarios AS SELECT * FROM read_json('usuarios.json');
```















### Engines 
Engines são motores de processamento de dados responsáveis por interpretar e executar operação. No contexto de engenharia de dados e big data,  usamos engines para processar, transformar e analisar grandes volumes de dados. As engines são otimizadas para processar grandes volumes de dados e são altamente eficientes para operações pesadas como agregação, ordenação, join, etc.

### Segundo passo
Existema algumas maneiras de resolver o desafio e processar esses dados, vamos falar sobre algumas delas:
1. Utilizando pandas (Não recomendada) -> O pandas é uma biblioteca muito poderosa para análise de dados, mas não é otimizada para processar grandes volumes de dados. Ela é muito lenta para processar 1 bilhão de linhas e pode travar o computador. No arquivo src/pandas_solution.py, temos um exemplo de como processar os dados com pandas, mas cuidado, não execute ele.




### Duvidas sobre o projeto
1. Essa base de dados é big data?
    R. Não, para ser considerado big data, deve ter os 3 pilares (Volume, Velocidade e Variedade) e nesse caso, temos apenas o volume, vamos falar sobre cada um deles:
        - Volume: Refere-se à quantidade massiva dos dados, do processamento e do armazenamento. 
        - Velocidade: Refere-se à rapidez com que os dados são gerados, processados e analisados.
        - Variedade: Envolve os diferentes tipos e formatos de dados que precisam ser gerenciados, como: banco de dados, xml, json, imagens, videos, etc.
    Esses 3 pilares é o que forma o BIG DATA
