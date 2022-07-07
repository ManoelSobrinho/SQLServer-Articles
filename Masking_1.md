### Mascaramento utilizando funções do SQL Server

# 1) Criando as tabelas

Serão criadas duas tabelas, uma que não será mascarada e outra que será.

```
CREATE TABLE Clientes (
	ID_Cliente INT PRIMARY KEY IDENTITY(1,1),
	Nome VARCHAR(20),
	Sobrenome VARCHAR(20),
	Telefone VARCHAR(10),
	Email VARCHAR(20)
)
```

```
CREATE TABLE Mask_Clientes (
	ID_Cliente INT PRIMARY KEY IDENTITY(1,1),
	Nome VARCHAR(20) MASKED WITH (FUNCTION = 'partial(1,"XXXXXXX",0)') NULL,
	Sobrenome VARCHAR(20) MASKED WITH (FUNCTION = 'partial(2,"XXXXXXX",0)') NULL,
	Telefone VARCHAR(10) MASKED WITH (FUNCTION = 'default()') NULL,
	Email VARCHAR(20) MASKED WITH (FUNCTION = 'email()') NULL
)
```

# 2) Verificando se existe alguma tabela mascarada e quais campos estão mascarados

```
SELECT SMC.Name, ST.Name AS Table_Name, SMC.Is_Masked, SMC.Masking_Function  
FROM sys.masked_columns AS SMC  
JOIN sys.tables AS ST   
    ON SMC.[Object_Id] = ST.[Object_Id]  
WHERE Is_Masked = 1;
```

![image](https://user-images.githubusercontent.com/25832508/177662963-12803e69-ec4f-4ced-9391-0e599fcfedfa.png)

Como resultado da consulta temos o retorno dos campos que estão mascadados e em qual tabela

# 3) Inserindo os mesmos dados em ambas tabelas

```
INSERT INTO Clientes(Nome, Sobrenome, Telefone, Email)
VALUES
	('Pedro','Cardozo','8899-7788','pedro.ca@gmail.com'),
	('João','Paulo','9878-6699','jpaulo@outlook.com'),
	('Maria','Clara','9877-1222','mclara@gmail.com'),
	('Francisco','Carlos','9969-9632','fcarlos@outlook.com'),
	('Cátia','Cardozo','8844-0124','ccardozo@gmail.com')
```

```
INSERT INTO Mask_Clientes(Nome, Sobrenome, Telefone, Email)
VALUES
	('Pedro','Cardozo','8899-7788','pedro.ca@gmail.com'),
	('João','Paulo','9878-6699','jpaulo@outlook.com'),
	('Maria','Clara','9877-1222','mclara@gmail.com'),
	('Francisco','Carlos','9969-9632','fcarlos@outlook.com'),
	('Cátia','Cardozo','8844-0124','ccardozo@gmail.com')
```

# 4) Fazendo um SELECT na tabela Mask_Clientes

```
SELECT * FROM Mask_Clientes
```

![image](https://user-images.githubusercontent.com/25832508/177663118-95c85844-1e74-49db-bb56-c585e1471df3.png)

É possível ver que nenhum campo está mascarado, isso pelo fato de o login que estou utilizando tem permissões suficientes para ver dados até mesmo em uma tabela mascarada. Logo, vamos criar um usuário que não precise de login para ver se o mascaramento realment está funcionando.

# 5) Criando um usuário para testar resultado do mascaramento

Criando o usuário com permissão para fazer SELECT nas duas tabelas.

```
CREATE USER Teste WITHOUT LOGIN;  
GRANT SELECT ON Clientes TO Teste;  
GRANT SELECT ON Mask_Clientes TO Teste;
```

# 6) SELECT utilizando o usuário criado

```
EXECUTE AS USER = 'Teste';  
SELECT * FROM Clientes; 
SELECT * FROM Mask_Clientes; 
REVERT; 
```

![image](https://user-images.githubusercontent.com/25832508/177663468-f86d23de-342e-4a3d-828d-37202ca99d2a.png)

É possível notar que realmente o mascaramento funcionou.

Agora que vimos o funcionamento, podemos fazer alterações caso necessário.

# 7) Alterando ou adicionando o mascaramento de uma coluna

```
ALTER TABLE Mask_Clientes  
ALTER COLUMN Sobrenome VARCHAR(20) MASKED WITH (FUNCTION = 'default()'); 
```

![image](https://user-images.githubusercontent.com/25832508/177663666-212fc23c-04d3-40e8-81d8-2e7b0b025fa5.png)

É possível notar que agora a coluna Sobrenome só mostra x para qualquer registro.

# 8) Dando permissão para um usuário ver os dados de uma tabela mascarada sem a máscara

```
GRANT UNMASK TO Teste;  
EXECUTE AS USER = 'Teste';  
SELECT * FROM Mask_Clientes;  
REVERT;
```

![image](https://user-images.githubusercontent.com/25832508/177663798-07e5128e-b9c8-49c5-93b1-f2f4cf829bae.png)

# 9) Retirando permissão de um usuário ver os dados de uma tabela mascarada sem a máscara.

```
REVOKE UNMASK TO Teste; 
EXECUTE AS USER = 'Teste';  
SELECT * FROM Mask_Clientes;  
REVERT;
```

![image](https://user-images.githubusercontent.com/25832508/177663907-cf843946-dd9e-41ba-8a7a-352806c3e8a3.png)

# 10) Eliminando a máscara de uma coluna mascarada

```
ALTER TABLE Mask_Clientes   
ALTER COLUMN Sobrenome DROP MASKED; 
```

```
EXECUTE AS USER = 'Teste';  
SELECT * FROM Mask_Clientes;  
REVERT;
```

![image](https://user-images.githubusercontent.com/25832508/177664317-62da1aab-b8ce-43d2-822c-d487d3ad2199.png)
