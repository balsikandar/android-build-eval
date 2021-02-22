### Intro

This repository is for community collaboration.

It includes Uber-agnostic auto-generated projects (via Android Studio Poet), with comparable complexity to existing Uber Production mobile apps.(*)

These projects are buildable on Buck, Bazel and Gradle, therefore it enables build time benchmarking across different build systems.

Any PR to enable speeding up any of these builds is welcome!

### Project details

Auto-generated project(s) offer similar complexity to existing Uber Production mobile apps in terms of :

- number of modules, classes and (some) resources
- interdependency between modules
- java Vs kotlin code
- dependencies on external libraries

When building these project, we voluntarily disable lint, error prone (**) and annotation processors for each build system for better build time measurement parity.

Also, we're not using build network cache, nor remote build execution (Bazel).

### Setup

- Make sure Bazel 3.7+ is installed, and is on the PATH (<https://docs.bazel.build/versions/master/install.html>)
- Change directory to mobile app folder : `cd mobile_app1`
- Generate OkBuck files : `./setup_buck.sh`

### How to build a project

- (Gradle) `./gradlew rootModule:assembleDebug`
- (Bazel) `bazel build rootModule`
- (Buck) `./buckw build rootModule:src_debug`

### How to run benchmark for a project

- `./benchmark.sh`

Build time report will be generated in the /output/<timestamp> folder.

Note :

- In order to 'clean' builds between runs for Bazel and Buck, we're using a fork for android-profiler that will be installed automatically from <https://github.com/sunyal/gradle-profiler>

### Scenario description

We used special naming for a couple of modules, these can be used to simulate different incremental builds :

- rootModule : the top project module (i.e any module in the project is a transitive dependency on this module)
- leafModule(Min|Avg|Max) : these are modules on which some other modules depend on. Min = only a few modules depends on this module, Max = a lot of modules depends on this module (i.e ABI change of this module will cause a somehow large part of the build graph to be rebuilt)

To simulate incremental builds (ABI and non-ABI changes alike), you can modify manually source files of these modules.

Benchmark scenario are as follow :

- fullLocalCache : clean build of the app (with external artifacts/toolchain already cached locally)
- abiChangeWith(Min|Avg|Max)LeafLocalCache : abi-change incremental build with (large|medium|small) part of the build graph rebuilt
- nonAbiChangeWithMaxLeafLocalCache : non abi-change incremental build
- androidResourceChangeWithRootLocalCache : incremental build with resources changes to root module
- noOpLocalCache : no-op build

### Benchmark results

#### Conditions

- Bazel : 3.7.0, persistent workers, sandboxing disabled, JAVA header generation disabled
- Gradle : 6.8, file watcher and configuration cache enabled
- Buck : latest version (as specified by OkBuck)
- Java : Java 8 compiler is used (pulled from JAVA_HOME env variable)
- Host machine : benchmark is ran on powerful linux server (***). Slower build times are expected on Macbook Pros, for instance.
- Last run : 02/02/2021 (code changes made after that are not reflected yet in result below)

#### Current results

![apps_presidio_helix_app results](/results/mobile_app1.png)

### License

```
Copyright 2020 Uber Technologies

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except
in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License
is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express
or implied. See the License for the specific language governing permissions and limitations under
the License.
```

(*) The generated code is not representative of Uber's existing production mobile apps.

(**) For Bazel, Error Prone is still running as it doesn't seem possible to disable it.

(***) Architecture:       x86_64
CPU op-mode(s):     32-bit, 64-bit
CPU(s):             96
Thread(s) per core: 2
Core(s) per socket: 24
Model name:         Intel(R) Xeon(R) CPU @ 2.00GHz
RAM:                396gb
