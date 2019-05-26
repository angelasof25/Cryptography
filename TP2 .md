
<h3>Trabalho 2</h3> <br/>
<br/>
No primeiro exercício, é pedido para construir uma cifra híbrida combinando RSA e a cifra simétrica AES (com autenticação) de acordo com um protocolo descrito no enunciado do trabalho.
<br/>
Num <i>esquema de cifra RSA</i> estão definidos três algoritmos:<br/>
<ol>
    <li> <u>Geração de Chaves</u>;</li><br/>
    <li> <u>Cifra</u> (tem acesso à chave pública e à mensagem e gera o criptograma);</li><br/>
    <li> <u>Decifra</u> (tem acesso à chave privada e ao criptograma e recupera a mensagem).</li> 
</ol>
O AES (Advanced Encryption Standard) corresponde a um algoritmo de cifra por blocos, que é um modo de autenticação das operações das cifras simétricas. O protocolo a seguir é o seguinte:
<br/>
<br/>
<b>a)</b> O Emissor  determina a  chave da cifra simétrica, gerando um número aleatório  e envia-o ao Recetor, usando a cifra  <i>RSA</i>  recorrendo à chave pública do Recetor; <br/>
<b>b)</b> O Receptor e o Emissor estabelecem uma sessão, neste caso uma troca de mensagens, usando a chave acordada em a);<br/>
<b>c)</b> Cada comunicação é vista como uma comunicação assíncrona.


```python
import os
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import serialization, hashes
import cryptography
import getpass, os, io
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from BiConn import BiConn
from Auxs import hashs, mac, kdf
```

Geração dos parâmetros comuns a todas as técnicas criptográficas e públicos:


```python
class RSA(object):
    def __init__(self,security=2048, exponent=65537):

        self.private_key = rsa.generate_private_key(
                            public_exponent= exponent,
                            key_size= security,
                            backend=  default_backend())

        self.public_key = self.private_key.public_key()
```

O código seguinte representa a <b>geração das chaves</b>. Cada uma das chaves, pública e privada, é guardada num ficheiro específico, que se encontram no formato PEM, que consiste num formato de ficheiros para armazenamento e envio de chaves criptográficas, certificados, e outros dados. Estas chaves serão enviadas através da BiConn, pelo que é preciso serializá-las, o que é feito também de seguida.
É pedida uma password para garantir que o utilizador pode aceder aos ficheiros criados.


```python
def KeyGen(pk_file="pk.pem",pub_file="pub.pem"):
    
    rsa_object = RSA()
    password = bytes(getpass.getpass('password '),'utf-8')

    pk_pem = rsa_object.private_key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.PKCS8,
        encryption_algorithm=serialization.BestAvailableEncryption(password))

    pub_pem = rsa_object.public_key.public_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PublicFormat.SubjectPublicKeyInfo)
    
    
    fd = open(pk_file,"wb"); fd.write(pk_pem); fd.close()
    
    fd = open(pub_file,"wb"); fd.write(pub_pem); fd.close()
    
KeyGen()
```

    password ········


Após os passos efetuados acima para gerar as <i>key's</i> e os respetivos ficheiros, pertencentes ao método RSA, passou-se então para a construção da cifra híbrida em si.

Chamando o ficheiro com o código correspondente à criação da coneção entre o Emitter e o Receiver, criamos então duas funções com esses mesmos nomes para realizar o que pretendemos.

No Emitter começa então por ser gerada uma mensagem aleatória com 16 bytes, um vetor de inicialização a que demos o nome de <b>iv </b> (initiation vector) também de 16 bytes e também um número aleatório de 32 bytes que vai ser a chave secreta acordada entre os dois agentes. 

Depois, vamos buscar a chave pública ao ficheiro onde esta está guardada e para ser lida.

Passamos então para a cifra da secret key com RSA, utilizando a chave pública do recetor. O resultado desta cifra é o chamado ciphertext1.
É também cifrada a mensagem aleatória com AES, com uso do vetor de inicialização iv gerado anteriormente.
 
Depois disto, tanto o ciphertext1 como o vetor inicialização iv e a mensagem cifrada ct são enviados num objeto (dicionário) através da coneção.

