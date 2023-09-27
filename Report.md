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


Pesquisando, encontrei que o erro ocorre quando você realiza mais de um SELECT na mesma query e eles retornam uma quantidade diferente de colunas. Nesse caso, conseguimos deduzir que o primeiro SELECT está buscando duas colunas (name, surname). Portanto, temos que buscar duas colunas também.
```
id=1 UNION SELECT user, password FROM users&Submit=Submit
```


