# Local Development Setup

## Prerequisites

- Java Development Kit (JDK) 17
- Apache Maven
- Git

## Verify the Environment

```bash
java -version
javac -version
mvn -version
git --version
```

Ensure the Java version used by Maven is compatible with the project's Java 17 configuration.

## Build and Test

From the project root, where `pom.xml` is located:

```bash
mvn clean test
mvn clean package
```

The packaged application is generated in the `target/` directory.

## Development Notes

- Keep source code under `src/main/java`.
- Keep automated tests under `src/test/java`.
- Do not commit generated build output or local logs.
- Review `git status` before committing changes.