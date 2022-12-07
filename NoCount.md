# No Count

É um parâmetro em nível de instância que permite evitar que o SQL Server imprima uma mensagem para cada linha afetada do comando. Ao configurar o SET NOCOUNT como ON fornecemos um aumento significativo de desempenho, porque o tráfego de rede é reduzido. O parâmetro também pode ser configurado em nível de sessão se estiver habilitado.

OBS: A função @@ROWCOUNT é atualizada mesmo quando o SET NOCOUNT é ON.

## Como configurar em nível de sessão

Execute a query:

```TSQL
EXEC sp_configure 'user options'
```


<p align="center">
<img src="[https://user-images.githubusercontent.com/25832508/180337386-afe785d2-234b-49cb-bb0b-3d1f161cf540.png](https://user-images.githubusercontent.com/25832508/206198897-c6b09c40-3b27-4526-833c-a05e8209db2f.png)">
</p>




