# Synthetic Test Support

This library provides a programmatic way to manage Datadog synthetic browser tests. This library contains all the necessary functions that are required to manage the synthetic browser tests with different steps and configurations.

## Dependencies

Synthetic Test Support library builds up on top of the Datadog's API [Java client library](https://github.com/DataDog/datadog-api-client-java/) and provides Kotlin implementation of functions to create and edit synthetic browser tests.

There are 2 ways to provide the API credentials in order to communicate with Datadog API:

* fetching from the environment variables: DD_API_KEY and DD_APP_KEY
* fetching from [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)

## Configuration

There are some variables like Datadog API host, library and default test settings that need to be configured in order to have the Synthetic Test library up and running. Follow the steps below to setup your configuration:

- Copy the file `configuration.yaml.sample` into the `resources` folder in your source set.
- Rename the file to `configuration.yaml`.
- Update the variables in the file with your own specifications or specify the environment variables (and optionally default values to use when variables are not set using `${<VAR_NAME>:-<default-value>}` [syntax](https://commons.apache.org/proper/commons-text/apidocs/org/apache/commons/text/StringSubstitutor.html#:~:text=Providing%20Default%20Values)). If the datadogCredentialsAwsArn is set in the configuration, the library will default to using the AWS Secrets Manager to fetch Datadog API credentials, alternatively ddApiKey and ddAppKey need to be provided in the configuration.

### Configuring credentials provider

You can use AWS Secrets Manager to store and fetch Datadog API credentials to enable the library to publish the test changes to Datadog.

You need to include an additional AWS package dependency, depending on how you login to AWS:
- if you use [AWS SSO](https://docs.aws.amazon.com/sdkref/latest/guide/access-sso.html), please add [`software.amazon.awssdk:sso`](https://mvnrepository.com/artifact/software.amazon.awssdk/sso) dependency
- if you use [AWS STS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html), please add [`software.amazon.awssdk:sts`](https://mvnrepository.com/artifact/software.amazon.awssdk/sts) dependency

Please check the currently supported AWS SDK version in [`build.gradle.kts`](build.gradle.kts).

## Getting started

This library is compiled with JDK 17. Your project should be on JDK 17+ in order to use it. 

Start using the library in a gradle project by following the steps below:

1. In the gradle project, add a new source set and Gradle task to run test generation in `build.gradle.kts` file.
```kotlin
    sourceSets {
        create("syntheticTest")
    }
    
    task("publishSyntheticTests", JavaExec::class) {
        group = "syntheticTest"
        mainClass.set("<yourPackageName>.<kotlinFileWithMainClass>")
        classpath = sourceSets["syntheticTest"].runtimeClasspath
    }

    val syntheticTestImplementation: Configuration by configurations
```

2. Add dependencies:
```kotlin
    dependencies {
        // check out https://github.com/personio/datadog-synthetic-test-support/releases to get the latest version
        syntheticTestImplementation("com.personio:datadog-synthetic-test-support:x.x.x")
        // check out https://github.com/JetBrains/kotlin/releases to get the latest version
        syntheticTestImplementation("org.jetbrains.kotlin:kotlin-stdlib:x.x.x")
    }
```

3. Setup configuration as described in the [Configuration](https://github.com/personio/datadog-synthetic-test-support#configuration) section.

4. Implement your synthetic browser tests, for example like this:
```kotlin
    fun `add a synthetic test`() {
       syntheticBrowserTest("Test Login to the app") {
          tags(listOf("env:qa"))
          baseUrl(URL("https://synthetic-test.personio.de"))
          browserAndDevice(SyntheticsDeviceID.CHROME_LAPTOP_LARGE)
          publicLocation(Location.FRANKFURT_AWS)
          testFrequency(5.minutes)
          retry(1, 600.milliseconds)
          useGlobalVariable("PASSWORD")
          inputTextStep(
             stepName = "Enter username",
             targetElement = TargetElement("[name='email']"),
             text = "test@personio.de"
          )
          inputTextStep(
             stepName = "Enter password",
             targetElement = TargetElement("[name='password']"),
             text = "{{ PASSWORD }}"
          )
          clickStep(
             stepName = "Click login button",
             targetElement = TargetElement("[name='login']")
          ) {
             waitBeforeDeclaringStepAsFailed(30.seconds)
          }
          pageContainsTextAssertion(
             stepName = "Check Welcome title",
             expectedText = "Welcome"
          )
       }
    }
```
   
5. Create a new kotlin file with the `main` function and call the function created in Step 3 to publish the tests, for example like this:
```kotlin
    fun main() {
        `add a synthetic test`()
    }
```

6. To publish the browser tests to Datadog, use the following command:
```bash
   ./gradlew publishSyntheticTests`
```

Note:
- Test name is a unique identifier which is used during test publishing. If a test with the given name already exists in Datadog, it will be overwritten.

## Build the library

The following are some useful commands for working with the project.

- Run `./gradlew build` to build the whole project.
- Run `./gradlew ktlintFormat` to format the project.
- Run `./gradlew test` to execute all the unit tests.
- Run `./gradlew e2eTest` to execute the e2e test.
- Run `./gradlew publishToMavenLocal` to publish to your local .m2 repository.

## Release process

A new version will be released each time there are some crucial changes or a bunch of useful improvements.

Release process is handled by the Code owners. 

The list of releases is available on the [Releases](https://github.com/personio/datadog-synthetic-test-support/releases) page.
Each release contains the list of changes from the Changelog file corresponding to the release version and zipped repository files.

## Changelog

This repository maintains a separate log for each change. Refer to `CHANGELOG.md` for changes to the `datadog-synthetic-test-support` library. Ensure that the `CHANGELOG.md` file has been updated to include the new version's changes & additions.

This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html) and the format is partially based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) with customised sections: Breaking changes, New features & improvements, Bug fixes, Dependencies.

## How to contribute and report issues

Feel free to contribute! Just create a new issue or a new pull request following the [Contributing Guidelines](CONTRIBUTING.md).

## License

The library is licensed under the [MIT License](https://github.com/personio/datadog-synthetic-test-support/blob/master/LICENSE), copyright 2023 Personio SE & Co. KG.
