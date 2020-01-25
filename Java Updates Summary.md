
***Just a few basic summations of the more recent and important enhancements to Java***

Java 8
------
- Lambda expressions e.g. 
	````
	(a,b) -> someMethod(a, b))
	````

	Lambdas are effectively implementations of single method interfaces

- Functional interfaces
	````
	@FunctionalInterface
	public interface Supplier<T> { 
		T get(); 
	}

	@FunctionalInterface
	public interface Consumer<T> { 
		void accept(T t); 
	}
	
	@FunctionalInterface
	public interface BiConsumer<T U> { 		
		void accept(T t, U u ); 
	}

	@FunctionalInterface
	public interface Predicate<T> { 
		boolean test(T t); 
	}

	@FunctionalInterface
	public interface Function<T, R> {
		R apply(T t); 
	}

	@FunctionalInterface
	public interface BiFunction<T, U, R> {
		R apply(T t, U u); 
	}
	````
	Predicates can be chained using **and**
	e.g. 
	```
	Predicate<String> p1 = s -> s.length() < 20;
	Predicate<String> p2 = s -> s.length() > 10;
	Predicate<String> p3 = s -> p1.and(p2);
	```
	Typically these are used to define the parameters to the Stream API methods (see below) which will be entered "inline" as lambdas

- Method references
	- e.g. Integer::compare, System.out::println
- Iterable<E> forEach

- Streams - .stream() (filter(), map(). reduce(), collect(), sort(), flatmap() etc) - a considerable change to enhance "*fluid*" programming

- Optionals
		e.g. Optional<Integer> max = stream.max(Comparator.naturalOrder())
		.isPresent(), .orElse() , .orElseThrow(), .get()

- Terminal reductions
	Principally for stream usage - max(), min(), count(), allMatch(), noneMatch(), anyMatch(), findFirst(), findAny()

- Date and Time API (Replacements for flawed java.util.Date, Calendar)
	- New package *java.time*
		- Instant
		- LocalDate
		- Period
		- DateAdjuster
		- LocalTime
		- ZonedTime

- Default methods on interfaces
	- a necessity really for adding the other functionality but also allows future trouble-free extension of interfaces
	- see e.g the Collector interface

- StringJoiner
		e.g. 

		StringJoiner sj = new StringJoiner(", ");
		sj.add("one").add("two").add("three");
		String result = sj.toString();

- Java I/O Enhancements

	For reading files etc now - better to use 
	e.g 
	Stream<String> streamLines = reader.lines(); // on a BufferedReader in a try-with-resources block
	
	````
	Path path = Paths.get("d", "tmp", "debug.log");
	try (Stream<String> logLines = Files.lines(path)) {
		...
	}
	````

- Collections API and Hashmap
	- Addition of stream(), parallelStream(), spliterator(), forEach(), removeIf(), replaceAll(), sort()
	e.g. list.sort(Comparator.naturalOrder())
			
- Map interface
	- forEach(), getOrDefault(), putIfAbsent(), replace(), replaceAll(),
	- remove(), compute(), computeIfPresent(), computeIfAbsent(), merge()

- Annotations
	Multiple now allowed e.g.
	````
	@TestCase(param=1, expected=false)
	@TestCase(param=2, expected=true)
	````		

	and on types
	````			
	private @NonNull List<@NonNull Person> persons = ...;
	````

- Java FX added to JDK (but see Java 11 changes !)
- Nashorn Javascript engine (but see Java 11 deprecation of..)

Java 9
------
- The new module system

		module java.base{
			exports java.lang;
			exports java.util;
			exports java.io;
			//etc
		}

		requires java.logging;

		$java --list-modules
		$java --describe-module java.sql

		Dependency inspection (from cmd line) e.g. to migrate a classpath based app
			$jdeps -jdkinternals Main.class

- JShell - Fully functional Java REPL
	- can also be run within an app (if desired - more of an IDE tools feature)
	- $jshell (numerous switches available including /save, /methods, /open, /vars)

- Collection Factory Methods
	- List.of (e.g. List<Integer> int = List.of(1,2,3);)
	- Set.of
	- Map.of(e.g. Map.of("Key1",1, "Key2", 2);) or (even better):-
	````
	Map<String, List> myMap = Map.ofEntries(Map.Entry("Key1", true), Map.Entry("Key2", false);)
	````

		
- Stream API improvements/additions
	- Stream<T> **takeWhile**(Predicate<? super T> predicate)
	- Stream<T> **dropWhile**(Predicate<? super T> predicate)
	- Stream<T> **ofNullable**(T t)
	- static Stream<T> **iterate**(T seed, Predicate<? super T> hasNext, UnaryOperator<T> next)

