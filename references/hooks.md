# Riverpod-V3 - Hooks 集成

**页数:** 1

---

## 关于 Hooks

**URL:** https://riverpod.dev/docs/concepts/about_hooks

**内容:**
- 关于 hooks
- 你应该使用 hooks 吗？
- 什么是 hooks？
- Hooks 的规则
- Hooks 与 Riverpod
  - 安装
  - 使用

本页面解释了什么是 hooks 以及它们与 Riverpod 的关系。

"Hooks" 是来自独立包的实用工具，独立于 Riverpod：flutter_hooks。尽管 flutter_hooks 是一个完全独立的包，与 Riverpod 没有任何关系（至少不是直接关系），但通常会将 Riverpod 和 flutter_hooks 配对使用。

Hooks 是一个强大的工具，但并不适合所有人。如果你是 Riverpod 的新手，请避免使用 hooks。

尽管有用，但 hooks 对于 Riverpod 来说不是必需的。你不应该因为 Riverpod 而开始使用 hooks。相反，你应该因为想要使用 hooks 而开始使用 hooks。

使用 hooks 是一种权衡。它们可以很好地生成健壮和可重用的代码，但它们也是一个需要学习的新概念，一开始可能会令人困惑。Hooks 不是 Flutter 的核心概念。因此，它们在 Flutter/Dart 中会感觉格格不入。

Hooks 是在 widgets 内部使用的函数。它们被设计为 StatefulWidgets 的替代方案，使逻辑更具可重用性和可组合性。

Hooks 是来自 React 的概念，flutter_hooks 只是 React 实现到 Flutter 的移植。因此，是的，hooks 在 Flutter 中可能会感觉有点格格不入。理想情况下，在未来我们会有一个专门为 Flutter 设计的解决方案来解决 hooks 解决的问题。

如果 Riverpod 的 providers 用于"全局"应用状态，那么 hooks 用于本地 widget 状态。Hooks 通常用于处理有状态的 UI 对象，例如 TextEditingController、AnimationController。它们还可以作为"builder"模式的替代品，用不涉及"嵌套"的替代方案替换 FutureBuilder/TweenAnimatedBuilder 等 widgets - 大大提高可读性。

一般来说，hooks 有助于：

例如，我们可以使用 hooks 手动实现淡入动画，其中 widget 开始时不可见并慢慢出现。

如果我们使用 StatefulWidget，代码将如下所示：

使用 hooks，等效代码为：

在这段代码中有几个有趣的地方需要注意：

没有内存泄漏。此代码不会在 widget 重建时重新创建新的 AnimationController，并且在 widget 卸载时控制器会被正确释放。

可以在同一个 widget 中多次使用 hooks。因此，如果我们想要，可以创建多个 AnimationController：

这会创建两个控制器，没有任何负面后果。

如果我们愿意，可以将此逻辑重构为一个单独的可重用函数：

然后我们可以在 widgets 中使用这个函数，只要该 widget 是 HookWidget：

注意我们的 useFadeIn 函数完全独立于我们的 FadeIn widget。如果我们愿意，可以在完全不同的 widget 中使用该 useFadeIn 函数，它仍然可以工作！

Hooks 有独特的约束：

它们只能在扩展 HookWidget 的 widget 的 build 方法中使用：

它们不能有条件地使用或在循环中使用。

有关 hooks 的更多信息，请参阅 flutter_hooks。

由于 hooks 独立于 Riverpod，因此需要单独安装 hooks。如果你想使用它们，仅安装 hooks_riverpod 是不够的。你仍然需要将 flutter_hooks 添加到依赖项中。有关更多信息，请参阅快速开始。

在某些情况下，你可能想要编写一个同时使用 hooks 和 Riverpod 的 Widget。但正如你可能已经注意到的，hooks 和 Riverpod 都提供了自己的自定义 widget 基类型：HookWidget 和 ConsumerWidget。但类一次只能扩展一个超类。

要解决这个问题，你可以使用 hooks_riverpod 包。此包提供了一个 HookConsumerWidget 类，它将 HookWidget 和 ConsumerWidget 组合成一个类型。因此，你可以子类化 HookConsumerWidget 而不是 HookWidget：

或者，你可以使用两个包提供的"builders"。例如，我们可以坚持使用 StatelessWidget，并同时使用 HookBuilder 和 Consumer。

这种方法无需使用 hooks_riverpod 即可工作。只需要 flutter_riverpod。

如果你喜欢这种方法，hooks_riverpod 通过提供 HookConsumer 来简化它，它是两个 builders 的组合：

**示例:**

示例 1 (dart):
```dart
class FadeIn extends StatefulWidget {  const FadeIn({Key? key, required this.child}) : super(key: key);  final Widget child;  @override  State<FadeIn> createState() => _FadeInState();}class _FadeInState extends State<FadeIn> with SingleTickerProviderStateMixin {  late final AnimationController animationController = AnimationController(    vsync: this,    duration: const Duration(seconds: 2),  );  @override  void initState() {    super.initState();    animationController.forward();  }  @override  void dispose() {    animationController.dispose();    super.dispose();  }  @override  Widget build(BuildContext context) {    return AnimatedBuilder(      animation: animationController,      builder: (context, child) {        return Opacity(          opacity: animationController.value,          child: widget.child,        );      },    );  }}
```

示例 2 (dart):
```dart
class FadeIn extends HookWidget {  const FadeIn({Key? key, required this.child}) : super(key: key);  final Widget child;  @override  Widget build(BuildContext context) {    // 创建 AnimationController。当 widget 卸载时，控制器会自动销毁。    final animationController = useAnimationController(      duration: const Duration(seconds: 2),    );    // useEffect 相当于 initState + didUpdateWidget + dispose。    // 传递给 useEffect 的回调在第一次调用 hook 时执行，    // 然后在作为第二个参数传递的列表发生变化时执行。    // 由于我们在这里传递了一个空的 const 列表，这严格等同于 `initState`。    useEffect(() {      // 在 widget 首次渲染时启动动画。      animationController.forward();      // 我们可以选择在这里返回一些"dispose"逻辑      return null;    }, const []);    // 告诉 Flutter 在动画更新时重建此 widget。    // 这相当于 AnimatedBuilder    useAnimation(animationController);    return Opacity(      opacity: animationController.value,      child: child,    );  }}
```

示例 3 (dart):
```dart
@overrideWidget build(BuildContext context) {  final animationController = useAnimationController(    duration: const Duration(seconds: 2),  );  final anotherController = useAnimationController(    duration: const Duration(seconds: 2),  );  ...}
```

示例 4 (dart):
```dart
double useFadeIn() {  final animationController = useAnimationController(    duration: const Duration(seconds: 2),  );  useEffect(() {    animationController.forward();    return null;  }, const []);  useAnimation(animationController);  return animationController.value;}
```

---
