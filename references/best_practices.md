# Riverpod-V3 - 最佳实践

**页数:** 1

---

## DO/DON'T（推荐/不推荐）

**URL:** https://riverpod.dev/docs/root/do_dont

**内容:**
- DO/DON'T
- 避免在 widget 中初始化 providers
- 避免将 providers 用于临时状态
- 不要在 provider 初始化期间执行副作用
- 优先使用静态已知 providers 的 ref.watch/read/listen（和类似 API）
- 避免动态创建 providers

为了确保代码的良好可维护性，以下是使用 Riverpod 时应遵循的良好实践列表。

此列表并非详尽无遗，并且可能会更改。如果你有任何建议，请随时提出问题。

此列表中的项目没有特定顺序。

这些建议中的很大一部分可以通过 riverpod_lint 强制执行。有关安装说明，请参阅快速开始。

Providers 应该自己初始化。它们不应该由外部元素（如 widget）初始化。

如果不这样做，可能会导致可能的竞态条件和意外行为。

这个问题没有"一刀切"的解决方案。如果你的初始化逻辑依赖于 provider 外部的因素，通常正确的位置是在触发导航的按钮的 onPressed 方法中：

Providers 旨在用于共享业务状态。它们不应该用于临时状态，例如：

如果你正在寻找一种处理本地 widget 状态的方法，请考虑使用 flutter_hooks。

不鼓励这样做的一个原因是，这种状态通常限定在路由范围内。如果不这样做，可能会破坏应用的返回按钮，因为新页面会覆盖前一页的状态。

例如，假设我们要在 provider 中存储当前选定的书籍：

我们可能面临的一个挑战是，导航历史可能如下所示：

在这种情况下，当按下返回按钮时，我们应该期望返回到 /books/42。但是，如果我们使用 selectedBookProvider 来存储选定的书籍，选定的 ID 不会重置为其先前的值，我们会继续看到 /books/21。

Providers 通常应该用于表示"读取"操作。你不应该将它们用于"写入"操作，例如提交表单。

将 providers 用于此类操作可能会产生意外行为，例如如果执行了先前的副作用，则跳过副作用。

如果你正在寻找一种处理副作用的加载/错误状态的方法，请参阅 Mutations（实验性）。

Riverpod 强烈建议启用 lint 规则（通过 riverpod_lint）。但是，为了使 lints 有效，你的代码应该以可静态分析的方式编写。

如果不这样做，可能会使发现错误变得更加困难，或者导致 lints 出现误报。

Providers 应该专门是顶级 final 变量。

允许将 providers 创建为 static final 变量，但代码生成器不支持。

**示例:**

示例 1 (dart):
```dart
class WidgetState extends State<MyWidget> {  @override  void initState() {    super.initState();    // 不好：provider 应该自己初始化    ref.read(provider).init();  }}
```

示例 2 (dart):
```dart
ElevatedButton(  onPressed: () {    ref.read(provider).init();    Navigator.of(context).push(...);  },  child: Text('Navigate'),)
```

示例 3 (dart):
```dart
final selectedBookProvider = StateProvider<String?>((ref) => null);
```

示例 4 (dart):
```dart
/books/books/42/books/21
```

---
