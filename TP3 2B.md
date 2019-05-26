
Na <i> alínea b) </i> do segundo exercício foi pedido para escrever uma implementação do protocolo de acordo de chaves <b> DH </b>, na sua versão básica: sem autenticação nem confirmação.


```python
import os
from random import randint
```


```python
def rprime(l):
    return random_prime(2**l-1,True,2**(l-1))
```


```python
l = 1024

q = rprime(l)
p = rprime(l+1)
 
R = IntegerModRing(p)

secret = randint(0,500)
public = R(q)**(secret)


secret2 = randint(0,500)
public2 = R(q)**(secret2)
```


```python
shared_key = R(public2)**secret

shared_key2 = R(public)**secret2

print(shared_key == shared_key2)

```

    True

