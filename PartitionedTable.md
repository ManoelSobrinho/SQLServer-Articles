# Particionamento de tabelas

Particionar uma tabela significa dividi-la logicamente em partes menores (partições), com base em um valor de coluna (ex: data, ID, região etc.). Porém, aos olhos do usuário, continua sendo uma única tabela.

# Passo a passo de como particionar uma tabela

## Criar uma função de particionamento

Ela define a regra de divisão das partições.

```TSQL
CREATE PARTITION FUNCTION pf_Tempos (INT)
AS RANGE LEFT FOR VALUES (2020, 2021, 2022, 2023, 2024, 2025);
```

Isso cria 5 partições:

  - Menor ou igual a 2020
    
  - 2021
    
  - 2022
    
  - 2023
    
  - 2024
    
  - Maior ou igual a 2025

## Criar um esquema de particionamento

```TSQL
CREATE PARTITION SCHEME ps_Tempos
AS PARTITION pf_Tempos
ALL TO ([PRIMARY]);  -- Pode distribuir em diferentes filegroups se quiser
```

## Criar a tabela particionada

```TSQL
CREATE TABLE Tempos (
    Id INT IDENTITY(1,1),
    Nome VARCHAR(50),
    Ano INT
)
ON ps_Tempos(Ano);
```

## Inserir dados normalmente

Aqui vamos inserir 10.000 linhas com valores aleatórios.

```TSQL
WITH NomesBase AS (
    SELECT TOP 1000 
        ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS N,
        CHAR(65 + ABS(CHECKSUM(NEWID())) % 26) + 
        CHAR(65 + ABS(CHECKSUM(NEWID())) % 26) + 
        CHAR(97 + ABS(CHECKSUM(NEWID())) % 26) + 
        CHAR(97 + ABS(CHECKSUM(NEWID())) % 26) + 
        CHAR(97 + ABS(CHECKSUM(NEWID())) % 26) AS NomeAleatorio
    FROM sys.all_objects
),
Registros AS (
    SELECT TOP 10000
        NomeAleatorio,
        2020 + ABS(CHECKSUM(NEWID())) % 6 AS AnoAleatorio
    FROM NomesBase
    CROSS APPLY (SELECT TOP 10 1 AS X FROM sys.all_objects) AS T -- Multiplica os nomes
)
INSERT INTO Tempos (Nome, Ano)
SELECT NomeAleatorio, AnoAleatorio
FROM Registros;
```

Esse script vai inserir na tabela Tempos 10.000 registros com Nome aleatório e Ano variando entre 2020 e 2025.

## Visualizar partições

Você pode usar a DMV para ver como os dados estão distribuídos:

```TSQL
SELECT 
    $PARTITION.pf_Tempos(Ano) AS Particao,
    COUNT(*) AS Qtde
FROM Tempos
GROUP BY $PARTITION.pf_Tempos(Ano);
```

<p align="center">
<img src="https://github.com/user-attachments/assets/5540c262-7140-42ec-9f9d-5b72246dcd0d">
</p>

## Truncar uma partição específica

Caso não seja mais necessária determinada partição, ela pode ser excluída da seguinte forma:

```TSQL
TRUNCATE TABLE Tempos WITH (PARTITIONS (3));
-- Apaga só a partição 3 (por exemplo, 2022)
```

## Mover dados com SWITCH

É possívem também mover dados de uma partição para outra tabela com o seguinte comando:

```TSQL
ALTER TABLE Tempos SWITCH PARTITION 3 TO Tempos_Arquivadas;
```

Para isso a tabela Tempos_Arquivadas deve estar criada com a mesma estrutura da tabela Tempos. Outro detalha importante, a tabela Tempos_Arquivadas não pode ser uma tabela particionada.

# Filegroups diferentes

Para criar a mesma tabela particionada porém em filegroups diferentes as etapas são semelhantes, a diferença é o esquema de partição que deve seguir esse exemplo:

```TSQL
CREATE PARTITION SCHEME ps_Tempos
AS PARTITION pf_Tempos
TO (FG_2020, FG_2021, FG_2022, FG_2023, FG_2024, FG_2025);
```

# Visualizar em qual partição está indo o registro

Podemos ver com os eguinte script:

```TSQL
SELECT 
    t.name AS Tabela,
    i.name AS Indice,
    p.partition_number,
    pfv.value AS ValorLimiteParticao,
    p.rows AS Linhas
FROM 
    sys.partitions p
    JOIN sys.indexes i ON p.object_id = i.object_id AND p.index_id = i.index_id
    JOIN sys.tables t ON i.object_id = t.object_id
    JOIN sys.partition_schemes ps ON i.data_space_id = ps.data_space_id
    JOIN sys.partition_functions pf ON ps.function_id = pf.function_id
    JOIN sys.partition_range_values pfv ON pf.function_id = pfv.function_id AND pfv.boundary_id = p.partition_number
WHERE 
    t.name = 'Tempos' AND i.index_id <= 1  -- Foco na tabela e índice clustered
ORDER BY 
    p.partition_number;
```

# Tabela particionada com campo PK

É totalmente possível criar uma tabela particionada com uma chave primária (PK) no SQL Server — mas existe uma exigência importante: a chave primária deve incluir a coluna de particionamento.

Exemplo prático:

```TSQL
CREATE PARTITION FUNCTION pfAno (INT)
AS RANGE LEFT FOR VALUES (2020, 2021, 2022, 2023, 2024, 2025);

CREATE PARTITION SCHEME psAno
AS PARTITION pfAno ALL TO ([PRIMARY]);

```

```TSQL
CREATE TABLE Tempos (
    Id INT NOT NULL,
    Nome VARCHAR(100),
    Ano INT NOT NULL,
    CONSTRAINT PK_Tempos PRIMARY KEY (Ano, Id)  -- Ano incluído na PK!
) ON psAno(Ano);  -- A tabela é particionada por 'Ano'

```

O SQL Server exige isso para garantir localidade dos dados e consistência no acesso, já que a chave primária precisa ser única em toda a tabela, e sem saber a partição correta, ele não conseguiria verificar isso de forma eficiente.

# Vantagens do particionamento

- Consultas mais rápidas (porque podem acessar apenas a partição relevante).

- Manutenção facilitada (ex: apagar dados antigos com SWITCH ou TRUNCATE de uma partição).

- Melhor gerenciamento de índices.

- Cargas e ETL mais otimizadas (quando combinadas com operações minimamente logadas).
