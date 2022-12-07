# No Count

É um parâmetro em nível de instância que permite evitar que o SQL Server imprima uma mensagem para cada linha afetada do comando. Ao configurar o SET NOCOUNT como ON fornecemos um aumento significativo de desempenho, porque o tráfego de rede é reduzido. O parâmetro também pode ser configurado em nível de sessão se estiver habilitado.

OBS: A função @@ROWCOUNT é atualizada mesmo quando o SET NOCOUNT é ON.

## Configuração em nível de instância

Execute a query:

```TSQL
EXEC sp_configure 'user options'
```

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/206200856-29bdae48-d16c-4ff8-8db8-d159c56e79c2.png">
</p>

Agora observe a coluna run_value, se o valor for 0 então o parâmetro está desabilitado em nível de instância. Para configurá-lo basta utilizar o seguinte comando:

```TSQL
EXEC sp_configure 'user options', 512
GO
RECONFIGURE
GO
```

E verificando novamente vemos que foi alterado corretamente.

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/206201233-322f1609-a7eb-42ee-9195-c104d102bea1.png">
</p>

## Configuração em nível de sessão

Agora em nível de sessão, vamos exemplificar utilizando a base AdventureWorks2017.

```TSQL
USE AdventureWorks2017;  
GO  
SET NOCOUNT OFF;  

SELECT TOP(100) FirstName  
FROM [Person].[Person] 
```

Execute a query e verifique a aba de Messages.

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/206203601-dc35495e-ffb9-49f8-a15d-149f02b9e4c6.png">
</p>

É possível ver que a contagem foi feita e exibida.

Agora para habilitar no mesmo exemplo de query basta trocar o SET NOCOUT OFF por SET NOCOUNT ON.

```TSQL
SET NOCOUNT ON;  

SELECT TOP(100) FirstName  
FROM [Person].[Person] 
```

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/206203374-1c44c4cd-3c37-4e6e-b268-40ff671fc111.png">
</p>

Sem contagem!
