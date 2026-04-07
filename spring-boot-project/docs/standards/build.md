# Project Build & Maintenance Guide
This document defines the standard operations for building, testing, and packaging this Spring Boot application. Use these commands via the run_terminal_command tool.

## 1. Core Development Commands
Use these during active coding and debugging.

- Clean and Build:

```Bash
./gradlew clean build -x test
```
Note: Use -x test to skip tests for rapid prototyping, but ensure tests pass before any PR.

- Run Application:

```Bash
./gradlew bootRun
```

- Check Dependencies: Useful for resolving version conflicts.

```Bash
./gradlew dependencies
```

## 2. Testing Suites
In our DDD structure, we distinguish between unit and integration tests.

- Run All Tests:

```Bash
./gradlew test
```
- Run Domain Unit Tests Only:

```Bash
./gradlew test --tests "com.example.project.domain.*"
```

- Run Integration Tests (Testcontainers):
Requires Docker to be running.

```Bash
./gradlew test --tests "com.example.project.infrastructure.*"
```

- Architecture Guard (ArchUnit):

```Bash
./gradlew test --tests "com.example.project.architecture.*"
```

## 3. Packaging & Artifacts
Standard Spring Boot executable JAR generation.

- Build Executable JAR:

```Bash
./gradlew bootJar
```
Output location: build/libs/[project-name]-[version].jar

- Generate Static Analysis Report: (If Checkstyle/Jacoco is enabled)

```Bash
./gradlew check
```

## 4. Troubleshooting & Cache
Use these if you encounter "Unsupported class file major version 62" or IDE sync issues.

- Refresh Dependencies: Force download from Maven Central.

```Bash
./gradlew build --refresh-dependencies
```

- Stop Gradle Daemon: Reset the build environment.

```Bash
./gradlew --stop
```

## 5. Instructions for AI (Claude Code)

When performing tasks, follow these operational rules:
1. Before Refactoring: Run ./gradlew test to establish a baseline.
2. After Adding Entities: Run ./gradlew classes to ensure MapStruct/Lombok processors generate necessary code.
3. Dependency Changes: After modifying build.gradle, always run ./gradlew help (or any simple task) to trigger a synchronization check.
4. Error Handling: If a build fails due to Java version, verify JAVA_HOME points to JDK 18.