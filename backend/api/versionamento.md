# Versionamento de APIs

> Um "não" é temporário, mas um "sim" é para sempre.

Você já sofreu, eu já sofri, todos nós sofremos. Esse é um problema comum em desenvolvimento de software, afinal os requisitos mudam com frequência e muitas vezes com uma velocidade maior do que o desejado. Portanto, o software precisa acompanhar estas mudanças e aí que começam os problemas.

## Definição de API

Apesar de muitas vezes, na era de aplicações web, a palava API seja comumente relacionada com uma REST API, esse termo é mais abrangente do que isso. Afinal API significa: `Application Programming Interface` e isso não se limita à comunicação pela Web, mas as APIs é tudo que um software expõe para que outros desenvolvedores façam uma integração com outro software. Portanto podemos dizer que utilizamos uma API quando:

- Chamamos uma função ou utilizamos uma classe built-in da linguagem;
- Chamamos uma função ou utilizamos uma classe de uma biblioteca instalada.
- Fazemos algum tipo de comunicação pela internet com algum serviço de terceiro.

Em todos esses casos, é possível que alterações sejam feitas pelos atores, e sempre que falamos de uma API, temos pelo menos dois atores:
* O criador da API, também conhecido como `mantenedor`, ou `maintainer`;
* O utilizador da API, que é um software feito por um desenvolvedor.

Abaixo estão algumas das definições de cada um dos atores:

