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

### Objetos dependentes

- Aplicações, jobs, procedures, views, scripts, linked servers, conexões de aplicações e outros podem estar apontando para o nome antigo.

- Esses objetos não são atualizados automaticamente. Você precisará revisar e ajustar tudo que usa o nome da base.

### Backups & Restore

Backups antigos ainda funcionarão, mas durante o restore, o nome da base pode ser diferente — pode causar confusão se não documentado corretamente.

### Replicação / Log Shipping / Always On

- Em ambientes de alta disponibilidade, o renomear não é suportado diretamente se:

    - A base participa de replicação

    - Está configurada com log shipping

    - Faz parte de um grupo de disponibilidade Always On

- Nesses casos, o nome da base é parte da configuração e:

    - Você precisa remover a base do grupo, renomear, e reconfigurar o ambiente.

    - Pode causar interrupção na alta disponibilidade.
