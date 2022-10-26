# Introdução

Este artigo tem como finalidade demonstrar como o parâmetro Optimize for Ad Hoc Workloads pode ser configurado na sua instância do SQL Server, qual sua utilidade e o que a não configuração desse pode causar.

# O que é uma Ad Hoc query?

É uma query alterada de acordo com uma necessidade específica. Imagine um usuário que vai acessar a base e fazer um select qualquer, esse select é considerado uma Ad Hoc query.

# Ad Hoc query e Cache

A imagem a seguir mostra de uma forma geral o funcionamento de uma consulta qualquer em uma determinada tabela no SQL Server. O Buffer manager é o responsável por gerenciar a troca de informação entre o HD e a memória, e dentro do Plan cache é onde ficam armazenados os planos de execução das queries.

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/198142735-2f01a60c-fad4-4776-ab37-a85a908b1e5c.png">
</p>

# Parâmetro Optimize for Ad Hoc Workloads

Agora vamos entender o que o Optimize for Ad Hoc Workload faz; Observe a imagem abaixo onde temos a representação de quando o parâmetro não está habilitado.
 
<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/198143249-4fbcd3f8-1cd4-4d3c-a4cc-6d9deeab6018.png">
</p>

Cada Ad Hoc query vai ter seu plano armazenado no plan cache, e em um cenário onde existem muitas consultas Ad Hoc sendo executadas, podemos ter um cenário de pressão de memória.

Agora vamos ver o mesmo cenário só que com o parâmetro habilitado.

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/198143348-b46a9631-b7dc-4c0b-9a8c-637aeceb8804.png">
</p>

## Quais as diferenças?

Ainda temos a mesma quantidade de queries e a mesma quantidade de planos, só que diferente de antes, agora temos planos menores, é esse é o objetivo do Optimize for Ad Hoc Workload, diminuir o tamanho dos planos que serão armazenados no Plan cache.

# Como habilitar 

Acessando as propriedades da instância você pode verificar e alterar, observe a imagem.

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/198143455-96cfe87f-309a-4e89-8973-6d423847195f.png">
</p>

Você também pode verificar essa configuração fazendo um select na  sys.configurations da seguinte forma:

```TSQL
SELECT name  
, value  
, description  
FROM sys.configurations  
WHERE Name = 'optimize for ad hoc workloads' 
```
<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/198143653-040218af-beff-404d-adc4-2e633579e23a.png">
</p>

# Como verificar se há muitas consultas Ad Hoc utilizando o cache?

Podemos verificar através dos seguintes scripts:

```TSQL
SELECT  
AdHoc_Plan_MB, Total_Cache_MB,  
AdHoc_Plan_MB*100.0 / Total_Cache_MB AS 'AdHoc %'  
FROM (  
SELECT SUM(CASE  
WHEN objtype = 'adhoc'  
           THEN CONVERT(BIGINT,size_in_bytes)  
ELSE 0 END) / 1048576.0 AdHoc_Plan_MB,  
           SUM(CONVERT(BIGINT,size_in_bytes)) / 1048576.0 Total_Cache_MB  
FROM sys.dm_exec_cached_plans) T  
```

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/198143818-5c549b01-57f3-430c-b041-ce4d3ae8b68f.png">
</p>

Onde podemos ver a quantidade em MB de Ad Hoc plans armazenados, a quantidade total em MB armazenado em cache e a porcentagem que os Ad Hoc plans exercem sobre o total de memória no cache.

E 

```TSQL
SELECT SUM(c.usecounts) AS count  
, c.objtype  
FROM sys.dm_exec_cached_plans AS c  
CROSS APPLY sys.dm_exec_sql_text(plan_handle) AS t  
CROSS APPLY sys.dm_exec_query_plan(plan_handle) AS q  
GROUP BY c.objtype
```

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/198143934-ce13ae7d-ec3c-4fe3-be59-8b77202f70f2.png">
</p>

Onde podemos ver que temos apenas quatro Ad Hoc plans armazenados em cache.

# Como obter os planos que estão armazenados?

```TSQL
SELECT    cplan.usecounts  
                , cplan.objtype  
                , qtext.text  
                , qplan.query_plan  
FROM sys.dm_exec_cached_plans AS cplan 
CROSS APPLY sys.dm_exec_sql_text(plan_handle) AS qtext 
CROSS APPLY sys.dm_exec_query_plan(plan_handle) AS qplan
ORDER BY cplan.usecounts DESC
```

# Como limpar o cache?

```TSQL
DBCC FREEPROCCACHE
```
