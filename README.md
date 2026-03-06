# Dynamis Parent

Parent POM for the **Dynamis Engine ecosystem**.

This project provides the shared Maven configuration used by all Dynamis libraries, ensuring consistent builds, dependency management, signing, and publishing across the ecosystem.

It is published to Maven Central as:

```
org.dynamisengine:dynamis-parent
```

---

## Purpose

`dynamis-parent` centralizes common build configuration for the Dynamis platform.

It provides:

- Java **25** build configuration
- consistent **plugin versions**
- **GPG signing** for Maven Central releases
- **source and javadoc** artifact generation
- **Sonatype Central publishing configuration**
- shared build conventions across all Dynamis modules

Using a shared parent keeps every project aligned and avoids repeating build configuration in each repository.

---

## Ecosystem

The Dynamis platform is composed of multiple libraries built on this parent:

```
org.dynamisengine
├── vectrix            (high-performance math / SIMD primitives)
├── meshforge          (mesh processing and geometry pipelines)
├── dynamis-physics    (physics engine integration layer)
├── dynamis-light-engine (rendering engine)
├── dynamis-fx         (JavaFX / rendering integration)
└── dynamis-ai         (AI and agent systems)
```

All of these inherit their Maven configuration from `dynamis-parent`.

---

## Using the Parent

Add the parent to your `pom.xml`:

```xml
<parent>
    <groupId>org.dynamisengine</groupId>
    <artifactId>dynamis-parent</artifactId>
    <version>1.0.0</version>
</parent>
```

Then define your artifact:

```xml
<artifactId>vectrix</artifactId>
```

This automatically applies:

* compiler configuration
* plugin versions
* artifact signing
* release configuration

---

## Building

Local build:

```bash
mvn clean install
```

---

## Publishing

Publishing to Maven Central:

```bash
mvn clean deploy -Prelease
```

This uses the **Sonatype Central Publishing Portal** and requires valid credentials in:

```
~/.m2/settings.xml
```

---

## Requirements

* Java **25**
* Maven **3.9+**
* GPG signing key configured
* Sonatype Central account with namespace `org.dynamisengine`

---

## License

Apache License 2.0

---

## Author

Larry Mitchell