Por sua vez, no Receiver, começa-se por pedir que este insira uma palavra-passe. Depois, este recebe a informação que foi enviada pelo Emitter através da coneção e vai buscar ao ficheiro correspondente a chave privada.

Depois, na <b>Decifra</b>, é obtido o plaintext através do RSA e o cipher1 através do AES.


```python
def Emitter(conn):  
    

    mensagem = os.urandom(16)
    print("Mensagem:", mensagem )
    
    #geração das chaves
    secret_key=os.urandom(32)
    print("Secret key:", secret_key)
    iv=os.urandom(16)
    
    backend= default_backend()
    
    #carregamento das chaves + serialização
    with open("pub.pem", "rb") as fd:
        public_key = serialization.load_pem_public_key(
            fd.read(),
            backend=default_backend())
    fd.close()
    
   
    
    #cifra da secret_key com a chave pública RSA do recetor
    ciphertext1=public_key.encrypt(
        secret_key, 
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA1()),
            algorithm=hashes.SHA1(),
            label=None
        )
    )
    print("Secret_key cifrada: ", ciphertext1)
    
    
   #cifrar a mensagem com AES
    cipher = Cipher(algorithms.AES(secret_key), modes.CBC(iv), backend= default_backend())
    encryptor = cipher.encryptor()
    ct = encryptor.update(mensagem) + encryptor.finalize()
    print("Mensagem cifrada: ", ct)
    

    obj={'ct' : ct , 'Dados cifrados RSA' : ciphertext1, 'Vetor inicialização': iv }
    
    conn.send(obj)
    
    conn.close()
    
def Receiver(conn):
    password = bytes(getpass.getpass('password '),'utf-8')
    
#Recuperar a informação
    obj = conn.recv()
    
    ciphertext = obj['Dados cifrados RSA']
    iv = obj['Vetor inicialização']
    ct = obj['ct']
    

    with open("pk.pem", "rb") as fd:
        private_key = serialization.load_pem_private_key(
            fd.read(),
            password=password,
            backend=default_backend())
    fd.close()
    

    
    plaintext = private_key.decrypt(
    ciphertext,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA1()),
        algorithm=hashes.SHA1(),
        label=None
        )
    )
    print("Secret key decifrada: ", plaintext)
    
    cipher1 = Cipher(algorithms.AES(plaintext), modes.CBC(iv), backend=default_backend())
    decryptor = cipher1.decryptor()
    ct1 = decryptor.update(ct) + decryptor.finalize()
    print("Mensagem decifrada: ", ct1)

    conn.close()

    print("Válido.")
    
    
BiConn(Emitter,Receiver).manual()
 
```

    Mensagem: b'\xf4\xdcuM\xb3\x89\xddz-\x0e\xc8\xb2\xfd\x8b\x1dq'
    Secret key: b'\x93\xa0\xfd\xa3}\xf2\xca\xb9\xe0\x00\x01\x05P\xbb\xff\xe8\x1a\x92N\xa1_\xfd\r\xa7=\x17\xce\xf3\xcb\xa8R\x86'
    Secret_key cifrada:  b',\xcb.\xb01%\x1b\xc1\x0b\xc9\x89\n\xb2p\n\x1f\x80\xe6\'\x17\x0fN\xb3\xce\xa7V0\x7fq\xe5+\xceZp\xdf:\xa7\xbf$\xaai\x8d\xa1C>{r\x8e|B:\xbe\xe8\x97\x8e\x9e\xbc7\xfah\x01\xcf9\xfb\x86\x8c\xf2\x0b\x90\xd0\x97\xb3\x10\xf0\xbcaih!9$\x96\x8b\xfbOo<\xaa.\xa9\x19kt\x19\x14_\xec\x04\'a}\xec\xbf\xd5\x8d\xcd@\xba\x89\x86\xe6\x17\xa1%\xc1\xdf\x0ec\xaf\xa4\xd6\'(1y\x08\xfa~\x8f\x15\x9a\xa7C5F1jC\xb85>\xa7I@)7\xdd\xb9\x10\xd9An\x9f\x14\xfb\xf6\xaf\x8a\x13\xc5\x17\x15\x92\x186I\xd2\xc6F_y\xadG\x8b\xe9\xf5\x1a\xaacn\xf4\x17\xb4H\x95\xea\x17s\xbc\x90\x1bX\xc9\x0c\xcd\x85\xef\xcc\xc6\xde\xe3\xeb\xcd\xd7\xb8\x9f\x190dupv\x10~\xc1\xbd\xc9p&.V\xa0\xe3d\xc2-\x9f\xc0\xfb*u\xc3N\x12?\xda\xb0\xd2\xab\xd7f\xe5\xc0M\xf8.*\xda\xe6{\x12\xda8\x90L"'
    Mensagem cifrada:  b'r\xbb\r\x8a\x1el~"\xd5\xd2d\x94x=/\r'
    password ········
    Secret key decifrada:  b'\x93\xa0\xfd\xa3}\xf2\xca\xb9\xe0\x00\x01\x05P\xbb\xff\xe8\x1a\x92N\xa1_\xfd\r\xa7=\x17\xce\xf3\xcb\xa8R\x86'
    Mensagem decifrada:  b'\xf4\xdcuM\xb3\x89\xddz-\x0e\xc8\xb2\xfd\x8b\x1dq'
    Válido.


