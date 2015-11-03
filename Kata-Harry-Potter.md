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