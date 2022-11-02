# 使用 Bazel 构建 Java 项目

---

## 准备工作

- 使用 Bazelisk 安装 Bazel `brew install bazelisk`
- 安装 IDEA Bazel 插件 `https://plugins.jetbrains.com/plugin/8609-bazel`
- 使用 Maven 初始化一个项目

## Bazel 基本操作

- 项目根目录添加 WORKSPACE
- 每个要构建的类包添加 BUILD 文件

```
java_binary ( # 规则的名称；它需要额外的属性来构建二进制文件
    name = "java-bazel-demo", # 构建目标的名称
    srcs = glob(["src/main/java/org/example/*.java"]), # 一个文件位置模式数组，用于告诉要构建哪些 Java 文件
    main_class = "src.main.java.org.example.App", # 应用程序主类的名称（可选）
)
```

- 使用 Bazel 构建项目 `bazel build //:java-bazel-demo`
- 使用 IDEA Bazel 插件构建项目
- 编译后的文件在 `bazel-bin` 目录下
- 如果构建项目使用 _deploy.jar 后缀`java -jar bazel-bin/java-bazel-demo_deploy.jar`，将会自动打包所有依赖的
  jar 包

## Bazel 管理内部依赖

- 创建一个新类 Say ，并在 App 类中调用 Say 类
- 在 BUILD 文件中添加依赖

```
java_binary ( # 规则的名称；它需要额外的属性来构建二进制文件
    ...
    deps = ["//:say"], # 依赖项列表
)

java_library( # 依赖项
    name = "say",
    srcs = glob(["src/main/java/org/example/say/*.java"]),
    visibility = ["//visibility:public"], # 包的可见性
)
```

- 使用插件或 Bazel 构建项目 `bazel build //:say`
- 可以看到 `bazel-bin` 目录下多了一个 `libsay.jar`

## Bazel 管理外部依赖

- 比如添加外部依赖 `apache-commons-lang`，在 WORKSPACE 文件中添加

```
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_jar")

http_jar (
    name = "apache-commons-lang",
    url = "https://repo1.maven.org/maven2/org/apache/commons/commons-lang3/3.12.0/commons-lang3-3.12.0.jar"
)
```

- 最后在 BUILD 文件中添加外部依赖

```
java_binary ( # 规则的名称；它需要额外的属性来构建二进制文件
    ...
    deps = [..., "@apache-commons-lang//jar"], # 依赖项列表
)
```

- 管理单个 jar 是乏味的，可以使用 WORKSPACE 文件中的 `rules_jvm_external` 规则配置 Maven 仓库
- 先删除 WORKSPACE 文件中的 `http_jar` 规则
- 再使用 WORKSPACE 文件中的 `http_archive` 规则从远程位置导入 `rules_jvm_external` 规则：

```
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

RULES_JVM_EXTERNAL_TAG = "2.0.1"
RULES_JVM_EXTERNAL_SHA = "55e8d3951647ae3dffde22b4f7f8dee11b3f70f3f89424713debd7076197eaca"

http_archive(
    name = "rules_jvm_external",
    strip_prefix = "rules_jvm_external-%s" % RULES_JVM_EXTERNAL_TAG,
    sha256 = RULES_JVM_EXTERNAL_SHA,
    url = "https://github.com/bazelbuild/rules_jvm_external/archive/%s.zip" % RULES_JVM_EXTERNAL_TAG,
)
```

- 然后使用 WORKSPACE 文件中的 `maven_install` 规则并配置 Maven 存储库 URL 和所需的工件：

```
load("@rules_jvm_external//:defs.bzl", "maven_install")

maven_install(
    artifacts = ["org.apache.commons:commons-lang3:3.12.0"],
    repositories = ["https://repo1.maven.org/maven2"],
)
```

- 最后在 BUILD 文件中添加外部依赖，删除之前的 `@apache-commons-lang//jar` 依赖

```
java_binary ( # 规则的名称；它需要额外的属性来构建二进制文件
    ...
    deps = [..., "@maven//:org_apache_commons_commons_lang3"], # 依赖项列表，它使用下划线 (_) 字符解析工件的名称
)
```