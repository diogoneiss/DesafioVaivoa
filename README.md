# Implementação da API

Não possuo conhecimentos em asp;net core, apenas em C#, portanto foi tudo muito desafiante pra mim, principalmente pela questão do tempo curto.

O projeto foi desenvolvido utilizando ASP.NET Core 5, juntamente de entity framework e swagger. Para o desenvolvimento em si utilizei o Visual Studio 2019, com o template de Web API.


## Código base

Me baseei no tutorial do [Balta](https://www.youtube.com/watch?v=but7jqjopKM&t=692s&ab_channel=balta.iobalta.io), onde uma api de produtos é criada. Serão necessárias modificações no projeto, que entrarei em detalhes em breve

## Modelo de Dados
Utilizaremos um banco de dados contendo diferentes entradas de Usuários, cada uma identificada por seu email e contendo a lista de cartões de crédito. DataAnnotations do Net Core são importantes para melhorar a robustez do dado.

Chave principal: string de email
Lista de cartões: string contendo todos os cartões concatenados, separados por vírgulas, de modo a facilitar o armazenamento. Inserir um novo cartão é super rápido, bastando concatenar o separador e o novo valor.
Lista de timestamps: string contendo todas as timestamps em ordem, mapeadas 1:1 na lista de cartões, de modo que a enésima timestamp seja o enésimo cartão da lista.  A mesma técnica de serialização com separação por vírgulas será usada.

## Lógica do programa

Teremos uma função gerar cartão, que gerará a sequência de caracteres de um cartão aleatório e retornará uma string com o número. É importante estar em string no caso do 0 no começo, que seria ignorado no caso de um inteiro.

### Post de Cartão
Essa requisição receberá como body o email do usuário, que será procurado dentro do context com o seguinte código:

    var foundItem = await context.Users.FirstOrDefaultAsync(item => item.Email == email);

 - Caso foundItem seja nulo, significa que não há um usuário com esse email cadastrado, sendo necessário criar esse usuário no banco.


- Caso seja encontrado, adicionaremos o novo cartão no usuário recuperado.
Ao criar um novo cartão, essa função será chamada e o separador decimal e resultado concatenado nos cartões existentes do usuário, de modo a serializar o objeto. 

Em seguida, será armazenada a data atual e adicionada na lista de timestamps, do mesmo modo que o novo cartão: utilizando uma lista serializada por , em formato de string.

Por fim, a API retornará o último cartão associado a esse usuário, que foi o que acabamos de criar.

### Get de Cartão
Essa requisição receberá como body o email do usuário, que será procurado dentro do context com o seguinte código:

    var foundItem = await context.Users.FirstOrDefaultAsync(item => item.Email == email);

 - Caso foundItem seja nulo, significa que não há um usuário com esse email cadastrado, retornando um json vazio.


- Caso seja encontrado, deserializaremos a lista, usando um split em vírgula, tanto para a lista de cartões quanto os timestamps. Uma lista de pares <Cartão, Timestamp> será criada, combinando os itens em ordem. Não é necessário ordenar pois a estrutura de dados já está em ordem, já que utiliza uma fila para inserção de dados. Dependendo do desejo do usuário colocaremos a timestamp convetida em data no retorno ou não. A implementação solicita especifica apenas os cartões em ordem, então retornaremos a lista de cartões deserializada.

O retorno da variável depende do email já ter sido cadastrado previamente ou não, podendo ser nulo ou 
