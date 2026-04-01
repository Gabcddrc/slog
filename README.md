# Slog
Slog (reads "S-log") is a simple, fast, and cross-language structured logging. It's like Glog but allows structure via tags and async handling.

# Importing
## Into a C++ project built by Bazel
Requirements:
* C++14

Instructions: 
* Follow the example from `test_import/example_project_cc_via_bazel` directory.

## Into a Python project built by Bazel
Requirements:
* All C++ requirements from above
* Python 3.8
* pybind11 and pybind11_bazel

Instructions: 
* Follow the example from `test_import/example_project_py_via_bazel` directory.

## Into a Java project built by Bazel
Requirements:
* All C++ requirements from above
* Java 8+

In your `WORKSPACE` file, add slog as a dependency (same as for C++). Then in your `BUILD` file:
```python
java_binary(
    name = "my_app",
    srcs = ["MyApp.java"],
    deps = ["@slog//:slog_java"],
)
```

Usage:
```java
import com.woven.slog.Slog;

Slog.info("Hello from Java");
Slog.info("with tags", Slog.tags("key", "value"));

try (SlogScope scope = Slog.scope("my_scope")) {
    Slog.info("inside scope");
}
```

## Into a Java project built by Gradle
Requirements:
* All C++ requirements from above
* Java 8+
* Bazel (to build the native libraries)

Gradle cannot build the JNI native code directly, so you first build the artifacts with Bazel, then use them from Gradle.

**Step 1.** Build the slog Java JAR and native `.so` files with Bazel:
```bash
bazel build //slog_java:libslog_jni.so //slog_java:slog_java
```

**Step 2.** Copy the artifacts into your Gradle project:
```bash
mkdir -p libs/native
cp bazel-bin/slog_java/libslog_java.jar libs/slog_java.jar
cp bazel-bin/slog_java/libslog_jni.so libs/native/
# Also copy dependent shared libraries:
cp bazel-bin/slog_java/libslog_jni.so.runfiles/__main__/_solib_k8/*.so libs/native/
```

**Step 3.** In your `build.gradle`, add the JAR as a dependency and configure the native library path:
```groovy
dependencies {
    implementation files('libs/slog_java.jar')
}

test {
    def nativeDir = file("${projectDir}/libs/native").absolutePath
    systemProperty 'java.library.path', nativeDir
    environment 'LD_LIBRARY_PATH', nativeDir
}
```

A complete working example is in `test_import/example_project_java_via_gradle`, including a `prepare.sh` script that automates steps 1-2.

# Contributing
## Installing tools for build
* `scripts/setup_dev_env.sh`

## Running tests
* `bazelisk test --test_output=errors //...` -- run all unit tests.

## Code formatting
* For the first time, install linters with `./scripts/install_linters.sh `
* `scripts/lint.sh ./ -i` -- automatically format all code.

## Releases.
CI automatically uploads a zip-archive to Artifactory. This .zip archive could be imported into another bazel project. The .zip file naming and content are matching a .zip file that could be created via GitHub releases.