- New Collectors (for Streams)
	- groupingBy e.g 
		````
		Map<Integer, List<Integer> ints = Stream.of(1, 2, 3, 3)
		.collect(groupingBy(i -> i%2, toList() ) );
		````
	- filtering e.g.
		````
		Map<Set<String>, Set<Book> booksByAuthors = 
			books.collect(
				groupingBy(Book::getAuthor,
					filtering(b -> b.getPrice() > 10,
					toSet() )
					)
			);
		````

	- flatMapping e.g.
		````
		Map<Double, Set<String>> authorsSellingForPrice = 
			books.collect(
					groupingBy(
						Book::getPrice,
						flatMapping(b -> b.getAuthors().stream(),
						toSet())
						)
			);
		````

- Optional - added functionality
	- void ifPresentOrElse(Consumer<T> action, Runnable emptyAction)
	- Optional<T> or(Supplier<Optional<T>> supplier)  [allows chaining]
	- Stream<T> stream()

- New APIs
	- ProcessHandle e.g.
		````
		ProcessHandle.allProcesses()
				.map(ProcessHandle::info)
				.sorted(Comparator.comparing( 
						info -> info.startInstant.orElse(Instant.MAX)))
				.forEach(ListProcesses::printInfo)
		````

- HttpClient (experimental only !!)
	- send(), sendAsync() - CompletableFuture, thenAccept(), join() etc..

- jlink - allows production of a runtime image e.g. for deployment (if required)

- Applets deprecated (but see Java 11)

- Private interface methods

- Reactive Streams
	- an event driven way of handling streams of data
	- has support for backpressure
	- Flow API: interfaces added to JDK
	- interoperability for reactive projects (e.g. RxJava, AkkaStreams, Spring5 )
	- not meant as an end user API
	- Publisher/Subscriber interfaces

- Stackwalker API
	- walk(), forEach()
	
		e.g.
		````
		StackWalker walker = StackWalker.getInstance();
		walker.forEach(System.out::println);
		````

- GC changes
	- Concurrent Mark Sweep GC deprecated
	- G1 Garbage Collector now default
		- divides heaps into cells not just 3 big regions
		- an incremental GC
		- parallel marking
		- low pause, tunable
		- slightly more CPU intensive but well suited for large heaps

- Compact Strings
	- lower mem usage & effective without any code changes

