### Objetivo

Eliminar registros iguais em uma tabela.

## 1ª Etapa: Criando uma procedure que gera um par de valores aleatórios entre dois números <a name="introduction"></a>

```
USE [Teste]
GO

CREATE PROCEDURE [dbo].[SP_GerarCoordenada]
	@maior INT,
	@menor INT
AS
	SET @menor = 100 ---- menor número
	SET @maior = 999 ---- maior número
	SELECT ROUND(((@maior - @menor -1) * RAND() + @menor), 0) AS x, ROUND(((@maior - @menor -1) * RAND() + @menor), 0) AS y
GO
```
