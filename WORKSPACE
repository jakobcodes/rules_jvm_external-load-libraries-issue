workspace(name = "rules_jvm_external-test")

load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

RULES_JVM_EXTERNAL_TAG = "6.0"
RULES_JVM_EXTERNAL_SHA = "85fd6bad58ac76cc3a27c8e051e4255ff9ccd8c92ba879670d195622e7c0a9b7"

http_archive(
    name = "rules_jvm_external",
    strip_prefix = "rules_jvm_external-master",
    url = "https://github.com/bazelbuild/rules_jvm_external/archive/refs/heads/master.zip",
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

#load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_file")
#
#http_file(
#    name = "com_almworks_sqlite4java_libsqlite4java_linux_amd64",
#    sha256 = "da3d7d21eef476baf644026e449b392dbd738bf9246fca48ce072987264c3aca",
#    urls = ["https://repo1.maven.org/maven2/com/almworks/sqlite4java/libsqlite4java-linux-amd64/1.0.392/libsqlite4java-linux-amd64-1.0.392.so"],
#    downloaded_file_path = "libsqlite4java-linux-amd64-1.0.392.so"
#)