# Kata Legacy Code Base

## Léger refactor

Afin de rendre le code testable, on doit légèrement refactorer la classe :

- Rendre la méthode non statique (faire un `delegate` pour la méthode statique existante).
- Extraire la récupération du System.out et du System.in dans des méthodes protected.

Voici le code de prod à la fin de cette étape :

```java
public class SecurityManager {

	public static void createUser() {
        new SecurityManager().createUserNonStatic();
    }

	public void createUserNonStatic() {
		Scanner scanner = new Scanner(inputStream());
        PrintStream printStream = printStream();

        printStream.print("Enter a username:");
		String input1 = scanner.next();
		printStream.print("Enter your full name:");
		String input2 = scanner.next();
		printStream.print("Enter your password:");
		String input3 = scanner.next();
		printStream.print("Re-enter your password:");
		String input4 = scanner.next();
		scanner.close();

		if (!input3.equals(input4)) {
			printStream.println("The passwords don't match !");
			return;
		}

		if (input3.length() < 8) {
			printStream.println("Password must be at least 8 characters in length !");
			return;
		}

		// Encrypt the password (just reverse it, should be secure)
		char[] array = input3.toCharArray();
		ArrayUtils.reverse(array);

		printStream.println(String.format("Saving Details for User (%s, %s, %s)\n",
                input1,
                input2,
                Arrays.toString(array)));
	}

    protected PrintStream printStream() {
        return System.out;
    }

    protected InputStream inputStream() {
        return System.in;
    }

}
```

## Ecrire le harnais de test

Le code étant testable, on peut écrire le harnais de test. Voici le code de test à la fin de cette étape

```java
public class SecurityManagerTest {

    private class SecurityManagerUnderTest extends SecurityManager {

        public PrintStream printStream;
        public InputStream inputStream;

        @Override
        protected PrintStream printStream() {
            return printStream;
        }

        @Override
        protected InputStream inputStream() {
            return inputStream;
        }
    }

    @Test
    public void should_stop_if_password_dont_match() {

        PrintStream printStream = mock(PrintStream.class);
        InputStream inputStream = inputStreamFromResponses("Reno", "Humbert", "passwd", "Password");

        SecurityManagerUnderTest securityManager = newSecurityManager(printStream, inputStream);

        securityManager.createUserNonStatic();

        verifyQuestions(printStream);
        verify(printStream).println("The passwords don't match !");
    }

    @Test
    public void should_stop_if_password_less_than_8_char() {
        PrintStream printStream = mock(PrintStream.class);
        InputStream inputStream = inputStreamFromResponses("Reno", "Humbert", "passwd", "passwd");

        SecurityManagerUnderTest securityManager = newSecurityManager(printStream, inputStream);

        securityManager.createUserNonStatic();

        verifyQuestions(printStream);
        verify(printStream).println("Password must be at least 8 characters in length !");
    }

    @Test
    public void should_print_final_details() {
        PrintStream printStream = mock(PrintStream.class);
        InputStream inputStream = inputStreamFromResponses("Reno", "Humbert", "Admin01!", "Admin01!");

        SecurityManagerUnderTest securityManager = newSecurityManager(printStream, inputStream);

        securityManager.createUserNonStatic();

        verifyQuestions(printStream);
        verify(printStream).println("Saving Details for User (Reno, Humbert, [!, 1, 0, n, i, m, d, A])\n");
    }

    private void verifyQuestions(PrintStream printStream) {
        verify(printStream).print("Enter a username:");
        verify(printStream).print("Enter your full name:");
        verify(printStream).print("Enter your password:");
        verify(printStream).print("Re-enter your password:");
    }

    private SecurityManagerUnderTest newSecurityManager(PrintStream printStream, InputStream inputStream) {
        SecurityManagerUnderTest securityManager = new SecurityManagerUnderTest();
        securityManager.printStream = printStream;
        securityManager.inputStream = inputStream;
        return securityManager;
    }

    private InputStream inputStreamFromResponses(String... responses) {
        return new ByteArrayInputStream(String.join("\n", responses).getBytes());
    }
}
```
