# devopsexam
 here are **all the steps** for Exercises 1 to 10, in simple words. 💪 Your record uses someone else's values (the `Parkbong7` repo, port `8081`, JDK-25, Maven 3.9.16), so I've used **your own** setup: `esaikaviya1612`, port `8080`, JDK 21, Maven 3.10.0. Don't copy their screenshots; use yours.

## Exercise 1: Lab setup
1. Install JDK, Git, Maven and Eclipse.
2. In CMD, run `java -version` and `javac -version` (**Fig 1.1**).
3. Run `git --version`, `mvn -version` and `where mvn` (**Fig 1.2**).
4. In Eclipse, create `Calculator.java` and run it. It prints Addition 15, Subtraction 5, Multiplication 50, Division 2 (**Fig 1.3**).
5. Convert the project to Maven: right-click the project → Configure → Convert to Maven Project. Use Group ID `com.esaikaviya`, Artifact ID `Calculator`.
6. In the project folder, run `mvn clean install` and get `BUILD SUCCESS` (**Fig 1.4**).
7. Start SonarQube and open its dashboard (**Fig 1.5**), then open Jenkins (**Fig 1.6**).

## Exercise 2: Git with Eclipse (EGit)
1. Create a repository on GitHub.
2. In Eclipse, create the Java project and run the calculator. **Screenshot:** program output.
3. Right-click the project → **Team → Share Project → Git**. This connects it to a local Git repository.
4. Right-click → **Team → Commit**. Write a message like "Initial Commit" and commit. **Screenshot:** Eclipse commit.
5. Right-click → **Team → Push Branch** to upload to GitHub. **Screenshot:** the GitHub repository.
6. In **another workspace**: File → Import → Git → Projects from Git → Clone URI, and paste your GitHub URL.
7. Clone, import, and open `Calculator.java`. **Screenshot:** cloned project.

## Exercise 3: SonarQube project
1. Start SonarQube with `StartSonar.bat` (wait for "SonarQube is operational").
2. Open `http://localhost:9000` and log in.
3. Create a project (name and key `calculator`).
4. Choose **Locally**, generate a **token** and copy it. Don't show it in screenshots.
5. In the project folder, run this as **one line**:
   `mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=calculator -Dsonar.projectName=calculator -Dsonar.host.url=http://localhost:9000 -Dsonar.token=YOUR_TOKEN`
6. **Screenshot:** the dashboard with Security, Reliability, Maintainability, Coverage and Duplications.

## Exercise 4: Static analysis
1. Run the same Maven Sonar command and wait for `BUILD SUCCESS`. **Screenshot:** CMD.
2. Open the SonarQube project dashboard. **Screenshot.**
3. Open the **Issues** page. You'll see issues like "Replace System.out by a logger" and "Move file to a named package". **Screenshot.**
4. Fix one. You fixed the package issue by moving `Calculator.java` into the package `com.esaikaviya`.
5. Run the analysis again. The issue count drops. **Screenshot:** updated results.

## Exercise 5: Local repository in Artifactory
1. Start Artifactory and open `http://localhost:8082`. **Screenshot:** login/home.
2. Log in as `admin`. The first login asks you to set a new password.
3. Go to **Administration → Repositories → Repositories → Add Repositories → Local Repository**. **Screenshot.**
4. Choose package type **Maven**.
5. Enter the Repository Key `calculator-local` and click **Create Local Repository**. **Screenshot.**
6. Open **Application → Artifactory → Artifacts** and check the repository appears. **Screenshot.**

## Exercise 6: Maven build automation
From the project folder, run these one by one and take a screenshot of each result:
1. `mvn clean` removes old build files (`BUILD SUCCESS`).
2. `mvn compile` compiles the code (`BUILD SUCCESS`).
3. `mvn test` runs the tests (`BUILD SUCCESS`).
4. `mvn package` creates `target\Calculator-0.0.1-SNAPSHOT.jar`.

