# Introdução

Este artigo tem como finalidade demonstrar como o parâmetro Optimize for Ad Hoc Workloads pode ser configurado na sua instância do SQL Server, qual sua utilidade e o que a não configuração desse parâmetro pode causar.

# O que é uma Ad Hoc query?

É uma query alterada de acordo com uma necessidade específica. Imagine um usuário que vai acessar a base e fazer um select qualquer, esse select é considerado um Ad Hoc query.

# Ad Hoc query e Cache

A imagem a seguir mostra de uma forma geral o funcionamento de uma consulta qualquer em uma determinada tabela no SQL Server. O Buffer manager é o responsável por gerenciar a troca de informação entre o HD e a memória, e dentro do Plan cache é onde ficam armazenados os planos de execução das queries.

<p align="center">
<img src="![image](https://user-images.githubusercontent.com/25832508/198142614-a4f60456-0341-4907-8db7-37e2a42c51dc.png)">
</p>