De seguida encontra-se criada a sessão, ou seja, o diálogo com troca de mensagens entre os dois agentes Emitter e Receiver (Alice e Bob).

Decidiu-se guardar as mensagens em listas para que seja mais fácil aceder às mesmas posteriormente, ao longo da conversa. Assim, começou-se por inicializar ambas as listas.
Foi então feito um ciclo <u>While</u> que recebia os inputs correspondentes às mensagens de cada Agente e os acrescentava à lista respetiva. O critério de paragem escolhido foi que ambos os agentes digitassem a string "exit".

Após este passo, foram criadas as funções Emitter e Receiver.

<b>No Emitter</b>
<br/>
É criado um ciclo <u>FOR</u> que percorre as mensagens introduzidas anteriormente pelo emissor na lista_alice.
Para cada mensagem, é efetuada a sua <u>cifra</u> (AES). No entanto, é necessário quecada mensagem seja autenticada, pelo que é feito um hash da secret key para gerar uma tag, e com essa tal e com a mensagem cifrada anteriormente é feita a autenticação.
<br/>
Posto isto, o Emissor envia ao Recetor através da coneção a mensagem cifrada e a respetiva autenticação.
<br/>
Por sua vez, o Recetor recebe estes parâmetros e vai realizar a autenticação do seu lado da ligação. Caso este resultado seja igual ao efetuado no "Emissor", significa que a mensagem está autenticada e, portanto, o Recetor pode efetuar a <u>decifra</u> e ler o que lhe foi enviado.
<br/>
<br/>
<b>No Receiver</b>
<br/>
Começa por ir buscar a chave pública ao ficheiro.
Depois, recebe o ciphertext do "Emissor" e processa à sua <u>decifra</u> para recuperar a secret key.
Depois, à semelhança do que é feito no Emissor, é feito um ciclo <u>FOR</u> que percorre todas as mensagens que estão na lista_bob:
Após receber os parâmetros enviados pelo Emissor, como foi explicado anteriormente, o Recetor faz a autenticação através do hash e do mac. Mais uma vez, caso as tags sejam iguais, significa que a mensagem está autenticada, pelo que este pode decifrar a mensagem que recebeu, de modo a poder lê-la, e cifrar a mensagem que pretende enviar, antes de a enviar.

<b>Inicialização das Listas <b>


```python
iv=os.urandom(16)

lista_alice = []
lista_bob = []

while True:
    mensagem_alice=bytes(input("Alice, insira a sua mensagem:" ), 'utf-8')
    if mensagem_alice == bytes('exit','utf-8'):
        break
    lista_alice.append(mensagem_alice)
    mensagem_bob=bytes(input("Bob, insira a sua mensagem:" ), 'utf-8')
    if mensagem_bob == bytes('exit','utf-8'):
        break
    lista_bob.append(mensagem_bob)

```

    Alice, insira a sua mensagem:ola jjjjj
    Bob, insira a sua mensagem:hkjhkhjk
    Alice, insira a sua mensagem:exit


<b>Definição das Funções Emitter e Receiver<b>


