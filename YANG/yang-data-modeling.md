# Yang Data Modeling
Sumário 
1. [Parte 1](#parte-1---yang-schema)
2. [Parte 2](#parte-2---cabeçalho)
3. [Parte 3](#parte-3---descrevendo-dados-com-precisão)
4. [Parte 4](#parte-4)
5. [Parte 5](#parte-5)
6. [Parte 6 ](#parte-6---separando-dados-em-categorias.)
7. [Parte 7 ](#parte-7---descrevendo-possíveis-eventos)

# Exemplo
## Parte 1 - YANG Schema 
Uma Rede de livrarias fictícia decidiu definir uma interface própria e consistente para cada loja para que assim cada aplicação cliente possa navegar pelo inventário de delas é a aplicação de gerenciamento central possa adicionar novos títulos e autores nas lojas afiliadas
**Dados**:
- Nome da livraria : Book Zone
- informações: título, ISBN autor e preço de cada livro

| title                                                                                                        | isbn          | author            | price |
|--------------------------------------------------------------------------------------------------------------|---------------|-------------------|-------|
| The Neverending Story                                                                                     | 9780140386332 | Michael Ende      | 8.50  |
| What We Think About When We Try Not To Think About Global Warming: Toward a New Psychology of Climate Action | 9781603585835 | Per Espen Stoknes | 16.00 |
| The Hitchhiker’s Guide to the Galaxy                                                                         | 0330258648    | Douglas Adams     | 22.00 |
| The Art of War                                                                                               | 160459893X    | Sun Tzu           | 12.75 |
| I Am Malala: The Girl Who Stood Up for Education and Was Shot by the Taliban                                 | 9780297870913 | Malala Yousafzai  | 19.50 |

YANG Schema Representation of the Book Catalog
## Representação do catalogo de livros em um YANG schema
```yang 
container books {
    list book {
      key title;

      leaf title {
        type string;
      }
      leaf isbn {
        type string;
        mandatory true;
      }
      leaf author {
        type string;
      }
      leaf price {
        type decimal64 {
          fraction-digits 2;
        }
        units sim-dollar;
      }
    }
  }
```
Cada linha no YANG consiste em uma palavra chave (*keyword*)(`container, list, key, leaf, type, mandatory`) seguida por um nome (_books_, _book_, _title_, _isbn_, or _author_) ou valor com significado predefinido (`string, true`). Cada linha é seguida por um ponto e virgula (`;`) ou por um bloco cercado por chaves (`{...}`).

- `container` é usando uando você tem uma coleção de elementos de informação que pertencem um ao outro.
- `list` assim como um container, coisas relacionadas vão juntas, exceto você pode ter muitas instancias do que está dentro da lista, Dentro da lista _book_ haverão muitos _titles_. Uma YANG list pode ser comparada como uma tabela com colunas.
- ao colocar `leaf` dentro da lista quatro vezes, você terá uma tabela com quatro colunas. Cada leaf tem um `type` dentro que define o tipo de dado dessa leaf em particular. No [exemplo](#exemplo)  três _leafs_ são _strings_ e uma é um numero com duas casas decimais a leaf _isbn_ foi marcada com ` mandatory true `. Isso significa ue deve ter um valor, por padrão as leafs são opcionais, a não ser ue sejam marcadas com keys ou mandatory. No exemplo _isbn_ e _title_ são obrigatórias e _author_ e _price_ são opcionais.  
- `key` no _title_ significa que a coluna é a coluna chave da lista. Ela identifica  qual entry esta em qual lista 
[Exemplo de update utilizando lista]
Ao escolher _title_ para ser a key, você tem um identificador com significado pra utilizar. Keys com menos significados aumentam a chance de erros passarem sem serem detectados. Use chaves com significados, quando possível, e permita que o usuário utilize uma string qualquer como identificador, ao invés de números, por exemplo. q
Se houvesse uma forte exigência de suporte a vários livros com títulos idênticos, obviamente _title_ não funcionaria como uma chave. Nesse caso _isbn_ poderia ser uma opção melhor. Ao menos há um valor de identificação no mundo real
## Parte 2 - Cabeçalho
Antes do YANG no [exemplo](#exemplo) ser utilizado, é necessário um cabeçalho. O cabeçalho identifica exclusivamente o modulo e indica a data de revisão.
[Código no github](https://github.com/janlindblad/bookzone/blob/master/1-intro/bookzone-example.yang)
```yang
module bookzone-example {
  yang-version 1.1;
  namespace 'http://example.com/ns/bookzone';
  prefix bz;

  revision 2018-01-01 {
    description 'Initial revision. A catalog of books.';
  }

  container books {
    list book {
      key title;

      leaf title {
        type string;
      }
      leaf isbn {
        type string;
        mandatory true;
      }
      leaf author {
        type string;
      }
      leaf price {
        type decimal64 {
          fraction-digits 2;
        }
        units sim-dollar;
      }
    }
  }
}
```
Cada módulo YANG deve começar com a palavra chave `module`, seguida pelo nome do módulo. o nome do arquivo do módulo deve ter o mesmo nome, junto da extensão `.yang`.  A próxima linha deve declarar que o módulo está escrito em YANG 1.1 (ou alguma outra versão futura).
Cada módulo YANG deve ter um namespace name único no mundo. definido pelo `namespace`. Não devem existir dois módulos YANG com o mesmo namespace. É recomendado que o namespace seja a URL da organização e alguma que string que assegure que o namespace será único na organização.
namespaces começando com `url:` são utilizados pela IETF e devem ser registrados na [IANA](https://en.wikipedia.org/wiki/Internet_Assigned_Numbers_Authority).
Como namespaces precisam ser unicos(e longos), existe o _prefix_ que é essencialmente uma abreviação do nome do módulo. Em teoria esses prefixos não precisam ser únicos mas é recomendado ue sejam para evitar confusões. O `revision` não é obrigatório mas é uma boa prática adicionar um toda vez que uma nova versão  do módulo é "publicada". Um módulo YANG é considerado publicado quando é acessível à pessoas fora do time responsável pelo desenvolvimento.

## Parte 3 - Descrevendo dados com precisão
A parte 3 do exemplo adiciona algumas novas linhas ao YANG para gerenciamento de usuários cada usuário tem um _user-id_ único no sistema e um nome que ocasionalmente pode ser o mesmo para múltiplos usuários.
```yang
container users {
  list user {
    key user-id;

    leaf user-id {
      type string;
    }
    leaf name {
      type string;
    }
  }
}
```
## Parte 4
A parte quatro adiciona os autores. Nesse exemplo foi decidido que todos os autores serão referenciados pelo nome completo, e não usarão IDs ou números. Cada autor deve ter um número de conta para pagamentos. A palavra-chave `max` em `range` significa o maior número possível que este tipo pode representar.
```yang
container authors {
  list author {
    key name;

    leaf name {
      type string;
    }
    leaf account-id {
      type uint32 {
        range 1001..max;
      }
    }
  }
}
```
Os containers _users_ e  _authors_ vão fora do _book_. 
>"Tree representation" do módulo YANG:
```
module: bookzone-example
    +--rw authors
    |  +--rw author* [name]
    |     +--rw name          string
    |     +--rw account-id?   uint32
    +--rw books
    |  +--rw book* [title]
    |     +--rw title       string
    |     +--rw isbn        string
    |     +--rw author?     string
    |     +--rw price?      decimal64
    +--rw users
       +--rw user* [user-id]
          +--rw user-id    string
          +--rw name?      string
```

## Parte 5
`leafref` aponta para uma coluna em uma YANG list, e qualquer valor que existe na coluna é válido. 
```yang
leaf author {
  type leafref {
    path /authors/author/name;
  }
}
```
A expressão no `path` começa na root do YANG tree (/)  e vai pelo container _authors_ e list _author_. Finalmente aponta a leaf _name_ na list.
Qualquer leaf (string) em  /_authors_/_author_/_name_ é um valor válido para a  leafref /_books_/_book_/_author_, que anteriormente também era uma string.
Outra informação importante a ser adicionada em _books_ é qual lingua os livros são escritos, para que as aplicações clientes possam procurar e mostrar os livros mais relevantes aos usuários.
Nesse caso é so adicionar uma leaf chamada _language_ com uma string para a lingua. 
É importante ser específico sobre os os valores permitidos quando possível. Aqui é apropriado fazer uma enumeração das opções de idiomas.
É possível fazer isso dentro da leaf _language_ mas como escolha de idiomas pode ser relevante em outras partes do model, é interessante definir um tipo reutilizável para os idiomas. 
Em /_books_/_book_ list:
```yang
leaf language {
  type language-type;
}
```
E em algum lugar do YANG module é necessário definir o language-type
```yang
typedef language-type {
  type enumeration {
    enum arabic;
    enum chinese;
    enum english;
    enum french;
    enum moroccan-arabic;
    enum swahili;
    enum swedish;
    // List not exhaustive in order to save space
  }
  description
    'Primary language the book consumer needs to master '+
    'in order to appreciate the book's content';
}
```
É convenção em YANG é ter todos os identificadores em minúsculo (com traços se necessário). YANG é _case-sensitive_ então `enum ARABIC` e `enum arabic` são dois valores distintos .
De maneira similar, ISBN para os livros é uma string, mas como ISBNs tem um formato definido (2 na verdade ISBN 10 e ISBN 13), o YANG module deve suportar ambos. 

Em /_books_/_book_ list:
```yang
leaf isbn {
  type ISBN-10-or-13;
}
```

Por convenção tipos são definidos no topo do módulo.
```yang
typedef ISBN-10-or-13 {
  type union {
    type string {
      length 10;
      pattern '[0-9]{9}[0-9X]';
    }
    type string {
      length 13;
      pattern '97[89][0-9]{10}';
    }
  }
  description
    'The International Standard Book Number (ISBN) is a unique
     numeric commercial book identifier.

     An ISBN is assigned to each edition and variation (except
     reprintings) of a book. [source: wikipedia]';
  reference
    'https://en.wikipedia.org/wiki/International_Standard_Book_Number';
}
```
A  `union` YANG construct permite múltiplos tipos listados dentro dela.
A palavra-chave `reference` é utilizada para ajudar o leitor a encontrar mais informação sobre o tópico.
## Parte 6 - Separando dados em categorias.
Um aspecto a considerar é que não há um ISBN por título, mas sim um ISBN por título e formato. Se cada formato tem um ISBN diferente seja capa dura, EPUB, etc. Com o modelo atual, é necessário ter entries separadas para cada livro, dependendo do formato dele. Aos invés disso é possível fazer uma lista de formatos e ISBNs dentro da listra de livros.
	
   
    |  +--formats
    |     +--isbn  
    |     +--format 
    |     +--price
    |  +--title
    |  +--author
  
 List _format_ no container _books_:
```yang
  container books {
    list book {
      key title;

      leaf title {
        type string;
      }
	  ...
      list format {
        key isbn;
        leaf isbn {
          type ISBN-10-or-13;
        }
        leaf format-id {
		...
        }
      }
```
Sobre a leaf _format-id_, poderia ser uma string. Permite todos os formatos possíveis, atuais e futuros, Porém, também permite erros, erros de digitação ou entries ambíguas.  enumeração (`enumeration`) encaixa melhor aqui, o contra é que enumerações são difíceis de expandir com o tempo e não lidam bem com variantes.
Por exemplo, em _language_, Moroccan-Arabic é uma variante de Arabic, mas na enumeração os valores são distintos. Se existem expressões YANG que dependem se o idioma é Arabic ou não, teriam que ser testadas em ambos _arabic_ e _moroccan-arabic_. E, se outra variante for adicionada, a expressão pode necessitar de atualização para levar em consideração essa variante.

Este conceito de alguns valores de enumeração pertencentes é comum em redes e no mundo em geral. Pense em quantos tipos de interface existem e quantas variantes de Ethernet existem agora, de 10-Base-T a 1000GE. Em YANG, essas enumerações com relações de tipo são modeladas usando a palavra-chave `identity`. Se você estiver familiarizado com os princípios de design orientado a objetos, isso é conhecido como subclasse nesse contexto.
Abaixo estão listados os formatos escolhidos.
```yang
identity format-idty {
  description 'Root identity for all book formats';
}
identity paper {
  base format-idty;
  description 'Physical book printed on paper';
}
identity audio-cd {
  base format-idty;
  description 'Audiobook delivered as Compact Disc';
}
identity file-idty {
  base format-idty;
  description 'Book delivered as a file';
}
identity paperback {
  base paper;
  description 'Physical book with soft covers';
}
identity hardcover {
  base paper;
  description 'Physical book with hard covers';
}
identity mp3 {
  base file-idty;
  description 'Audiobook delivered as MP3 file';
}
identity pdf {
  base file-idty;
  description 'Digital book delivered as PDF file';
}
identity epub {
  base file-idty;
  description 'Digital book delivered as EPUB file';
}
```
Cada `identity ` tem uma `base`. Identity  _paperback_  tem uam base de _paper_, o que significa que é uma espécie de livro de papel. Identity _paper_ por sua vez, tem uma base de format-idty, que significa que _paper_ é uma espécie de formato de livro. O sufixo -_idty_ é só uma abreviação adicionada aqui para cada uma das root identities que deixam mais facil de lembrar que é uma  `identitiy`.
Agora que você tem um numero de formatos definidos, retorne a leaf _format-id_. Ao especificar o tipo `identitiyref` e uma root identity, essa leaf pode assumir o valor de qualquer identity que é diretamente ou indiretamente baseada nessa root identity. Caso identities adicionais forem definidas em outros módulos, elas imediatamente se tornam valores válidos para a leaf _format-id_.
```yang
leaf format-id {
  mandatory true;
  type identityref {
    base format-idty;
  }
}
```
Ao especificar o tipo identityref base  _format-idty_, todos os formatos de livros listados previamente se tornam valores validos para essa leaf, porque eles são baseados, diretamente ou indiretamente, em _format-idty_. Leaf _format-id_ foi definida como `mandatory` já que ISBN é um formato dado . Você não quer permitir que um livro seja listado com um ISBN sem dizer qual o formato do livro. 

#### Yang Module completo:
[Github](https://github.com/janlindblad/bookzone/blob/master/2-config/bookzone-example.yang)
```yang
  description 'Physical book printed on paper';
  }
  identity audio-cd {
    base format-idty;
    description 'Audiobook delivered as Compact Disc';
  }
  identity file-idty {
    base format-idty;
    description 'Book delivered as a file';
  }
  identity paperback {
    base paper;
    description 'Physical book with soft covers';
  }
  identity hardcover {
    base paper;
    description 'Physical book with hard covers';
  }
  identity mp3 {
    base file-idty;
    description 'Audiobook delivered as MP3 file';
  }
  identity pdf {
    base file-idty;
    description 'Digital book delivered as PDF file';
  }
  identity epub {
    base file-idty;
    description 'Digital book delivered as EPUB file';
  }
  typedef ISBN-10-or-13 {
    type union {
      type string {
        length 10;
        pattern '[0-9]{9}[0-9X]';
      }
      type string {
        length 13;
        pattern '97[89][0-9]{10}';
      }
    }
    description
      'The International Standard Book Number (ISBN) is a unique
       numeric commercial book identifier.

       An ISBN is assigned to each edition and variation (except
       reprintings) of a book. [source: wikipedia]';
    reference
      'https://en.wikipedia.org/wiki/International_Standard_Book_Number';
  }

  container authors {
    list author {
      key name;

      leaf name {
        type string;
      }
      leaf account-id {
        type uint32 {
          range 1001..max;
        }
      }
    }
  }

  container books {
    list book {
      key title;

      leaf title {
        type string;
      }
      leaf author {
        type leafref {
          path /authors/author/name;
        }
      }
      leaf language {
        type language-type;
      }
      list format {
        key isbn;
        leaf isbn {
          type ISBN-10-or-13;
        }
        leaf format-id {
          mandatory true;
          type identityref {
            base format-idty;
          }
        }
        leaf price {
          type decimal64 {
            fraction-digits 2;
          }
          units sim-dollar;
        }
      }
    }
  }

  container users {
    list user {
      key user-id;

      leaf user-id {
        type string;
      }
      leaf name {
        type string;
      }
    }
  }
}
```

## Parte 7 - Descrevendo possíveis eventos
Além das mudanças de configurações que flui do cliente para o servidor, eventos podem ocorrer em cada lado, e a outra parte pode precisar reagir a eles. Informações sobre esses eventos também precisam ser parte do YANG contract. Quando um cliente informa um server, é chamado `action` ou `rpc`. Os eventos que ocorrem em um servidor são enviados ao cliente como uma `notification`.
#### Actions e RPCs
O model que você tem pode servir bem como um catálogo de livro e para guardar informação sobre os usuários e autores, mas não permite uma aplicação cliente fazer uma compra.
Para isso, é necessário uma YANG`action` ou `rpc`. RPC significa _remote procedure call_ ([chamada de procedimento remoto](https://pt.wikipedia.org/wiki/Chamada_de_procedimento_remoto)). Um termo da ciência da computação para um client pedindo ao server para performar uma operação específica.
No YANG,  a unica diferença entre um _rpc_ e uma _action_é que um _rpc_ so pode ser declarado no _top level_ do modelo YANG (fora dos conteiners, lists, etc). Uma _action_, por outro lado, não pode ser usada no _top level_, mas somente dentro dos containers, lists e etc. Fora isso significam a mesma coisa
Use `action` quando a operação age em um node(object) específico na YANG tree e use `rpc` se a operação não pode ser associada a um node específico.
Quando definir uma `action` ou (rpc) em um modelo YANG, você pode especificar uma seção de parametros `input` e `output` dentro da `action` 
Se você escolheu fazer uma `rpc`, que deve ficar fora dos containers e lists, a seção input para a ação de _purchase_ deve conter uma leafref para o comprador (_user_), uma leafred para os _title_ , _format_ e _number-of-copies_.
rpc/_purchase_ com o input:
- _user_: "janl"
- _title_: "The Neverending Store"
- _format_: "bz:paperback"
- _number-of-copies_: 1

Caso tenha escolhido fazer uma `action`, você pode definir a action dentro da _list_ _user_. Dessa forma, o usuário que comprar o livro é  especificado no _path_ da action.

/_users_/_user[name="janl"]_/_purchase_:
-   _title_: “The Neverending Story”
    
-   _format_: “bz:paperback”
    
-   _number-of-copies_: 1

Não há grande diferenças entre as duas formas, você pode sempre transformar uma action em uma rpc adicionando uma leafrerf para o object que em que a action opera.
O parametro `output` lista as informações que vem da action.
O código abaixo mostra a _purchase_ action completa. A definição da action fica dentro da list _user_.  Também poderia ser um rpc (add leafref ao _user_) ou definida dentro do _book_ (remover a leafref ao _title_ e _format_ e adcionar uma leafred ao _user_)

```yang
action purchase {
  input {
    leaf title {
      type leafref {
        path /books/book/title;
      }
    }
    leaf format {
      type leafref {
        path /books/book/format/format-id;
        // Reviewer note: This should be improved
      }
    }
    leaf number-of-copies {
      type uint32 {
        range 1..999;
      }
    }
  }
  output {
    choice outcome {
      case success {
        leaf success {
          type empty;
          description
            'Order received and will be sent to specified user.
             File orders are downloadable at the URL given below.';
        }
        leaf delivery-url {
          type string;
          description
            'Download URL for file deliveries.';
        }
      }
      leaf out-of-stock {
        type empty;
        description
          'Order received, but cannot be delivered at this time.
           A notification will be sent when the item ships.';
      }
      leaf failure {
        type string;
        description
          'Order cancelled, for reason stated here.';
      }
    }
  }
}
```
o output foi modelado como uma `choice`.  uma choice lista o número de alternativas, as quais(no máximo) uma pode ser apresentada. Cada uma das alternativas podem ser listadas como um `case` dento da choice. A palavra-chave `case` pode ser pulado se o case contem um único yang elemento, como uma leaf,  container, ou list.
No exemplo a `choice` tem três diferentes cases que a action pode retornar:
1.  `leaf`  _success_  and/or  _delivery-url_.
    
2.  `leaf`  _out-of-stock_
    
3.  `leaf`  _failure_

Na verdade há um quarto case. Nesse modelo YANG, retornar nada também seria válido, pois a `choice` não foi marcada com `mandatory true`.
Choice e case nodes são _schema nodes_ do YANG, Isso significa que são invisíveis na _data tree_.  quando uma action retorna, ela simplesmente retorna os conteúdos de um dos _case statements_, e nunca faz menção à nada sobre `choice` _outcome_ ou `case` _success_. 

Em YANG, leafs do tipo `empty` não tem valor, mas podem existir ou não, Uma leaf do tipo `empty` pode ser criada e deletada, mas não atribuída. Leafs do tipo `empty` podem geralmente são utilizadas para indicar condições de sinalização como _success_ ou _enabled_.  _delivery-url_ e _failure_ são modeladas como strings, diferente de `empty`, pois retornam a URL  e uma razão para o erro, respectivamente.

#### Notificações
Na action _purchase_ existe a possibilidade da loja estar sem estoque no momento, Nesse cenário, o pedido ainda é registrado e será entregue posteriormente. Nesse caso, uma notificação é enviada ao comprador quando a entrega está prestes a acontecer. Esta notificação _shipping_ inclui uma referência ao _user_ comprador, o _title_ e o _format_ de entrega do livro, bem como o número de cópias.
Notic

`notification` são sempre definidas no _top level_, fora de ualuer container, list, etc, como os rpcs, são independentes de ualuer objeto YANG.
```yang
notification shipping {
  leaf user {
    type leafref {
      path /users/user/name;
    }
  }
  leaf title {
    type leafref {
      path /books/book/title;
    }
  }
  leaf format {
    type leafref {
      path /books/book/format/format-id;
      // Reviewer note: This should be improved
    }
  }
  leaf number-of-copies {
    type uint32;
  }
}
```

Antes de publicar um modelo, é interessante adicionar uma `revision` 
```yang
revision 2018-01-03 {
  description
    'Added action purchase and notification shipping.';
}
```
