# Riverpod-V3 - 高级特性

**页数:** 3

---

## Mutations（实验性）

**URL:** https://riverpod.dev/docs/concepts2/mutations

**内容:**
- Mutations（实验性）
- 定义 mutation
- 监听 mutation
  - 限定 mutation 作用域
  - 触发 mutation
  - 不同的 mutation 状态及其含义
  - mutation 启动后，如何将其重置为空闲状态？

Mutations 是实验性的，API 可能会在没有主版本号更新的情况下以破坏性方式更改。

在 Riverpod 中，Mutations 是使用户界面能够对状态变化做出反应的对象。一个常见的用例是在提交表单时显示加载指示器。

简而言之，mutations 用于实现这样的效果：!

如果没有 mutations，你必须将表单提交的进度直接存储在 provider 的状态中。这并不理想，因为它会用 UI 关注点污染 provider 的状态；并且涉及大量样板代码来处理加载状态、错误状态和成功状态。

Mutations 旨在以更优雅的方式处理这些问题。

Mutations 是 Mutation 对象的实例，存储在某处的 final 变量中。

通常，此变量将是全局变量或 Notifier 上的 static final 变量。

一旦我们定义了 mutation，就可以开始在 Consumers 或 Providers 中使用它。为此，我们需要一个 Refs 并选择我们选择的监听方法（通常是 Ref.watch）。

一个典型的例子是：

有时，你可能希望拥有同一 mutation 的多个实例。

这可以包括 id 或使 mutation 唯一的任何其他参数。

如果你想拥有同一 mutation 的多个实例，例如删除列表中的特定项，这很有用。

只需使用唯一键调用 mutation：

有时，这些 mutations 具有泛型返回类型，例如如果 api 响应可能根据输入参数具有不同的响应类型，例如反序列化。

到目前为止，我们已经监听了 mutation 的状态，但实际上还没有发生任何事情。

要触发 mutation，我们可以使用 Mutation.run，传递我们的 mutation，并提供一个异步回调来更新我们想要的任何状态。最后，我们需要返回一个与 mutation 的泛型类型匹配的值。

Mutations 可以处于以下状态之一：

你可以使用 switch 语句切换不同的状态：

Mutations 在以下情况下会自然地将自己重置为 MutationIdle：

这类似于自动销毁的工作方式，但用于 mutations。

或者，你可以通过调用 Mutation.reset 方法手动将 mutation 重置为其空闲状态：

**示例:**

示例 1 (dart):
```dart
// 用于跟踪"添加待办事项"操作的 mutation。
// 泛型类型是可选的，可以指定以使 UI 能够与 mutation 的结果交互。
final addTodo = Mutation<Todo>();
```

示例 2 (dart):
```dart
class Example extends ConsumerWidget {
  const Example({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 我们监听"addTodo" mutation 的当前状态。
    // 监听这个本身不会执行任何副作用。
    final addTodoState = ref.watch(addTodo);
    return Row(
      children: [
        ElevatedButton(
          style: ButtonStyle(
            // 如果有错误，我们将按钮显示为红色
            backgroundColor: switch (addTodoState) {
              MutationError() => const WidgetStatePropertyAll(Colors.red),
              _ => null,
            },
          ),
          onPressed: () {
            addTodo.run(ref, (tsx) async {
              // todo
            });
          },
          child: const Text('Add todo'),
        ),
        // 操作正在进行中，让我们显示一个进度指示器
        if (addTodoState is MutationPending) ...[
          const SizedBox(width: 8),
          const CircularProgressIndicator(),
        ],
      ],
    );
  }
}
```

示例 3 (dart):
```dart
final removeTodo = Mutation<void>();
final removeTodoWithId = removeTodo(todo.id);
```

示例 4 (dart):
```dart
final create = Mutation<ApiResponse>();
final createTodo = create<CreatedResponse<Todo>>('create_todo');
Future<void> executeCreateTodo(MutationTarget ref) async {
  await createTodo.run(ref, (tsx) async {
    final client = tsx.get(apiProvider);
    final response = client.post('/todos', data: {'title': 'Eat a cookie'});
    return CreatedResponse<Todo>.fromJson(response.data, Todo.fromJson);
  });
}
```

---

## 自动重试

**URL:** https://riverpod.dev/docs/concepts2/retry

**内容:**
- 自动重试
- 自定义重试逻辑
  - 禁用重试
- 关于默认重试逻辑
- 等待重试完成