```python
def Emitter(conn):  
    
    #geração das chaves
    secret_key=os.urandom(32)
    backend= default_backend()
    
    #carregamento da chave pública
    with open("pub.pem", "rb") as fd:
        public_key = serialization.load_pem_public_key(
            fd.read(),
            backend=default_backend())
    fd.close()
    
    
    #cifra da secret_key com a chave pública RSA do recetor
    ciphertext1=public_key.encrypt(
        secret_key, 
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA1()),
            algorithm=hashes.SHA1(),
            label=None
        )
    )
    conn.send(ciphertext1)
    
    
    #CONVERSA
    
    
    for mensagem_alice in lista_alice:
         
        #cifrar
        cipher = Cipher(algorithms.AES(secret_key), modes.CTR(iv), backend= default_backend())
        encryptor = cipher.encryptor()
        ciphertext= encryptor.update(mensagem_alice) + encryptor.finalize()
        print('O que a Alice cifrou: ', ciphertext)
        
        #criar a tag + autenticação do lado do "emissor"
        hkey = hashs(secret_key)
        h_alice = mac(hkey,ciphertext)
        
        #enviar a mensagem cifrada
        conn.send((ciphertext,h_alice))
        
        #receber
        ct_bob, h_bob = conn.recv()
        
        #autenticação do lado do "recetor"
        h_alice = mac(hkey,ct_bob)
        
        #verificação
        if h_alice == h_bob:
            print("A mensagem da Alice está autenticada.")
            #decifra
            decipher = Cipher(algorithms.AES(secret_key), modes.CTR(iv), backend=default_backend())
            decryptor = decipher.decryptor()
            mensagembob = decryptor.update(ct_bob) + decryptor.finalize()
            print("O que a Alice decifrou: ", mensagembob)
        else:
            print("Mensagem não autenticada.")
   
    conn.close()

password = bytes(getpass.getpass('password '),'utf-8')

def Receiver(conn):
    
    #carregamento da chave privada
    with open("pk.pem", "rb") as fd:
        private_key = serialization.load_pem_private_key(
            fd.read(),
            password=password,
            backend=default_backend())
    fd.close()
    
    
    ciphertext=conn.recv()
    
    #recuperar a secret key
    secret_key = private_key.decrypt(
    ciphertext,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA1()),
        algorithm=hashes.SHA1(),
        label=None
        )
    )
    
    for mensagem_bob in lista_bob:
        ct_alice, h_alice=conn.recv()
        
        #autenticação
        hkey= hashs(secret_key)
        h_bob = mac(hkey, ct_alice)
        
        if h_bob == h_alice:
            print("A mensagem do Bob está autenticada.")
            #decifra
            decipher2 = Cipher(algorithms.AES(secret_key), modes.CTR(iv), backend=default_backend())
            decryptor = decipher2.decryptor()
            mensagemalice = decryptor.update(ct_alice) + decryptor.finalize()
            print("O que o Bob decifrou: ", mensagemalice)
        else:
            print("Mensagem não autenticada.")
        #cifra
        cipher3 = Cipher(algorithms.AES(secret_key), modes.CTR(iv), backend= default_backend())
        encryptor = cipher3.encryptor()
        ciphertext = encryptor.update(mensagem_bob) + encryptor.finalize()
        print("O que o Bob cifrou: ", ciphertext)
        h_bob = mac(hkey,ciphertext)
        conn.send((ciphertext, h_bob))
    
    conn.close()

    
BiConn(Emitter,Receiver).auto()
 
```

    password ········
    O que a Alice cifrou:  b'!\x98\x03l$;\xd6\xba\xa8'
    A mensagem do Bob está autenticada.
    O que o Bob decifrou:  b'ola jjjjj'
    O que o Bob cifrou:  b'&\x9f\x08$%9\xd6\xbb'
    A mensagem da Alice está autenticada.
    O que a Alice decifrou:  b'hkjhkhjk'


<h3>Exercício 2</h3>

Neste exercício era pretendido construir uma versão do protocolo de acordo de chaves Diffie-Hellman com verificação da chave, como foi apresentado na aula teórica, mas também com autenticação dos agentes. Para a autenticação mútua utilizou-se o esquema de assinaturas DSA.

