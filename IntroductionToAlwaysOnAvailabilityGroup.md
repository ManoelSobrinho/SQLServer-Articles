### Always On e Alta Disponibilidade

Em um ambiente de banco de dados, a perda de dados seja ela por qual motivo for é um tema muito importante de ser abordado. Nos para bancos de dados SQL Server temos algumas soluções para esse tema, e uma delas é essa. Vamos abordas alguns pontos importantes desse tema.

# 1) Grupos de disponibilidade

Um grupo de disponibilidade dá suporte a um ambiente de replicação de banco de dados, permitindo o failover entre nós para garantir a disponibilidade dos bancos de dados em casos de desastres.

# 2) Réplicas de disponibilidade

Existem dois tipos de réplica: primária e secundária, sendo a primária uma réplica única. É possível ter entre uma e oito réplicas secundárias.

# 3) Como é feita a sincronização dos dados

A réplica primária é responsável por enviar os registros de log de transação para as demais réplicas. Cada réplica recebe esse registro de log de transação e armazena no seu cache para posteriormente aplicar em seu banco de dados.









<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/183068404-ecc869c9-9f9b-48ff-91bb-567fca922020.png">
</p>