在 Riverpod 中，Providers 在失败时会自动重试。

当 provider 的计算过程中抛出异常时，会尝试重试。重试逻辑可以在每个 provider 的基础上或全局为所有 providers 自定义。

默认情况下，provider 最多可以重试 10 次，指数退避从 200ms 到 6.4 秒。有关默认重试逻辑的完整详细信息，请参阅 retry。

可以为整个应用程序或特定 provider 提供自定义重试逻辑。

两种情况的实现相同：自定义重试逻辑是一个函数，预期返回 Duration? 值；它指示下次重试之前的延迟（或 null 以停止重试）。

以下实现了一个自定义重试函数，它将重试最多 5 次，指数退避从 200ms 开始，并忽略 ProviderExceptions：

然后可以在 providers 中使用此函数来更新该特定 provider 的重试逻辑：

或者通过将其传递给 ProviderContainers/ProviderScopes 来全局使用：

禁用重试就像在重试函数中始终返回 null 一样简单。如果你希望为所有应用程序禁用重试，请执行以下操作：

默认重试逻辑被设计为比简单的"如果失败，重试"更聪明。特别是，它不会重试 Errors 和 ProviderExceptions。

不会重试 Errors，因为它们是不可恢复的。它们表示代码中的错误，重试无济于事。在这些情况下重试只会用无用的重试尝试污染日志。

至于 ProviderExceptions，不会重试它们，因为它们表示 provider 没有失败，而是从失败的 provider 重新抛出异常。考虑：

在此示例中，尽管 myProvider 失败，但它不对失败负责。重试它无济于事。相反，应该重试 failedProvider。

这意味着如果你为 failedProvider 禁用重试，那么 myProvider 也不会被重试。

你可能知道可以通过使用 FutureProvider.future 来等待异步 providers 完成：

但你可能想知道自动重试如何与此交互。

简而言之，当异步 provider 失败并重试时，关联的 future 将继续等待，直到：

这确保了 await ref.watch(myProvider.future) 跳过中间失败。

**示例:**

示例 1 (dart):
```dart
Duration? myRetry(int retryCount, Object error) {
  // 在 ProviderException 上停止重试
  if (retryCount >= 5) return null;
  // 忽略 ProviderException
  if (error is ProviderException) return null;
  return Duration(milliseconds: 200 * (1 << retryCount)); // 指数退避}
```

示例 2 (dart):
```dart
final myProvider = Provider<int>(retry: myRetry, (ref) => 0);
```

示例 3 (dart):
```dart
@Riverpod(retry: myRetry)
int myProvider(MyProviderRef ref) {
  return 0;
}
```

示例 4 (dart):
```dart
// 对于纯 Dart 代码
final container = ProviderContainer(
retry: myRetry,);
...// 对于 Flutter 代码runApp(
ProviderScope(
retry: myRetry,
child: MyApp(),
),);
```

---

## 离线持久化（实验性）

**URL:** https://riverpod.dev/docs/concepts2/offline

**内容:**
- 离线持久化（实验性）
- 创建 Storage
- 持久化 provider 的状态
  - 使用简化的 JSON 序列化（代码生成）
- 理解持久化键
- 更改缓存持续时间
- 使用"销毁键"进行简单的数据迁移
- 等待持久化解码
- 测试持久化

离线持久化是将 Providers 的状态存储在用户设备上的能力，以便即使用户离线或应用重启时也可以访问它。

Riverpod 独立于用于存储数据的底层数据库或协议。但默认情况下，Riverpod 提供 riverpod_sqflite 以及基本的 JSON 序列化。

Riverpod 的离线持久化被设计为数据库的简单包装器。它不是为了完全替代与数据库交互的代码。

你可能仍然需要手动与数据库交互以：

离线持久化使用两个部分工作：

在开始持久化 notifiers 之前，我们需要实例化一个实现 Storage 接口的对象。此对象将负责将 Riverpod 与你的数据库连接。

你需要：

如果使用 SQFlite，可以使用 riverpod_sqflite：

然后，你可以通过实例化 JsonSqFliteStorage 来创建 Storage：

一旦我们创建了 Storage，就可以开始持久化 providers 的状态。目前，只有"Notifiers"可以被持久化。有关它们的更多信息，请参阅 Providers。

要持久化 notifier 的状态，你通常需要在 notifier 的 build 方法中调用 AnyNotifier.persist。

