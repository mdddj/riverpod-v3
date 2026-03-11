# Riverpod-V3 - 代码生成

**页数:** 1

---

## 关于代码生成

**URL:** https://riverpod.dev/docs/concepts/about_code_generation

**内容:**
- 关于代码生成
- 我应该使用代码生成吗？
- 使用代码生成有什么好处？
- 语法
  - 定义 provider
  - 启用/禁用 autoDispose
  - 向 provider 传递参数 (family)
- 从非代码生成版本迁移

代码生成是使用工具为我们生成代码的想法。在 Dart 中，它的缺点是需要额外的步骤来"编译"应用程序。尽管这个问题可能在不久的将来得到解决，因为 Dart 团队正在研究潜在的解决方案。

在 Riverpod 的上下文中，代码生成是关于稍微改变定义 "provider" 的语法。例如，不使用：

使用代码生成，我们会写：

在使用 Riverpod 时，代码生成是完全可选的。完全可以在不使用代码生成的情况下使用 Riverpod。同时，Riverpod 拥抱代码生成并推荐使用它。

有关如何安装和使用 Riverpod 的代码生成器的信息，请参阅快速开始页面。确保在文档侧边栏中启用代码生成。

代码生成在 Riverpod 中是可选的。考虑到这一点，你可能想知道是否应该使用它。

答案是：只有当你已经在其他地方使用代码生成时。（参考 Freezed、json_serializable 等）当 Dart 团队在开发一个名为 "macros" 的功能时，使用代码生成是使用 Riverpod 的推荐方式。不幸的是，这些已经被取消了。

虽然代码生成带来了许多好处，但目前它仍然相当慢。Dart 团队正在努力提高代码生成的性能，但目前还不清楚何时可用以及会有多大改进。因此，如果你的项目中还没有使用代码生成，那么仅仅为了 Riverpod 而开始使用它可能不值得。

同时，许多应用程序已经使用 Freezed 或 json_serializable 等包进行代码生成。在这种情况下，你的项目可能已经设置好了代码生成，使用 Riverpod 应该很简单。

你可能想知道："如果代码生成在 Riverpod 中是可选的，为什么要使用它？"

与包一样：让你的生活更轻松。这包括但不限于：

在使用代码生成定义 provider 时，记住以下几点会很有帮助：

使用代码生成时，providers 默认是 autoDispose 的。这意味着当没有监听器附加到它们时（ref.watch/ref.listen），它们会自动销毁自己。这个默认设置更符合 Riverpod 的理念。最初在非代码生成版本中，autoDispose 默认是关闭的，以适应从 package:provider 迁移的用户。

如果你想禁用 autoDispose，可以通过向注解传递 keepAlive: true 来实现。

使用代码生成时，我们不再需要依赖 family 修饰符来向 provider 传递参数。相反，我们的 provider 的主函数可以接受任意数量的参数，包括命名参数、可选参数或默认值。但请注意，这些参数仍应具有一致的 ==。这意味着要么缓存值，要么参数应该重写 ==。

使用非代码生成版本时，需要手动确定 provider 的类型。以下是过渡到代码生成版本的相应选项：

**示例:**

示例 1 (dart):
```dart
final fetchUserProvider = FutureProvider.autoDispose.family<User, int>((
  ref,
  userId,
) async {
  final json = await http.get('api/user/$userId');
  return User.fromJson(json);
});
```

示例 2 (dart):
```dart
@riverpod
Future<User> fetchUser(Ref ref, {required int userId}) async {
  final json = await http.get('api/user/$userId');
  return User.fromJson(json);
}
```

示例 3 (dart):
```dart
@riverpod
String example(Ref ref) {
  return 'foo';
}
```

示例 4 (dart):
```dart
@riverpod
class Example extends _$Example {
  @override
  String build() {
    return 'foo';
  }
  // 添加方法来修改状态}
```

---
