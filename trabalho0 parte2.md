**Corpos finitos primos**

Neste ponto, é-nos pedido que sejam criados três corpos finitos primos, sendo eles p=31, 127, 257. 
Para executar esta alínea, abriu-se um separador com o Sagemath. A criação dos corpos finitos é bastante simples, uma vez que apenas é necessário realizar a função GF (numero_primo). A propriedade destes corpos e respetivos números primos é que o corpo com um número finito de elementos.

De seguida, construiu-se um gráfico para o corpo criado anteriormente, através da ferramenta plot. Para tal fez-se o import da biblioteca matplotlib. Em seguida, utilizou-se o modelo disponibilizado no jupyter, ou seja, a função utilizada foi pyp.plot(). Para terminar, de modo a que o gráfico aparecesse, utilizou-se a função pyp.show().

```python
c1=GF(31)
c2=GF(127)
c3=GF(257)

from matplotlib import pyplot as pyp 

graf=pyp.plot([graf**2 for graf in c1])

pyp.show(graf)

```


![png](output_0_0.png)



```python
graf2=pyp.plot([graf2**2 for graf2 in c2])
pyp.show(graf2)
    
```


![png](output_1_0.png)



```python
graf3=pyp.plot([graf3**2 for graf3 in c3])
pyp.show(graf3)
```


![png](output_2_0.png)

