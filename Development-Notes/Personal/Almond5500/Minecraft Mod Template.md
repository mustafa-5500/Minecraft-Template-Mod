## Overview
**Motivation:** A lot of our Minecraft Plugin Ideas can also be written as Client side mods, for example the Git plugin and Lore linking Plugin. Furthermore integrating ROS nodes into the client side for Minecraft will also follow the Minecraft Mod structure, therefore it will be helpful to have a template set up with all the DevOps tools required for making a Client Side Mod.

**Idea:** Have a template repository with the Gradle tools set up for Minecraft Fabric Mods, later on we can also create separate branches for different Minecraft Mod Loaders.

## DevOps
**DevOps Tools:** There is a breif [DevOps Guide](DevOps.md) describing the main gradle calls set up, and custom scripts in the `Utils/` submodule.

### Gradle
**Description:** Gradle is the main packaging and compiling tool for the project. 

#### `build.gradle`
**Description:** The main files describing the dependencies and Gradle tasks.

**Plugins:** Gradle plugins to add more tasks, the fabric loom plugin adds developer launching the mod. The maven publish plugin allows task to publish to maven repositories.
```gradle
plugins {
    // The -remap variant is used for obfuscated Minecraft versions such as 1.21.x.
    // Use 'net.fabricmc.fabric-loom' for the unobfuscated 26.x versions.
    id 'net.fabricmc.fabric-loom-remap' version "${loom_version}"
    id 'maven-publish'
}
```

**Mod Properties:** The `project` prefix means the values are coming from the `gradle.properties` file. The `base` block uses the Gradle default base plugin which defines values for the entire project, in this case we are defining the name for the project out files.
```gradle
version = project.mod_version
group = project.maven_group

base {
    archivesName = project.archives_base_name
}
```

**Dependencies:** If we have any other repositories we depend upon.
```gradle
// Loom adds the Minecraft, Mojang and Fabric repositories for you. Add extra
// repositories here only when you depend on other mods or libraries.
repositories {
    // maven {
    //     name = 'Example Maven'
    //     url = 'https://example.com/maven'
    // }
}
```

### Github Actions
```yml
# Build the mod on every push and pull request. Pushing a v* tag also creates a
# GitHub Release with the built jars attached.
name: build

# When to do the actions, pushes on branchs or tags, as well as pull requests
on:
  push:
    branches:
      - main
      - master
      - develop
    tags:
      - 'v*'
  pull_request:

# Give the action write permissions 
permissions:
  contents: write

# The Jobs done by the action
jobs:
  # Build on the ubuntu machine, with 30 minute limit
  build:
    runs-on: ubuntu-24.04
    timeout-minutes: 30

	# The Job Steps
    steps:
      - name: Checkout repository
        uses: actions/checkout@v6

      - name: Validate Gradle wrapper
        uses: gradle/actions/wrapper-validation@v6

      - name: Set up JDK 25
        uses: actions/setup-java@v5
        with:
          java-version: '25'
          distribution: 'temurin'
          cache: 'gradle'

      - name: Make Gradle wrapper executable
        run: chmod +x ./gradlew

      - name: Build
        run: ./gradlew build --stacktrace

      - name: Upload build artifacts
        uses: actions/upload-artifact@v7
        with:
          name: mod-jars
          path: build/libs/

      - name: Create GitHub Release
        if: startsWith(github.ref, 'refs/tags/v')
        env:
          GH_TOKEN: ${{ github.token }}
        run: gh release create "$GITHUB_REF_NAME" build/libs/*.jar --generate-notes

```
