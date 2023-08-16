---
title: How to do bazel java_export with desired change only
date: 2023-08-16 20:26:10
categories: BackEnd
tags:
    - bazel
top:
---
# How to to bazel java_export with desired change only


Faced one interesting issue when using bazel, the background is we have multiple repository to host code — SOA. For sharing, then we need to export the library and publish to company wide artifactory. 

# Background

We use bazel mainly because we have code on different languages, like RoR, Java, Kotlin, Scala, Clojure, etc. Bazel makes it easier to manage it cross different language in a monorepo. 

For the export library purpose, we heavily rely on the [rules_jvm_external](https://github.com/bazelbuild/rules_jvm_external/blob/master/private/rules/java_export.bzl) , which has built-in java_export commend we could leverage on directly. 

```jsx
def java_export(
        name,
        maven_coordinates,
        deploy_env = [],
        excluded_workspaces = {name: None for name in DEFAULT_EXCLUDED_WORKSPACES},
        pom_template = None,
        visibility = None,
        tags = [],
        testonly = None,
        **kwargs)
```

Basically we could follow the pattern here to define our srcs, deps, maven_coordinates, and then bazel could help us export it and publish to selected artifactory. 


# Issue

On the other end, we have a service built on kotlin with gradle. When we implement the library, we found inside the jar, besides the code in the export places, it also contain directory from `com.google.protobuf.*` , and unfortunately the protobuf package version in the lib mismatch with what we use in our own service, as the version in the lib is lower, it breaks the compilation of our code base, especially when we have couple extension functions in protobuf to enrich the conversion, whereas the old protobuf version does not support. 

So basically we need to find a way to not include undesired directory inside the jar. In our case, remove those from `com.google.protobuf.*` 

# Solution

[https://bazel.build/reference/be/java#java_library](https://bazel.build/reference/be/java#java_library) 

Add those into runtime_deps, then the export jar would not contain the directory from the pkg. It solves our problem perfect. 