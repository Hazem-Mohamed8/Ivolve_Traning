## 1. Install Java and Maven

```bash
sudo apt update && sudo apt install -y openjdk-17-jdk maven
java -version
```
## 2. Clone the source code

```bash
git clone https://github.com/Ibrahim-Adel15/CalculatorMavenApp.git
cd CalculatorMavenApp
```

![clone](screenshots/02-clone.png)

## 3. Run the unit tests

```bash
mvn test
```

All tests should pass (`BUILD SUCCESS`).

![unit tests](screenshots/03-test.png)

## 4. Build the application

```bash
mvn package
ls target/calculator.jar
```

Maven compiles the code, runs the tests again, and packages the artifact at `target/calculator.jar`.

![build](screenshots/04-package.png)
![artifact](screenshots/05-artifact.png)

## 5. Run the application and verify it works

```bash
java -jar target/calculator.jar
```

![run](screenshots/06-run.png)
