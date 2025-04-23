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


# Vantagens do particionamento

- Consultas mais rápidas (porque podem acessar apenas a partição relevante).

- Manutenção facilitada (ex: apagar dados antigos com SWITCH ou TRUNCATE de uma partição).

- Melhor gerenciamento de índices.

- Cargas e ETL mais otimizadas (quando combinadas com operações minimamente logadas).
