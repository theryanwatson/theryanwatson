# Suggested Java Coding Conventions

## Common Style Guides

I suggest following widely-used Java conventions, like [Google's Java Style Guide](https://google.github.io/styleguide/javaguide.html), which expands on Sun/Oracle conventions and reflects best practices.

In addition to using a common style, following common coding principles is also suggested:

1. "Don't Repeat Yourself" ([DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself))
2. "Keep It Short and Simple" ([KISS](https://en.wikipedia.org/wiki/KISS_principle))
3. "Single Responsibility" ([SRP](https://en.wikipedia.org/wiki/Single-responsibility_principle), [Curly's Law](https://blog.codinghorror.com/curlys-law-do-one-thing/))
4. "You Aren't Going To Need It" ([YAGNI](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it))
5. "Unified Language" ([Java for Everything](https://www.teamten.com/lawrence/writings/java-for-everything.html))

## Naming Guide

In addition to the [Goggle Java Naming Guide](https://google.github.io/styleguide/javaguide.html#s5-naming) naming suggestions:

1. Prefer clearer class/field names over abbreviated names
    1. ✅ `Configuration`
    2. ❌ `Conf`, `Config`
2. Name classes for what they are and their subject-matter
    1. If many implementations exist, then start using more specific names
    2. ✅ `ItemController`, `ItemService`, `ItemRepository`
3. Prefer descriptive and specific class/package names
    1. ✅ `UserDetailLookupService`, `UserServiceConfiguration`
    2. ❓ `SomeTypeUtility` only if the class is follows Utility class patterns, see also [@UtilityClass](https://projectlombok.org/features/experimental/UtilityClass)
    3. ❌ `UserHelper`, `UserBeans` (technically any code is a "helper" and any Component in Spring is a "bean")
4. Name interfaces and abstract-classes for what they represent, like classes, without using [special prefixes](https://google.github.io/styleguide/javaguide.html#s5.1-identifier-names), to encourage refactoring and prevent leaking implementation (see also [class-names](https://google.github.io/styleguide/javaguide.html#s5.2.2-class-names))
    1. ✅ `MyNode`, `Searchable`, `List`, `Map`
    2. ❌ `IMyNode`, `AMyNode`, `ISearchable`, `AList`, `IMap`
5. Organize class packages by what the class/component is first, not by its subject-matter
    1. ✅ `controllers`, `models`, `services`
    2. ❌ `workbooks`, `datasources`, `grpc`
    3. If the package is very full, then subject-matter sub-packages can be used if they are really needed: `clients.grpc`, `clients.rest`
6. Name class packages plurally when they represent a collection of classes
    1. ✅ `clients`, `configurations`, `models`, `services`
    2. ❌ `client`, `config`
7. Avoid 🔵 "Smurf Naming" fields and methods to allow for potential super-class or interface
    1. ✅ `node.getId()`, `database.getName()`
    2. ❌ `node.getNodeId()`, `database.getDatabaseName()`

## Reusable Code

1. Prefer encapsulation (make package/public only when needed) to encourage behavioral tests and easier refactoring
    1. ✅ `private`
    2. ❌ `public`
3. Prefer private fields with getter methods (standard Java style)
    1. ✅ `private String myValue; public String getMyValue();`
    2. ❌ `public String myValue;`
4. Only create an interface/abstract-class when it is needed (avoid over-interfacing)
    1. Just like any other code, only add it when it's needed. Defaulting to always creating interfaces for shared/autowired classes results in unneeded code that will need to be maintained. At the point when an interface is needed, one can be added. If also following the naming conventions suggested here, adding an interface can be totally invisible to dependent classes
    2. ✅ `class MyImplementedClass`
    3. ❌ `class MyImplementedClass implements MyUsedOnceInterface`
5. Classes for fields/parameters use highest viable interface
    1. e.g. If a method is looping over a `List`, not using `.get(0)`, then have the method accept a `Collection` or `Iterable` instead. It will allow for `List`, `Set`, and any other `Collection` to be used as input without needing to wrap them in another class first
    2. Prefer `List` over `ArrayList`, prefer `Collection` over `List`, prefer `Iterable` over `Collection`
    3. This guideline does not need to apply to return values. If the method returns a `Set`/`List`/etc, that type can be returned, but if assigning the return value to a variable, still prefer the higher interface
    4. ✅ `myMethod(Collection<Integer> ids, Collection<String> names)`
    5. ❌ `myMethod(Set<Integer> ids, ArrayList<String> names)`
6. Prefer returning `Optional` over "maybe null" objects
    1. ✅ `public Optional<MyMaybeClass> getMyMaybeClass(){...}`
    2. ❌ `public MyMaybeClass getMyMaybeClass(){...}`
7. Use `@Nullable` `@NonNull` annotations for method parameters and return value (if not using `Optional`)
    1. ✅ `@Nullable Collection input, @NonNull Integer id`
    2. ❌ `List input`
8. Single shared location for dependency versions (Parent POM, BOM)

## Immutable/Unmodifiable Objects

Prefer immutable everything because it is more likely to be thread-safe and less likely to create unexpected mutation bugs

1. Final fields can not be pointed to a new object once set
    1. ✅ `final String name`
    2. ❌ `String name`
2. Java 8+ `List.of`, `Set.of`, and `Map.of` are all unmodifiable collections
    1. ✅ `List.of(x, y, x)`, `Set.of(x, y, z)`, `Map.of(k, v)`
    2. ❌ `Lists.newArrayList(x, y, x)`, `Arrays.asList(x, y, z)`
3. Java 10+ streams can be collected to unmodifiable. `Collectors.toUnmodifiableList()`, `Collectors.toUnmodifiableSet()`, and `Collectors.toUnmodifiableMap(...)`
    1. ✅ `stream.collect(Collectors.toUnmodifiableList())`
    2. ❌ `stream.collect(Collectors.toList())`
4. Lombok `@lombok.Value` creates an unmodifiable class. Adding `@Builder` can make the object easier to instantiate
    1. ✅ `MyUnmodifiableClass.builder().name(...).id(...).moreThings(...).build();`
    2. ❌ `c = new MyMutableClass(); c.setName(...); c.setId(...);`

## Thread-Safety

1. Declaring a field `final` prevents changing its pointer, but does not mean it is thread-safe. If the object is mutable/modifiable (like `ArrayList`, `HashMap`, etc.), then you may have concurrent modification issues if used in threaded code
    1. ✅ `final List<String> myList = List.of(...);`
    2. ❌ `final List<String> myList = new ArrayList<String>();`
2. Method-local variables are thread-safe. Declaring a new mutable object inside of a method means it is thread-safe as long as it is inside that method
    1. ✅ `myMethod() { List<String> localList = new ArrayList<String>(); }`
    2. ❌ `myMethod() { this.sharedList.add(...); }`
3. If mutable objects _must_ be used in a shared scope, prefer Thread-Safe implementations. `Map` → `ConcurrentHashMap`. Keep performance degradation and deadlock scenarios in mind with synchronized classes
    1. ✅ `private final Map<String, Object> sharedMap = new ConcurrentHashMap<>();`
    2. ❌ `private final Map<String, Object> sharedMap = new HashMap<>();`
4. If no thread-safe version is available, Atomic classes can be used (like `AtomicReference`). If all-else fails, `ThreadLocal`
    1. ✅ `private final AtomicInteger counter = new AtomicInteger(); counter.incrementAndGet();`
    2. ❌ `private int = 0; int++;`
5. In the rare case that the above guidelines can not be followed, appropriately use the `volatile` keyword

## Leaking resources

1. Prefer using try-with with `Closeable` resources
    1. ✅ `try(MyResource myResource = new MyResource()) { myResource.doThings(); }`
    2. ❌ `MyResource myResource = new MyResource(); myResource.doThings();`
2. If implementing a class which manages external resources prefer implementing the `Closeable` or `AutoCloseable` interface. This is especially critical for file handles, sockets, stateful streams, and OS level resources
    1. ✅ `class MyResource implements AutoCloseable { public void close(){...} }`
    2. ❌ `class MyResource { public void cleanUpHandles(){...} }`
3. Objects used as HashMap keys _must_ implement `hashCode()` or they will not function as a key. The result will be that anything put into the map can not be retrieved. If using a Map for deduplication, it will not deduplicate
    1. ✅ `map.put(myHashCodeClassObject, someValue)`
    2. ❌ `map.put(noHashCodeClassObject, willNeverFindThisValue)`
    3. `@lombok.Value` classes, `String`, `Integer`, and many other Java types will have `hashCode()` implemented
4. Keep in mind [semantic memory leaks](http://ithare.com/java-vs-c-trading-ub-for-semantic-memory-leaks-same-problem-different-punishment-for-failure/). While Java's GC can generally detect cycles it cannot automatically free objects that were left in a custom cache. Which is why try-with-resources can help to manage life cycle for short-lived caches.

## Spring

1. Hard-coded/magic values parameterized as `@Value` (Spring's Value annotation, not Lombok's)
    1. ✅ `@Value("${my.config.name:default value here}") String someValue`
    2. ❌ `String someValue = "default value here";`
2. Prefer using Spring to autowire properties, instead of wrapping and/or passing around a `PropertyResolver`
    1. ✅ `@Value("${my.property}") String myProperty`
    2. ❌ `String myProperty = propertyResolver.getString("my.property")`
3. For properties or configured instances, prefer using `@Configuration` class with `@Bean`, rather than `@Component` a wrapper class
    1. ✅ `@Bean MyPropertiesBean myPropertiesBean(){...}`
    2. ❌ `@Component class MyPropertiesBean(){...}`
4. Prefer specialized annotations for Component classes, if one applies:
    1. ✅ `@Configuration`, `@Controller`, `@Service`, `@RestController`, `@Repository`
    2. ❌ `@Component`
5. Prefer [AutoConfiguration](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration) and [Conditional](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration.condition-annotations) beans over hard-coded Package scans
    1. ✅ `@ConditionalOnProperty("my.feature.enabled"), @ConditionalOnExpression(...), @ConditionalOnBean(...)`
        1. `WebApplicationContextRunner` can quickly build test context for many different property/bean conditions
    2. ❌ `@SpringBootApplication(scanBasePackages = "my.specific.package")`
6. Prefer a constructed `@Bean` instance over simple Factory beans
    1. ✅ `this.thing = thingBean;`
    2. ❌ `this.thing = factoryBean.makeThing();`
7. Use Spring bean creation for Singleton instances. `SCOPE_SINGLETON` is the default `@Scope` for any component/bean
    1. Other scopes: `SCOPE_PROTOTYPE`, `SCOPE_REQUEST`, `SCOPE_SESSION`
8. Prefer standard `application.properties` and using Spring's many property injection options over custom properties files
    1. ✅ `application.properties`
    2. ❌ `custom.name.properties`

## Lombok

1. Use only one Lombok config at the top-level of the project
    1. `lombok.config` file should only exist at the top of the project so that the behavior is consistent throughout the project
2. Use Lombok `@Slf4J` for any static logger creation
    1. ✅ `@Slf4j class MyResource {...}`
    2. ❌ `private final Logger log = LoggerFactory.getLogger(MyResource.class);`
3. `@lombok.Value` for domain classes
    1. The `@lombok.Value` annotation will create an unmodifiable class with all fields set as private final, required args constructor, getters, `equals()`, `hashCode()`, and `toString()` all created at pre-compile time
    2. Additional annotations exist to modify the default behavior of all of this generated code
    3. Prefer using the full path `@lombok.Value`, since the lombok annotation clashes with Spring's `@Value` annotation
4. `@Builder` to make instantiation easier
    1. `@Builder` allows for partial/mutable construction of an unmodifiable class
    2. Additional annotations, like `@Singular` can make builders even more useable
    3. The Builder class can be partially implemented, if custom logic is needed with setting fields or building
5. `@Data` annotation can be used, if a mutable class is needed
    1. Like `@lombok.Value`, it can be paired with `@Builder` and will create a required args constructor, getters, `equals()`, `hashCode()`, and `toString()` at pre-compile time
6. `@SneakyThrows` can be used in situations when a handled-exception is caught, wrapped in a `RuntimeException`, and thrown
7. If it seems like a handwritten Builder is needed, it's very likely that the Lombok Builder can be expanded
    1. Check the [Lombok documentation](https://projectlombok.org/features/Builder)
    2. `@SuperBuilder` may be needed if using builders with inherited classes

## Unit and Partial-Integration Tests

1. Test packages match the subject-class package
2. Test classes named as `[SubjectClass]Test` (Singular, not plural). (In IntelliJ, Ctrl+Shift+T will automatically create a test class named this way)
3. Prefer AssertJ for assertions over Hamcrest or JUnit
4. Use Mockito instead of JMockit because JMockit has been abandoned
5. When testing `@Component` classes, prefer `@SpringBootTest(classes=MyClass.class)` and `@MockBean` (Instead of `MockitoAnnotations.openMocks(this)`)
6. Replace NoOp single-method interfaces with `() → {}`

## Structured Logging

Prefer Structured Logging patterns: `<statement>`. `<key-value-pairs>`. Splunk [Best Practices](https://dev.splunk.com/enterprise/docs/developapps/addsupport/logging/loggingbestpractices/) suggests Structured Logging because it makes most queries much easier; Rather than using regex to extract variables, each variable has a key. Other systems, like Elastic Stack, also suggest using Structured Logging `key=value` format. CLI tools like grep and awk can quickly take advantage of the `key=value` format.

1. Log a fixed statement, followed by `key=value` variables
    1. ✅ `log.info("Created entry. name={}, time={}", entryName, now);`
    2. ❌ `log.info("Created an entry called {} at {}.", entryName, now);`
2. Adding or reordering variables will not cause value extraction to fail or result in strange run-on sentences
    1. ✅ `log.info("Created entry. name={}, result={}, duration={}, time={}", ...);`
    2. ❌ `log.info("Created an entry called {} at {}, which was {} and took {}.", ...);`

## Documentation

1. Public classes and methods should have useful documentation. Document how to configure the class or what methods to prefer. Document what is not immediately apparent about the usage, method, parameters, and/or return type
    1. ✅ `generateQuery(...) /** formats the parameter section of a query. Expected to be joined to a "where" clause. @return e.g. "p1 = y AND p2 = b" */`
    2. ❌ `generateQuery(...) /** generates a query. @return the query */`
2. Deprecated classes and methods should document what the dev should use instead
    1. ✅ `/** @deprecated Use {@link OtherClass} for this functionality */
            @Deprecated`
    2. ❌ `@Deprecated`
3. Prefer self-documenting code and de-mystifying code over comments. Add comments where an explanation is needed. Avoid commenting on things that are clearly obvious from the code
    1. ✅ `// This uses bitwise comparison of the hash bytes to validate ...`
    2. ❌ `// The Constructor`
4. Keep documentation and comments up-to-date. Any refactor should also include updating documentation. It causes undue confusion when functionality changes and documentation doesn't
    1. ✅ `/** Use the {@link #generateQuery(...)} method to generate .... */`
    2. ❌ `/** Use the missing method to do things */`

## Code Formatting

1. If all Devs are using the same IDE (IntelliJ, Eclipse, VSCode), just use the built-in formatter
    1. All IDEs have a "Format" hotkey; learn it, use it
    2. If there is one stand-out using a different IDE, they should adapt to the commonly used IDE's format
2. More important than any guide or preference is that the formatting be consistent throughout the project
    1. Format preferences may vary, follow the common format
    2. Prefer automatic formatting, format checkers that fail builds are a waste of Dev time
    3. Format battles are a waste of energy (Tabs vs Spaces, 4-Space-Tabs vs 8-Space-Tabs, 2-Spaces vs 4-Spaces)
        1. See also: "Tabs vs Spaces" scene from the series "Silicon Valley" (no link provided)
        2. See also: "Tabs vs Spaces vs Both" on Reddit (no link provided)
3. Avoid reformatting untouched files, reformat as necessary
    1. If nothing has changed in a class/file, avoid reformatting it to maintain the commit history
    2. If a class if being refactored, consider if reformatting the whole class will make the review easier or harder
4. Import Order
    1. Just use the built-in order
    2. Do not break builds if in the wrong order, it is a waste of Dev time
    3. Automatic import sanitization 
    4. Prefer no wildcards in imports (but, don't break the build over it)

## Containerization

1. Containerize any/all deployable war/jar, etc. with the standard container (Docker)

