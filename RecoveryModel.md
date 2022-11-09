# O que é o Recovery Model

Recovery model ou modelo de recuperação de dados é uma propriedade que permite controlar a forma com que as transações são registradas, como será o comportamento do transaction log com relação às operações, e que tipo de restauração é permitida.

# Quais são os modelos de recuperação possíveis?

São três modelos: Simple, Full e Bulk logged. Dentre esses três os mais comuns de serem utilizados são o Simple e o Full.

# Entendendo cada um dos tipos de revorecy model

| Modelo  |  |  |  |
| ------------- | ------------- | ------------- | ------------- |
| Simple | Não requer backup de log  | Internamente o SQL Server realiza a limpeza recorrente do transaction log. | Algumas funções não são possíveis para neste modo, exemplo: Log shipping, Always On, Database mirroring, recuperação de mídia sem perda de dados e restaurações pontuais. |
| Full | Requer backup de log  | Todas as transações no banco de dados são registradas no transaction log. | Permite a recuperação de dados um ponto arbitrário de tempo. |
| Bulk logged | Requer backup de log |  | É um modelo complementar do modelo full. Permite operações de cópia em massa de alto desempenho e reduz o uso do transaction log utilizando o mínimo necessário para a maioria das operações. |

# Onde usar cada modelo?

Geralmente em ambientes de desenvolvimento, onde é aceitável a perda de dados, utilizamos o modelo Simple, outro motivo para utilizar ele é que com ele é menos comum ter problemas de crescimento desenfreado do transaction log, já que o próprio SQL Server automaticamente faz a limpeza do mesmo. 

Em ambiente de produção, onde não é permitida a perda de dados de forma alguma ou até em ambientes de QA que visam similar um ambiente produtivo é recomendado utilizar o modelo Full. Nesse caso é muito importante garantir uma frequência adequada dos backups de log, pois eles serão os responsáveis por garantir a restauração de mídia em um ponto específico, e também regular o tamanho do transaction log.
