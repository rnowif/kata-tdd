# Kata Harry Potter

## Création de la feature
Dans le fichier feature, ajouter la feature et le scénario de base.

```
Feature: Calculer le prix d'un panier de livres Harry Potter

Scenario: Calculer le prix d'un seul livre
Given Un panier
When J'ajoute 1 livre de Harry Potter Tome 1
Then Le prix du panier est 8.0€

```

## Préparation des steps
Lancer, le test. Cucumber va proposer un squelette pour la step. Il doit être un peu modifié pour gérer le prix en tant que double.

```java
public class BasketPriceStepDefs {

    private Basket basket;

    @Given("^Un panier$")
    public void aBasket() {
        basket = new Basket();
    }

    @When("^J'ajoute (\\d+) livre de Harry Potter Tome (\\d+)$")
    public void addBooks(int nbBooks, int bookNumber) throws Throwable {
        for (int i = 0; i < nbBooks; i++) {
            basket.addBook(new Book(bookNumber));
        }
    }

    @Then("^Le prix du panier est (\\d+.\\d+)€$")
    public void totalCost(double expectedCost) throws Throwable {
        assertThat(basket.totalPrice()).isCloseTo(
                BigDecimal.valueOf(expectedCost),
                Percentage.withPercentage(0.01)
        );
    }
}
```

Les classes `Basket` et `Book` sont créées vides, tout comme les méthodes utilisées. Le test Cucumber ne passe pas.

## Développement de Basket
Un panier vide coute 0€.
En TDD, créer une classe pour tester ce cas.

```java
public class BasketTest {

    @Test
    public void should_cost_0_when_no_book() {
        Basket basket = new Basket();

        assertThat(basket.totalPrice()).isEqualTo(BigDecimal.ZERO);
    }
}
```

Pour tester l'intégration de `Basket` avec `Book`, on va utiliser le test Cucumber. On va donc implémenter la méthode `addBooks(Book)` et `totalPrice`.

```java
public class Basket {

    private Book book;

    public void addBook(Book book) {
        this.book = book;
    }

    public BigDecimal totalPrice() {
        return book == null ? BigDecimal.ZERO : book.price();
    }
}
```
Le prix du livre va être délocalisé dans `Book`. On va donc coder la méthode `price` de `Book` en TDD.

```java
public class BookTest {

    @Test
    public void should_cost_8() {
        Book book = new Book(1);
        assertThat(book.price()).isEqualTo(BigDecimal.valueOf(8.0));
    }
}
```

Une fois que cette méthode est fonctionnelle, le test Cucumber passe.

## Ajout d'un premier discount

La nouvelle feature est la suivante :
```
Scenario: Une remise de 5% pour deux livres différents
Given Un panier
When J'ajoute 1 livre de Harry Potter Tome 1
And J'ajoute 1 livre de Harry Potter Tome 2
Then Le prix du panier est 15.20€
```

Passer du calcul du prix pour un livre à une réduction me parait une marche trop haute. Je vais donc tout d'abord calculer le prix pour deux livres en TDD.

```java
    @Test
    public void should_cost_two_books_price_when_two_books() {
        Basket basket = new Basket();
        Book tome1 = new Book(1);
        Book tome2 = new Book(2);

        basket.addBook(tome1);
        basket.addBook(tome2);

        assertThat(
                basket.totalPrice()
        ).isEqualTo(
                tome1.price().add(tome2.price())
        );
    }
```

La classe `Basket` devient alors :

```java
public class Basket {

    private List<Book> books = new ArrayList<>();

    public void addBook(Book book) {
        books.add(book);
    }

    public BigDecimal totalPrice() {
        return books.stream()
                .map(Book::price)
                .reduce(BigDecimal::add)
                .orElse(BigDecimal.ZERO);
    }
}
```

Ensuite, je modifie mon test pour prendre en compte la discount.

```java
    @Test
    public void should_apply_5_pct_discount_when_two_different_books() {
        Basket basket = new Basket();
        Book tome1 = new Book(1);
        Book tome2 = new Book(2);

        basket.addBook(tome1);
        basket.addBook(tome2);

        assertThat(
                basket.totalPrice()
        ).isCloseTo(
                tome1.price().add(tome2.price()).multiply(BigDecimal.valueOf(0.95)),
                Percentage.withPercentage(0.1)
        );
    }
```
Et mon code de `Basket` devient alors :

```java
    public BigDecimal totalPrice() {
        BigDecimal total = books.stream()
                .map(Book::price)
                .reduce(BigDecimal::add)
                .orElse(BigDecimal.ZERO);

        if (books.size() == 2) {
            total = total.multiply(BigDecimal.valueOf(0.95));
        }

        return total;
    }
```

Le test Cucumber passe au vert. Je ne fais pas encore de refacto dans `Basket` car il n'y a pas de duplication évidente.

## Ajout du discount de 10%

On ajoute la feature suivante :

```
Scenario: Une remise de 10% pour trois livres différents
Given Un panier
When J'ajoute 1 livre de Harry Potter Tome 1
And J'ajoute 1 livre de Harry Potter Tome 2
And J'ajoute 1 livre de Harry Potter Tome 3
Then Le prix du panier est 21.60€
```

Le code pour la faire passer est le suivant

```java
    public BigDecimal totalPrice() {
        BigDecimal total = books.stream()
                .map(Book::price)
                .reduce(BigDecimal::add)
                .orElse(BigDecimal.ZERO);

        if (books.size() == 2) {
            total = total.multiply(BigDecimal.valueOf(0.95));
        }

        if (books.size() == 3) {
            total = total.multiply(BigDecimal.valueOf(0.90));
        }

        return total;
    }
```

On voit une répétition, donc on va extraire ce comportement dans une `Map` pour contenir les discounts. La classe Basket devient alors la suivante :

```java
public class Basket {

    private List<Book> books = new ArrayList<>();
    private Map<Integer, BigDecimal> discounts = new HashMap<>();

    public Basket() {
        discounts.put(2, BigDecimal.valueOf(0.95));
        discounts.put(3, BigDecimal.valueOf(0.90));
    }

    public void addBook(Book book) {
        books.add(book);
    }

    public BigDecimal totalPrice() {
        BigDecimal total = books.stream()
                .map(Book::price)
                .reduce(BigDecimal::add)
                .orElse(BigDecimal.ZERO);

        return discounts.entrySet().stream()
                .filter(entry -> entry.getKey().equals(books.size()))
                .map(entry -> total.multiply(entry.getValue()))
                .findFirst()
                .orElse(total);
    }
}
```