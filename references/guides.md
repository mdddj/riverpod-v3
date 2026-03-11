# Riverpod-V3 - 指南

**页数:** 1

---

## 你的第一个 Riverpod 应用

**URL:** https://riverpod.dev/docs/tutorials/first_app

**内容:**
- 你的第一个 Riverpod 应用
- 关键要点
- 设置项目
  - 创建 Flutter 项目
  - 创建模拟 UI
  - 将 Riverpod 添加到项目
  - （可选）添加 riverpod_lint
  - 在 main 函数中添加 ProviderScope
- 创建模型类
- 编写调用 API 的函数

在本教程中，我们将使用 Riverpod 构建一个随机笑话生成器应用：

首先，让我们创建一个新的 Flutter 项目：

然后，在你喜欢的编辑器中打开项目。

在开始编写任何形式的逻辑之前，让我们创建应用的 UI。我们将从静态数据开始，而不是使用真实的 API。

让我们在项目的 lib 目录中创建一个名为 home.dart 的新文件。在其中，你可以粘贴以下代码：

然后，我们可以更新 main.dart 文件以使用这个新的 HomeView widget：

如果你现在运行应用，你应该看到以下内容：

创建项目后，我们需要将 Riverpod 添加为依赖项。

我们将使用 Riverpod 和 Flutter，因此我们将安装 flutter_riverpod 包。同样，我们将使用 Dio 包执行网络请求，因此我们也将安装它。

你可以在终端中输入以下命令来执行此操作：

这将把最新版本的 Riverpod 和 Dio 添加到你的项目中。

为了帮助你编写更好的 Riverpod 代码，你可以安装 riverpod_lint 包。此包提供了一组重构，以更轻松地编写 Riverpod 代码，以及一组 lints 来帮助你避免常见错误。

Riverpod_lint 使用 analysis_server_plugin 实现。因此，它通过 analysis_options.yaml 安装。

简而言之，在 pubspec.yaml 旁边创建一个 analysis_options.yaml 文件并添加：

为了让 Riverpod 工作，我们需要更新 main 函数以包含 ProviderScope。你可以在 ProviderContainers/ProviderScopes 部分了解这些对象。

以下是更新后的 main 函数：

在本教程中，我们将从随机笑话生成器 API 获取数据。

此 API 返回一个如下所示的 JSON 对象：

为了在我们的应用中表示这些数据，我们将创建一个名为 Joke 的模型类。

为此，让我们在项目的 lib 目录中创建一个名为 joke.dart 的新文件。Joke 类如下所示：

注意 fromJson 工厂构造函数。由于我们的 API 返回一个 JSON 对象，我们需要一种将 JSON 数据转换为 Joke 类的方法。此构造函数接受一个 Map<String, Object?> 并返回一个 Joke 实例。

现在我们有了模型类，我们可以编写一个从 API 获取数据的函数。我们将在这里使用 Dio 包，因为如果请求失败，它会自然地抛出异常，这对我们的用例很方便。但你可以使用任何你喜欢的 HTTP 客户端。

我们可以将该逻辑放在我们刚刚创建的 joke.dart 文件中，因为这个逻辑与 Joke 类密切相关。

注意我们没有捕获 API 调用的任何错误。这是故意的。Riverpod 会为我们处理错误，所以我们不需要手动处理。

现在我们有了一个查询 API 的函数，我们可以创建一个负责缓存该 API 结果的 "provider"。有关它们的更多信息，请参阅 Providers。

由于我们的 fetchRandomJoke 函数返回一个 Future<Joke>，我们将使用 FutureProvider。我们可以将 provider 放在同一个 joke.dart 文件中，因为它也与 Joke 类相关。

通过这样做，fetchRandomJoke 的执行将被缓存，无论我们访问该值多少次，网络请求只会执行一次。

fetchRandomJoke 函数和 randomJokeProvider 之间的分离不是强制性的。如果你愿意，可以直接在 provider 中编写 fetchRandomJoke 的内容：

现在我们有了一个 provider，是时候更新我们的 HomeView widget 以动态加载数据了。

为此，我们需要 Riverpod 的另一个功能：Consumer widget。这个 widget 允许我们读取 provider 的值，并在值更改时重建 UI。它的使用方式让人想起 StreamBuilder 等 widget。

具体来说，我们将希望将 Stack 封装在 Consumer widget 中。如果你在前面的步骤中安装了 riverpod_lint，你可以使用其中一个内置的重构：