## Exercise 7: Jenkins setup
1. **Manage Jenkins → Tools → JDK installations → Add JDK.** Give a name, set JAVA_HOME to your JDK 21 path, and untick *Install automatically*.
2. **Maven installations → Add Maven.** Give a name, set MAVEN_HOME to your Maven folder, untick *Install automatically*, and Save. **Screenshot:** JDK and Maven.
3. **Manage Jenkins → System → Global Properties.** Tick *Environment variables* and add `JAVA_HOME` and `M2_HOME`. Save. **Screenshot.**
4. **New Item →** name `calculator-CI` → **Freestyle project → OK.**
5. Source Code Management: **Git**, URL `https://github.com/esaikaviya1612/Calculator.git`, branch `*/master`.
6. Build step: **Invoke top-level Maven targets**, goals `clean package`. **Screenshot:** configuration.
7. Save → **Build Now.** **Screenshot:** build #1 with a green tick.
8. Open the build → **Console Output**, ending in `BUILD SUCCESS` and `Finished: SUCCESS`. **Screenshot.**

## Exercise 8: Plugins
1. **Manage Jenkins → Plugins → Available plugins.**
2. Search and tick: **Copy Artifact, JaCoCo, Artifactory, Build Pipeline, Deploy to container.** **Screenshot.**
3. Click **Install** and wait until every plugin shows **Success.** **Screenshot.**
4. Open **Installed plugins** and check all five are listed. **Screenshot.**

## Exercise 9: Central CI pipeline

**Preparation first:**
1. In `Calculator.java`, put add, subtract, multiply and divide into separate methods and call them from `main`.
2. Create `CalculatorTest.java` with 5 JUnit 5 tests (add, subtract, multiply, divide, divide by zero).
3. Update `pom.xml`: packaging `war`, `failOnMissingWebXml` false, JUnit 5.11.4 (test scope), `testSourceDirectory` set to `test`, surefire 3.5.2, and JaCoCo 0.8.14 (goals `prepare-agent` and `report`).
4. Commit and push to GitHub. Then run `calculator-CI` to confirm it still builds.

**Create 5 Freestyle jobs.** In every job, set Git to your repo, branch `*/master`, and your Maven version.
1. **Checkout:** no build step. Post-build → *Build other projects* → `Compile` (trigger only if stable).
2. **Compile:** goals `clean compile`. Next job: `UnitTest`.
3. **UnitTest:** goals `test`. Post-build: *Publish JUnit test result report* with `**/target/surefire-reports/*.xml`. Next job: `Coverage`.
4. **Coverage:** goals `clean test`. Post-build: *Record JaCoCo coverage report* with exec `**/target/jacoco.exec`, classes `**/target/classes`, sources `**/src`. Next job: `StaticAnalysis`.
   - **Our fix:** type `0` in **every** threshold box, including the bottom **delta** row.
5. **StaticAnalysis:** one-line goals (with your own token):
   `clean test org.sonarsource.scanner.maven:sonar-maven-plugin:5.0.0.4389:sonar -Dsonar.projectKey=calculator -Dsonar.projectName=calculator -Dsonar.host.url=http://localhost:9000 -Dsonar.token=YOUR_TOKEN -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml`

**Run and take screenshots:**
1. Click **Build Now** on Checkout and watch each job start the next.
2. Take screenshots of: each job's configuration page, the dashboard with the jobs green, the Test Result page (5 passed), the JaCoCo Coverage Report, and the SonarQube project page after StaticAnalysis.

## Exercise 10: Copy and move jobs
**Copy:**
1. **New Item →** name `calculator-CI-copy`.
2. Choose **Duplicate an existing item** and type `calculator-CI`. **Screenshot.**
3. Click **OK**, then **Save**.
4. **Screenshot:** dashboard with the original and the copy.

**Move:**
1. **New Item →** name `MyFolder` → **Folder → OK → Save.**
2. Open `calculator-CI-copy` → **Move** (left menu).
3. Select **Jenkins » MyFolder**. **Screenshot:** Move page.
4. Click **Move**, then open MyFolder and see the job inside it. **Screenshot.**

