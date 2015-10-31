# Kata Legacy Sprout Method

## Léger refacto

Pour débuter, on va rendre la méthode non statique.

```java
public class SecurityManager {

	public static void createUser() {
        new SecurityManager().createUserNonStatic();
    }

	private void createUserNonStatic() {
		Scanner scanner = new Scanner(System.in);
		System.out.print("Enter a username:");
		String username = scanner.next();
		System.out.print("Enter your full name:");
		String userFullName = scanner.next();
		System.out.print("Enter your password:");
		String password = scanner.next();
		System.out.print("Re-enter your password:");
		String confirmPassword = scanner.next();
		scanner.close();

		if (!password.equals(confirmPassword)) {
			System.out.println("The passwords don't match !");
			return;
		}

		if (password.length() < 8) {
			System.out.println("Password must be at least 8 characters in length !");
			return;
		}

		// Encrypt the password (just reverse it, should be secure)
		char[] array = password.toCharArray();
		ArrayUtils.reverse(array);

		System.out.println(String.format("Saving Details for User (%s, %s, %s)\n",
				username,
				userFullName,
				Arrays.toString(array)));
	}

}
```

## Ajout de la fonctionnalité

La fonctionnalité à ajouter est de pouvoir envoyer les informations de l'utilisateur à une application tierce en HTTP. Pour ce faire, on va injecter une dépendance dans la méthode.

```java
private void createUserNonStatic(UserSender userSender) {
   // [Snip]
   userSender.sendUser(new UserIdentity(username, userFullName), new UserPassword(array));
   // [Snip]
}
```
Il faut modifier l'appel de la méthode pour lui injecter une implémentation de UserSender.

```java
public static void createUser() {
    new SecurityManager().createUserNonStatic(new RestUserSender());
}
```

## Développement en TDD de RestUserSender

Désormais, il est possible de développer le RestUserSender en TDD.