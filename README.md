# Clean Code em Dart
Este repositório é uma adaptação do projeto [felipe-augusto/clean-code-javascript](https://github.com/felipe-augusto/clean-code-javascript) para a linguagem Dart.

Repositório original em inglês: [ryanmcdermott/clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript)

**This repo in English**: [williambarreiro/clean-code-dart](https://github.com/williambarreiro/clean-code-dart)

## Índice
  1. [Introdução](#introdução)
  2. [Variáveis](#variáveis)
  3. [Funções](#funções)
  4. [Objetos e Estruturas de Dados](#objetos-e-estruturas-de-dados)
  5. [Classes](#classes)
  6. [SOLID](#solid)
  7. [Testes](#testes)
  8. [Concorrência](#concorrência)
  9. [Tratamento de Erros](#tratamento-de-erros)
  10. [Formatação](#formatação)
  11. [Comentários](#comentários)

## Introdução
![Imagem humorística da estimativa de qualidade do software baseado na contagem de quantos palavrões você gritou enquanto lia o código.](http://www.osnews.com/images/comics/wtfm.jpg)

Princípios da Engenharia de Software, do livro de Robert C. Martin
[*Código Limpo*](https://www.amazon.com.br/C%C3%B3digo-Limpo-Habilidades-Pr%C3%A1ticas-Software/dp/8576082675),
adaptados para Dart. Isto não é um guia de estilos. É um guia para se produzir código [legível, reutilizável e refatorável](https://github.com/ryanmcdermott/3rs-of-software-architecture).

Nem todo princípio demonstrado deve ser seguido rigorosamente, e ainda menos são um consenso universal. Estes princípios são orientações e nada mais, entretanto, foram codificados durante muitos anos de experiência coletiva pelos autores de *Código limpo*.

Nosso ofício de engenharia de software tem pouco mais de 50 anos e ainda estamos aprendendo muito. Quando a arquitetura de software for tão velha quanto a própria arquitetura, talvez então tenhamos regras mais rígidas para seguir. Por enquanto, deixe que estas orientações sirvam como critério para se avaliar a qualidade de código Dart que tanto você e o seu time produzirem.

Mais uma coisa: aprender isto não irá lhe transformar imediatamente em um desenvolvedor de software melhor, trabalhar com eles por muitos anos não quer dizer que você não cometerá erros. Toda porção de código começa com um rascunho, como argila molhada sendo moldada em sua forma final. Finalmente, talhamos as imperfeições quando revisamos com nossos colegas. Não se sinta culpado pelos primeiros rascunhos que ainda precisam de melhorias. Ao invés, desconte em seu código.

## **Variáveis**
### Use nomes de variáveis que tenham significado e sejam pronunciáveis

**Ruim:**
```dart
final yyyymmdstr = DateFormat('yyyy/MM/dd').format(DateTime.now());
```

**Bom:**
```dart
final currentDate = DateFormat('yyyy/MM/dd').format(DateTime.now());
```
**[⬆ voltar ao topo](#Índice)**

### Use o mesmo vocabulário para o mesmo tipo de variável

**Ruim:**
```dart
getUserInfo();
getClientData();
getCustomerRecord();
```

**Bom:**
```dart
getUser();
```
**[⬆ voltar ao topo](#Índice)**

### Use nomes pesquisáveis
Nós iremos ler mais código que escrever. É importante que o código que escrevemos seja legível e pesquisável. *Não* dando nomes em variáveis que sejam significativos para entender nosso programa, machucamos nossos leitores. Torne seus nomes pesquisáveis.

**Ruim:**
```dart
// Para que diabos serve 32?
Future.delayed(Duration(minutes: 32), launch);
```

**Bom:**
```dart
// Declare-as como const se o valor for conhecido em tempo de compilação;
// Declare-as como final se o valor for atribuído apenas uma vez;
// Use lowerCamelCase;
// OBS: setupTimeInMinutes será int, pois o tipo será inferido.
const setupTimeInMinutes = 32;

Future.delayed(Duration(minutes: setupTimeInMinutes), launch);
```
**[⬆ voltar ao topo](#Índice)**

### Use variáveis explicativas
**Ruim:**
```dart
const address = <String>['One Infinite Loop', 'Cupertino', '95014'];
saveCityZipCode(address[1], address[2]);
```

**Bom:**
```dart
const address = <String>['One Infinite Loop', 'Cupertino', '95014'];
final city = address[1];
final zipCode = address[2];
saveCityZipCode(city, zipCode);
```
**[⬆ voltar ao topo](#Índice)**

### Evite Mapeamento Mental
Explicito é melhor que implícito.

**Ruim:**
```dart
const locations = <String>['Austin', 'New York', 'San Francisco'];
locations.forEach((l) {
  doStuff();
  doSomeOtherStuff();
  // ...
  // ...
  // ...
  // Espera, para que serve o `l` mesmo?
  dispatch(l);
});
```

**Bom:**
```dart
const locations = <String>['Austin', 'New York', 'San Francisco'];
locations.forEach((location) {
  doStuff();
  doSomeOtherStuff();
  // ...
  // ...
  // ...
  dispatch(location);
});
```
**[⬆ voltar ao topo](#Índice)**

### Não adicione contextos desnecessários
Se o nome de sua classe/objeto já lhe diz alguma coisa, não as repita nos nomes de suas variáveis.

**Ruim:**
```dart
final car = Car(
  carMake: 'Honda',
  carModel: 'Accord',
  carColor: 'Blue',
);

void paintCar(Car car, String color) {
  car.carColor = color;
}
```

**Bom:**
```dart
final car = Car(
  make: 'Honda',
  model: 'Accord',
  color: 'Blue',
);

void paintCar(Car car, String color) {
  car.color = color;
}
```
**[⬆ voltar ao topo](#Índice)**

### Quando possível, use argumentos padrões ao invés de curto circuitar ou usar condicionais

Argumentos padrões são geralmente mais limpos do que curtos circuitos. Porém esses argumentos devem ser const, não sendo possível usá-los em todos os casos.

**Ruim:**
```dart
void createMicrobrewery({String? name}) {
  final breweryName = name ?? 'Hipster Brew Co.';
  // ...
}
```

**Bom:**
```dart
void createMicrobrewery({String breweryName = 'Hipster Brew Co.'}) {
  // ...
}
```
**[⬆ voltar ao topo](#Índice)**

## **Funções**
### Argumentos de funções (idealmente 2 ou menos)
Limitar a quantidade de parâmetros de uma função é incrivelmente importante porque torna mais fácil testá-la. Ter mais que três leva a uma explosão combinatória onde você tem que testar muitos casos diferentes com cada argumento separadamente.

Um ou dois argumentos é o caso ideal, e três devem ser evitados se possível. Qualquer coisa a mais que isso deve ser consolidada. Geralmente, se você tem mais que dois argumentos então sua função está tentando fazer muitas coisas. Nos casos em que não está, na maioria das vezes um objeto é suficiente como argumento.

Para tornar mais óbvio quais as propriedades que as funções esperam, você pode usar parâmetros nomeados. Eles possuem algumas vantagens:

1. Quando alguém olha para a assinatura de uma função, fica imediatamente claro quais propriedades são usadas.
2. Linters podem te alertar sobre propriedades não utilizadas se elas forem `required`.

**Ruim:**
```dart
Menu getMenu(String title, String body, String buttonText, bool cancellable) {
  // ...
}
```

**Bom:**
```dart
Menu getMenu({
  required String title,
  required String body,
  required String buttonText,
  required bool cancellable,
}) {
  // ...
}

final menu = getMenu(
  title: 'Foo',
  body: 'Bar',
  buttonText: 'Baz',
  cancellable: true,
);
```
**[⬆ voltar ao topo](#Índice)**


### Funções devem fazer uma coisa
Essa é de longe a regra mais importante em engenharia de software. Quando funções fazem mais que uma coisa, elas se tornam difíceis de serem compostas, testadas e raciocinadas. Quando você pode isolar uma função para realizar apenas uma ação, elas podem ser refatoradas facilmente e seu código ficará muito mais limpo. Se você não levar mais nada desse guia além disso, você já estará na frente de muitos desenvolvedores.

**Ruim:**
```dart
void emailClients(List<Client> clients) {
  for(final client in clients) {
    final clientRecord = database.lookup(client);
    if (clientRecord.isActive()) {
      email(client);
    }
  }
}
```

**Bom:**
```dart
void emailActiveClients(List<Client> clients) {
  clients
    .where(isActiveClient)
    .forEach(email);
}

bool isActiveClient(Client client) {
  final clientRecord = database.lookup(client);
  return clientRecord.isActive();
}
```
**[⬆ voltar ao topo](#Índice)**

### Nomes de funções devem dizer o que elas fazem

**Ruim:**
```dart
void addToDate(DateTime date, int months) {
  // ...
}

final currentDate = DateTime.now();

// É difícil dizer pelo nome da função o que é adicionado
addToDate(currentDate, 1);
```

**Bom:**
```dart
void addMonthsToDate(int months, DateTime date) {
  // ...
}

final currentDate = DateTime.now();
addMonthsToDate(1, currentDate);
```
**[⬆ voltar ao topo](#Índice)**

### Funções devem ter apenas um nível de abstração
Quando você tem mais de um nível de abstração sua função provavelmente esta fazendo coisas demais. Dividir suas funções leva a reutilização e testes mais fáceis.

**Ruim:**
```dart
void parseBetterAlternative(String code) {
  const regexes = [
    // ...
  ];

  final statements = code.split(' ');
  final tokens = [];
  for (final regex in regexes) {
    for (final statement in statements) {
      tokens.add( /* ... */ );
    }
  }

  final ast = <Node>[];
  for (final token in tokens) {
    ast.add( /* ... */ );
  }

  for (final node in ast) {
    // parse...
  }
}
```

**Bom:**
```dart
List<String> tokenize(String code) {
  const regexes = [
    // ...
  ];

  final statements = code.split(' ');
  final tokens = <String>[];
  for (final regex in regexes) {
    for (final statement in statements) {
      tokens.add( /* ... */ );
    }
  }

  return tokens;
}

List<Node> lexer(List<String> tokens) {
  final ast = <Node>[];
  for (final token in tokens) {
    ast.add( /* ... */ );
  }
  
  return ast;
}

void parseBetterAlternative(String code) {
  final tokens = tokenize(code);
  final ast = lexer(tokens);
  for (final node in ast) {
    // parse...
  }
}
```
**[⬆ voltar ao topo](#Índice)**

### Remova código duplicado
Faça absolutamente seu melhor para evitar código duplicado. Código duplicado quer dizer que existe mais de um lugar onde você deverá alterar algo se precisar mudar alguma lógica.

Imagine que você é dono de um restaurante e você toma conta do seu estoque: todos os seus tomates, cebolas, alhos, temperos, etc. Se você tem multiplas listas onde guarda estas informações, então você terá que atualizar todas elas quando servir um prato que tenha tomates. Se você tivesse apenas uma lista, teria apenas um lugar para atualizar!

Frequentemente, você possui código duplicado porque você tem duas ou mais
coisas levemente diferentes, que possuem muito em comum, mas suas diferenças lhe forçam a ter mais duas ou três funções que fazem muito das mesmas coisas. Remover código duplicado significa criar uma abstração que seja capaz de lidar com este conjunto de coisas diferentes com apenas uma função/módulo/classe.

Conseguir a abstração correta é crítico, por isso que você deveria seguir os princípios SOLID descritos na seção *Classes*. Abstrações ruins podem ser piores do que código duplicado, então tome cuidado! Dito isto, se você puder fazer uma boa abstração, faça-a! Não repita a si mesmo, caso contrário você se pegará atualizando muitos lugares toda vez que precisar mudar qualquer coisinha.

**Ruim:**
```dart
Widget buildDeveloperCard(Developer developer) {
  return CustomCard(
    expectedSalary: developer.calculateExpectedSalary(),
    experience: developer.getExperience(),
    projectsLink: developer.getGithubLink(),
  );
}

Widget buildManagerCard(Manager manager) {
  return CustomCard(
    expectedSalary: manager.calculateExpectedSalary(),
    experience: manager.getExperience(),
    projectsLink: manager.getMBAProjects(),
  );
}
```

**Bom:**
```dart
Widget buildEmployeeCard(Employee employee) {
  String projectsLink;

  if (employee is Manager) {
    projectsLink = manager.getMBAProjects();
  } else if (employee is Developer) {
    projectsLink = developer.getGithubLink();
  }

  return CustomCard(
    expectedSalary: employee.calculateExpectedSalary(),
    experience: employee.getExperience(),
    projectsLink: projectsLink,
  );
}
```
**[⬆ voltar ao topo](#Índice)**

### Não use flags como parâmetros de funções
Flags falam para o seu usuário que sua função faz mais de uma coisa. Funções devem fazer apenas uma coisa. Divida suas funções se elas estão seguindo caminhos de código diferentes baseadas em um valor booleano.

**Ruim:**
```dart
void createFile(String name, bool temp) {
  if (temp) {
    File('./temp/${name}').create();
  } else {
    File(name).create();
  }
}
```

**Bom:**
```dart
void createFile(String name) {
  File(name).create();
}

void createTempFile(String name) {
  File('./temp/${name}').create();
}
```
**[⬆ voltar ao topo](#Índice)**

### Evite Efeitos Colaterais (parte 1)
Uma função produz um efeito colateral se ela faz alguma coisa que não seja receber um valor de entrada e retornar outro(s) valor(es). Um efeito colateral pode ser escrever em um arquivo, modificar uma variável global, ou acidentalmente transferir todo seu dinheiro para um estranho.

Agora, você precisa de efeitos colaterais ocasionalmente no seu programa. Como no exemplo anterior, você pode precisar escrever em um arquivo. O que você quer fazer é centralizar aonde está fazendo isto. Não tenha diversas funções e classes que escrevam para um arquivo em particular. Tenha um serviço que faça isso. Um e apenas um.

O ponto principal é evitar armadilhas como compartilhar o estado entre objetos sem nenhuma estrutura, usando tipos de dados mutáveis que podem ser escritos por qualquer coisa, e não centralizando onde seu efeito colateral acontece. Se você conseguir fazer isto, você será muito mais feliz que a grande maioria dos outros programadores.

**Ruim:**
```dart
// Variável global referenciada pela função seguinte
// Se tivéssemos outra função que usa esse nome, então seria um vetor (array) e poderia quebrar seu código
dynamic name = 'Ryan McDermott';

void splitIntoFirstAndLastName() {
  name = name.split(' ');
}

splitIntoFirstAndLastName();

print(name); // ['Ryan', 'McDermott'];
```

**Bom:**
```dart
List<String> splitIntoFirstAndLastName(name) {
  return name.split(' ');
}

final name = 'Ryan McDermott';
final newName = splitIntoFirstAndLastName(name);

print(name); // 'Ryan McDermott';
print(newName); // ['Ryan', 'McDermott'];
```
**[⬆ voltar ao topo](#Índice)**

### Evite Efeitos Colaterais (parte 2)
Em Dart, tipos primitivos são passados por valor e objetos/vetores são passados por referência. No caso de objetos e vetores, se sua função faz uma mudança em um vetor de um carrinho de compras, por exemplo, adicionando um item para ser comprado, então qualquer outra função que use o vetor `cart` também será afetada por essa adição. Isso pode ser ótimo, mas também pode ser ruim. Vamos imaginar uma situação ruim:

O usuário clica no botão "Comprar", botão que invoca a função `purchase` que dispara uma série de requisições e manda o vetor `cart` para o servidor. Devido a uma conexão ruim de internet, a função `purchase` precisa fazer novamente a requisição. Agora, imagine que nesse meio tempo o usuário acidentalmente clique no botão `Adicionar ao carrinho` em um produto que ele não queria antes da requisição começar. Se isto acontecer e a requisição for enviada novamente, então a função `purchase` irá enviar acidentalmente o vetor com o novo produto adicionado porque existe uma referência para o vetor `cart` que a função `addItemToCart` modificou adicionando um produto indesejado.

Uma ótima solução seria que a função `addCartToItem` sempre clonasse o vetor `cart`, editasse-o, e então retornasse seu clone. Isso garante que nenhuma outra função que possua uma referência para o carrinho de compras seja afetada por qualquer mudança feita.

Duas ressalvas desta abordagem:

  1. Podem haver casos onde você realmente quer mudar o objeto de entrada, mas quando você adota este tipo de programação, você vai descobrir que estes casos são bastante raros. A maioria das coisas podem ser refatoradas para não terem efeitos colaterais.

  2. Clonar objetos grandes pode ser bastante caro em termos de desempenho. Com sorte, na prática isso não é um problema, porque existem [ótimas bibliotecas](https://facebook.github.io/immutable-js/) que permitem que este tipo de programação seja rápida e não seja tão intensa no uso de memória quanto seria se você clonasse manualmente objetos e vetores.


**Ruim:**
```dart
void addItemToCart(List<int> cart, int item) {
  cart.add(item);
} 

final cart = <int>[1, 2];
addItemToCart(cart, 3);

print(cart); // [1, 2, 3]
```

**Bom:**
```dart
List<int> addItemToCart(List<int> cart, int item) {
  return [...cart, item];
}

final cart = <int>[1, 2];
final newCart = addItemToCart(cart, 3);

print(cart); // [1, 2]
print(newCart); // [1, 2, 3]
```
**[⬆ voltar ao topo](#Índice)**

### Favoreça programação funcional sobre programação imperativa
Dart não é uma linguagem funcional da mesma forma que Haskell é, mas possui algumas características funcionais. Linguagens funcionais são mais limpas e fáceis de se testar. Favoreça esse tipo de programação quando puder.

**Ruim:**
```dart
final programmerOutput = <Programmer>[
  Programmer(name: 'Uncle Bobby', linesOfCode: 500),
  Programmer(name: 'Suzie Q', linesOfCode: 1500),
  Programmer(name: 'Jimmy Gosling', linesOfCode: 150),
  Programmer(name: 'Gracie Hopper', linesOfCode: 1000),
];

var totalOutput = 0;

for (var i = 0; i < programmerOutput.length; i++) {
  totalOutput += programmerOutput[i].linesOfCode;
}
```

**Bom:**
```dart
final programmerOutput = <Programmer>[
  Programmer(name: 'Uncle Bobby', linesOfCode: 500),
  Programmer(name: 'Suzie Q', linesOfCode: 1500),
  Programmer(name: 'Jimmy Gosling', linesOfCode: 150),
  Programmer(name: 'Gracie Hopper', linesOfCode: 1000),
];

final totalOutput = programmerOutput.fold<int>(
    0, (previousValue, programmer) => previousValue + programmer.linesOfCode);
```
**[⬆ volta ao topo](#Índice)**

### Encapsule condicionais

**Ruim:**
```dart
if (programmer.language == 'dart' && programmer.projectsList.isNotEmpty) {
  // ...
}
```

**Bom:**
```dart
bool isValidDartProgrammer(Programmer programmer) {
  return programmer.language == 'dart' && programmer.projectsList.isNotEmpty;
}

if (isValidDartProgrammer(programmer)) {
  // ...
}
```
**[⬆ voltar ao topo](#Índice)**

### Evite negações de condicionais

**Ruim:**
```dart
bool isFileNotValid(File file) {
  // ...
}

if (!isFileNotValid(file)) {
  // ...
}
```

**Bom:**
```dart
bool isFileValid(File file) {
  // ...
}

if (isFileValid(file)) {
  // ...
}
```
**[⬆ voltar ao topo](#Índice)**

### Evite condicionais
Esta parece ser uma tarefa impossível. Da primeira vez que as pessoas escutam isso, a maioria diz, “como eu supostamente faria alguma coisa sem usar `if`? ” A resposta é que você pode usar polimorfismo para realizar a mesma tarefa em diversos casos. A segunda questão é geralmente, “bom, isso é ótimo, mas porque eu deveria fazer isso?” A resposta é um conceito de código limpo aprendido previamente: uma função deve fazer apenas uma coisa. Quando você tem classes e funções que tem declarações `if`, você esta dizendo para seu usuário que sua função faz mais de uma coisa. Relembre-se, apenas uma coisa.

**Ruim:**
```dart
class Airplane {
  // ...
  double getCruisingAltitude() {
    switch (type) {
      case '777':
        return getMaxAltitude() - getPassengerCount();
      case 'Air Force One':
        return getMaxAltitude();
      case 'Cessna':
        return getMaxAltitude() - getFuelExpenditure();
    }
  }
}
```

**Bom:**
```dart
class Airplane {
  // ...
}

class Boeing777 extends Airplane {
  // ...
  double getCruisingAltitude() {
    return getMaxAltitude() - getPassengerCount();
  }
}

class AirForceOne extends Airplane {
  // ...
  double getCruisingAltitude() {
    return getMaxAltitude();
  }
}

class Cessna extends Airplane {
  // ...
  double getCruisingAltitude() {
    return getMaxAltitude() - getFuelExpenditure();
  }
}
```
**[⬆ voltar ao topo](#Índice)**

### Remova código morto
Código morto é tão ruim quanto código duplicado. Não existe nenhum motivo para deixá-lo em seu código. Se ele não estiver sendo chamado, livre-se dele. Ele ainda estará a salvo no seu histórico de versionamento se ainda precisar dele.

**Ruim:**
```dart
Future<void> oldRequest(url) {
  // ...
}

Future<void> newRequest(url) {
  // ...
}

await newRequest();
```

**Bom:**
```dart
Future<void> newRequest(url) {
  // ...
}

await newRequest();
```
**[⬆ voltar ao topo](#Índice)**

## **Objetos e Estruturas de Dados**
### Use getters e setters apenas quando necessário
Diferente de outras linguagens, no Dart o recomendado é usar getters e setters apenas quando existe alguma lógica antes da utilização do atributo. Quando a ideia é puramente pegar ou editar o atributo, o ideal é não usá-los.


**Ruim:**
```dart
class BankAccount {
  // "_" configura como privado
  int _balance;

  int get balance => _balance;

  set balance(int amount) => _balance = amount;

  BankAccount({
    int balance = 0,
  }) : _balance = balance;
}

final account = BankAccount();
account.balance = 100;
```

**Bom:**
```dart
class BankAccount {
  int balance;
  // ...

  BankAccount({
    this.balance = 0,
    // ...
  });
}

final account = BankAccount();
account.balance = 100;
```
**[⬆ voltar ao topo](#Índice)**


### Utilize métodos e atributos privados quando necessário
Se um método ou atributo deve ser utilizado apenas dentro de uma classe, ele deve ser privado.

**Ruim:**
```dart
class Employee {
  String name;

  Employee({required this.name});
}

final employee = Employee(name: 'John Doe');
print(employee.name); // John Doe
employee.name = 'Uncle Bob';
print(employee.name); // Uncle Bob
```

**Bom:**
```dart
class Employee {
  String _name;

  Employee({required String name}) : _name = name;
}

final employee = Employee(name: 'John Doe');
print(employee.name); // Não é possível acessar de fora da classe.
```
**[⬆ voltar ao topo](#Índice)**

## **Classes**
### Use encadeamento de métodos (cascade notation)
Este padrão permite que seu código seja expressivo e menos verboso. Por esse motivo, eu digo, use encadeamento de métodos e dê uma olhada em como o seu código ficará mais limpo.

**Ruim:**
```dart
class Car {
  String make;
  String model;
  String color;

  Car({
    required this.make,
    required this.model,
    required this.color,
  });

  save() => print('$make, $model, $color');
}

final car = Car(make: 'Ford', model: 'F-150', color: 'red');
car.color = 'pink';
car.save();
```

**Bom:**
```dart
class Car {
  String make;
  String model;
  String color;

  Car({
    required this.make,
    required this.model,
    required this.color,
  });

  save() => print('$make, $model, $color');
}

final car = Car(make: 'Ford', model: 'F-150', color: 'red')
  ..color = 'pink'
  ..save();
```
**[⬆ voltar ao topo](#Índice)**

### Prefira composição ao invés de herança
Como dito famosamente em [*Padrão de projeto*](https://pt.wikipedia.org/wiki/Padr%C3%A3o_de_projeto_de_software) pela Gangue dos Quatro, você deve preferir composição sobre herança onde você puder. Existem muitas boas razões para usar herança e muitas boas razões para se usar composição. O ponto principal para essa máxima é que se sua mente for instintivamente para a herança, tente pensar se composição poderia modelar melhor o seu problema. Em alguns casos pode.

Você deve estar pensando então, "quando eu deveria usar herança?" Isso depende especificamente do seu problema, mas essa é uma lista decente de quando herança faz mais sentido que composição:

1. Sua herança representa uma relação de "isto-é" e não uma relação de "isto-tem" (Human→Animal vs. User->UserDetails)
2. Você pode reutilizar código de classes de base (Humanos podem se mover como todos os animais).
3. Você quer fazer mudanças globais para classes derivadas mudando apenas a classe base. (Mudar o custo calórico para todos os animais quando se movem).

**Ruim:**
```dart
class Employee {
  String name;
  String email;

  Employee({
    required this.name,
    required this.email,
  });

  // ...
}

// Ruim porque Employees (Empregados) "tem" dados de impostos. EmployeeTaxData não é um tipo de Employee.
class EmployeeTaxData extends Employee {
  String ssn;
  double salary;

  EmployeeTaxData({
    required this.ssn,
    required this.salary,
    required super.name,
    required super.email,
  });

  // ...
}
```

**Bom:**
```dart
class EmployeeTaxData {
  String ssn;
  double salary;

  EmployeeTaxData({
    required this.ssn,
    required this.salary,
  });

  // ...
}

class Employee {
  String name;
  String email;
  EmployeeTaxData? taxData;

  Employee({
    required this.name,
    required this.email,
  });

  void setTaxData(String ssn, double salary) {
    taxData = EmployeeTaxData(ssn: ssn, salary: salary);
  }

  // ...
}
```
**[⬆ voltar ao topo](#Índice)**

## **SOLID**
### Princípio da Responsabilidade Única (SRP)
Como dito em Código Limpo, "Nunca deveria haver mais de um motivo para uma classe ter que mudar". É tentador empacotar uma classe em excesso com muitas funcionalidades, como quando você pode levar apenas uma mala em seu voo. O problema com isso é que sua classe não será conceitualmente coesa e dar-lhe-á diversos motivos para mudá-la. Minimizar o número de vezes que você precisa mudar uma classe é importante, porque, se muitas funcionalidades estão em uma classe e você mudar uma porção dela, pode ser difícil entender como isto afetará outras módulos que dependem dela no seu código.

**Ruim:**
```dart
class UserSettings {
  String user;
  
  UserSettings({
    required this.user,
  });

  void changeSettings(Settings settings) {
    if (verifyCredentials()) {
      // ...
    }
  }

  bool verifyCredentials() {
    // ...
  }
}
```

**Bom:**
```dart
class UserAuth {
  String user;

  UserAuth({
    required this.user,
  });

  bool verifyCredentials() {
    // ...
  }
}

class UserSettings {
  String user;
  UserAuth auth;

  UserSettings({
    required this.user,
  }) : auth = UserAuth(user: user);

  void changeSettings(Settings settings) {
    if (auth.verifyCredentials()) {
      // ...
    }
  }
}
```
**[⬆ voltar ao topo](#Índice)**

### Princípio do Aberto/Fechado (OCP)
Como foi dito por Bertrand Meyer, "entidades de software (classes, módulos, funções, etc.) devem se manter abertas para extensões, mas fechadas para modificações." Mas o que isso significa? Esse princípio basicamente diz que você deve permitir que usuários adicionem novas funcionalidades sem mudar código já existente.

**Ruim:**
```dart
double getArea(Shape shape) {
  if (shape is Circle) {
    return getCircleArea(shape);
  } else if (shape is Square) {
    return getSquareArea(shape);
  }
}

double getCircleArea(Shape shape) {
  // ...
}

double getSquareArea(Shape shape) {
  // ...
}
```

**Bom:**
```dart
abstract class Shape {
  double getArea();
}

class Circle extends Shape {
  @override
  double getArea() {
    // ...
  }
}

class Square extends Shape {
  @override
  double getArea() {
    // ...
  }
}

// ...
final area = shape.getArea();
```
**[⬆ voltar ao topo](#Índice)**


### Princípio de Substituição de Liskov (LSP)
Esse é um termo assustador para um conceito extremamente simples. É formalmente definido como “Se S é um subtipo de T, então objetos do tipo T podem ser substituídos por objetos com o tipo S (i.e., objetos do tipo S podem substituir objetos do tipo T) sem alterar nenhuma das propriedades desejáveis de um programa (corretude, desempenho em tarefas, etc.).” Esta é uma definição ainda mais assustadora.

A melhor explicação para este conceito é se você tiver uma classe pai e uma classe filha, então a classe base e a classe filha pode ser usadas indistintamente sem ter resultados incorretos. Isso ainda pode ser confuso, então vamos dar uma olhada no exemplo clássico do Quadrado-Retângulo (Square-Rectangle). Matematicamente, um quadrado é um retângulo, mas se você modelá-lo usando uma relação “isto-é” através de herança, você rapidamente terá problemas.

**Ruim:**
```dart
class Rectangle {
  double width;
  double height;

  Rectangle({
    this.width = 0,
    this.height = 0,
  });

  // setWidth e setHeight usados apenas para este exemplo
  void setWidth(double value) => width = value;

  void setHeight(double value) => height = value;

  double getArea() {
    return width * height;
  }
}

class Square extends Rectangle {
  Square({
    super.width = 0,
    super.height = 0,
  });

  @override
  void setWidth(double value) {
    width = value;
    height = value;
  }

  @override
  void setHeight(double value) {
    width = value;
    height = value;
  }
}

final rectangles = [Rectangle(), Rectangle(), Square()];

for (final rectangle in rectangles) {
  rectangle.setWidth(4);
  rectangle.setHeight(5);

  final area = rectangle.getArea();
  print(area); // RUIM: Retorna 25 para o Quadrado. Mas acaba não sendo o height x width esperado.
}
```

**Bom:**
```dart
abstract class Shape {
  double getArea();
}

class Rectangle extends Shape {
  double width;
  double height;

  Rectangle({
    required this.width,
    required this.height,
  });

  @override
  double getArea() {
    return width * height;
  }
}

class Square extends Shape {
  double length;

  Square({
    required this.length,
  });

  @override
  double getArea() {
    return length * length;
  }
}

final rectangles = [
  Rectangle(width: 4, height: 5),
  Rectangle(width: 4, height: 5),
  Square(length: 4),
];

for (final rectangle in rectangles) {
  final area = rectangle.getArea();
  print(area); // Mostra os valores corretamente: 20, 20, 16.
}
```
**[⬆ voltar ao topo](#Índice)**

### Princípio da Segregação de Interface (ISP)
O ISP diz que "Clientes não devem ser forçados a depender de métodos que eles não usam". O correto é sempre criar interfaces mais específicas ao invés de criar apenas uma interface genérica. Ou seja, se a sua classe que implementa uma interface usa o famoso `throw UnimplementedError()`, ela provavelmente não respeita o princípio.

**Ruim:**
```dart
abstract class Book {
  int getNumberOfPages();
  void download();
}

class EBook implements Book {
  @override
  int getNumberOfPages() {
    // ...
  }

  @override
  String download() {
    // ...
  }
}

class PhysicalBook implements Book {
  @override
  int getNumberOfPages() {
    // ...
  }

  @override
  void download() {
    throw UnimplementedError(); // Livro físico não tem download.
  }
}
```

**Bom:**
```dart
abstract class Book {
  int getNumberOfPages();
}

abstract class DownloadableBook {
  void download();
}

class EBook implements Book, DownloadableBook {
  @override
  int getNumberOfPages() {
    // ...
  }

  @override
  void download() {
    // ...
  }
}

class PhysicalBook implements Book {
  @override
  int getNumberOfPages() {
    // ...
  }
}
```
**[⬆ voltar ao topo](#Índice)**

### Princípio da Inversão de Dependência  (DIP)
Este princípio nos diz duas coisas essenciais:
1. Módulos de alto nível não devem depender de módulos de baixo nível. Ambos devem depender de abstrações.
2. Abstrações não devem depender de detalhes. Detalhes devem depender de abstrações.

Você já pode ter visto uma implementação deste princípio na forma de injeção de dependência (DI). Apesar de não serem conceitos idênticos, DIP não deixa módulos de alto nível saber os detalhes de seus módulos de baixo nível, assim como configurá-los. Isso pode ser alcançado através de DI. Um grande benefício é que reduz o acoplamento entre os módulos. Acoplamento é um padrão de desenvolvimento muito ruim porque torna seu código mais difícil de ser refatorado.

**Ruim:**
```dart
class InventoryRequester {
  void requestItem(item) {
    // ...
  }
}

class InventoryTracker {
  final requester = InventoryRequester(); // InventoryTracker dependendo de um módulo de baixo nível.
  List<String> items;

  InventoryTracker({
    required this.items,
  });

  void requestItems() {
    for (var item in items) {
      requester.requestItem(item);
    }
  }
}

final inventoryTracker = InventoryTracker(items: ['apples', 'bananas']);
inventoryTracker.requestItems();
```

**Bom:**
```dart
class InventoryTracker {
  List<String> items;
  InventoryRequester requester;

  InventoryTracker({
    required this.items,
    required this.requester,
  });

  void requestItems() {
    for (var item in items) {
      requester.requestItem(item);
    }
  }
}

abstract class InventoryRequester {
  void requestItem(item);
}

class InventoryRequesterV1 implements InventoryRequester {
  @override
  void requestItem(item) {
    // ...
  }
}

class InventoryRequesterV2 implements InventoryRequester {
  @override
  void requestItem(item) {
    // ...
  }
}

// Construindo nossas dependências externamente e injetando-as, podemos facilmente
// substituir nosso módulo de request por um mais novo.
final inventoryTracker = InventoryTracker(
  items: ['apples', 'bananas'],
  requester: InventoryRequesterV2(),
);
inventoryTracker.requestItems();
```
**[⬆ voltar ao topo](#Índice)**

## **Testes**
Testes são mais importantes que entregas. Se você não possui testes ou uma quantidade inadequada deles, então toda vez que você entregar seu código você não terá certeza se você não quebrou alguma coisa. Decidir o que constitui uma quantidade adequada é responsabilidade do seu time, mas ter 100% de cobertura (todas as sentenças e branches) é a maneira que se alcança uma alta confiança e uma paz de espírito no desenvolvimento.

Sempre escreva testes para cada nova funcionalidade/módulo que você introduzir. Se seu método preferido for Desenvolvimento Orientado a Testes (TDD), isso é ótimo, mas o ponto principal é apenas ter certeza que você está alcançando suas metas de cobertura antes de lançar qualquer funcionalidade, ou refatorar uma já existente.

### Um conceito por teste

**Ruim:**
```dart
import 'package:test/test.dart';

test('String', () {
  var string = 'foo,bar,baz';
  expect(string.split(','), equals(['foo', 'bar', 'baz']));

  string = '  foo ';
  expect(string.trim(), equals('foo'));
});
```

**Bom:**
```dart
import 'package:test/test.dart';

group('String', () {
  test('.split() splits the string on the delimiter', () {
    const string = 'foo,bar,baz';
    expect(string.split(','), equals(['foo', 'bar', 'baz']));
  });

  test('.trim() removes surrounding whitespace', () {
    const string = '  foo ';
    expect(string.trim(), equals('foo'));
  });
});
```
**[⬆ voltar ao topo](#Índice)**

## **Concorrência**
### Utilize async/await no lugar de then
Utilizar async/await deixa seu código mais simples de entender.

**Ruim:**
```dart
final albumTitle = await client
    .get(Uri.parse('https://jsonplaceholder.typicode.com/albums/1'))
    .then((response) {
  // ...
  return title;
});
```

**Bom:**
```dart
Future<String> getAlbumTitle() async {
  final response = await client
      .get(Uri.parse('https://jsonplaceholder.typicode.com/albums/1'));

  // ...

  return title;
}
```
**[⬆ voltar ao topo](#Índice)**

## **Tratamento de Erros**
`throw error` é uma coisa boa! Eles significam que o programa identificou
com sucesso quando algo deu errado e está permitindo que você saiba disso parando
a execução da função no processo atual, fechando o processo, e
notificando você no console com a pilha de processos.

### Não ignore erros capturados
Não fazer nada com um erro capturado não te dá a habilidade de resolvê-lo ou
reagir ao erro informado. Exibir um log no console(`log`) não é muito
melhor porque muitas vezes ele pode ficar perdido entre um monte de outras
coisas impressas no console. Se você envolver qualquer pedaço de código em um
`try/catch` isso significa que você acredita que um erro pode ocorrer lá e então
você deveria ter um plano, ou criar caminho de código para quando isso ocorrer.

**Ruim:**
```dart
try {
  functionThatMightThrow();
} catch (error) {
  print(error);
}
```

**Bom:**
```dart
try {
  functionThatMightThrow();
} on Exception catch (e, s) {
  // Opção 1:
  log('Error description...', error: e, stackTrace: s);
  // Opção 2:
  notifyUserOfError(e, s);
  // Opção 3:
  reportErrorToService(e, s);
}
```
**[⬆ voltar ao topo](#Índice)**

### Não ignore erros nos Futures
Usar await e try/catch é muito melhor do que usar future/then. Mas, caso queira usar o future/then, lembre-se de também tratar os erros.

**Ruim:**
```dart
functionThatMightThrow().then((value) {
  // ...
}).onError((e, s) {
  print(e);
});
```

**Bom:**
```dart
functionThatMightThrow().then((value) {
  // ...
}).onError((e, s) {
  // Opção 1:
  log('Error description...', error: e, stackTrace: s);
  // Opção 2:
  notifyUserOfError(e, s);
  // Opção 3:
  reportErrorToService(e, s);
});
```
**[⬆ voltar ao topo](#Índice)**

## **Formatação**
Formatação é subjetiva. Como muitas regras aqui, não há nenhuma regra que você TEM que seguir. O ponto principal é NÃO DISCUTA sobre formatação.
Aconselho dar uma lida no [Effective Dart](https://dart.dev/guides/language/effective-dart/style), lá são mostrados vários padrões a serem seguidos, mas nada é obrigatório.

### Utilize a capitalização correta

**Ruim:**
```dart
const DAYS_IN_WEEK = 7;

const Bands = ['AC/DC', 'Led Zeppelin', 'The Beatles'];

void restore_database() {}

class animal {}

typedef predicate<T> = bool Function(T value);
```

**Bom:**
```dart
// lowerCamelCase para consts
const daysInWeek = 7;
const bands = ['AC/DC', 'Led Zeppelin', 'The Beatles'];

// lowerCamelCase para funções
void restoreDatabase() {}

// UpperCamelCase para classes, enum types, typedefs, and type parameters
class Animal {}
typedef Predicate<T> = bool Function(T value);
```
**[⬆ voltar ao topo](#Índice)**

### Funções e chamadas de funções devem estar próximas
Se uma função chamar outra, mantenha estas funções verticalmente próximas no arquivo
fonte. Em um cenário ideal, manter a chamada logo acima da função. Nós tendemos a
ler códigos de cima para baixo, como em um jornal. Por causa disso, faça o seu código
desta maneira.

**Ruim:**
```dart
class Smartphone {
  // ...

  String getOS() {
    // ...
  }

  void showPlatform() {
    final os = getOS();
    final chipset = getChipset();
    // ...
  }

  String getResolution() {
    // ...
  }

  void showSpecifications() {
    showPlatform();
    showDisplay();
  }

  String getChipset() {
    // ...
  }

  void showDisplay() {
    final resolution = getResolution();
    // ...
  }
}
```

**Bom:**
```dart
class Smartphone {
  // ...

  void showSpecifications() {
    showPlatform();
    showDisplay();
  }

  void showPlatform() {
    final os = getOS();
    final chipset = getChipset();
    // ...
  }

  String getOS() {
    // ...
  }

  String getChipset() {
    // ...
  }

  void showDisplay() {
    final resolution = getResolution();
    // ...
  }

  String getResolution() {
    // ...
  }
}
```
**[⬆ voltar ao topo](#Índice)**

## **Comentários**
### Apenas comente coisas que tenham complexidade de lógica de negócio.
Comentários são uma desculpa, não um requisito. Um bom código documenta-se, *a maior parte*, por si só.

**Ruim:**
```dart
List<String> getCitiesNames(List<String> cities) {
  // Lista com os nomes das cidades
  final citiesNames = <String>[];

  // Percorre todas as cidades
  for (final city in cities) {
    // Pega apenas a string que está antes da vírgula
    final filteredCityName = city.split(',')[0];

    // Adiciona o nome da cidade filtrado
    citiesNames.add(filteredCityName);
  }

  // Retorna a lista com os nomes das cidades
  return citiesNames;
}
```

**Bom:**
```dart
List<String> getCitiesNames(List<String> cities) {
  final citiesNames = <String>[];

  for (final city in cities) {
    // Pega apenas a string que está antes da vírgula
    final filteredCityName = city.split(',')[0];

    citiesNames.add(filteredCityName);
  }

  return citiesNames;
}
```
**[⬆ voltar ao topo](#Índice)**

### Não deixe código comentado na sua base de código
Controle de versão existe por um motivo: deixar códigos velhos no seu histórico.

**Ruim:**
```dart
doStuff();
// doOtherStuff();
// doSomeMoreStuff();
// doSoMuchStuff();
```

**Bom:**
```dart
doStuff();
```
**[⬆ voltar ao topo](#Índice)**

### Não comente registro de alterações
Lembre-se, utilize controle de versão! Não tem necessidade em deixar códigos
inutlizados, códigos comentados e especialmente registros de alterações.
Utilize `git log` para pegar o histórico!

**Ruim:**
```dart
/**
 * 2016-12-20: Removidas monads, não entendia elas (RM)
 * 2016-10-01: Melhoria utilizando monads especiais (JP)
 * 2016-02-03: Removido checagem de tipos (LI)
 * 2015-03-14: Adicionada checagem de tipos (JR)
 */
int combine(int a, int b) {
  return a + b;
}
```

**Bom:**
```dart
int combine(int a, int b) {
  return a + b;
}
```
**[⬆ voltar ao topo](#Índice)**

### Evite marcadores de posição
Eles geralmente criam ruídos. Deixe que as funções e nomes de variáveis em conjunto
com a devida identação e formatação deem a estrutura visual para o seu código.

**Ruim:**
```dart
////////////////////////////////////////////////////////////////////////////////
// Intanciação do Programmer
////////////////////////////////////////////////////////////////////////////////
final programmer = Programmer(
  name: 'Jack',
  linesOfCode: 500,
);

////////////////////////////////////////////////////////////////////////////////
// Implementação do startProject
////////////////////////////////////////////////////////////////////////////////
void startProject() {
  // ...
};
```

**Bom:**
```dart
final programmer = Programmer(
  name: 'Jack',
  linesOfCode: 500,
);

void startProject() {
  // ...
};
```
**[⬆ voltar ao topo](#Índice)**
