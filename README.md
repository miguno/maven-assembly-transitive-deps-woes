# Maven Assembly Woes: Transitive Dependencies and excludes directive

## Problem

Dependency graph:

```
 child1 --> A --> B

 child2 --> B
```

* Project `child1` depends on library `A` directly, which in turns depends on library `B`.
* Project `child2` depends on library `B` directly.
* Once we exclude `child2` in the assembly setup for `child1` (with `useTransitiveFiltering` set to `true`),
  then library `B` will be also excluded from the assembly of `child1` (even though library `A`,
  on which `child1` depends directly, still depends on library `B`).  Doh!


## Example

When you package this project via `mvn clean package`, maven assembles the following directory:

```shell
# Actual + WRONG directory tree after packaging
#
$ tree package/target/package-1.0-bin/share/java/
package/target/package-1.0-bin/share/java/
├── child1
│   ├── avro-1.7.7.jar
│   ├── child1-1.0.jar
│   ├── commons-compress-1.4.1.jar
│   ├── jackson-core-asl-1.9.13.jar   <<< jackson-mapper-asl should also be here but isn't
│   ├── junit-3.8.1.jar
│   ├── paranamer-2.3.jar
│   ├── slf4j-api-1.6.4.jar
│   ├── snappy-java-1.0.5.jar
│   └── xz-1.0.jar
└── child2
    ├── avro-1.7.7.jar
    ├── child2-1.0.jar
    ├── commons-compress-1.4.1.jar
    ├── jackson-annotations-2.5.0.jar
    ├── jackson-core-2.5.4.jar
    ├── jackson-core-asl-1.9.13.jar
    ├── jackson-databind-2.5.4.jar
    ├── jackson-mapper-asl-1.9.13.jar
    ├── paranamer-2.3.jar
    ├── slf4j-api-1.6.4.jar
    ├── snappy-java-1.0.5.jar
    └── xz-1.0.jar
```

Here, `jackson-mapper-asl` (our library `B` in this example) is a dependency of `avro` (our library `A`).
Because `child2` depends directly on `jackson-mapper-asl`, and we `<exclude>` the project `child2` from
the assembly of `child1, the assembled contents of `child1` will now NOT include `jackson-mapper-asl`
even though `child1` does require it for proper functioning (because `child1` depends on `avro`, and
`avro` won't work without `jackson-mapper-asl`).

The expected correct directory tree looks as follows:

```
# Expected + correct directory tree after packaging
#
$ tree package/target/package-1.0-bin/share/java/
package/target/package-1.0-bin/share/java/
├── child1
│   ├── avro-1.7.7.jar
│   ├── child1-1.0.jar
│   ├── commons-compress-1.4.1.jar
│   ├── jackson-core-asl-1.9.13.jar
    ├── jackson-mapper-asl-1.9.13.jar
│   ├── junit-3.8.1.jar
│   ├── paranamer-2.3.jar
│   ├── slf4j-api-1.6.4.jar
│   ├── snappy-java-1.0.5.jar
│   └── xz-1.0.jar
└── child2
    ├── avro-1.7.7.jar
    ├── child2-1.0.jar
    ├── commons-compress-1.4.1.jar
    ├── jackson-annotations-2.5.0.jar
    ├── jackson-core-2.5.4.jar
    ├── jackson-core-asl-1.9.13.jar
    ├── jackson-databind-2.5.4.jar
    ├── jackson-mapper-asl-1.9.13.jar
    ├── paranamer-2.3.jar
    ├── slf4j-api-1.6.4.jar
    ├── snappy-java-1.0.5.jar
    └── xz-1.0.jar
```
