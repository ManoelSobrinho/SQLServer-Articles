# Introdução

É possível alterar o nome de uma base de dados no SQL Server? Quais os procedumentos necessários para fazer isso? Quais os possíveis problemas?

Vamos aprofundar nos aprofundar nessas questões.

## É possível alterar o nome da base de dados?

Sim, é possível alterar o nome de uma base de dados no SQL Server com o comando:

```TSQL
ALTER DATABASE NomeAntigo MODIFY NAME = NomeNovo;
```

## Implicações e cuidados ao renomear uma base de dados:

### Conexões ativas

- O comando só será executado se não houver conexões ativas na base.

- Para garantir isso, você pode colocá-la em modo single user antes:

```TSQL
ALTER DATABASE NomeAntigo SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
ALTER DATABASE NomeAntigo MODIFY NAME = NomeNovo;
ALTER DATABASE NomeNovo SET MULTI_USER;
```