**Where you are now:** Exercises 1 to 8 and 10 are done. Exercise 9 needs the Coverage fix, a successful **Build Now**, and then StaticAnalysis. 🚀

//here

# SB8050 DevOps Practical – Commands and Code (Exercises 1 to 12)

Reference sheet for my own setup:

| Item | Value |
|---|---|
| GitHub repo | `https://github.com/esaikaviya1612/Calculator.git` (branch `master`) |
| Package / Group ID | `com.esaikaviya` |
| Artifact ID | `Calculator` |
| JDK | 21 (`C:\Program Files\Java\jdk-21.0.12.1`) |
| Maven | 3.10.0 (`C:\apache-maven-3.10.0-bin\apache-maven-3.10.0`) |
| Jenkins | `http://localhost:8080` |
| SonarQube | `http://localhost:9000` (`D:\sonarqube-26.9.0.129388`) |
| Artifactory | `http://localhost:8082` |

> **Never commit a real token to GitHub.** Everywhere below, `YOUR_TOKEN` must be replaced with your own SonarQube token.
> **Windows tip:** in CMD every command must be on **one line**. The `\` line-break (Linux style) does not work.

---

## Exercise 1 – Testing lab set up

**Aim:** install and check JDK, Git, Maven, Eclipse; create the Calculator project and build it with Maven.

Check the tools (CMD):

```
java -version
javac -version
git --version
mvn -version
where mvn
echo %MAVEN_HOME%
```

`Calculator.java` (first version):

```java
public class Calculator {
    public static void main(String[] args) {
        int a = 10, b = 5;
        System.out.println("Addition: " + (a + b));
        System.out.println("Subtraction: " + (a - b));
        System.out.println("Multiplication: " + (a * b));
        System.out.println("Division: " + (a / b));
    }
}
```

Output: `Addition: 15`, `Subtraction: 5`, `Multiplication: 50`, `Division: 2`

Convert to Maven: Eclipse → right-click project → Configure → Convert to Maven Project (Group ID `com.esaikaviya`, Artifact ID `Calculator`, version `0.0.1-SNAPSHOT`, packaging `jar`). Then build:

```
cd C:\Users\acer\eclipse-workspace-devops\Calculator
mvn clean install
```

Expected: `BUILD SUCCESS` and `target\Calculator-0.0.1-SNAPSHOT.jar`.

---

## Exercise 2 – Git operations using Eclipse EGit

**Aim:** share, commit, push and clone a project using EGit.

Eclipse (GUI) steps:
1. Right-click project → **Team → Share Project → Git** (create local repository)
2. Right-click project → **Team → Commit** (message: `Initial Commit`)
3. Right-click project → **Team → Push Branch** (push to GitHub)
4. Other workspace: **File → Import → Git → Projects from Git → Clone URI** (paste the GitHub URL)

Same thing with commands (CMD, inside the project folder):

```
git init
git add .
git commit -m "Initial Commit"
git branch -M master
git remote add origin https://github.com/esaikaviya1612/Calculator.git
git push -u origin master
```

Clone:

```
git clone https://github.com/esaikaviya1612/Calculator.git
```

---

## Exercise 3 – Creating a project in SonarQube

**Aim:** create a project in SonarQube and analyse it.

Start SonarQube:

```
cd D:\sonarqube-26.9.0.129388\bin\windows-x86-64
StartSonar.bat
```

Wait for `SonarQube is operational`, then open `http://localhost:9000` (default login `admin` / `admin`, it asks for a new password).

In SonarQube: Create project (name and key) → **Locally** → generate a **token**.

Run the analysis from the project folder (**one line**):

```
mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=Calculator -Dsonar.projectName=Calculator -Dsonar.host.url=http://localhost:9000 -Dsonar.token=YOUR_TOKEN
```

Expected: `ANALYSIS SUCCESSFUL` and `BUILD SUCCESS`. Screenshot: project dashboard (Security, Reliability, Maintainability, Coverage, Duplications).

