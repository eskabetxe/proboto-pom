# proboto-pom

`pro.boto:boto` is a shared parent POM for pro.boto Java projects. It centralizes build
configuration — compiler settings, code formatting, version enforcement, and Maven Central
publishing — so individual projects don't have to redeclare it. The structure and intent are
modeled after the [Apache Software Foundation parent POM](https://maven.apache.org/pom/asf/).

## Using it

Inherit from it in your project's `pom.xml`:

```xml
<parent>
    <groupId>pro.boto</groupId>
    <artifactId>boto</artifactId>
    <version>1-SNAPSHOT</version>
</parent>
```

## What it provides

### Properties

| Property | Default | Purpose |
|---|---|---|
| `project.jvm.version` | `21` | Java language level used by the compiler and enforced at build time. Override in your project to target a different JDK. |
| `project.encoding` | `UTF-8` | Source and resource file encoding. |
| `maven.deploy.skip` | `false` | Set to `true` in a module to opt it out of deployment. |

### Build plugins

| Plugin | What it does |
|---|---|
| [`maven-compiler-plugin`](https://maven.apache.org/plugins/maven-compiler-plugin/) | Compiles sources/targets/release at `project.jvm.version`. |
| [`spotless-maven-plugin`](https://github.com/diffplug/spotless) | Enforces code style with [palantir-java-format](https://github.com/palantir/palantir-java-format), import ordering, trailing-whitespace and newline rules. Runs on `validate` and fails the build on violations. |
| [`maven-enforcer-plugin`](https://maven.apache.org/enforcer/maven-enforcer-plugin/) | Fails the build early if Maven is older than 3.9, the JDK doesn't match `project.jvm.version`, or a dependency version is duplicated across the POM. |
| [`maven-deploy-plugin`](https://maven.apache.org/plugins/maven-deploy-plugin/) | Deploys artifacts; gated by `maven.deploy.skip`. |
| [`central-publishing-maven-plugin`](https://central.sonatype.org/publish/publish-portal-maven/) | Publishes releases to Maven Central via the Sonatype Central Portal API. |

### Publishing to Maven Central

Publishing releases and snapshots works differently, matching how the Central Portal itself
handles them:

- **Snapshots** are deployed the traditional way — `mvn deploy` pushes to the
  `snapshotRepository` declared in `distributionManagement`
  (`https://central.sonatype.com/repository/maven-snapshots/`). No validation is performed and
  snapshots expire after 90 days.
- **Releases** go through the Central Portal Publisher API via `central-publishing-maven-plugin`,
  which validates and bundles the artifacts before publishing. Because releases are validated as
  a signed, immutable bundle, they must include sources, javadoc, and a GPG signature — Central
  rejects releases missing any of these.

To cut a release, activate the `release` profile, which attaches the required artifacts:

```xml
<profile>
    <id>release</id>
    <!-- maven-source-plugin, maven-javadoc-plugin, maven-gpg-plugin -->
</profile>
```

```bash
mvn clean deploy -Prelease
```

Prerequisites:

1. A GPG key pair (`gpg --gen-key`), with the public key published to a keyserver
   (e.g. `keys.openpgp.org`), so Central can verify signatures.
2. A `<server id="central">` entry in `~/.m2/settings.xml` with your Central Portal token, and
   your GPG passphrase available via `settings.xml` or an environment variable — never committed
   to the POM.

### Requirements

- Maven 3.9+
- JDK 21+ (enforced via `project.jvm.version`)

## License

Licensed under the [Apache License, Version 2.0](LICENSE).
