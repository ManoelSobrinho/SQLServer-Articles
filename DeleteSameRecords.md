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

## 4ª Etapa: Verificando se existem registros iguais

```
SELECT COUNT(*) AS Quantidade, CONVERT(CHAR,x) + CONVERT(CHAR,y) AS Registro
FROM Coordenadas
GROUP BY CONVERT(CHAR,x) + CONVERT(CHAR,y) 
ORDER BY 1 DESC
```

![image](https://user-images.githubusercontent.com/25832508/175531239-e5358c23-846c-47b0-822c-3faec363db60.png)

Na primeira coluna estão a quantidade de vezes que o registro se repete e na segunda a coordenada x e y agrupada em uma só. Se na primeira coluna o valor for maior que 1, logo o valor se repetiu. Nesse caso, para 10000 registros 62 estão repetidos.

![image](https://user-images.githubusercontent.com/25832508/175531552-97d37b5f-b3a5-4fed-8b0a-f8c0a87b04a4.png)

