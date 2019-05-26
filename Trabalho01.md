

Num ponto inicial, foi-nos pedido que instalássemos o software indicado. Escolhendo a segunda opção, a Opção Máquina Virtual.

**Criação da Matriz de bits aletaória**

Geração de duas matrizes aleatórias, A e B, de dimensão 1024x1024. A matriz deve conter valores booleanos (1 e 0). Para isso executou-se a função do \"Random Sampling\" numpy.random.randint, que por si só devolve uma lista aleatória, apenas com números inteiros; no entanto, estas especificações não são suficientes e por isso dentro dos parêntesis deve-se colocar um 2 (pois só queremos utilizar os dois primeiros números 0 e 1) e ainda o tamanho que desejamos size=(1024x1024). 

```python
import numpy as ny
A=ny.random.randint(2,size=(1024,1024))#randint(low, high, size, type)
print(A)

```

    [[1 0 1 ..., 0 1 1]
     [1 1 1 ..., 0 1 0]
     [0 0 1 ..., 0 0 1]
     ..., 
     [1 1 0 ..., 1 0 0]
     [0 1 1 ..., 1 1 1]
     [0 0 1 ..., 1 0 1]]

**Matriz B** 

```python

B=ny.random.randint(2,size=(1024,1024))
print(B)

```

    [[0 1 1 ..., 1 1 0]
     [1 1 0 ..., 1 0 1]
     [0 1 1 ..., 0 1 1]
     ..., 
     [0 0 0 ..., 1 1 0]
     [1 0 0 ..., 1 1 0]
     [0 1 0 ..., 0 1 0]]



Neste ponto é pedido para explorarmos as operações de compactação e descompactação de matrizes e a sua comparação depois de efetuadas operações binárias.
Assim sendo, realizou-se o numpy.packbits para a compactação. Esta funcionalidade compacta os bits alinhando-os ao byte. 

Depois realizou se a função XOR entre as matrizes A e B e depois compactadas. Esta função é executada pelo operador ^ . Em seguida, refez-se este passo com as matrizes compactadas e decompactou se o resultado da operação.

Comparou se e o resultado foi o esperado (TRUE na comparação). 

**Compactar a matriz A**

```python
compA=ny.packbits(A,axis=-1)
print (compA)
```

    [[186 171 199 ...,  81 217  59]
     [237 132 113 ..., 101  64 122]
     [ 46  42  52 ...,  27 255 137]
     ..., 
     [214  80  79 ..., 173 122  12]
     [123 133 113 ..., 188 134 215]
     [ 44  47 143 ...,  30  92 221]]

**Compactar matriz B**

```python
compB=ny.packbits(B,axis=-1)
print (compB)
```

    [[111 126 156 ...,  55  17  30]
     [216 121 250 ..., 103 226  13]
     [ 96 235 223 ..., 102 155 107]
     ..., 
     [  5 125 175 ...,  90 252 246]
     [149 250  82 ..., 113 145  22]
     [ 87 151 195 ..., 112 249  66]]

**Operação binária XOR**

```python
opbin=A^B
print opbin
```

    [[1 1 0 ..., 1 0 1]
     [0 0 1 ..., 1 1 1]
     [0 1 0 ..., 0 1 0]
     ..., 
     [1 1 0 ..., 0 1 0]
     [1 1 1 ..., 0 0 1]
     [0 1 1 ..., 1 1 1]]

**Operação binária com matrizes compactadas**

```python
opbincomp=compA^comp
print opbincomp
```

    [[213 213  91 ..., 102 200  37]
     [ 53 253 139 ...,   2 162 119]
     [ 78 193 235 ..., 125 100 226]
     ..., 
     [211  45 224 ..., 247 134 250]
     [238 127  35 ..., 205  23 193]
     [123 184  76 ..., 110 165 159]]

**Descompactar a matriz resultante da operação**