Common mistakes:
- Command ends with `\` → remove it, use one line.
- `sonar.organization` error → `-Dsonar.host.url` was missing (it then talks to SonarQube Cloud).

---

## Exercise 4 – Using SonarQube with Sonar-Runner (static analysis)

**Aim:** find issues with static analysis, fix them, and analyse again.

1. Run the same Maven Sonar command as Exercise 3 → `BUILD SUCCESS`.
2. SonarQube → project → **Issues** page.
3. Fix an issue. Examples:
   - "Move this file to a named package" → put `Calculator.java` in package `com.esaikaviya` (add `package com.esaikaviya;` as the first line).
   - "Replace System.out by a logger" → use `java.util.logging`:

```java
package com.esaikaviya;

import java.util.logging.Logger;

public class Calculator {
    private static final Logger LOGGER = Logger.getLogger(Calculator.class.getName());

    public static void main(String[] args) {
        int a = 10, b = 5;
        LOGGER.info("Addition: " + (a + b));
    }
}
```

4. Run the analysis again and compare the Issues count (it goes down).

---

## Exercise 5 – Creating a local repository in Artifactory

**Aim:** create a local Maven repository to store build files (`.jar` / `.war`).

No code, only the web page (`http://localhost:8082`):

1. Log in as `admin` (first login asks for a new password).
2. **Administration → Repositories → Repositories → Add Repositories → Local Repository**
3. Package type: **Maven**
4. Repository Key: `calculator-local`
5. **Create Local Repository**
6. **Application → Artifactory → Artifacts** → check `calculator-local` is in the tree.

---

## Exercise 6 – Build automation using Maven

**Aim:** understand the Maven lifecycle: clean → compile → test → package.

Run from the project folder (screenshot each):

```
mvn clean
mvn compile
mvn test
mvn package
```

| Command | What it does |
|---|---|
| `mvn clean` | deletes the old `target` folder |
| `mvn compile` | compiles Java code to `.class` files |
| `mvn test` | runs the unit tests |
| `mvn package` | creates the `.jar` (or `.war`) in `target` |

`pom.xml` holds the project configuration, dependencies and build information.

---

## Exercise 7 – Jenkins installation and system configuration

**Aim:** configure Jenkins for Continuous Integration (`http://localhost:8080`).

**Part A – JDK:** Manage Jenkins → Tools → JDK installations → Add JDK
- Name: `JDK-21`
- JAVA_HOME: `C:\Program Files\Java\jdk-21.0.12.1`
- Untick *Install automatically*

**Part B – Maven:** Maven installations → Add Maven
- Name: `Maven-3.10.0`
- MAVEN_HOME: `C:\apache-maven-3.10.0-bin\apache-maven-3.10.0`
- Untick *Install automatically* → Save

**Part C – Global variables:** Manage Jenkins → System → Global properties → tick *Environment variables*
- `JAVA_HOME` = `C:\Program Files\Java\jdk-21.0.12.1`
- `M2_HOME` = `C:\apache-maven-3.10.0-bin\apache-maven-3.10.0`

**Part D – CI job:**
1. New Item → `calculator-CI` → Freestyle project
2. Source Code Management: Git, URL `https://github.com/esaikaviya1612/Calculator.git`, branch `*/master`
3. Build step → Invoke top-level Maven targets → Maven Version `Maven-3.10.0`, Goals: `clean package`
4. Save → Build Now → Console Output ends with `BUILD SUCCESS` and `Finished: SUCCESS`

Restart Jenkins service when needed: `Windows + R` → `services.msc` → Jenkins → Restart.

---

## Exercise 8 – Download the plugins in Jenkins

**Aim:** install the plugins needed for the CI pipeline.

Manage Jenkins → Plugins → Available plugins. Search, tick and Install:

- Copy Artifact
- JaCoCo
- Artifactory
- Build Pipeline
- Deploy to container

Check all five under **Installed plugins**.

---

