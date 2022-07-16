### Objetivo

Eliminar registros iguais em uma tabela.

## 1ª Etapa: Criando uma procedure que gera um par de valores aleatórios entre dois números

```TSQL
CREATE PROCEDURE [dbo].[SP_GerarCoordenada]
	@menor INT,
	@maior INT
AS
	SET @menor = 100 ---- menor número
	SET @maior = 999 ---- maior número
	SELECT ROUND(((@maior - @menor -1) * RAND() + @menor), 0) AS x, ROUND(((@maior - @menor -1) * RAND() + @menor), 0) AS y
GO
```

## 2ª Etapa: Criando uma tabela para armazenar pares de coordenadas

```TSQL
CREATE TABLE Coordenadas (
	x INT,
	y INT
)
```

## 3ª Etapa: Inserindo 10000 registros na tabela

```TSQL
INSERT INTO coordenadas
EXEC SP_GerarCoordenada 100, 999
GO 10000
```

## 4ª Etapa: Verificando se existem registros iguais

```TSQL
SELECT COUNT(*) AS Quantidade, CONVERT(CHAR,x) + CONVERT(CHAR,y) AS Registro
FROM Coordenadas
GROUP BY CONVERT(CHAR,x) + CONVERT(CHAR,y) 
ORDER BY 1 DESC
```

![image](https://user-images.githubusercontent.com/25832508/175531239-e5358c23-846c-47b0-822c-3faec363db60.png)

Na primeira coluna está a quantidade de vezes que o registro se repete e na segunda a coordenada x e y agrupada em uma só. Se na primeira coluna o valor for maior que 1, logo o registro se repete. Nesse caso, para 10000 registros 62 estão repetidos.

![image](https://user-images.githubusercontent.com/25832508/175531552-97d37b5f-b3a5-4fed-8b0a-f8c0a87b04a4.png)

## 5ª Etapa: Eliminando os valores repetidos

```TSQL
WITH CTE AS 
(SELECT x, y, ROW_NUMBER() OVER(PARTITION BY x, y ORDER BY x DESC) Linha
FROM Coordenadas)
DELETE FROM cte WHERE Linha > 1
```

![image](https://user-images.githubusercontent.com/25832508/175531960-a32793aa-ed6d-495e-8d09-3a5d5626df87.png)

Logo, como existiam 62 registros iguais, na mensagem de saída teremos 62 registros apagados, ficando apenas um para os que estavam repetidos.

## 6ª Etapa: Verificando o resultado

Agora basta novamente verificar se existem ainda valores repetidos

```TSQL
SELECT COUNT(*) AS Quantidade, CONVERT(CHAR,x) + CONVERT(CHAR,y) AS Registro
FROM Coordenadas
GROUP BY CONVERT(CHAR,x) + CONVERT(CHAR,y) 
ORDER BY 1 DESC
```

![image](https://user-images.githubusercontent.com/25832508/175532357-12c5b82d-3c20-4e2f-8da6-e92872abfa85.png)

OBS: Os mesmo procedimento pode ser feito para registros que se repetem mais de 2 vezes, para um teste com uma quantidade maior de registros repetidos basta apenas aumentar a quantidade de registros inseridos pela ```PROCEDURE``` ou inserir registros com intervalo menor. 
