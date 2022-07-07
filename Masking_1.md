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

# Verificando se existe alguma tabela mascarada e quais campos estão mascarados

```
SELECT SMC.Name, ST.Name AS Table_Name, SMC.Is_Masked, SMC.Masking_Function  
FROM sys.masked_columns AS SMC  
JOIN sys.tables AS ST   
    ON SMC.[Object_Id] = ST.[Object_Id]  
WHERE Is_Masked = 1;
```

![image](https://user-images.githubusercontent.com/25832508/177662963-12803e69-ec4f-4ced-9391-0e599fcfedfa.png)

Como resultado da consulta temos o retorno para os campos que estão mascadados e em qual tabela

# Inserindo os mesmos dados em ambas tabelas

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





