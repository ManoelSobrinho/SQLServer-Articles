### Funcionamento básico do Change Data Capture (CDC)

É muito importante que um ambiente de banco de dados seja monitorado e auditado, para isso existem muitos conceitos e ferramentas, e uma delas é o Change Data Capture.

# O que é Change Data Capture?

De uma forma bem simplificada, o Change Data Capture é um processo que verifica a alteração de dados. No SQL Server existe uma forma de implementar essa funcionalidade desde a versão do SQL Server 2008. Uma grande vantagem de implementar o Change Data Capture é a facilidade, vamos ver na prática.

# 1) Verificando em quais databases o CDC está habilitado.

Para verificar é simples, basta utilizar o comando:

```TSQL
SELECT [name], is_cdc_enabled 
FROM sys.databases
```

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/180337386-afe785d2-234b-49cb-bb0b-3d1f161cf540.png">
</p>

# 1) Criando uma database e habilitando o CDC.

```TSQL
CREATE DATABASE Teste_CDC
```

```TSQL
USE Teste_CDC 
GO
 
EXEC sys.sp_cdc_enable_db 
GO
```

Agora podemos verificar novamente as databases que estão com CDC habilitado.

```TSQL
SELECT [name], is_cdc_enabled 
FROM sys.databases
WHERE is_cdc_enabled = 1
```

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/180337999-141af854-9d02-4596-9dfc-55ed1a8971b9.png">
</p>

É importante também saber em quais tabelas o CDC está habilitado, porém nesse caso ainda não habilitamos.

```TSQL
SELECT [name], is_tracked_by_cdc
FROM sys.tables
```

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/180338294-cfc3fff2-e21e-4115-90ea-4c079e96eeb8.png">
</p>

Agora precisamos criar uma tabela e habilitar o CDC nela.

Criando a tabela:

```TSQL
CREATE TABLE Pessoas (
	ID_Pessoa INT PRIMARY KEY IDENTITY(1,1),
	Nome VARCHAR(20)
)
```

Habilitando o CDC:

```TSQL
USE Teste_CDC
GO 
 
EXEC sys.sp_cdc_enable_table 
@source_schema = N'dbo', 
@source_name   = N'Pessoas', 
@role_name     = NULL 
GO
```

Após alguns segundos você receberá a mensagem quando for habilitado:

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/180338747-817231c8-78c1-466c-ae3d-360292ab8865.png">
</p>

Agora é possível ver que existem novas tabelas de sistema relacionadas ao CDC

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/180339010-2d11592d-a77f-46ef-a15c-680415268873.png">
</p>

Também é possível ver que o schema CDC foi criado.

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/180339141-cc54064c-98da-4612-ad15-bebc10604fea.png">
</p>

# 2) Alterando dados e verificando se o CDC está funcionando

Inicialmente a tabela do CDC estará vazia.

```TSQL
SELECT * FROM [cdc].[dbo_Pessoas_CT]
```

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/180339516-641b1d25-adb1-48ef-aad6-758d2549a993.png">
</p>

Podemos ver também que o CDC criou as colunas de metadados.

Agora vamos inserir alguns dados na tabela [dbo].[Pessoas]

```TSQL
INSERT INTO [dbo].[Pessoas] (Nome)
	VALUES
		('Tiago'),
		('Maria'),
		('João'),
		('Pedro'),
		('Carla')
```

Em seguida vamos verificar se algo foi anterado na tabela [cdc].[dbo_Pessoas_CT]

```TSQL
SELECT * FROM [cdc].[dbo_Pessoas_CT]
```

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/180339837-ec2ff4ad-cac3-438f-b555-e12abb2086b9.png">
</p>

Agora vamos fazer alguns DELETES e ALTERS na [dbo].[Pessoas] e novamente verificar a tabela do CDC.

```TSQL
UPDATE [dbo].[Pessoas]
SET Nome = 'Paulo' WHERE ID_Pessoa = 5
```

Como resultado na tabela CDC tivemos: 

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/180340237-cb8eaa0b-28cf-4a21-a990-4dc5723cc87e.png">
</p>

```TSQL
DELETE FROM [dbo].[Pessoas]
WHERE ID_Pessoa = 1
```

Como resultado na tabela CDC tivemos: 

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/180340354-da1ea6a7-204e-45db-97b1-fc98838552cb.png">
</p>

# 3) Explicando o básico para entender uma tabela de CDC

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/180341310-4c2cb9e6-701b-45a6-8bec-513b86bbc25f.png">
</p>

Podemos ver que ao inserir na tabela todos os valores da coluna __$operation são iguais a 2, o que significa que foi feita uma inserção. (Bloco Vermelho)

Na coluna __$command_id tivemos valores de 1 até 5, pois foram feitas 5 insersões. (Bloco Vermelho)

Ao alterar dados tivemos na coluna __$operation os valores 3 e 4. 3 indicando o valor antigo e 4 o valor alterado. É possível ver isso pela coluna ID_Pessoa, que é 5 para amboos os registros. (Bloco Amarelo)

Por fim, ao deletar, podemos que que na coluna __$operation obtivemos o valor 1, que indica que aquele registro foi deletado. (Bloco Verde)