### Mantenedor
Pensando como um mantenedor, é possível que seus usuários solicitem novas funcionalidades, ou exponham um bug em sua API, e o desafio é adicionar novas funcionalidades, ou corrigir bugs sem que afete o funcionamento dos demais utilizadores da sua API. A maneira que um mantenedor realiza esse "jogo de cintura" define em muito a experiência dos utilizadores da API.
No caso de um mantenedor que frequentemente faz alterações que "quebram" a integração dos utilizadores, é bem provável que seus utilizadores não se sintam felizes. Afinal isso irá introduzir bugs no software do utiizador. Mas um bom `mantenedor` evita ao máximo introduzir `breaking changes` ou mudanças críticas em suas APIs. Portanto é comum que em software se utilize [semantic versioning](https://semver.org/) para garantir que os usuários possam utilizar uma versão específica do software, e aos poucos possam ir alterando para as versões mais novas.
É claro que um bom mantenedor também irá garantir que seus usuários tenham um `Migration Guide` para fazer atualização para as novas versões que contenham breaking changes. Neste documento, um [exemplo](#um-exemplo-prático-da-importância-de-versionar) irá tentar ilustrar este tema. No caso de não ser possível manter a integrações dos utilizadores, um bom mantenedor irá definir uma [política de descontinuação](#política-de-descontinuação) (deprecation) da funcionalidade, ou de um parâmetro. Garantindo assim que os desenvolvedores tenham tempo para se atualizar para a nova funcionalidade.

### Utilizador

Como um utilizador da API, você quer garantir que sua integração se mantenha íntegra, ou seja, que com o passar do tempo não existirão mudanças que afetarão o funcionamento atual do seu software. É possível atingir isto por: Se prender (lock) a uma versão específica do software. Alguns casos: 
* A versão da sua linguagem de programação pode incluir mudanças da versão `a` para a versão `b`.
* A versão de uma biblioteca pode alterar uma função, classe ou outra estrutura da versão `a` para a versão `b`. Por isso que você quer garantir que suas dependências de bibliotecas de terceiros estejam "presas" a uma versão específica. Exemplo: `Django==4.2`
* Um fornecedor de um serviço de terceiro torna um parâmetro obrigatório  da versão `a` para a versão `b`.

Caso seja necessário integrar uma nova funcionalidade, ou se adequar a uma mudança o utilizador deverá ficar atento à quaisquer avisos de descontinuação de uma funcionalidade, ou de mudanças críticas nas APIs que utiliza.


## Um exemplo prático da importância de versionar

Nesse exemplo, você será o mantenedor, e dois personagens serão os utilizadores de uma API que você criou para eles: João e Marcos.
Portanto você é o mantenedor de uma série de funções de utilidade de uma biblioteca interna da sua empresa. E você notou que é comum para os desenvolvedores João e Marcos repetirem certo trecho de código: remover a formatação de um texto. Então você cria uma função que facilita e remove essa duplicidade. Digamos que a função que você criou faça o seguinte:

> Dada uma string, e uma lista de caracteres, retorne a string sem os caracteres da lista.

Em python essa função poderia se parecer com esta:
```python
from typing import Sequence

def remove_occurrences(text: str, chars: Sequence[str]) -> str:
  new = text
  for char in chars:
    new = new.replace(char, "")
  return new
```
Esmiuçando:
Foi criada uma função chamada `remove_occurrences` que recebe dois argumentos/parâmetros:
* `text`: A string que será usada como base para que os caracteres sejam removidos;
* `chars`: Uma sequência de strings, no python isso significa que é qualquer objeto que seja possível iterar (`for x in obj`) sobre, e onde seus items sejam strings.
Essa função então utiliza da função built-in do python: [replace](https://docs.python.org/3/library/stdtypes.html#str.replace) para remover as ocorrências de cada um dos caracteres.

Agora os desenvolvedores podem utilizar sua função para não mais duplicar essa funcionalidade.
Eles utilizariam sua função da seguinte maneira:
```python
from company.utils import remove_occurrences

formatted_doc = "111.222.333-44"
doc = remove_occurrences(formatted_doc, ".-")
# Outras maneiras de escrever esse código seriam, passando uma tuple ou list
# doc = remove_occurrences(formatted_doc, [".", "-"])
# doc = remove_occurrences(formatted_doc, (".", "-"))
print(doc)
# > 11122233344
```
Perfeito, o caso de uso parece bem definido. Então você informa seus colegas: João e Marcos que agora eles podem utilizar essa função para fazer remover a formatação de um documento. João e Marcos imediatamente começam a utilizar essa função para remover a formatação de documentos em outros softwares.
**Parabéns, você acabou de criar uma API !** Mas com isso vem uma série de responsabilidades, como garantir que essa API está funcionando corretamente (esse tópico não será abordado) com testes, e garantindo que João e Marcos possam continuar utilizando sua API para fazer suas rotinas.

Mas é claro que depois de 1 hora, Marcos chega até você para informar o seguinte:
*Em alguns documentos eu só posso remover a primeira ocorrência de cada um dos caracteres*. Portanto no caso do documento: 111.222.333-44 ao chamar a função eu gostaria que ao chamar função assim, eu tenha o resultado "111222.33344", como nesse exemplo abaixo:
```python
formatted_doc = "111.222.333-44"
doc = remove_occurrences(formatted_doc, ".-")
assert doc == "111222.33344
```

Você balança a cabeça e não acredita que de fato exista esse caso de uso, mas como mantenedor da API agora você tem a responsabilidade com o utilizador. Vocês criaram um "contrato" e um vínculo um com o outro. Mas como mantenedor, nesse caso cabe a você decidir se a alteração será inclusa ou não na sua API. Em outros casos, é possível que a decisão de realizar a alteração não seja sua, e você seja obrigado a incluir a alteração. Portanto, nesse caso a alteração deverá ser incluída na sua API.

#### Objetivo de versionar
Nesse momento você para e lembra que João não pediu por essa alteração, então você precisa garantir que sua API ainda funcione para João. Portanto o que você não pode fazer é adicionar um parâmetro obrigatório nessa função, afinal João não solicitou essa alteração, e é provável que nem saiba que esta alteração irá ocorrer. Então você pensa em algumas possibilidades que garantam as duas coisas:

* Que Marcos possa ter o comportamento que ele deseja;
* Que João possa continuar utilizando a sua API como já tem feito até então.

Nesse caso existem algumas coisas que podem ser feitas:
* É possível **criar uma nova função** com outro nome e que contenha um **parâmetro adicional obrigatório**;
* É possível **alterar a função já existente** e adicionar um **parâmetro adicional opcional**.

Em cada caso talvez existam novos casos a se pensar e considerar, bem como soluções que estejam disponíveis. A decisão fica a cargo do desenvolvedor mantenedor. Neste exemplo vamos ver os aspectos técnicos de cada uma das decisões e das consequências de cada uma delas.

##### Criar uma nova função com um parâmetro obrigatório:

Talvez o mantenedor pense que criar uma nova função com um parâmetro obrigatório seja o caminho mais fácil para que ambos os [objetivos](#objetivo-de-versionar) sejam atingidos. Portanto o mantenedor simplesmente copia a função `remove_occurrences`, lhe dá um novo nome e adiciona um parâmetro obrigatório. Veja uma possível implementação:

```python
from typing import Sequence

def remove_occurrences(text: str, chars: Sequence[str]) -> str:
  new = text
  for char in chars:
    new = new.replace(char, "")
  return new

## Nova função definida abaixo
def remove_occurrences_limited(text: str, chars: Sequence[str], limit: int) -> str:
  new = text
  for char in chars:
    new = new.replace(char, "", limit)
    # Que bom que a função replace aceita um limitador!
  return new
```

Perfeito! João e Marcos agora podem usar as funções de acordo com sua necessidade. Mas o código das duas funções são realmente muito similares, alterando apenas uma linha de uma função para a outra:
```diff
-  new = new.replace(char, "")
+  new = new.replace(char, "", limit)
```
Agora será necessário que toda a implementação seja testada novamente para a nova função. Nesse caso parece que a solução ficou "apressada" ou "grosseira".

##### Atualizar a nova função com um parâmetro opcional:

Talvez o mantenedor tenha decidido alterar a função para receber um novo parâmetro opcional. Nesse caso, João poderá continuar chamando a função como já fazia e Marcos poderá chamar a função com o novo parâmetro. Veja um exemplo de como esse parâmetro poderia ser adicionado à função:

```diff
from typing import Sequence

- def remove_occurrences(text: str, chars: Sequence[str]) -> str:
+ def remove_occurrences(text: str, chars: Sequence[str], limit: int = -1) -> str:
  new = text
  for char in chars:
-   new = new.replace(char, "")
+   new = new.replace(char, "", limit)
  return new
```

Ótimo, agora só é necessário testar a função com o novo parâmetro e tanto João e Marcos ainda estarão integrados com nossa API.

##### Pós-Decisão

Após a decisão de implementação ter sido tomada, você acabou de criar uma nova versão da sua API, nessa versão nenhuma alteração crítica (breaking change) foi adicionada, portanto não será necessário realizar nenhuma ação adicional além de avisar Marcos que agora ele poderá utilizar esse novo parâmetro.
Mas é claro, nem sempre as coisas são tão simples!
2 semanas depois seu líder técnico informa que a função que você escreveu será repassada para um outro algoritmo de uma outra fonte, e que todos os utilizadores da função atual deverão passar a utilizar o novo algoritmo, só que neste novo algoritmo não será possível utilizar um limitador da quantidade de caracteres, porém devido à criticidade do negócio e de uma parceria com outra empresa a decisão final é essa. Portanto, você começa agora precisa informar João e Marcos sobre essa alteração na sua API, afinal essa alteração será uma mudança crítica (breaking change). Nesse caso Marcos precisará dar um outro jeito de ter a funcionalidade que utilizava em sua API.

Portanto, pensando nos seus utilizadores, você informa seu gestor que será necessário um período de 15 dias para que todos os utilizadores migrem para a nova versão, sendo:
1 dia para implementação do novo algoritmo, lançamento da versão;
1 dia para coletar os usuários que utilizam a funcionalidade e avisá-los que a funcionalidade será alterada na nova versão, e o que será alterado;
13 dias para que os utilizadores tenham tempo de realizar as mudanças necessárias.

**Parabéns, você acaba de definir uma política de descontinuação!**

> É claro que esse prazo é variável de acordo com cada necessidade e cada caso. Cabe ao mantenedor definir o prazo.

Portanto você implementa a função, talvez se pareça algo como:
```diff
from typing import Sequence
+ from partner import utils

def remove_occurrences(text: str, chars: Sequence[str], limit: int = -1) -> str:
- new = text
- for char in chars:
-   new = new.replace(char, "", limit)
- return new
+ if limit > 0:
+   print(
+     "Warning: the limit parameter is",
+     "deprecated and will be removed on the next version"
+   )
+   new = text
+   for char in chars:
+     new = new.replace(char, "", limit)
+   return new
+ return utils.remove_occurrences_faster(text, chars)
```


No caso acima, o parâmetro `limit` não foi removido. Nesse caso, essa seria uma versão que não quebraria a integração com Marcos, já que tudo estaria funcionando normalmente para ele. Porém um aviso é emitido para o utilizador informando que esse parâmetro utilizado está descontinuado e será removido em breve. Isso dará para Marcos tempo para que ele faça as alterações em seu software para atender as exigências do mantenedor.

Portanto, Marcos estaria ciente de que deve alterar em breve seu software para poder continuar integrado. Já João não precisa realizar nada para que seu software continue integrado. Numa próxima versão, assim que o prazo definido na política de descontinuação for atingido, o mantenedor pode então fazer uma mudança crítica (breaking change) na API. Uma possível implementação da nova funcionalidade seria:
```python
from typing import Sequence
from partner import utils

def remove_occurrences(text: str, chars: Sequence[str]) -> str:
  return utils.remove_occurrences_faster(text, chars)
```

Nessa situação o mantenedor cria uma nova versão da API sem o parâmetro descontinuado. Todos os utilizadores que quiserem utilizar a nova versão da API deverão seguir o novo contrato.
Portanto o mantenedor garantiu que todos os utilizadores da API conseguiram manter a integração com sua API sem nenhuma alteração crítica que afetasse seus softwares.

##### Considerações finais do exemplo

Neste exemplo vimos que:
* Softwares passam por mudanças frequentemente, sejam por decisões de negócio ou por solicitações de novas funcionalidades, ou problemas encontrados.
* Quando mudanças precisam ocorrer, deve se dar preferência por não incluir mudanças críticas (se for possível) para que utilizadores continuem integrados à API do mantenedor.
* Quando mudanças críticas precisam ocorrer, o mantenedor deve informar os utilizadores da mudança que irá acontecer e como se ajustar a elas, criando uma política de descontinuação.
* É responsabilidade do utilizador se manter atualizado com a versão mais recente da API e estar atento à informes de descontinuação ou mudanças críticas.

No exemplo foi citado um caso onde um parâmetro foi adicionado, e posteriormente removido. Na vida real haverão casos onde um parâmetro será adicionado e será obrigatório, uma informação que era obrigatória não será mais coletada, e diversos outros cenários são possíveis. O mais importante é que o mantenedor saiba da importância de manter seus utilizadores integrados o maior tempo possível e que tenham tempo para se adaptar as mudanças.


## Política de descontinuação

Uma política de descontinuação é um termo que utiliza-se para informar que um mudança acontecerá em breve. Em cada caso caberá aos desenvolvedores envolvidos definir a política de acordo e seus termos de acordo com cada situação e urgência.
Mas é vital que quando mudanças críticas acontecerem, esteja definido na política:
* O que foi alterado;
* O que deve ser feito para se adequar às mudanças;
* Quanto tempo há para se adequar às mudanças;

E é claro que é vital que os utilizadores recebam essa informação. Essa informação pode ser feita por meio de um informe por email.
Um exemplo prático abaixo:
Em uma aplicação web, um novo parâmetro será adicionado e será obrigatório em um endpoint da REST API da aplicação. Foi definido que um novo endpoint será criado para que o parâmetro seja recebido. Uma política de descontinuação foi definida e então o código é enviado, o endpoint agora possui duas versões: a versão 1 que não possui o parâmetro, e a versão 2 onde o parâmetro é obrigatório.
Verifica-se nos logs do servidor quais clientes estão usando o endpoint na versão 1, após isso, identifica-se os endereços de email dos utilizadores (desenvolvedores) responsáveis. Um e-mail é enviado para estes emails informando sobre a mudança, e a política definida.

## Considerações finais

O versionamento de software é muito importante, e para uma melhor experiência de todos, é necessário que haja responsabilidade pelos mantenedores de API. Na [Plathanus](https://plathanus.com.br) a forma mais comum de ser um mantenedor de uma API é por ser o responsável por uma REST API que possui utilizadores, sendo os utilizadores as aplicações mobile ou web feitas pelos desenvolvedores da própria empresa ou de outras instituições. Portanto garantir que todos os utilizadores continuarão a ter a integração funcionando até que todos atualizem para a última versão é de suma importância.
Caso esse processo não seja seguido, é comum acontecerem os seguintes necessários:
* Um usuário utiliza uma versão desatualizada de um aplicativo, e uma mudança na REST API faz com que seu aplicativo não funcione mais.
* Um usuário está utilizando uma ferramenta de outra instituição que se comunica com a REST API que teve alterações e a ferramenta de outra instituição para de funcionar.

Nesses casos gera-se atrito para os:
* Clientes (Usuários): Seus aplicativos que até ontem funcionavam param de funcionar inesperadamente, fazendo com que muitas vezes seja difícil de identificar a causa do problema.
* Utilizadores (Desenvolvedores internos/externos): Tenham que fazer alterações as pressas para que a integração seja restabelecida.

Por outro lado, quando se seguem a política de descontinuação e se dá valor ao versionamento das APIs os desenvolvedores tem tempo para:
* Informar uma versão mínima nas lojas de aplicativos que contenham a nova alteração;
* Identificar facilmente clientes que estejam utilizando uma versão antiga do aplicativo.

Cada projeto contém seus diversos desafios, e a necessidade de se definir regras simples ou complexas depende de diversos fatores. Portanto, caso não saiba quais regras aplicar, simplesmente comece: Uma regra é melhor do que nenhuma, e duas regras são melhores que uma.

## Considerações REST API

Diante do cenário em que as mudanças são inevitáveis, o desenvolvedor deve garantir o versionamento desde a primeira versão, sendo esta a versão: `1`.
Diante do cenário de REST API a comunicação ser feita via HTTP existem algumas maneiras diferentes de se versionar uma API, são elas:

* Query Parameter: É possível definir a versão do endpoint através de um paramêtro de query na URL. Por exemplo: `https://example.com/api/products?version=1`.
* Cabeçalho/Header HTTP: É possível definir a versão do endpoint através de um cabeçalho HTTP. Normalmente isso é feito através do `Accept` header. Tenha em mente que este cabeçalho também é utilizado para realizar **Content Negotiation**, em alguns frameworks o valor definido neste cabeçalho pode definir o formato de saída. Exemplo: `Accept: application/json; version=1` O formato do valor no cabeçalho pode alterar de framework para framework.
* Caminho URL: É possível definir a versão do endpoint através do caminho da URL. Por exemplo: `https://example.com/api/v1/products`.

Na [Plathanus](https://plathanus.com.br) geralmente damos preferência a última maneira. Isso por que dentre todos este é o mais comum e mais explícito entre as outras maneiras.

Considere criar uma nova versão de um endpoint sempre que:

**Um campo de entrada/saída tem seu nome alterado**: Neste caso é possível que os utilizadores dependam do nome do campo previamente definido. Exemplo, um campo chamado `name` é atualizado para `full_name`:
```diff
{
-  "name": "John doe"
+  "full_name": "John doe"
}
```

**Um campo de entrada/saída é removido**: Neste caso é possível que o campo tenha um efeito colateral que utilizadores esperavam (no caso de entrada) e continuem a enviar esse campo (que geralmente é ignorado). Ou que esperava-se que um certo campo estivesse presente no retorno. Exemplo, um campo chamado `name` é atualizado e agora encontra-se dentro de um outro objeto:
```diff
{
  "id": 1,
- "name": "John doe"
+ "profile": {
+   "name": "John doe"
+ }
}
```

**Um campo de entrada obrigatório é adicionado**: Neste caso os utilizadores não estarão enviando um campo, causando uma resposta provavelmente inesperada, ou que não se tenha como resolver até que o utilizador realize uma atualização no seu aplicativo. Exemplo, um campo chamado `birth_date` é obrigatório:
```diff
{
- "name": "John doe"
+ "name": "John doe",
+ "birth_date": "2023-11-20"
}
```

Nestes casos seria prudente adicionar uma nova versão do endpoint. Caso a versão anterior do endpoint não seja mais suficiente para atender aos requisitos, inicie o processo de Descontinuação do endpoint. Caso contrário mantenha as duas versões até que os dados recebidos no endpoint sejam suficientes para atender aos requisitos da aplicação, ou até que se tenha certeza que não existe nenhum cliente utilizando a versão anterior. Esse seria o caso onde os utilizadores atualizaram seus softwares para utilizar a versão mais recente, e todos os "clientes": aplicativos, web já estão na última versão.

## Referências

* [Versionamento Semântico](https://semver.org/) em inglês;
* [Como versionar uma REST API](https://www.freecodecamp.org/news/how-to-version-a-rest-api/) em inglês;
* [Política descontinução REST API IBM](https://www.ibm.com/docs/en/security-verify?topic=apis-api-compatibility-policy-deprecation-policies) em inglês;
* [Descontinuação API](https://document360.com/blog/api-deprecation/) em inglês;

Abaixo estão alguns exemplos do projeto Open-Source Django, um projeto que já tem cerca de 20 anos de open-source, e como lidam com mudanças de API.
* [Estabilidade APIs Django](https://docs.djangoproject.com/en/4.2/misc/api-stability/) em inglês;
* [Linha do tempo Descontinuação APIs Django](https://docs.djangoproject.com/en/4.2/internals/deprecation/) em inglês;