更新后的 home.dart 代码应该如下所示：

现在我们有了一个 Consumer，我们可以使用它的 ref 参数来读取我们的 provider。使用这个对象，我们可以调用 ref.watch(randomJokeProvider) 来获取 provider 的当前值。但还有其他与 providers 交互的方法！有关更多信息，请参阅 Refs。

我们更新后的 Consumer 应该如下所示：

通过这一行，Riverpod 将自动从我们的 API 获取笑话并缓存结果。我们现在可以使用 randomJoke 变量在 UI 中显示笑话。

我们之前创建的 randomJoke 变量不是 Joke 类型，而是 AsyncValue<Joke> 类型。AsyncValue 是一个 Riverpod 类型，表示异步操作的状态，例如网络请求。它包括有关加载、成功和错误状态的信息。AsyncValue 在许多方面类似于 StreamBuilder 中使用的 AsyncSnapshot 类型。

处理不同状态的一种便捷方法是使用 Dart 的 switch 功能。它类似于 if/else if 链，但专门用于处理一个特定对象的条件。

与 AsyncValue 结合使用时的常见方法如下：

操作顺序很重要！如果使用上面使用的语法，重要的是在检查错误之前检查值，并最后处理加载状态。

如果使用不同的顺序，你可能会看到不正确的行为，例如在请求已经完成时显示进度指示器。

我们现在可以更新我们的 Stack 以根据 randomJoke 的状态显示笑话、加载指示器或错误消息：

在这个阶段，我们的应用程序已连接到互联网，并在应用启动时显示一个随机笑话！

目前，我们在应用启动时显示一个随机笑话，但单击按钮什么也不做。让我们更新按钮以在单击时获取新笑话。

我们可以使用类似于 ChangeNotifier 的模式并手动处理状态。Riverpod 支持这种模式，但在这里不是必需的。

相反，我们可以告诉 Riverpod 在单击按钮时重新执行 provider 的逻辑。这可以通过使用 Ref.invalidate 来完成，如下所示：

这就是我们需要做的全部！当单击按钮时，Riverpod 将重新执行 randomJokeProvider 的逻辑，这将从 API 获取新笑话并相应地更新 UI。

你可能已经注意到，当单击"获取另一个笑话"按钮时，应用不会显示任何加载指示器。

这是因为当我们调用 Ref.invalidate 时，现有的缓存不会被销毁。相反，在获取新笑话时，我们保留有关先前笑话的信息。这使我们能够在获取新笑话时显示先前的笑话。

但是，UI 可能希望处理这些情况并同时显示加载指示器和先前的笑话。LinearProgressIndicator 是一种常见的方法。要添加此指示器，我们可以检查 AsyncValue.isRefreshing。当旧数据可用且正在进行新请求时，此标志为 true。

我们更新后的 Stack 应该如下所示：

就是这样！我们现在有一个功能齐全的随机笑话生成器应用，它从 API 获取笑话并在 UI 中显示它们。我们已经处理了所有边缘情况，例如加载和错误状态。

注意我们从未编写过 try/catch 或编写诸如 isLoading = true/false 之类的代码。

**示例:**

示例 1 (unknown):
```unknown
flutter create first_app
```

示例 2 (dart):
```dart
import 'package:flutter/material.dart';class HomeView extends StatelessWidget {  const HomeView({super.key});  @override  Widget build(BuildContext context) {    return Scaffold(      appBar: AppBar(title: const Text('Random Joke Generator')),      body: SizedBox.expand(        child: Stack(          alignment: Alignment.center,          children: [            const SelectableText(              'What kind of bagel can fly?\n\n'              'A plain bagel.',              textAlign: TextAlign.center,              style: TextStyle(fontSize: 24),            ),            Positioned(              bottom: 20,              child: ElevatedButton(                onPressed: () {},                child: const Text('Get another joke'),              ),            ),          ],        ),      ),    );  }}
```

示例 3 (dart):
```dart
import 'package:flutter/material.dart';import 'home.dart';void main() {  runApp(const MyApp());}class MyApp extends StatelessWidget {  const MyApp({super.key});  @override  Widget build(BuildContext context) {    return const MaterialApp(home: HomeView());  }}
```

示例 4 (unknown):
```unknown
flutter pub add flutter_riverpod dio
```

---