## Exercise 9 – Creating the central CI pipeline

**Aim:** five linked Freestyle jobs: Checkout → Compile → UnitTest → Coverage → StaticAnalysis.

### Preparation (code)

`src/main/java/com/esaikaviya/Calculator.java`:

```java
package com.esaikaviya;

public class Calculator {

    public int add(int a, int b) {
        return a + b;
    }

    public int subtract(int a, int b) {
        return a - b;
    }

    public int multiply(int a, int b) {
        return a * b;
    }

    public int divide(int a, int b) {
        if (b == 0) {
            throw new ArithmeticException("Cannot divide by zero");
        }
        return a / b;
    }

    public static void main(String[] args) {
        Calculator calc = new Calculator();
        System.out.println("Addition: " + calc.add(10, 5));
        System.out.println("Subtraction: " + calc.subtract(10, 5));
        System.out.println("Multiplication: " + calc.multiply(10, 5));
        System.out.println("Division: " + calc.divide(10, 5));
    }
}
```

`test/com/esaikaviya/CalculatorTest.java` (5 JUnit 5 tests):

```java
package com.esaikaviya;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;

import org.junit.jupiter.api.Test;

class CalculatorTest {

    private final Calculator calc = new Calculator();

    @Test
    void testAdd() {
        assertEquals(15, calc.add(10, 5));
    }

    @Test
    void testSubtract() {
        assertEquals(5, calc.subtract(10, 5));
    }

    @Test
    void testMultiply() {
        assertEquals(50, calc.multiply(10, 5));
    }

    @Test
    void testDivide() {
        assertEquals(2, calc.divide(10, 5));
    }

    @Test
    void testDivideByZero() {
        assertThrows(ArithmeticException.class, () -> calc.divide(10, 0));
    }
}
```

`pom.xml` (reference version – compare with your own working file, your file is the one that already builds):

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.esaikaviya</groupId>
  <artifactId>Calculator</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>war</packaging>

  <properties>
    <maven.compiler.release>21</maven.compiler.release>
    <failOnMissingWebXml>false</failOnMissingWebXml>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>5.11.4</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <testSourceDirectory>test</testSourceDirectory>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.5.2</version>
      </plugin>
      <plugin>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <version>0.8.14</version>
        <executions>
          <execution>
            <goals>
              <goal>prepare-agent</goal>
            </goals>
          </execution>
          <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
              <goal>report</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

Commit and push, then run `calculator-CI` to check it still builds:

```
git add .
git commit -m "Add calculator methods, JUnit tests and JaCoCo coverage"
git push origin master
```

### The five jobs

In **every** job: Source Code Management = Git, URL `https://github.com/esaikaviya1612/Calculator.git`, branch `*/master`, Maven Version `Maven-3.10.0`.

| Job | Maven goals | Post-build actions |
|---|---|---|
| **Checkout** | (no build step) | Build other projects: `Compile` (trigger only if build is stable) |
| **Compile** | `clean compile` | Build other projects: `UnitTest` |
| **UnitTest** | `test` | Publish JUnit test result report: `**/target/surefire-reports/*.xml`; Build other projects: `Coverage` |
| **Coverage** | `clean test` | Record JaCoCo coverage report; Build other projects: `StaticAnalysis` |
| **StaticAnalysis** | (see below) | – |

**Coverage – JaCoCo settings:**
- Path to exec files: `**/target/jacoco.exec`
- Path to class directories: `**/target/classes`
- Path to source directories: `**/src`
- **Type `0` in ALL threshold boxes, including the last (delta) row.** An empty box gives `NumberFormatException: empty String` and the build fails.

**StaticAnalysis – goals (one line):**

```
clean test org.sonarsource.scanner.maven:sonar-maven-plugin:5.0.0.4389:sonar -Dsonar.projectKey=Calculator -Dsonar.projectName=Calculator -Dsonar.host.url=http://localhost:9000 -Dsonar.token=YOUR_TOKEN -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
```

Run **Checkout → Build Now** and check that each job starts the next one.

