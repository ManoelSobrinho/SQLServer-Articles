# Alteração do nome de uma base de dados e suas implicações

## Introdução

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

Com o comando abaixo você pode melhorar esse procedimento utilizando um cursor para realizar o kill de todas as sessões ativas na base e alterar o nome da mesma.

```TSQL
USE master;  
GO  
SET DEADLOCK_PRIORITY HIGH;
GO

DECLARE @session_id INT, @host_name VARCHAR(128), @tsql VARCHAR(2048);

DECLARE cursor_sessions CURSOR LOCAL FAST_FORWARD READ_ONLY FOR
SELECT s.session_id
FROM sys.dm_exec_sessions s
	INNER JOIN sys.databases AS d ON d.database_id = s.database_id
WHERE s.database_id = DB_ID('NomeAntigo')

AND s.session_id <> 57 -- sua sessao que está rodando o multi_user 

OPEN cursor_sessions  
FETCH NEXT FROM cursor_sessions INTO @session_id

WHILE @@FETCH_STATUS = 0  
BEGIN  

	  BEGIN TRY

      SET @tsql = ' kill ' + cast(@session_id  as varchar(64));

		EXEC(@tsql);
		--PRINT(@tsql);

	  END TRY
	  BEGIN CATCH
		PRINT( @tsql + ' | ' + ERROR_MESSAGE());
	  END CATCH

      FETCH NEXT FROM cursor_sessions INTO @session_id
END 

CLOSE cursor_sessions  
DEALLOCATE cursor_sessions 

ALTER DATABASE NomeAntigo SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
GO

ALTER DATABASE NomeAntigo MODIFY NAME = NomeNovo;
GO  
ALTER DATABASE NomeNovo SET MULTI_USER;
GO
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

## Riscos e perigos

| Risco                                              | Descrição                                                                                          |
| -------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| ❌ **Desconexão de aplicações**                     | Aplicações com connection strings fixas vão falhar até serem ajustadas.                            |
| 🔁 **Perda de integridade em jobs / dependências** | Jobs do SQL Server Agent, Linked Servers, SSIS, etc. podem referenciar o nome antigo.              |
| 🔐 **Segurança e permissões**                      | Permissões permanecem, mas podem haver scripts automatizados vinculados ao nome da base.           |
| 🔒 **Ambientes HA**                                | Pode interromper replicação, failover automático ou causar corrupção na configuração do Always On. |

## Boas práticas

- Evite renomear bases em produção, especialmente em ambientes de alta disponibilidade.

- Faça isso em manutenções agendadas e com validação posterior.

- Comunique todos os times (infra, dev, suporte) antes da alteração.

- Se for necessário, reconfigure todos os componentes de HA (Always On, log shipping, etc.) após o rename.

Se você estiver em um ambiente de Always On, o recomendado mesmo é criar uma nova base com o nome desejado, restaurar um backup, e reconfigurar o grupo de disponibilidade.
