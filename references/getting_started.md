# Riverpod-V3 - 快速开始

**页数:** 1

---

## 快速开始

**URL:** https://riverpod.dev/docs/introduction/getting_started

**内容:**
- 快速开始
- 在线试用 Riverpod
- 安装包
- 启用 riverpod_lint
- 使用示例: Hello world
- 进阶: 安装代码片段

要体验 Riverpod，可以在 Dartpad 或 Zapp 上在线试用：

Riverpod 由一个独立的主包 "riverpod" 组成，并配有可选的代码生成包（关于代码生成）和 hooks 包（关于 hooks）。

确定要安装哪些包后，只需一行命令即可添加依赖：

或者，你也可以在 pubspec.yaml 中手动添加依赖：

然后使用 flutter pub get 安装包。

然后使用 dart pub get 安装包。

如果使用代码生成，现在可以运行代码生成器：

完成！你已经将 Riverpod 添加到应用中了。

Riverpod 提供了可选的 riverpod_lint 包，它提供 lint 规则来帮助你编写更好的代码，并提供自定义重构选项。

Riverpod_lint 使用 analysis_server_plugin 实现。因此，它通过 analysis_options.yaml 安装。

简而言之，在 pubspec.yaml 旁边创建一个 analysis_options.yaml 文件并添加：

现在，如果你在使用 Riverpod 时犯了错误，IDE 中应该会显示警告。

要查看完整的警告和重构列表，请访问 riverpod_lint 页面。

现在我们已经安装了 Riverpod，可以开始使用它了。

以下代码片段展示了如何使用新依赖来创建一个 "Hello world"：

然后使用 flutter run 启动应用。这将在你的设备上渲染 "Hello world"。

然后使用 dart lib/main.dart 启动应用。这将在控制台打印 "Hello world"。

如果你使用 Flutter 和 VS Code，可以考虑使用 Flutter Riverpod Snippets

如果你使用 Flutter 和 Android Studio 或 IntelliJ，可以考虑使用 Flutter Riverpod Snippets

**示例:**

示例 1 (bash):
```bash
flutter pub add flutter_riverpod
```

示例 2 (bash):
```bash
flutter pub add hooks_riverpod
flutter pub add flutter_hooks
```

示例 3 (bash):
```bash
flutter pub add flutter_riverpod
flutter pub add riverpod_annotation
flutter pub add dev:riverpod_generator
flutter pub add dev:build_runner
```

示例 4 (bash):
```bash
flutter pub add hooks_riverpod
flutter pub add flutter_hooks
flutter pub add riverpod_annotation
flutter pub add dev:riverpod_generator
flutter pub add dev:build_runner
```

---