---

## Exercise 10 – Copying and moving jobs

**Aim:** copy and move jobs in Jenkins.

**Copy:**
1. New Item → name `calculator-CI-copy`
2. Select **Duplicate an existing item** → type `calculator-CI`
3. OK → Save

**Move:**
1. New Item → name `MyFolder` → **Folder** → OK → Save
2. Open `calculator-CI-copy` → **Move** (left menu)
3. Select `Jenkins » MyFolder` → Move
4. URL becomes `http://localhost:8080/job/MyFolder/job/calculator-CI-copy/`

---

## Exercise 11 – Creating pipeline view in Jenkins

**Aim:** show the job chain as a visual pipeline.

1. Dashboard → click **+** next to the **All** tab
2. View name: `Calculator-Pipeline` → **Build Pipeline View** → Create
3. Upstream / downstream config → Select Initial Job: `Checkout` → OK
4. Click **Run** and watch: Checkout → Compile → UnitTest → Coverage

(Needs the **Build Pipeline** plugin from Exercise 8 and the Exercise 9 jobs.)

---

## Exercise 12 – Configuring gating conditions

**Aim:** the pipeline moves to the next job **only** when a quality condition is met. The gate is **Build other projects → Trigger only if build is stable**.

### Gate A – Unit testing job

1. UnitTest → Configure → Publish JUnit test result report → **Advanced** → Health report amplification factor = `1.0` → Save
2. Run the pipeline (all green).
3. Break one test in `CalculatorTest.java`, for example `assertEquals(16, calc.add(10, 5));`
4. Push:

```
git add .
git commit -m "Break test on purpose"
git push origin master
```

5. Run the pipeline → UnitTest fails, Coverage does **not** start.
6. Restore the assertion (`15`), push again, pipeline is green.

### Gate B – Code coverage job

1. Coverage → Configure → Record JaCoCo coverage report → tick **Change build status according to the defined thresholds**
2. Smooth build: storm-cloud row = `0` → build stays green.
3. Broken build: set **Instruction** and **Line** in the storm-cloud row to `80` → build becomes **unstable**, report shows red values.
4. Reset the thresholds to `0` and run again.

### Gate C – Static analysis job (SonarQube quality gate)

1. SonarQube → **Quality gates** → copy **Sonar way** as `CALQG` → **Add Condition**, for example:
   - Issues is greater than `0`
   - Coverage is less than `80`
   - Duplicated Lines (%) is greater than `3`
   - Security Hotspots Reviewed is less than `100%`
2. Project `Calculator` → set the quality gate to `CALQG` (Projects → move into the **With** list).
3. Build breaker – add this at the end of the StaticAnalysis goals:

```
-Dsonar.qualitygate.wait=true
```

4. Strict conditions → gate fails → Jenkins build fails.
5. Relax the conditions (Issues greater than `10`, Coverage less than `10`) → gate passes → build succeeds.

---

## Quick fixes I met during the practical

| Problem | Fix |
|---|---|
| `BUILD FAILURE ... sonar.organization` | Command was cut at `\`; use one line with `-Dsonar.host.url=http://localhost:9000` |
| SonarQube won't start: "paging file is too small" | System Properties (`sysdm.cpl`) → Advanced → Performance → Settings → Advanced → Virtual memory → custom size on a drive with free space (for example D: 4096 to 8192 MB), restart |
| Jenkins Built-In Node offline: "Disk space is below threshold of 1.00 GiB" | Free space on C: (Settings → System → Storage → Temporary files), then Manage Jenkins → Nodes → Built-In Node → Bring this node back online |
| JaCoCo: `NumberFormatException: empty String` | Put `0` in every threshold box, including the delta row |
| Forgot Jenkins password | Home folder `C:\ProgramData\Jenkins\.jenkins` → back up `config.xml` → set `<useSecurity>false</useSecurity>` → restart service → set new user and password → turn security back on |

---

## Additional exercises