```python
desc=ny.unpackbits(opbincomp, axis=-1)
print desc

```

    [[1 1 0 ..., 1 0 1]
     [0 0 1 ..., 1 1 1]
     [0 1 0 ..., 0 1 0]
     ..., 
     [1 1 0 ..., 0 1 0]
     [1 1 1 ..., 0 0 1]
     [0 1 1 ..., 1 1 1]]

**Verificar que as Matrizes são iguais**

```python
opbin==desc
```


    array([[ True,  True,  True, ...,  True,  True,  True],
           [ True,  True,  True, ...,  True,  True,  True],
           [ True,  True,  True, ...,  True,  True,  True],
           ..., 
           [ True,  True,  True, ...,  True,  True,  True],
           [ True,  True,  True, ...,  True,  True,  True],
           [ True,  True,  True, ...,  True,  True,  True]], dtype=bool)

**Serilização com Pickle**

Neste passo foi-nos pedido que explorássemos a serialização (funções dump e dumps) e a deserialização (função load), usando o Pickle  e o JSON. Tanto um método como o outro, na serialização, pegam numa estrutura de dados e transformam-na numa rede de bytes, preservando-a, sendo a deserialização o processo inverso. \n",
No Pickle, gravamos a informação num ficheiro e depois recuperar a partir desse mesmo ficheiro.
Enquanto que o Pickle é específico para o Python e por isso só pode ser utilizado em aplicações com esta linguagem, o JSON é universal. No entanto, o Pickle tem a vantagem de ser binário e muito mais compacto que o JSON.

Linha 2: wb: para escrever no ficheiro

Linha 3:  comando especifico para serializar

Linha4:depois de fechar, guarda o resultado da serielização no ficheiro


```python
import pickle
serie_pickle=open("resulserie.txt", "wb"            
pickle.dump(A,serie_pickle)
serie_pickle.close()

```

**Deserializar com Pickle**

rb: lê o ficheiro

Linha2: reverte a serialização.


```python
abrir_serie=open("resulserie.txt", "rb")
desserie=pickle.load (abrir_serie) 
print (desserie)
abrir_serie.close()

```

    [[1 0 1 ..., 0 1 1]
     [1 1 1 ..., 0 1 0]
     [0 0 1 ..., 0 0 1]
     ..., 
     [1 1 0 ..., 1 0 0]
     [0 1 1 ..., 1 1 1]
     [0 0 1 ..., 1 0 1]]

**Serilização com Json**

```python
import json
serie_json=open("resuljson.txt", "w")#
json.dump(A.tolist(), serie_json)
serie_json.close()

```

**Deserializar com Json**

Linha3: peculiaridade do json que obriga a criação de lista.


```python
abrir_seriej=open("resuljson.txt", "r")
desserie_j=json.load(abrir_seriej)
print(ny.asarray(desserie_j))
abrir_seriej.close()

```

    [[1 0 1 ..., 0 1 1]
     [1 1 1 ..., 0 1 0]
     [0 0 1 ..., 0 0 1]
     ..., 
     [1 1 0 ..., 1 0 0]
     [0 1 1 ..., 1 1 1]
     [0 0 1 ..., 1 0 1]]

Neste ponto foi-nos pedido que explorássemos cifrar objetos serializados. Neste sentido, utilizamos o Fernet em Cryptography, que garante que uma mensagem encriptada não pode ser lida nem manipulada sem uma chave (KEY). 

**Cifrar com Fernet e objetos serializados**

Linha3: gera a chave associada à cifra.

```python
from cryptography.fernet import Fernet
cifrarjson=open ("cifraaresul.txt", "w")
key=Fernet.generate_key()
f=Fernet(key)
token = f.encrypt(json.dumps(A.tolist()))
cifrarjson.write(token)
cifrarjson.close()
```

**Decifrar com Fernet e objetos serializados**


```python
decifrarjson=open("descifraresul.txt", "w")
decif=f.decrypt(token)
decifrarjson.write(decif)
decifrarjson.close()
```
