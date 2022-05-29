## Strategy Pattern with the Spring Framework

The [Spring Framework](https://spring.io/) (and its extension - Spring Boot) allows us to write Java code quicker, easier, and safer. It's no wonder, then, that through its [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) features it can make our lives simpler when implementing well-known _design patterns_ like the [Strategy Pattern](https://refactoring.guru/design-patterns/strategy).

There are multiple ways we could implement this with Spring - in this article I aim to showcase the simplest and most effective way I believe can be applied in the majority of use cases.

Notes: this article uses code written in Java 17, but equivalent solutions can be achieved with earlier versions.

---

### Using DI in the Strategy Pattern

#### Context

Say we need to write a service that can export data in multiple file formats: JSON, CSV, XML; the format comes as input from the user.

We can quickly create the basic components for this task:

- an `ExportFormat` enum containing the supported formats
- an `ExportFile` record class representing the file
- an `Exporter` interface as a blueprint for our services

```Java
public enum ExportFormat {
  JSON,
  CSV,
  XML
}

public record ExportFile(String name, byte[] content) {}

public interface Exporter {
  ExportFormat getType();

  ExportFile export(UUID id);
}

public class JsonExporter implements Exporter {

  @Override
  public ExportFormat getType() {
    return ExportFormat.JSON;
  }

  // other code omitted...
}
public class CsvExporter implements Exporter {}
public class XmlExporter implements Exporter {}
```

#### The Traditional Way

The most basic way to implement the Strategy Pattern would be to use another creational pattern to create our strategies - the [Factory Pattern](https://refactoring.guru/design-patterns/abstract-factory):

```Java
public class ExporterFactory {

  public Exporter create(final ExportFormat format) {
    return switch(format) {
      case JSON -> new JsonExporter();
      case CSV -> new CsvExporter();
      case XML -> new XmlExporter();
    };
  }
}
```

This approach works in many simple implementations, but quickly becomes less ideal when our services require other dependencies -> because we are instantiating our exporters here, we'd have to inject dependencies into the `ExporterFactory` and pass them to our exporters. Whenever we add or remove dependencies, we'd have to make changes to both the factory and our exporters (and associated test code).

#### The Spring DI Way

Instead of instantiating our exporters manually, we can annotate them with [@Component](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Component.html) or [@Service](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Service.html), which allows us to inject them in other classes.

Because our services extend the `Exporter` interface, Spring can provide all of them in the form of a `List` or `Set`. All we need to do is create a _resolver_ instead of a factory to use the correct exporter. This class will retrieve the right exporter from the `Set` based on the given format:

```Java
@Service
public class ExporterResolver {
  private final Set<Exporter> exporters;

  public ExporterResolver(final Set<Exporter> exporters) {
    this.exporters = exporters;
  }

  public Exporter create(final ExportFormat format) {
    return exporters.stream()
        .filter(e -> e.getType() == format)
        .findFirst()
        .orElseThrow(IllegalArgumentException::new);
  }
}
```

#### Putting Things Together

Lastly, our service will use the resolver to perform the task at hand:

```Java
@Service
public class ExportService {

  private final ExporterResolver resolver;

  public ExportService(final ExporterResolver resolver) {
    this.resolver = resolver;
  }

  public ExportFile export(final ExportFormat format, final UUID id) {
    return resolver.create(format).export(id);
  }
}
```

#### Happy Strategizing!
