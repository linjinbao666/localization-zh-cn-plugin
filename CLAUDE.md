# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Jenkins localization plugin** (`localization-zh-cn`) that provides Simplified Chinese translations for Jenkins core and popular Jenkins plugins. It packages as an HPI (Hudson Plugin Archive) using the Jenkins plugin parent POM.

## Build and Test Commands

```bash
# Build and install (skip tests)
mvn clean install -DskipTests

# Package the HPI
mvn clean package -DskipTests

# Run tests
mvn test

# Full build via Makefile
make install    # mvn clean install -DskipTests
make package    # mvn clean package -DskipTests
make clean      # rm -rf target
```

The Jenkinsfile tests on `linux+jdk21` and `windows+jdk17` using `buildPlugin()` from the Jenkins pipeline library.

## Architecture

### Directory Structure

```
.
├── pom.xml                          # Maven build, parent: org.jenkins-ci.plugins:plugin:4.86
├── src/
│   ├── main/java/io/jenkins/plugins/localization_zh_cn/
│   │   ├── LocalizationContributorImpl.java  # Core: loads translations via localization-support library
│   │   ├── CommunityPage.java                # UnprotectedRootAction at /chinese
│   │   ├── CommunityDecorator.java           # PageDecorator, conditionally shows Chinese community link
│   │   ├── UserCommunityProperty.java        # UserProperty for community link visibility (Chinese/Always/Never)
│   │   └── UpdateCenterAction.java           # RootAction for managing update center mirror certificate
│   ├── main/resources/
│   │   ├── index.jelly                       # UI for the Chinese community page
│   │   ├── io/                               # Plugin's own localized UI strings
│   │   └── mirror-adapter.crt                # Update center mirror CA certificate
│   └── test/java/.../CommunityPageTest.java  # JenkinsRule-based integration tests
├── core/src/main/resources/                  # Jenkins core translations (353 files)
│   ├── hudson/                               # hudson.* package translations
│   ├── jenkins/                              # jenkins.* package translations
│   └── lib/                                  # Jelly library translations
├── core/src/main/webapp/                     # Core webapp translations (help/)
└── plugins/                                  # 40 plugin localization directories
    ├── git-plugin/src/main/resources/
    ├── kubernetes-plugin/src/main/resources/
    ├── credentials-plugin/src/main/resources/
    └── ... (40 total)
```

### How Localization Works

1. **`LocalizationContributorImpl`** extends `io.jenkins.plugins.localization.support.LocalizationContributor` - this is the key class that registers translations with the Jenkins localization framework. It resolves resource paths via the classloader and handles `webapp/` prefixed paths for web resources.

2. **Translation files** follow the standard Java properties pattern with `*_zh_CN.properties` suffix. They mirror the package structure of the source they translate (e.g., `hudson/model/Messages_zh_CN.properties` translates `hudson.model.Messages`).

3. **All Chinese characters are native2ascii-encoded** (converted to `\uXXXX` ASCII). Use [native2ascii.net](https://native2ascii.net/) to convert.

4. **The `pom.xml`** declares each plugin's `src/main/resources` directory as a Maven resource, so all translations get bundled into the final HPI.

### Key Files

- **`LocalizationContributorImpl.java`**: The bridge between bundled resources and Jenkins' localization system. `getResource()` loads core/plugin translations; `getPluginResource()` loads webapp resources.
- **`CommunityDecorator.java`**: Detects Chinese locale via `Accept-Language` header; controls visibility of community links on pages.
- **`UpdateCenterAction.java`**: Manages installing/removing a mirror CA certificate for the Chinese update center mirror.

## Contributing Translation Files

1. Each plugin's translations live in `plugins/<plugin-name>/src/main/resources/`, mirroring the target plugin's package structure.
2. Core Jenkins translations live in `core/src/main/resources/`, mirroring `hudson.*` and `jenkins.*` packages.
3. All `.properties` values containing Chinese characters must be native2ascii-encoded.
4. Follow the [Jenkins Translation Specification](https://github.com/jenkins-zh/translation-spec/blob/master/specification.md).
5. PRs should use a dedicated branch (not `master`).

## Branch Strategy

- `master`: Main branch, tracks upstream jenkinsci
- `dev-cncec`: Current working branch (currently at parity with master)

## Dependencies

- **Parent POM**: `org.jenkins-ci.plugins:plugin:4.86`
- **Jenkins core version**: `2.387.3`
- **Localization support library**: `io.jenkins.plugins:localization-support:1.2`
- Java requirement: JDK 17+ (JDK 21 for Linux CI)