如果你使用 riverpod_sqflite 和代码生成，可以通过使用 JsonPersist 注解来简化 persist 调用：

在前面的一些代码片段中，我们向 AnyNotifier.persist 传递了一个 key 参数。该键使你的数据库能够知道在数据库中存储 provider 状态的位置。根据数据库，此键可能是唯一的行 ID。

指定 key 时，确保以下内容至关重要：

默认情况下，状态仅缓存 2 天。此默认值确保不会发生泄漏，并且已删除的 providers 不会无限期地保留在数据库中。

这通常是安全的，因为 Riverpod 被设计为主要用作 IO 操作（网络请求、数据库查询等）的缓存。但这样的默认值并不适合所有用例，例如如果你想存储用户首选项。

要更改此默认值，请像这样指定选项：

如果将缓存持续时间设置为无限，请确保在删除 provider 时手动从数据库中删除持久化状态。

为此，请参阅数据库的文档。

持久化数据时的一个常见挑战是处理数据结构更改时的情况。如果你更改对象的序列化方式，可能需要迁移存储在数据库中的数据。

虽然 Riverpod 不提供进行适当数据迁移的方法，但它确实提供了一种轻松用全新状态替换旧持久化状态的方法：销毁键。

销毁键通过使 Riverpod 能够知道何时应丢弃旧持久化状态来帮助进行简单的数据迁移。当发布具有不同 destroyKey 的新版本应用程序时，旧持久化状态将被丢弃，并且 provider 将被初始化，就好像它从未被持久化一样。

到目前为止，我们从未等待 AnyNotifier.persist 完成。这是自愿的，因为这允许 provider 尽快开始其网络请求。但是，这意味着 provider 在调用 persist 后无法轻松访问持久化状态。

在某些情况下，你可能希望使用持久化状态初始化 provider，而不是使用网络请求初始化 provider。

在这种情况下，你可以等待 persist 的结果，如下所示：

这使得可以在 build 中使用 this.state 访问持久化状态：

在测试应用程序时，使用真实数据库可能不方便。特别是，单元测试和 widget 测试将无法访问设备，因此无法使用数据库。

因此，Riverpod 提供了一种使用 Storage.inMemory 使用内存数据库的方法。要让测试使用此内存数据库，可以使用 Provider 覆盖：

**示例:**

示例 1 (bash):
```bash
dart pub add riverpod_sqflite sqflite
```

示例 2 (dart):
```dart
import 'package:flutter_riverpod/experimental/persist.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:path/path.dart';
import 'package:riverpod_sqflite/riverpod_sqflite.dart';
import 'package:sqflite/sqflite.dart';

final storageProvider = FutureProvider<Storage<String, String>>((ref) async {
  // 初始化 SQFlite。我们应该在 providers 之间共享 Storage 实例。
  return JsonSqFliteStorage.open(join(await getDatabasesPath(), 'riverpod.db'));
});
```

示例 3 (dart):
```dart
import 'package:flutter_riverpod/experimental/persist.dart';
import 'package:path/path.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'package:riverpod_sqflite/riverpod_sqflite.dart';
import 'package:sqflite/sqflite.dart';
part 'codegen.g.dart';

@riverpod
Future<Storage<String, String>> storage(Ref ref) async {
  // 初始化 SQFlite。我们应该在 providers 之间共享 Storage 实例。
  return JsonSqFliteStorage.open(join(await getDatabasesPath(), 'riverpod.db'));
}
```

示例 4 (dart):
```dart
class Todo {
  Todo({required this.task});
  final String task;
}

final todoListProvider = AsyncNotifierProvider<TodoList, List<Todo>>(
  TodoList.new,
);

class TodoList extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    persist(
      // 我们传入之前创建的 Storage。
      // 不要"await"这个。Riverpod 会为你处理它。
      ref.watch(storageProvider.future),
      // 此状态的唯一标识符。
      // 如果你的 provider 接收参数，请确保在键中也编码这些参数。
      key: 'todo_list',
      // 编码/解码状态。这里，我们使用基本的 JSON 编码。
      // 你可以使用任何你想要的编码，只要你的 Storage 支持它。
      encode: (todos) => todos.map((todo) => {'task': todo.task}).toList(),
      decode: (json) => (json as List)
          .map((todo) => Todo(task: todo['task'] as String))
          .toList(),
    );
    // 无论是否恢复了某些状态，我们都从服务器获取待办事项列表。
    return fetchTodosFromServer();
  }
}
```

---
