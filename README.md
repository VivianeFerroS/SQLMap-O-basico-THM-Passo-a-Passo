# SQLMap-O-basico-THM-Passo-a-Passo

## Tarefa 1
## Introdução
A injeção de SQL é uma vulnerabilidade prevalente e há muito tempo é um tópico quente em segurança cibernética. Para entender essa vulnerabilidade, precisamos primeiro aprender o que é um banco de dados e como os sites interagem com um banco de dados.

Um banco de dados é uma coleção de dados que podem ser armazenados, modificados e recuperados. Ele armazena dados de vários aplicativos em um formato estruturado, tornando o armazenamento, a modificação e a recuperação fáceis e eficientes. Você interage com vários sites diariamente. O site contém algumas das páginas da web onde a entrada do usuário é necessária. Por exemplo, um site com uma página de login pede que você insira suas credenciais e, depois que você as insere, ele verifica se as credenciais estão corretas e faz o login se estiverem. Como muitos usuários fazem login naquele site, como esse site registra todos os dados desses usuários e os verifica durante o processo de autenticação? Tudo isso é feito com a ajuda de um banco de dados. Esses sites têm bancos de dados que armazenam o usuário e outras informações e as recuperam quando necessário. Então, quando você insere suas credenciais na página de login de um site, o site interage com seu banco de dados para verificar se essas credenciais estão corretas. Da mesma forma, se você tem um campo de entrada para pesquisar algo, por exemplo, um campo de entrada de um site de livraria permite que você pesquise os livros disponíveis para venda. Quando você pesquisa qualquer livro, o site interage com o banco de dados para buscar o registro daquele livro e exibi-lo no site.

Agora, sabemos que o site pede ao banco de dados para recuperar, armazenar ou modificar quaisquer dados. Então, como essa interação acontece? Os bancos de dados são gerenciados por Database Management Systems (DBMS), como MySQL, PostgreSQL, SQLite ou Microsoft SQL Server. Esses sistemas entendem a Structured Query Language ( SQL ). Então, qualquer aplicativo ou site usa consultas SQL ao interagir com o banco de dados.

Esta sala ensinará a você os conceitos básicos de injeção de SQL e como usar uma ferramenta automatizada para executar injeção de SQL . Ela também mergulhará na prática no tópico por meio de um desafio prático.

Objetivos de aprendizagem
* Vulnerabilidade de injeção de SQL
* Caça à injeção de SQL através da ferramenta SQLMap
* Pré-requisitos do quarto
Embora seja útil ter um conhecimento sólido dos Fundamentos de SQL , não é obrigatório para concluir esta sala.

![image](https://github.com/user-attachments/assets/5dfad4a0-ad39-413c-8773-393c865a7697)


## Tarefa 2
## Vulnerabilidade de injeção de SQL

Na tarefa anterior, estudamos como sites e aplicativos interagem com bancos de dados para armazenar, modificar e recuperar seus dados de forma estruturada. Nesta tarefa, veremos como a interação entre um aplicativo e um banco de dados acontece por meio de consultas SQL e como os invasores podem aproveitar essas consultas SQL para executar ataques de injeção de SQL .

Observação : antes de prosseguir, certifique-se de tentar os métodos de injeção de SQL manuais ou automatizados somente após a permissão do proprietário do aplicativo.

Vamos dar um exemplo de uma página de login que pede para você digitar seu nome de usuário e senha para efetuar login. Vamos fornecer os seguintes dados:

Username: John

Password: Un@detectable444

Depois de inserir seu nome de usuário e senha, o site os receberá, fará uma consulta SQL com suas credenciais e as enviará ao banco de dados. 

SELECT * FROM users WHERE username = 'John' AND password = 'Un@detectable444';
Esta consulta será executada no banco de dados. Conforme esta consulta, o banco de dados verificará um usuário chamado  Johne a senha de  Un@detectable444. Se encontrar tal usuário, ele retornará os detalhes do usuário para o aplicativo. Observe que a consulta acima será bem-sucedida somente se o usuário e a senha fornecidos tiverem uma correspondência no banco de dados, pois são separados pelo booleano “AND”.

Às vezes, quando a entrada é higienizada incorretamente, o que significa que a entrada do usuário não é validada, os invasores podem manipular a entrada e escrever consultas SQL que seriam executadas no banco de dados e realizariam as ações desejadas pelo invasor. A injeção de SQL tem um efeito muito prejudicial neste mundo digital, pois todas as organizações armazenam seus dados, incluindo suas informações críticas, dentro dos bancos de dados, e um ataque de injeção de SQL bem-sucedido pode comprometer seus dados críticos.

Vamos supor que a página de login do site que discutimos acima não tenha validação de entrada e sanitização. Isso significa que ela é vulnerável à injeção de SQL . O invasor não sabe a senha do usuário John. Ele digitará a seguinte entrada nos campos fornecidos:

Username: John

Password: abc' OR 1=1;-- -

Desta vez, o atacante digitou uma string aleatória abc e uma string injetada ' OR 1=1;-- -. A consulta SQL que o site enviaria ao banco de dados agora se tornará a seguinte:

`SELECT * FROM users WHERE username = 'John' AND password = 'abc' OR 1=1;-- -';`
Esta declaração parece semelhante à consulta SQL anterior , mas agora adiciona outra condição com o operador **OR**. Esta consulta verá se há um usuário, **John**. Então, ela verificará se **John** tem a senha abc(o que ele não poderia ter porque o invasor inseriu uma senha aleatória). Idealmente, a consulta deve falhar aqui porque espera que tanto o nome de usuário quanto a senha estejam corretos, pois há um **AND** operador entre eles. Mas, esta consulta tem outra condição, **OR**, entre a senha e uma declaração **1=1** . Qualquer uma delas sendo verdadeira fará com que toda a consulta SQL seja bem-sucedida. A senha falhou, então a consulta verificará a próxima condição, que verifica se **1=1** . Como sabemos, 1=1é sempre verdadeiro, então ela ignorará a senha aleatória inserida antes disso e considerará esta declaração como verdadeira, o que executará esta consulta com sucesso. O -- -no final da consulta comentaria qualquer coisa depois de **1=1** , o que significa que a consulta seria executada com sucesso, e o invasor seria conectado à conta de usuário de **John** .

Uma das coisas importantes a serem notadas aqui é o uso de uma aspa simples 'depois de abc. Sem essa aspa simples, **'a string inteira 'abc OR 1=1;-- -'** seria considerada a senha, o que não é pretendido. No entanto, se adicionarmos uma aspa simples 'depois de abc, a senha ficaria parecida com **'abc' OR 1=1;---'** , que envolve a string original abc na consulta e nos permite introduzir uma condição lógica **OR 1=1**b , que é sempre verdadeira.
![image](https://github.com/user-attachments/assets/18f04f21-9152-4851-813f-13ccac8b0303)
EXPLICAÇÃO: O operador "OR" é usado quando queremos verificar se pelo menos uma das condições é verdadeira pra que a declaração inteira seja considerada verdadeira. Diferente do "AND", que exige que todas as condições sejam verdadeiras ao mesmo tempo, o "OR" só precisa que uma delas seja verdadeira.

![image](https://github.com/user-attachments/assets/a08d5450-45b0-4f2c-8b5e-3da0c8547bb6)
