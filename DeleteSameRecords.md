### Objetivo

Eliminar registros iguais em uma tabela.

## 1ª Etapa: Criando uma procedure que gera um par de valores aleatórios entre dois números

```
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

```
CREATE TABLE Coordenadas (
	x INT,
	y INT
)
```

## 3ª Etapa: Inserindo 10000 registros na tabela

```
INSERT INTO coordenadas
EXEC SP_GerarCoordenada 100, 999
GO 10000
```

SELECT COUNT(*) AS Quantidade, CONVERT(CHAR,x) + CONVERT(CHAR,y) AS Registro
FROM Coordenadas
GROUP BY CONVERT(CHAR,x) + CONVERT(CHAR,y) 
ORDER BY 1 DESC
```
