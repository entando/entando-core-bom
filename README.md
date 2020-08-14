# entando-core-bom
This project is an application of the [Maven Bill Of Materials pattern](https://dzone.com/articles/the-bill-of-materials-in-maven)
The goal of this project is to list all of the Maven library projects that could potentially be used in Entando App
Maven projects such as [entando-de-app](https://github.com/entando-k8s/entando-de-app). In addition, this project
can also be used in other internal Maven projects to manage upstream dependencies such as entando-engine and admin-console.

The pom.xml file of this project gets updated automatically from the individual pipelines. It will therefore always contain
the latest version of each library that has successfully completed all the quality gates in the pipeline.

# How do I use this project?
Step 1. In the dependencyManagement section of your Maven POM, insert the latest version of this dependency, but with "import"
scope and "pom" type:
```
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.entando</groupId>
                <artifactId>entando-core-bom</artifactId>
                <version>6.2.3</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
Step 2. In the dependencies section of your Maven POM, reference your required dependency to an internal library 
without specifying the version.
```
    <dependencies>
        <dependency>
            <groupId>org.entando.entando</groupId>
            <artifactId>entando-admin-console</artifactId>
            <classifier>classes</classifier>
        </dependency>
        <dependency>
            <groupId>org.entando.entando</groupId>
            <artifactId>entando-engine</artifactId>
            <type>test-jar</type>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.entando.entando</groupId>
            <artifactId>entando-admin-console</artifactId>
            <type>test-jar</type>
            <scope>test</scope>
        </dependency>
```
Notes: 
* there is no need to import entando-engine as it comes in as a transitive dependency of `entando-admin-console:classes`
* irrespective of the version of entando-engine imported transitvely by `entando-admin-console:classes`, the version 
     specified in the BOM project will override it because it controls it from the dependencyManagement section 
* test dependencies are not brought in transitively. Import them explicitly in your dependencies section if you need 
     the test dependencies

# I want to make changes to an upstream project and test those changes from my current project. How do I override the dependencies from my project pom to use the locally built version of the upstream project?
Step 1: verify that the version of your upstream project pom is a SNAPSHOT, e.g.: `<version>6.2.0-SNAPSHOT</version>`. This is the default, so ideally no changes required here
Step 2: in the dependencyManagment section of your current project, AFTER the element importing this BOM, add an explicit reference
to the local SNAPSHOT version of your upstream project. In this example we bring in the entando-admin-console:classes dependency:
```
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.entando</groupId>
                <artifactId>entando-core-bom</artifactId>
                <version>6.2.3</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>
            <dependency>
                <groupId>org.entando</groupId>
                <artifactId>entando-admin-console</artifactId>
                <version>6.2.0-SNAPSHOT</version>
                <classifier>classes</classifier>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
Step 3: install the upstream project from the command line using the local-dev profile: 
`mvn clean install -Plocal-dev`
This profile includes all the other build artifacts (war, test-jar, classes-jar) but excludes tests and signatures

Step 4. Refresh the dependencies in your IDE