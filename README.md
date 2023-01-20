# clean-code-dart

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
  12. [Traduções](#traduções)

## Introdução
![Imagem humorística da estimativa de qualidade do software baseado na contagem de quantos palavrões você gritou enquanto lia o código.](http://www.osnews.com/images/comics/wtfm.jpg)

Princípios da Engenharia de Software, do livro de Robert C. Martin
[*Código Limpo*](https://www.amazon.com.br/C%C3%B3digo-Limpo-Habilidades-Pr%C3%A1ticas-Software/dp/8576082675),
adaptados para Dart. Isto não é um guia de estilos. É um guia para se produzir código [legível, reutilizável e refatorável](https://github.com/ryanmcdermott/3rs-of-software-architecture) em Dart.

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
// Para que diabos serve 86400000?
Future.delayed(Duration(milliseconds: 86400000), blastOff);
```

**Bom:**
```dart
// Declare-as como final em lowerCamelCase.
// OBS: millisecondsPerDay será int, pois o tipo será inferido.
final millisecondsPerDay = 86400000;

Future.delayed(Duration(milliseconds: millisecondsPerDay), blastOff);
```
**[⬆ voltar ao topo](#Índice)**

### Use variáveis explicativas
**Ruim:**
```dart
final address = <String>['One Infinite Loop', 'Cupertino', '95014'];
saveCityZipCode(address[1], address[2]);
```

**Bom:**
```dart
final address = <String>['One Infinite Loop', 'Cupertino', '95014'];
final city = address[1];
final zipCode = address[2];
saveCityZipCode(city, zipCode);
```
**[⬆ voltar ao topo](#Índice)**

### Evite Mapeamento Mental
Explicito é melhor que implícito.

**Ruim:**
```dart
final locations = <String>['Austin', 'New York', 'San Francisco'];
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
final locations = <String>['Austin', 'New York', 'San Francisco'];
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

### Use argumentos padrões ao invés de curto circuitar ou usar condicionais

Argumentos padrões são geralmente mais limpos do que curto circuitos. Esteja ciente que se você usá-los, sua função apenas irá fornecer valores padrões para argumentos `null`.

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
void createMenu(String title, String body, String buttonText, bool cancellable) {
  // ...
}
```

**Bom:**
```dart
void createMenu({
  required String title,
  required String body,
  required String buttonText,
  required bool cancellable,
}) {
  // ...
}

createMenu(
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