---
layout: post
title:  "Jenkins Shared Libraries"
date:   2020-08-04 08:00:00 -0400
tags: [jenkins]
---

When you maintain a number of projects, each with their own Jenkinsfile, you will find you are copying the same 
code over and over again. Shared libraries provide a way to share code and simplify Jenkinsfile creation, allowing your 
team to produce pipelines faster. 

### How pipeline libraries work

A shared library can be loaded in your pipeline script with the `@Library` annotation
`@Library('shared-library@v1.0') _`

The library is freshly checked out from git at the start of every build.
The `@version` part corresponds with the name of the branch of your shared library repo. 
If it's omitted, the default version specified in the global- or folder configuration is loaded. There you can also 
specify whether or not you allow overriding that version.

#### Global or Folder level
A shared pipeline library can be configured either globally, or on a folder, if the Folders Plugin is installed. 
The behavior in both cases is different. A global shared library is considered *trusted*, and does not run in the 
sandbox. This means the library is able to, for instance, download binaries from the internet and run them on your 
master. It can use groovy's `@Grab` annotation to pull dependencies from artifactory. Jenkins admins should carefully 
evaluate whether or not this should be allowed on a master. 

A shared pipeline library creates a separate environment from the Jenkins master per-pipeline. This means that every 
library loads all of its resources (imports, variables, functions, classes, etc.) per pipeline rather than what 
happens with a plugin where it loads them once. Each pipeline loads its initial environment based on what is loaded 
using pipeline plugins and then loads your shared library on top of that. This means that each time you are doing 
something like a rest call, it has to load those resources for each pipeline and those loads could happen at the very 
same time. The grab annotation is single threaded, and can potentially cause a complete deadlock in pipeline execution.  

Shared libraries configured on a folder run in the sandbox, and are much more limited in what they can do and what 
resources they can access. It can't use `@Grab` or download content from the internet.

#### Flyweight and heavyweight executors
Jenkins defines flyweight and heavyweight executors. A flyweight executor is just a java thread running on the master. 
All of the pipeline code other than the steps themselves runs on the master using a flyweight executor. Steps may use 
heavyweight executors to do work on the agent if they are enclosed in a node block.

```groovy
def call(List list) {
  def result = list.collect {it -> it * 2}.sum()
  sh "echo $result"
}
```

In the step defined above, only the `echo $result part` is executed on the agent, the computation runs on the master. 
Don't put computationally intensive operations in your pipeline library, and don't use blocking I/O calls, or you may 
end up blocking the master. Cloudbees recommends using shell steps for all your resource intensive work. Cloudbees 
pecifically warns not to do things like XML or JSON parsing in the groovy code.

#### Groovy but not really
 
The syntax of pipeline scripts is groovy, but it is not executed as-is. It is transformed at compile time to to 
Continuation Passing Style (CPS), which adds safety, and makes the execution durable across master restarts, but it 
comes at a price. The code runs much slower than native groovy, and it supports only a small subset of groovy 
features. All variables used in CPS code must be serializable. For more complex code, use @NonCPS annotated methods. 
This still runs on the master, but is not transformed. In runs much faster and allows non-serializable local variables 
(parameters and return types must be serializable), but it will not survive being suspended and resumed, and it cannot 
use pipeline steps inside the `@NonCPS` method. A method annotated `@NonCPS` cannot call a method that has been 
CPS transformed.

#### Project structure
Jenkins Shared Library projects are expected to have the following directory structure:

```
(root)
+- src                     # Groovy source files
|   +- org
|       +- foo
|           +- Bar.groovy  # for org.foo.Bar class
+- vars
|   +- foo.groovy          # for global 'foo' variable
|   +- foo.txt             # help for 'foo' variable
+- resources               # resource files (external libraries only)
|   +- org
|       +- foo
|           +- bar.json    # static helper data for org.foo.Bar
```

#### src
The src directory contains class definitions that will be available on the classpath. In a scripted pipeline build, it 
is possible to instantiate these classes to run the containing code. If you do this, keep in mind that the groovy code 
in a scripted pipeline runs on the master, so avoid doing this for resource intensive code.
You can use classes defined under src in your scripted pipeline syntax, but always keep in mind that all this code, in 
every build that uses them, will run on the master. Technically you can use these in a declarative pipeline script too, 
but then you're no longer strictly using declarative syntax.

#### vars
The vars directory contains scripts that are available as variables in the pipeline. The name of the file is the name 
of the variable. In Jenkins groovy terms, 'variables' can refer to values, functions, closures etc. Anything you 
declare here will be available in the scope of your pipeline script.

`def doSomething(arg) { ... }`

The method above in the file `vars/foo.groovy` can be called as `foo.doSomething(argument)`. A method called call operates 
as a custom pipeline step that corresponds with the file name, if it has exactly this method signature: 

`def call(Map args = [:], Closure block) { ... }`

The arguments map and the closure block are optional. 

If the last parameter of a method call is a closure, groovy syntax allows you to place this closure outside the 
brackets of the method call. This means that a pipeline step defined in a file called `vars/foo.groovy` that contains 
a code block is defined as follows:

```groovy
def call(Closure block) {
    ...
    block.run()
}
```

can be used in a pipeline script with the following syntax:

```groovy
foo {
    sh 'echo step 1'
    sh 'echo step 2'
}
```

The groovy scripts under vars can be documented with corresponding `.txt` files with the same name. According to the 
documentation, these help texts support the system's configured markdown formatter, so it could be html, markdown etc, 
depending on which markup plugin is installed. You can select the system's markup formatter under `/configureSecurity`. 
The help texts you add show up under `pipeline-syntax/globals`.

Files under the `resources/` directory can be loaded by the built in `libraryResource` step, which will return the content 
as a string.

`def json = libraryResource 'org/foo/bar.json'`

The line above will load the content of the file `org/foo/bar.json` into the variable json.
Even though this json example can be found in the official cloudbees documentation, they're actually warning you to 
avoid json parsing in groovy, because of the associated cpu and memory load on the master. A better example might be 
this:

```groovy
def call() {
    def script = libraryResource('com/example/scripts/example.sh')
    sh "$script"
}
```

#### GDSL
A groovy DSL script can be downloaded under `$JENKINS_URL/pipeline-syntax/gdsl`. This gdsl file adds tab completion and 
syntax highlighting to editors that support it. In Intellj, place the gdsl file in the root of your groovy source folder.


#### Multiple shared libraries

All shared libraries declared by your pipeline script will be loaded in the same environment. They will share the same 
variables and class definitions in the same scope, and use each other's values and code. Multiple libraries can be 
declared in one statement:

`@Library(['library1', 'library2@branch']`

 
### Sources
Best practice from cloudbees

- <https://jenkins.io/doc/book/pipeline/shared-libraries/>
- <https://support.cloudbees.com/hc/en-us/articles/230922208-Pipeline-Best-Practices>
- <https://github.com/jenkinsci/pipeline-examples/blob/master/docs/BEST_PRACTICES.md> 
- <https://jenkins.io/blog/2017/02/01/pipeline-scalability-best-practice/>

Background info

- <https://jenkins.io/blog/2017/02/01/pipeline-scalability-best-practice/>
- <https://support.cloudbees.com/hc/en-us/articles/360012808951-Pipeline-Difference-between-flyweight-and-heavyweight-Executors>