Java 10
-------
- Local variable type inference (**var**)
	- this is a reserved type not a keyword
	- not allowed in lambdas (cf Java 11)
	- (essentially catching up with Scala, Kotlin, C#)
		````
		var name = "Sender";
		````
- Stop The World GC
	- Serial full GC in Java 9 but parallel full in 10
	- Tunable: -XX:ParallelGCThreads

- Java Shared Archive (JSA)
	- to improve VM startup time and reduce mem footprint if desired
	- via "jsa - shared mem"

- Improved Container awareness (e.g. for Docker etc)
	- queries the container instead of the host OS !



Java 11
-------
- Single file source code
	- can avoid the 2 step process of javac then java (java Hello.java is enough - will compile and run)
	- can also use executable Java script with #! (e.g. #!/user/bin/java -- source 11)

- Deprecations and removals
	- JAXB, JAXWS, CORBA, JTA,
	- Java Beans Activation
	- (can add dependencies for JAXB if needed for Spring via Maven: jaxb-api-jaxb-xml)
	- Thread methods destroy() and stop()
	- runFinalizersOnExit()
	- 4 methods in SecurityManager (AWT-related)
	- Applets - now completely removed along with the browser plugins
	- Nashorn - present but usage deprecated

- JavaFX
	- moved out of JDK into OpenJDK (so no longer tied to JDK)
	- published on Maven Central as "javafx"
	- Javapackager also removed (possibly to be supplanted by JPackager)

- HttpClient (java.net.http)
	- no longer "incubated" as in Java 9,10 (no requirement to use --add-modules jdk.incubator.httpclient)

- Lib improvements
	- String - repeat(), isBlank(), strip(), lines()
	- Files - readString(), writeString()
	- Optional<> - isEmpty()
		````
		var opt = Optional.ofNullable(null);
		opt.isEmpty()==true;

		````
	- Predicate::not 
		````
			.filter(Predicate.not(String::isBlank))
		````
	- Alignment with Unicode 10 (inc Bitcoin symbol)

- Local Variable Syntax for Lambda parameters
	````
	(var a, var b) -> a.concat(b)
	````
	- this is useful if e.g. you wish to annotate types
		````
		(@NonNull var a, @Nullable var b) -> a.concat(b)
		````

- Nest based access control
	- Inner classes calling private methods on containing classes - not previously possible but these now get declared as "nestmates"

- GC
	- G1GC still the default but now ~60% better
	- Epsilon GC added 
		- doesn't actually GC at all - useful for certain circumstances (!)
		- (-XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC)
	- Z GC
		- pause times under 10ms
		- no increase with heap size increase (scale to multi tbyte heaps)
		- coloured pointers
		- BUT only available on Linux


Java 12
-------

- String additional methods
	- indent()
	- transform() [e.g. transform(StringUtils::clean)]

- Compact number formats
	````
	NumberFormat shortNF = NumberFormat.getCompactNumberInstance();
	String short = shortNF.format(1000);
	(alt shortNF.setMaximumFractionDigits(2);)
	(alt .getCompactNumberInstance(Locale.GERMAN, Number.Format.Style.SHORT))
	````
	- can also customise
		````
		CompactNumberFormat(descPattern, symbols, compactPattern)
		````


- New Teeing Collector
	- an addition to e.g. Collectors.toList(), toSet(), counting()
	- it allows 2 collectors at the same time (teeing one after the other)
	- Collectors.teeing(c1, c2, combine)
		````
		var ints = Stream.of(10, 20, 30, 40);
		long average = ints.collect(
						Collectors.teeing(
							Collectors.summingInt(Integer::valueOf),
							Collectors.counting(),
							(sum, count) -> sum/count
						)
					);
		````

- Files::mismatch
	````
	Files.mismatch(Path.of("/file1"), Path.of("/file2")); 
	// -1 on first mismatching byte
	````

- Switch expressions 
	- (preview feature only at this point!!) - requires "--enable preview --release12" etc
	- e.g.
		````
		String monthName = switch(monthNumber) {
			case 1 -> "January";
			case 2 -> "February";
			case 3,4,5 -> "spring months";
			default -> "Unknown";
		};
		// Note that there is no fallthru !
		````
	- (playing catchup with Scala !?)

- Java Microbenchmarking Harness (JMH)
	- Performance measuring for small pieces of code e.g to compare alternatives or prevent performance regression
	- Reproducability - handles JVM warmup for consistent reporting etc.
	- Annotation based !
	- Initialise a benchmark project with Maven (mvn archetype:generate "@Benchmark")
	- @BenchmarkMode(Mode.AverageTime) - note similarity to approach in JUnit
	- Pitfalls !
		- Dead code elimination
		- Other compiler optimisations
		- Assumptions (making of)
	
	````
	@BenchmarkMode(Mode.AverageTime)
	@OutputTimeUnit(TimeUnit.NANOSECONDS)
	@Fork(1)
	public class SomeBenchmark {
		@Param({"1", "12345", "2147483647"})
		String toParse;

		@Benchmark
		public Integer parseInt() throws Exception {
			return Integer.parseInt(toParse);
		}
	}
	````
	- Get a normal jar and a benchmarks jar, "java -jar target/benchmarks.jar" to run
	- Does 5 warm up iterations, then 5 iterations, an overview of results is returned
	- possible to integrate into CI (and output CSV/JSON to a file)

- JVM changes
	- further improvements to G1GC (abortable mixed collections, collections in steps, prompt return of unused uncommitted memory)
	- New GC - Shenandoah
		- supports large heaps and has low pause times
		- uses Brooks pointers and reference forwarding
		- is EXPERIMENTAL (& Redhat OpenJDK) only !!

- JVM Constants API
	- low level, **not directly used**
	- concerns representation of classes partic the Constant pool - numeric literals, strings, method and class refs
	- mainly to pave way for future enhancements


Java 13
-------
- API updates
	- ByteBuffer - provides a view of bytes on/outside the heap
		````
		ByteBuffer get(int index, byte[] dst);
		ByteBuffer put(int index, byte[] src);
		````
		Can do bulk ops on multiple threads

- javax.security.cert removed
	- now use java.security.cert.Certificate (or X509Certificate)

- switch expressions tweak
	- use of break was not liked - e.g.:
		````
		case 1 -> {String month = "January";
					break month;
				}
		````
	- new **yield** keyword used to replace - i.e. "yield month;"
	- NOTE: switch ex is still a preview feature in J13

- Unicode 12.1 support - 4 new writing scripts supported

- Z Garbage Collector changes
	- now does Uncommit Unused Memory (to OS) -XX:ZUncommitDelay=x (in secs)
	- (experimental but performs comparatively well)

- Text blocks (note - preview feature)
	- seems similar to Python text blocks - e.g.:
		````
		String json = """
		{
			"firstName":"John",
			"lastName":"Evans"
		}
		"""
		````

- String - 3 new methods
	- these are mostly to facilitate the text block enhancement
	- stripIndent() - removes incidental whitespace
	- translateEscapes() - will apply the escape sequences to the string
	- formatted()
		````
		System.out.println("Hello %s!".formatted("World")); // Hello World!
		````

- Platform changes
	- reimplementation of SocketAPI (java.net.Socket, ServerSocket)
		- a more maintainable and modern implementation
		- not using Stack as I/O buffer
		- in readiness for the new Java concurrency modesl (project Loom)
	- sun.nio.ch.NioSocketImpl - reuses NIO native code and built using java.concurrent.Lock

- Class data sharing
	- to reduce class (& other metadata) loading times
	- JSA changes - dynamic App CDS archives

- Deprecated flag
	-XVerify:none & -noverify