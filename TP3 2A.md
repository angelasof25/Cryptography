
<h5> Exercício 2</h5> <br/>
Na <i> alínea a)</i> segundo exercício deste trabalho foi pedido para escrever em <b> SageMath </b> uma implementação de um esquema de cifra <b> RSA </b>.


```python
import os
```


```python
def rprime(l):
    return random_prime(2**l-1,True,2**(l-1))
```


```python
#seria ideal ter uma classe com uma funçao com os parâmetros e a função de cifrar e decifrar

l = 1024

q = rprime(l)
p = rprime(l+1)

N = p * q 
phi = (p-1)*(q-1)

G = IntegerModRing(phi) 
R = IntegerModRing(N)

e = G(rprime(512))
s = 1/e


```


```python
def cifrar(m):
    a = R(m)
    cm = a**e
    return cm
def decifrar(cm):
    b=R(cm)
    dm = b**s
    return dm
    
    
   
```


```python
m = ZZ.random_element(0,256)

print m
cm = cifrar(m)
dm = decifrar(cm)
print dm == m
```

    185
    True

