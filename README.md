# devopsexam
Okay bro, here are **all the steps** for Exercises 1 to 10, in simple words. 💪 Your record uses someone else's values (the `Parkbong7` repo, port `8081`, JDK-25, Maven 3.9.16), so I've used **your own** setup: `esaikaviya1612`, port `8080`, JDK 21, Maven 3.10.0. Don't copy their screenshots; use yours.

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

Want me to put this into a **Word or PDF file** so you can copy it into your record? And did Coverage turn green after adding the 0s?
