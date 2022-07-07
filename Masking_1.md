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
