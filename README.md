# rules_jvm_external-load-libraries-issue

I encountered an issue while trying to load a library in Bazel via `rules_jvm_external`. I found that the library cannot be loaded if there is no `.jar` file present and only `.so/.dylib/.dll` files are available.

The test code is as follows:
```
workspace(name = "rules_jvm_external-test")

load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

RULES_JVM_EXTERNAL_TAG = "6.0"
RULES_JVM_EXTERNAL_SHA = "85fd6bad58ac76cc3a27c8e051e4255ff9ccd8c92ba879670d195622e7c0a9b7"

http_archive(
    name = "rules_jvm_external",
    strip_prefix = "rules_jvm_external-%s" % RULES_JVM_EXTERNAL_TAG,
    sha256 = RULES_JVM_EXTERNAL_SHA,
    url = "https://github.com/bazelbuild/rules_jvm_external/releases/download/%s/rules_jvm_external-%s.tar.gz" % (RULES_JVM_EXTERNAL_TAG, RULES_JVM_EXTERNAL_TAG)
)

load("@rules_jvm_external//:repositories.bzl", "rules_jvm_external_deps")

rules_jvm_external_deps()

load("@rules_jvm_external//:setup.bzl", "rules_jvm_external_setup")

rules_jvm_external_setup()

load("@rules_jvm_external//:defs.bzl", "maven_install")
load("@rules_jvm_external//:specs.bzl", "maven")

maven_install(
    artifacts = [
        "com.almworks.sqlite4java:libsqlite4java-linux-amd64:1.0.392",
        "com.almworks.sqlite4java:libsqlite4java-osx:1.0.392",
    ],
    repositories = [
        "https://repo1.maven.org/maven2",
    ]
)
```

Here is a coursier fetch, which appears to show that it has downloaded jars, when in fact it hasn't. This could potentially be a topic for another issue 🤔
```
argsfile_content:
"-noverify"
"-jar"
"/private/var/tmp/_bazel_jakub_lubkowski/307cfbf9feb0e55d16011b457f38dc36/external/maven/coursier"
"fetch"
"com.almworks.sqlite4java:libsqlite4java-linux-amd64:1.0.392"
"--artifact-type"
"jar,json,aar,bundle,eclipse-plugin,exe,orbit,test-jar,hk2-jar,maven-plugin,scala-jar,src,doc"
"--verbose"
"--no-default"
"--json-output-file"
"dep-tree.json"
"--checksum"
"SHA-1,MD5"
"--repository"
"https://repo1.maven.org/maven2"
"--cache"
"v1"


/Library/Java/JavaVirtualMachines/amazon-corretto-17.jdk/Contents/Home/bin/java @/private/var/tmp/_bazel_jakub_lubkowski/307cfbf9feb0e55d16011b457f38dc36/external/maven/java_argsfile
OpenJDK 64-Bit Server VM warning: Options -Xverify:none and -noverify were deprecated in JDK 13 and will likely be removed in a future release.
  Dependencies:
com.almworks.sqlite4java:libsqlite4java-linux-amd64:1.0.392:
Downloading https://repo1.maven.org/maven2/com/almworks/sqlite4java/libsqlite4java-linux-amd64/1.0.392/libsqlite4java-linux-amd64-1.0.392.pom
Downloaded https://repo1.maven.org/maven2/com/almworks/sqlite4java/libsqlite4java-linux-amd64/1.0.392/libsqlite4java-linux-amd64-1.0.392.pom
Downloading https://repo1.maven.org/maven2/org/sonatype/oss/oss-parent/4/oss-parent-4.pom
Downloaded https://repo1.maven.org/maven2/org/sonatype/oss/oss-parent/4/oss-parent-4.pom
Downloading https://repo1.maven.org/maven2/com/almworks/sqlite4java/sqlite4java/1.0.392/sqlite4java-1.0.392.pom
Downloaded https://repo1.maven.org/maven2/com/almworks/sqlite4java/sqlite4java/1.0.392/sqlite4java-1.0.392.pom
Downloading https://repo1.maven.org/maven2/com/almworks/sqlite4java/libsqlite4java-linux-amd64/1.0.392/libsqlite4java-linux-amd64-1.0.392.jar
Downloading https://repo1.maven.org/maven2/com/almworks/sqlite4java/sqlite4java/1.0.392/sqlite4java-1.0.392.jar
Downloaded https://repo1.maven.org/maven2/com/almworks/sqlite4java/libsqlite4java-linux-amd64/1.0.392/libsqlite4java-linux-amd64-1.0.392.jar
Downloading https://repo1.maven.org/maven2/com/almworks/sqlite4java/libsqlite4java-linux-amd64/1.0.392/libsqlite4java-linux-amd64-1.0.392.jar.md5
Downloading https://repo1.maven.org/maven2/com/almworks/sqlite4java/libsqlite4java-linux-amd64/1.0.392/libsqlite4java-linux-amd64-1.0.392.jar.sha1
Downloaded https://repo1.maven.org/maven2/com/almworks/sqlite4java/libsqlite4java-linux-amd64/1.0.392/libsqlite4java-linux-amd64-1.0.392.jar.md5
Downloaded https://repo1.maven.org/maven2/com/almworks/sqlite4java/sqlite4java/1.0.392/sqlite4java-1.0.392.jar
Downloaded https://repo1.maven.org/maven2/com/almworks/sqlite4java/libsqlite4java-linux-amd64/1.0.392/libsqlite4java-linux-amd64-1.0.392.jar.sha1
v1/https/repo1.maven.org/maven2/com/almworks/sqlite4java/sqlite4java/1.0.392/sqlite4java-1.0.392.jar
```
Interestingly, in the `--artifact-type` flag, the `.so`, `.dylib`, or `.dll` types are not included. However, at this [location](https://github.com/bazelbuild/rules_jvm_external/blob/master/private/coursier_utilities.bzl#L20), we can see that they are present. 🤔 (Probably will be added in next release!)


Screenshot of missing files under `amd64` and `osx` paths:

![image](https://github.com/bazelbuild/rules_jvm_external/assets/85311656/d5396740-da5d-4e20-bfb4-dbab549ebca1)

To resolve this, I used http_files as a workaround like this:
```
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_file")

http_file(
    name = "com_almworks_sqlite4java_libsqlite4java_linux_amd64",
    sha256 = "da3d7d21eef476baf644026e449b392dbd738bf9246fca48ce072987264c3aca",
    urls = ["https://repo1.maven.org/maven2/com/almworks/sqlite4java/libsqlite4java-linux-amd64/1.0.392/libsqlite4java-linux-amd64-1.0.392.so"],
    downloaded_file_path = "libsqlite4java-linux-amd64-1.0.392.so"
)
```

Although this workaround is functional, I believe this is an issue with `rules_jvm_external` itself that should be addressed. Whenever a `.jar` file is not present, the library should still be loaded without any issues. Currently, it seems that `rules_jvm_external` only supports `.jar` files and not `.so/.dylib/.dll` files.

Probably resolved here: https://github.com/bazelbuild/rules_jvm_external/pull/959
