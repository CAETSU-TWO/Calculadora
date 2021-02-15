# Calculadora


## Tecnologias a Utilizar 
- PHP 
- Javascript (Recomendamos jQuery ou VueJS - outra coisa qualquer) 
- HTML/CSS (se preferires SASS/SCSS também pode ser)
- MySQL/SQLite/Postgre/MariaDB (qualquer coisa serve desde que seja relacional)
 
## Look&Feel
- O HTML utilizado para a calculadora deve ter valor semântico (e.g. não usar `DIV` quando se deveria usar `BUTTON`) 
- Em termos de CSS, usar Foundation / Boostrap ou outro equivalente. Não é necessário mais do que isso. A forma como o layout é organizado fica ao teu critério.

## Schema

Todos os campos são de preenchimento obrigatório. Não deverão ser permitidos valores `NULL`.

```
- id: auto increment 
- ip: varchar (nota: endereço de IP do utilizador)
- timestamp: inteiro (nota: unix timestamp timestamp da operação) 
- operation: varchar (nota: armazenar a operação completa e.g. `1+1/2*5`) 
- result: float 
- bonus: boolean/tinyint (nota: indicar se o resultado da operação acertou no número aleatório) 
- hash: varchar (nota: a composição da hash SHA1 é a seguinte: ip + timestamp + operation + result + bonus)
- parent_hash: varchar
```

O campo `parent_hash` é a hash do registo anterior da base de dados. Para simplificar assumimos que só existe um utilizador a usar a aplicação e portanto nunca existirão dois cálculos em simultâneo que poderiam gerar conflitos.

Se a tabela ainda estiver vazia não existe `parent_hash`. Como o campo não pode ser `NULL` deverá ser usado o valor `0000000000000000000000000000000000000000`.

## Aplicação
A aplicação será composta por dois ecrãs com os elementos indicados em baixo. Deverá existir um menu que permita alternar entre ambos os ecrãs.

### Ecrã Calculadora

- Um mostrador 
- 10 botões com os números (`0` a `9`) 
- 4 botões com as operações (`+`, `-`, `*`, `/`, `MOD`) 
- 1 botão de reset do resultado e do mostrador (coloca o mostrador vazio) 
- 1 botão para realizar a operação (`=`) 
- 1 botão para o separador decimal (`.`)

### Ecrã Explorador
O ecrã do explorador serve para navegar por todos os registos na base de dados começando no primeiro registo. Deverá ser mostrada toda a informação (i.e. todos os em BD) sobre a operação que está actualmente em visualização e deverão existir dois links - um para a operação anterior (caso exista) e outro para a operação seguinte (caso exista). O timestamp deverá ser devidamente formatado.

## Modo de Funcionamento 
- Quando o utilizador carrega num botão o seu valor correspondente (excepto o igual) é concatenado ao mostrador 
- Quando o utilizador carrega no botão de realizar a operação é feito um pedido AJAX para o servidor PHP com o valor que estiver naquele momento no mostrador 
- O script PHP calcula o valor da operação e retorna o resultado em formato JSON 
- O resultado é colocado no mostrador substituindo o que estava lá anteriormente. Este valor acumula até o utilizador carregar no reset. 
- Se o resultado for igual a um número aleatório gerado a cada pedido, juntamente com o resultado, deve ser devolvida uma mensagem para mostrar ao utilizador 
- Cada pedido de operação deve ser registado numa base de dados com o seguinte schema (id, ip, data_hora, operação, resultado, bonus, hash, parent_hash)

## Workflow 
- Passo 1: Clique no botão `1` 
- Passo 2: Concatenar ao mostrador o número `1`. O mostrador mostra `1` 
- Passo 3: Clique no botão `2` 
- Passo 4: Concatenar ao valor do mostrador o número `2`. O mostrador mostra `12` 
- Passo 5: Clique no botão `+` 
- Passo 6: Concatenar ao valor do mostrador o `+`. O mostrador mostra `12+` 
- Passo 7: Clique no botão `2` 
- Passo 8: Concatenar ao valor do mostrador o número `2`. O mostrador mostra `12+2` 
- Passo 9: Clique no botão `=` faz pedido AJAX para o script PHP 
- Passo 10: O servidor processa a operação pedida, coloca os valores necessários na base de dados e envia o resultado em `JSON` para o cliente. Se houver `bonus` esse valor deve também ser retornado para o frontend e mostrar a mensagem correspondente ao utilizador
- Passo 11: Colocar o resultado retornado pelo script no mostrador. O mostrador mostra apenas `14` 
- Passo 12: Voltar ao Passo 1. Não esquecer que o valor acumula portanto o utilizador pode carregar logo no `+` e ficar `14 + ...` 

## Casos de Teste Exemplo
- `1 + 2 * 3 = 9`
- `1 - 1 * 1 + 3 = 3`
- `2 / 2 * 3 + 1 - 1 = 3`

```
1º: 2 / 2 = 1
2º: 1 * 3 = 3
3º: 3 + 1 = 4
4º: 4 - 1 = 3

Resultado: 3 
```

- `5 + 3 * 5 = 40`

```
1º: 5 + 3 = 8
2º: 8 * 5 = 40
 
Resultado: 40
```

- `5 + 1 MOD 2` 

```
1º: 5 + 1 = 6
2º: 6 MOD 2 = 0
 
Resultado: 0
```

- `5 + 1 MOD 5 / 0`

```
1º: 5 + 1 = 6
2º: 6 MOD 5 = 1
3º:  1 / 0 = ERRO
 
Resultado: ERRO
```

## Edge Cases
No algoritmo de cálculo, se for detectada uma divisão por 0 (zero), deverá ser retornado erro imediatamente em vez de dar erro PHP e crashar!

## Notas Importantes 
- Não usar eval (ou truques equivalentes) para o cálculo em PHP 
- Não é necessário avaliar expressões complexas com parêntesis (exemplos de operações válidas: `1+1`, `5+3*5`, `1+1/2`). No caso de operações como `5+3*5` não é necessário executar primeiro a multiplicação. Neste caso o processamento da operação seria: 1º `5+3` » 2º `8*5` » 3º `40` 
- Cuidado com as queries (SQL Injection!)