O protocolo DH contém 3 algoritmos:
+ Geração dos parâmetros
+ Agente **Alice** gera uma chave privada, a respetica chave pública e envia-a ao agente **Bob**
+ Agente **Bob** procede de forma análoga, pelo que só foi criado um agente Alice e posteriormente chamado duas vezes.
+ Seguidamente ambos os agentes computam a chave partilhada e usam uma autenticação MAC para confirmar.


```python
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.asymmetric import dh, dsa
from cryptography.hazmat.primitives import serialization, hashes
from getpass import getpass
from cryptography.exceptions import *



# Gerar os parâmetros

parameters = dh.generate_parameters(generator=2, key_size=1024,
                                     backend=default_backend())

parameters_dsa = dsa.generate_parameters(key_size=1024, backend=default_backend())
```

Depois de gerar os parâmetros do protocolo Diffie Hellman, só é criada uma função, a que damos o nome de Alice, uma vez que o processo é análogo tanto no lado do Recetor como do Emissor. Assim, são feitas duas personificações "da Alice".
Depois de ter sido criada a função Alice, é gerada a chave privada e a chave pública, também a partir desta. Uma vez que a vamos enviar pelo canal de comunicação, esta precisa de ser gerada no formato PEM.
<br/>
O <b>peer_pub_key</b> lê do canal e recebe a chave pública enviada.
<br/>
O comando <i>pk.exchange</i> é o que faz a exponenciação final, o que resulta na <b>shared_key</b>, que como o nome indica, é a chave acordada entre ambos os intervenientes. <br/>
Depois disso, é realizada uma forma de confirmação da chave: de um dos lados da coneção, é criado o hash da <i>shared key</i> e este é enviado pela coneção. Do lado do "Recetor", este recebe o que lhe foi enviado e ambos são comparados nesa Fase de Confirmação num <b>IF</b>. Caso sejam iguais, significa que não ocorreram ataques durante a partilha da informação.<br/>
Posto isto, o pretendido era utilizar o <u>esquema de assinaturas DSA</u> para a autenticação dos intervenientes.<br/>
Para isso, é criada uma chave pública e uma privada (com o DSA) para se usar nas assinaturas. O "Emissor" então assina a chave pública e assim envia-a ao "Recetor", enviando também a assinatura.
Do outro lado é efetuada a verificação.


```python
from BiConn import BiConn
import Auxs   # import hashs, mac, kdf
import getpass, os, io

def Alice(conn):
    # agreement
    pk = parameters.generate_private_key()
    pub = pk.public_key().public_bytes(
            encoding=serialization.Encoding.PEM,
            format=serialization.PublicFormat.SubjectPublicKeyInfo)
    conn.send(pub)

    
    # shared_key calculation
    peer_pub_key = serialization.load_pem_public_key(
            conn.recv(),
            backend=default_backend())
    shared_key = pk.exchange(peer_pub_key)
    
    # confirmation
    my_tag = Auxs.hashs(bytes(shared_key))
    conn.send(my_tag)
    peer_tag = conn.recv()
    if my_tag == peer_tag:
        print('OK')
    else:
        print('FAIL')
        
    
    #ASSINAR
    
   #gerar as chaves privada e pública
    private_key_dsa = parameters_dsa.generate_private_key()
    pub_dsa = private_key_dsa.public_key().public_bytes( 
            encoding=serialization.Encoding.PEM,
            format=serialization.PublicFormat.SubjectPublicKeyInfo)
    
    #envia a chave pública
    conn.send(pub_dsa)
    
    #cálculo da assinatura
    signature = sig = private_key_dsa.sign(pub, hashes.SHA256())
   
    peer_pub_dsa =serialization.load_pem_public_key( #vou deserializa-la
        conn.recv(), #agora recebo o a^y
        backend=default_backend())  
    
    
    conn.send(signature)
    
    
    #verificar
    
    
    try:
        peer = conn.recv()
        peer_pub_dsa.verify = (peer, peer_pub_key, hashes.SHA256())
        print("ok")
    except InvalidSignature:
        print("fail")
        
```


```python
BiConn(Alice,Alice).auto()

```

    OK
    OK
    ok
    ok

