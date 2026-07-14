# ⚠️ Deprecation Notice: Moved to bcgov/actions/test-and-analyse

This repository (**bcgov/action-test-and-analyse-java**) has been **deprecated** and is no longer maintained. 

Java support has been integrated into the universal, multi-language action **[bcgov/actions/test-and-analyse](https://github.com/bcgov/actions/tree/main/test-and-analyse)**. We are using a single, unified action for testing and analysis across all projects, rather than maintaining a Java-specific utility.

## Why the Change?
The universal `test-and-analyse` action uses conditional runtime isolation:
- Runtimes (Node, Java, Python) are loaded dynamically.
- Java projects only run Java-related steps and dependencies, with no Node.js or Python overhead.
- Maintaining one action reduces maintenance overhead and centralizes security/compliance improvements.

---

## Migration Guide & Examples

To migrate, update your workflow files to point to `bcgov/actions/test-and-analyse@vX.Y.Z` and specify `language: java`.

### Example: Java (Maven)

```yaml
- name: Test and Analyze (Java Maven)
  uses: bcgov/actions/test-and-analyse@vX.Y.Z # Replace with the latest release version
  with:
    language: java
    java_version: "21" # Or your required version (e.g. "17")
    java_distribution: temurin # Default, supports corretto, openjdk, zulu, etc.
    commands: |
      mvn -B verify sonar:sonar \
        -Dsonar.organization=bcgov-sonarcloud \
        -Dsonar.projectKey=bcgov_your-project-key
    dir: backend
    sonar_token: ${{ secrets.SONAR_TOKEN }}
    triggers: ('backend/' 'pom.xml')
```

### Example: Java (Gradle)

```yaml
- name: Test and Analyze (Java Gradle)
  uses: bcgov/actions/test-and-analyse@vX.Y.Z # Replace with the latest release version
  with:
    language: java
    java_version: "21"
    java_distribution: temurin
    cache: gradle
    commands: |
      ./gradlew build sonarqube --info \
        -Dsonar.organization=bcgov-sonarcloud \
        -Dsonar.projectKey=bcgov_your-project-key
    dir: backend
    sonar_token: ${{ secrets.SONAR_TOKEN }}
    triggers: ('backend/' 'build.gradle' 'settings.gradle')
```

For more options and configuration details, please refer to the main **[bcgov/actions/test-and-analyse Documentation](https://github.com/bcgov/actions/tree/main/test-and-analyse)**.