To be added after doing them: adding custom rules to SonarQube, static analysis using SonarLint, binding SonarQube rules to SonarLint.

Sure bro! Here is **Additional Exercise 1** from start to finish, with every change and step, ready for your record. 💪

# Additional Exercise 1: Adding custom rules to SonarQube

**Aim:** To create our own custom rule in SonarQube from a rule template, activate it in a quality profile, and check that it finds issues in the code.

## Part 1: Create the custom rule
1. Start SonarQube (`StartSonar.bat` in `D:\sonarqube-26.9.0.129388\bin\windows-x86-64`) and wait for **SonarQube is operational**.
2. Open `http://localhost:9000` and log in as admin.
3. Click **Rules** in the top menu.
4. In the left panel, set **Language = Java**.
5. Open the **Template** filter and click **Show Templates Only**. The list shows **"Track uses of disallowed classes"** (Java, Rule Template, Maintainability, Info). 📸 *Screenshot 1*
6. Click that template. Scroll to **Custom Rules** and click **Create**.
7. Fill in the form:
   - **Name:** `Avoid java.util.Date`
   - **Key:** `NoJavaUtilDate` (the rule ID becomes `java:NoJavaUtilDate`)
   - **Description:** `Do not use java.util.Date, use java.time classes instead.`
   - **className:** `java.util.Date`
   - Severity and status: the defaults (it shows Medium, Maintainability).
8. Click **Create**. The rule appears under **Custom Rules**. 📸 *Screenshot 2*

## Part 2: Activate the rule in a quality profile
*(The built-in "Sonar way" profile can't be edited, so we copy it.)*

9. Click **Quality profiles** in the top menu.
10. On the **Java → Sonar way** row, click **⋮ → Copy**.
11. Name it `Calculator Profile` and click **Copy**.
12. On the `Calculator Profile` page, click **Activate More**.
13. Search `Avoid java.util.Date`, click **Activate**, then confirm.
14. Open the rule page again. Under **Quality profiles** it now lists **Calculator Profile**. 📸 *Screenshot 3*

## Part 3: Apply the profile to the Calculator project
15. In `Calculator Profile`, open the **Projects** tab → **Change projects**.
16. Select **Calculator**, then close the pop-up. (The project now uses this profile.)

## Part 4: Change the code so the rule has something to find
17. In Eclipse, open `Calculator.java` (`src/main/java/com/esaikaviya`).
18. **Change 1:** add this import **below** the `package com.esaikaviya;` line:
    ```java
    import java.util.Date;
    ```
19. **Change 2:** add this method **inside** the class, above `main`:
    ```java
    public Date getCurrentDate() {
        return new Date();
    }
    ```
20. Save with **Ctrl + S** (no red errors).

## Part 5: Run the analysis
21. Open CMD in the project folder and run (**one line**):
    ```
    cd C:\Users\acer\eclipse-workspace-devops\Calculator
    mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=Calculator -Dsonar.projectName=Calculator -Dsonar.host.url=http://localhost:9000 -Dsonar.token=YOUR_TOKEN
    ```
22. Expected: `ANALYSIS SUCCESSFUL` and `BUILD SUCCESS`. 📸 *Screenshot 4 (hide the token)*

## Part 6: Check the result
23. Open the rule page **Avoid java.util.Date**. At the bottom it shows **Issues (3)**.
24. Click **Issues (3)** to see the issues in `Calculator.java`. 📸 *Screenshot 5* (the one still pending)

**Result:** A custom rule **Avoid java.util.Date** was created from the template "Track uses of disallowed classes", activated in `Calculator Profile`, and the analysis reported **3 issues** where `java.util.Date` is used.

**Two small notes:**
- Keep the `Date` code until you've taken the last screenshot. If you remove it and run the analysis again, the issues disappear.
- Later, remove the extra method and import to keep your code clean.

Send me the **Issues (3)** screenshot, and then we'll start **Additional Exercise 2: SonarLint**. I can also add this exercise to your `.md` cheat sheet if you want. 🚀
