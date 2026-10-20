# Lab 2: Building and Packaging a Java Application with Gradle

Source code: https://github.com/Ibrahim-Adel15/CalculatorGradleApp.git

## 1. Install Gradle

The Gradle version available in Ubuntu's `apt` is too old for this project's `build.gradle`
(it failed with `Could not find method java()`), so Gradle 8.10.2 was installed with SDKMAN:

```bash
sudo apt install -y curl zip unzip
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install gradle 8.10.2
gradle -v
```

![gradle version](screenshots/01-versions.png)

## 2. Clone the source code

```bash
git clone https://github.com/Ibrahim-Adel15/CalculatorGradleApp.git
cd CalculatorGradleApp
```

![clone](screenshots/02-clone.png)

## 3. Run the unit tests

```bash
gradle test
```

![unit tests](screenshots/03-test.png)

## 4. Build the application

```bash
gradle build
ls build/libs/calculator.jar
```

The artifact is generated at `build/libs/calculator.jar`.

![build](screenshots/04-build.png)
![artifact](screenshots/05-artifact.png)

## 5. Run the application and verify it works

```bash
java -jar build/libs/calculator.jar
```

![run](screenshots/06-run.png)

## Result

The application was built with Gradle, the unit tests passed, and the generated `calculator.jar` runs correctly.
