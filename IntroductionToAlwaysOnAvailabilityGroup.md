### Always On e Alta Disponibilidade

Em um ambiente de banco de dados, a perda de dados seja ela por qual motivo for é um tema muito importante de ser abordado. Nos para bancos de dados SQL Server temos algumas soluções para esse tema, e uma delas é essa. Vamos abordas alguns pontos importantes desse tema.

# 1) Grupos de disponibilidade

Um grupo de disponibilidade dá suporte a um ambiente de replicação de banco de dados, permitindo o failover entre nós para garantir a disponibilidade dos bancos de dados em casos de desastres.

# 2) Réplicas de disponibilidade

Existem dois tipos de réplica: primária e secundária, sendo a primária uma réplica única. É possível ter entre uma e oito réplicas secundárias.

# 3) Como é feita a sincronização dos dados

A réplica primária é responsável por enviar os registros de log de transação para as demais réplicas. Cada réplica recebe esse registro de log de transação e armazena no seu cache para posteriormente aplicar em seu banco de dados.

# 4) Modos de disponibilidade

O modo de disponibilidade é a propriedade de uma réplica de disponibilidade. Ele indica se a réplica primária vai ou não esperar a confirmação das transações em uma réplica secundária. Existem dois modos de disponibilidade.

Asynchronous-commit (Assíncrono): A réplica primária confirma as transações sem precisar esperar a confirmação da réplica secundária.

Synchronous-commit (Síncrono): A réplica primária precisa esperar a confirmação da réplica secundária para confirmar a transação.

## Exemplo prático

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/183072787-d91c7539-0f12-4f90-804e-e7df4abdf798.png">
</p>

Na imagem acima temos um exemplo muito comum de como é um ambiente de alta disponibilidade.

Basicamente, temos duas réplicas que estão na mesma região com o modo de disponibilidade síncrono, e uma outra réplica no modo assíncrono que geralmente fica em outra região.

### Qual o porquê de regiões iguais ou regiões diferentes?

Se a comunicação entre duas réplicas é no modo síncrono, precisamos que elas tenham uma comunicação rápida, logo, é interessante ter ambas réplicas na mesma região.

Já a réplica assíncrona pode ou deve estar situada em uma região diferente.

### Pode ou deve?

Imagine que seus servidores estão hospedados em algum serviço de nuvem, neste caso pode ocorrer de acontecer algum problema e indisponibilizar seu servidor em determinada região, nesse caso, é importante ter uma réplica hospedada em uma região distante para não haver perda ou indisponibilidade de dados.

Bancos de dados são usados para processos de equipes diferentes e com objetivos distintos dentro de uma empresa, então é muito provável que serão extraídos relatórios desse banco de dados, relatórios esses muito pesados que podem gerar impacto no banco de dados, essa é uma das finalidades da utilização do modo assíncrono. Imagine uma extração em massa de dados para gerar um relatório feita em uma réplica secundária de modo síncrono, se o servidor de banco de dados não tiver recursos suficientes para retornar aquela consulta de forma rápida você terá um problema na replicação, pois como falado anteriormente, no modo síncrono a réplica primária depende da confirmação da secundária.

# 5) Funcionamento básico

<p align="center">
<img src="https://user-images.githubusercontent.com/25832508/183068404-ecc869c9-9f9b-48ff-91bb-567fca922020.png">
</p>

Na imagem acima temos um exemplo do funcionamento dessa forma de alta disponibilidade, porém é importante ressaltar alguns pontos importantes. 

Só um nó (node) pode ser primário, os demais são secundários.

Esse modo de alta disponibilidade utiliza um listener.




