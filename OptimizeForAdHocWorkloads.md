# Introdução

Este artigo tem como finalidade demonstrar como o parâmetro Optimize for Ad Hoc Workloads pode ser configurado na sua instância do SQL Server, qual sua utilidade e o que a não configuração desse parâmetro pode causar.

# O que é uma Ad Hoc query?

É uma query alterada de acordo com uma necessidade específica. Imagine um usuário que vai acessar a base e fazer um select qualquer, esse select é considerado um Ad Hoc query.

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







