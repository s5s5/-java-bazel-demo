java_binary ( # 规则的名称；它需要额外的属性来构建二进制文件
    name = "java-bazel-demo", # 构建目标的名称
    srcs = glob(["src/main/java/org/example/*.java"]), # 一个文件位置模式数组，用于告诉要构建哪些 Java 文件
    main_class = "src.main.java.org.example.App", # 应用程序主类的名称（可选）
#    deps = ["//:say", "@apache-commons-lang//jar"], # 依赖项列表
    deps = ["//:say", "@maven//:org_apache_commons_commons_lang3"], # 依赖项列表，它使用下划线 (_) 字符解析工件的名称
)

java_library( # 依赖项
    name = "say",
    srcs = glob(["src/main/java/org/example/say/*.java"]),
    visibility = ["//visibility:public"], # 包的可见性
)