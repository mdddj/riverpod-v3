---
name: riverpod-v3
description: Riverpod v3 - Flutter 响应式状态管理框架。包含 Provider、Consumer、自动销毁、Family、Mutations、离线持久化、代码生成等核心概念和最佳实践。
version: 3.1.0
keywords: flutter, dart, state management, riverpod, provider, reactive, notifier
---

# Riverpod v3 状态管理框架

Riverpod 是 Flutter/Dart 的响应式缓存和数据绑定框架，v3 版本带来了重大改进，包括简化的 API、实验性的 Mutations 和离线持久化支持。

> Riverpod = Provider 的完全重写版本，提供更强大的功能和更好的开发体验。

## 核心特性

### 基础能力
- **响应式状态管理**: 自动追踪依赖，状态变化时自动重建 UI
- **类型安全**: 编译时检查，避免运行时错误
- **可测试性**: 轻松 mock 和覆盖 providers，无需复杂配置
- **代码生成**: 可选的 `riverpod_generator` 简化代码编写

### 高级特性
- **自动销毁 (Auto Dispose)**: 智能管理资源生命周期，防止内存泄漏
- **Family**: 参数化 providers，支持动态参数
- **自动重试**: 失败时自动重试，指数退避策略
- **Ref.mounted**: 类似 BuildContext.mounted，检查 provider 是否仍然存活

### 实验性特性 (v3.0+)
- **Mutations**: 简化异步操作的加载/错误状态管理
- **离线持久化**: 本地缓存支持，应用重启后恢复状态

## 何时使用此技能

### 适用场景
- ✅ 使用 Riverpod v3 进行 Flutter 状态管理
- ✅ 创建和使用 Provider、Notifier、AsyncNotifier
- ✅ 处理异步数据 (FutureProvider, AsyncNotifier)
- ✅ 实现依赖注入和单元测试
- ✅ 从 v2 迁移到 v3
- ✅ 使用代码生成简化开发
- ✅ 实现离线缓存和数据持久化
- ✅ 处理复杂的异步操作和副作用

### 不适用场景
- ❌ 简单的临时 UI 状态（如表单输入、动画控制器）
- ❌ 仅限于单个 widget 的状态（考虑使用 StatefulWidget 或 flutter_hooks）

## 快速开始

### 安装依赖

#### 基础安装（不使用代码生成）
```bash
# Flutter 项目
flutter pub add flutter_riverpod
```

#### 推荐安装（使用代码生成）
```bash
# 添加核心依赖
flutter pub add flutter_riverpod riverpod_annotation

# 添加开发依赖
flutter pub add dev:riverpod_generator dev:build_runner dev:riverpod_lint

# 运行代码生成（监听模式）
dart run build_runner watch -d
```

### pubspec.yaml 配置

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^3.1.0
  riverpod_annotation: ^4.0.0

dev_dependencies:
  build_runner: ^2.4.0
  riverpod_generator: ^4.0.0+1
  riverpod_lint: ^3.0.0
  custom_lint: ^0.7.0
```

### 启用 riverpod_lint（推荐）

在项目根目录创建 `analysis_options.yaml`:

```yaml
analyzer:
  plugins:
    - custom_lint
```

### Hello World 示例

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 1. 创建一个 Provider（全局声明）
final helloWorldProvider = Provider<String>((ref) => 'Hello world');

void main() {
  runApp(
    // 2. 用 ProviderScope 包裹应用（必需）
    const ProviderScope(child: MyApp()),
  );
}

// 3. 使用 ConsumerWidget 代替 StatelessWidget
class MyApp extends ConsumerWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 4. 使用 ref.watch 监听 provider
    final String value = ref.watch(helloWorldProvider);

    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Riverpod Example')),
        body: Center(child: Text(value, style: const TextStyle(fontSize: 24))),
      ),
    );
  }
}
```

**关键点**:
- `ProviderScope` 必须包裹整个应用
- 使用 `ConsumerWidget` 或 `Consumer` 来访问 providers
- `ref.watch` 会在状态变化时自动重建 widget

## Provider 类型

Riverpod 提供 6 种 Provider 类型，根据返回值类型和是否可修改来选择：

| 类型 | 返回值 | 可修改 | 使用场景 |
|------|--------|--------|----------|
| `Provider` | 同步值 | ❌ | 配置、依赖注入、计算值 |
| `FutureProvider` | `Future<T>` | ❌ | 异步数据获取（一次性） |
| `StreamProvider` | `Stream<T>` | ❌ | 实时数据流 |
| `NotifierProvider` | 同步值 | ✅ | 可变状态管理 |
| `AsyncNotifierProvider` | `Future<T>` | ✅ | 异步可变状态 |
| `StreamNotifierProvider` | `Stream<T>` | ✅ | 流式可变状态 |

### 基础 Provider（不可修改）

#### Provider - 同步值

```dart
// 手动定义
final nameProvider = Provider<String>((ref) => 'John');

// 代码生成（推荐）
@riverpod
String name(Ref ref) => 'John';

// 依赖其他 provider
@riverpod
String greeting(Ref ref) {
  final name = ref.watch(nameProvider);
  return 'Hello, $name!';
}
```

#### FutureProvider - 异步数据

```dart
// 手动定义
final userProvider = FutureProvider<User>((ref) async {
  final response = await http.get(Uri.parse('api/user'));
  return User.fromJson(jsonDecode(response.body));
});

// 代码生成（推荐）
@riverpod
Future<User> user(Ref ref) async {
  final response = await http.get(Uri.parse('api/user'));
  return User.fromJson(jsonDecode(response.body));
}

// 在 Widget 中使用
class UserWidget extends ConsumerWidget {
  const UserWidget({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return switch (userAsync) {
      AsyncData(:final value) => Text('Hello ${value.name}'),
      AsyncError(:final error) => Text('Error: $error'),
      _ => const CircularProgressIndicator(),
    };
  }
}
```

#### StreamProvider - 实时数据流

```dart
// 手动定义
final messagesProvider = StreamProvider<List<Message>>((ref) {
  return firestore
      .collection('messages')
      .snapshots()
      .map(
        (snapshot) => snapshot.docs.map((doc) => Message.fromDoc(doc)).toList(),
      );
});

// 代码生成
@riverpod
Stream<List<Message>> messages(Ref ref) {
  return firestore
      .collection('messages')
      .snapshots()
      .map(
        (snapshot) => snapshot.docs.map((doc) => Message.fromDoc(doc)).toList(),
      );
}
```

### Notifier（可修改状态）

#### NotifierProvider - 同步可变状态

```dart
// 手动定义
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0; // 初始状态

  void increment() => state++;

  void decrement() => state--;

  void reset() => state = 0;
}

final counterProvider = NotifierProvider<CounterNotifier, int>(
  CounterNotifier.new,
);

// 代码生成（推荐）
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0; // 初始状态

  void increment() => state++;

  void decrement() => state--;

  void reset() => state = 0;
}

// 在 Widget 中使用
class CounterWidget extends ConsumerWidget {
  const CounterWidget({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 监听状态
    final count = ref.watch(counterProvider);

    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          // 调用方法修改状态
          onPressed: () => ref.read(counterProvider.notifier).increment(),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

#### AsyncNotifierProvider - 异步可变状态

```dart
// 代码生成
@riverpod
class TodoList extends _$TodoList {
  @override
  Future<List<Todo>> build() async {
    // 初始化：从 API 获取数据
    return fetchTodos();
  }

  // 添加 todo
  Future<void> addTodo(Todo todo) async {
    // 设置为加载状态
    state = const AsyncLoading();

    // 执行异步操作
    state = await AsyncValue.guard(() async {
      await saveTodo(todo);
      return [...state.value!, todo];
    });
  }

  // 删除 todo
  Future<void> removeTodo(String id) async {
    state = const AsyncLoading();

    state = await AsyncValue.guard(() async {
      await deleteTodo(id);
      return state.value!.where((todo) => todo.id != id).toList();
    });
  }

  // 切换完成状态
  Future<void> toggleTodo(String id) async {
    // 乐观更新：立即更新 UI
    final previousState = state;
    state = AsyncData(
      state.value!.map((todo) {
        if (todo.id == id) {
          return todo.copyWith(completed: !todo.completed);
        }
        return todo;
      }).toList(),
    );

    // 执行 API 调用
    try {
      await updateTodo(id);
    } catch (e) {
      // 失败时回滚
      state = previousState;
    }
  }
}

// 在 Widget 中使用
class TodoListWidget extends ConsumerWidget {
  const TodoListWidget({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todosAsync = ref.watch(todoListProvider);

    return switch (todosAsync) {
      AsyncData(:final value) => ListView.builder(
        itemCount: value.length,
        itemBuilder: (context, index) {
          final todo = value[index];
          return ListTile(
            title: Text(todo.title),
            trailing: IconButton(
              icon: const Icon(Icons.delete),
              onPressed: () {
                ref.read(todoListProvider.notifier).removeTodo(todo.id);
              },
            ),
          );
        },
      ),
      AsyncError(:final error) => Text('Error: $error'),
      _ => const CircularProgressIndicator(),
    };
  }
}
```

## 在 Widget 中使用

### ConsumerWidget（推荐）

```dart
class MyWidget extends ConsumerWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Scaffold(
      body: Center(child: Text('Count: $count')),
      floatingActionButton: FloatingActionButton(
        onPressed: () => ref.read(counterProvider.notifier).increment(),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### Consumer Builder（局部监听）

```dart
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('My App')),
      body: Column(
        children: [
          // 只有这部分会在 counter 变化时重建
          Consumer(
            builder: (context, ref, child) {
              final count = ref.watch(counterProvider);
              return Text('Count: $count');
            },
          ),
          // 这部分不会重建
          const Text('Static content'),
        ],
      ),
    );
  }
}
```

### StatefulWidget + ConsumerStatefulWidget

```dart
class MyWidget extends ConsumerStatefulWidget {
  const MyWidget({super.key});

  @override
  ConsumerState<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends ConsumerState<MyWidget> {
  @override
  void initState() {
    super.initState();
    // 可以在 initState 中使用 ref
    ref.read(myProvider);
  }

  @override
  Widget build(BuildContext context) {
    final value = ref.watch(myProvider);
    return Text('Value: $value');
  }
}
```

### 处理异步状态（AsyncValue）

```dart
class UserWidget extends ConsumerWidget {
  const UserWidget({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    // 方式 1: 使用 switch 表达式（推荐）
    return switch (userAsync) {
      AsyncData(:final value) => Text('Hello ${value.name}'),
      AsyncError(:final error) => Text('Error: $error'),
      _ => const CircularProgressIndicator(),
    };

    // 方式 2: 使用 when 方法
    return userAsync.when(
      data: (user) => Text('Hello ${user.name}'),
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );

    // 方式 3: 使用 maybeWhen（提供默认值）
    return userAsync.maybeWhen(
      data: (user) => Text('Hello ${user.name}'),
      orElse: () => const CircularProgressIndicator(),
    );
  }
}
```

## Ref 方法详解

### ref.watch - 监听并自动重建

**用途**: 在 `build` 方法中使用，状态变化时自动重建 widget

```dart
@override
Widget build(BuildContext context, WidgetRef ref) {
  // 监听整个状态
  final value = ref.watch(myProvider);

  // 使用 select 优化：只在特定字段变化时重建
  final name = ref.watch(userProvider.select((user) => user.name));

  return Text('Value: $value, Name: $name');
}
```

**注意事项**:
- ✅ 在 `build` 方法中使用
- ❌ 不要在回调函数中使用（会导致不必要的重建）

### ref.read - 一次性读取

**用途**: 在回调函数中使用，不会触发重建

```dart
ElevatedButton(
onPressed: () {
  // 读取当前值
  final value = ref.read(myProvider);
  print('Current value: $value');

  // 调用 notifier 方法
  ref.read(counterProvider.notifier).increment();
},
child: const Text('Increment'),
)
```

**注意事项**:
- ✅ 在回调函数中使用
- ❌ 不要在 `build` 方法中使用（不会响应变化）

### ref.listen - 监听副作用

**用途**: 监听状态变化并执行副作用（如显示 SnackBar、导航等）

```dart
@override
Widget build(BuildContext context, WidgetRef ref) {
  // 监听状态变化
  ref.listen<AsyncValue<void>>(myProvider, (previous, next) {
    // 显示错误提示
    if (next.hasError) {
      ScaffoldMessenger.of(
        context,
      ).showSnackBar(SnackBar(content: Text('Error: ${next.error}')));
    }

    // 成功后导航
    if (next.hasValue) {
      Navigator.of(context).pop();
    }
  });

  return const MyWidget();
}
```

**高级用法 - 暂停/恢复监听**:

```dart
final subscription = ref.listen(myProvider, (previous, next) {
  print('Value changed: $next');
});

// 暂停监听
subscription.pause();

// 恢复监听
subscription.resume();

// 关闭监听
subscription.close();
```

### ref.invalidate - 重置状态

**用途**: 重置 provider，下次读取时重新计算

```dart
ElevatedButton(
onPressed: () {
  // 重置 provider
  ref.invalidate(myProvider);

  // 重置并立即获取新值
  final newValue = ref.refresh(myProvider);
},
child: const Text('Refresh'),
)
```

### ref.refresh - 重置并读取

**用途**: `ref.invalidate` + `ref.read` 的语法糖

```dart
// 等价于
ref.invalidate(myProvider);
final value = ref.read(myProvider);

// 简写为
final value = ref.refresh(myProvider);
```

### ref.mounted - 检查是否存活（v3.0+）

**用途**: 在异步操作后检查 provider 是否仍然存活

```dart
@riverpod
Future<void> example(Ref ref) async {
  await Future.delayed(const Duration(seconds: 2));

  // 检查 provider 是否仍然存活
  if (!ref.mounted) return;

  // 安全地更新状态
  ref.state = 'Updated';
}
```

## 自动销毁 (Auto Dispose)

### 基本概念

自动销毁功能可以在 provider 不再被使用时自动清理资源，防止内存泄漏。

**代码生成默认行为**:
- ✅ 默认启用自动销毁
- 使用 `@Riverpod(keepAlive: true)` 禁用

**手动定义**:
- 使用 `.autoDispose` 修饰符启用

### 启用/禁用自动销毁

```dart
// 代码生成 - 默认启用
@riverpod
String hello(Ref ref) => 'Hello';

// 代码生成 - 禁用自动销毁
@Riverpod(keepAlive: true)
String hello(Ref ref) => 'Hello';

// 手动定义 - 启用自动销毁
final provider = Provider.autoDispose<String>((ref) => 'Hello');

// 手动定义 - 禁用自动销毁（默认）
final provider = Provider<String>((ref) => 'Hello');
```

### 条件保持存活 (ref.keepAlive)

```dart
@riverpod
Future<String> example(Ref ref) async {
  final response = await http.get(Uri.parse('api/data'));

  // 成功后保持存活，避免重复请求
  if (response.statusCode == 200) {
    ref.keepAlive();
  }

  return response.body;
}

// 高级用法：取消 keepAlive
@riverpod
Future<String> advanced(Ref ref) async {
  final link = ref.keepAlive();

  // 10 秒后恢复自动销毁
  Timer(const Duration(seconds: 10), link.close);

  return 'Data';
}
```

### 生命周期回调

```dart
@riverpod
Stream<int> example(Ref ref) {
  final controller = StreamController<int>();

  // 销毁时清理资源（必需）
  ref.onDispose(() {
    print('Disposing...');
    controller.close();
  });

  // 最后一个监听者移除时
  ref.onCancel(() {
    print('No listeners');
  });

  // 新监听者添加时
  ref.onResume(() {
    print('Listener added');
  });

  // 添加监听者移除回调
  ref.onRemoveListener(() {
    print('A listener was removed');
  });

  // 添加监听者添加回调
  ref.onAddListener(() {
    print('A listener was added');
  });

  return controller.stream;
}
```

### 实用示例：保持状态一段时间

```dart
// 扩展方法：保持状态 N 秒
extension CacheForExtension on Ref {
  void cacheFor(Duration duration) {
    final link = keepAlive();
    final timer = Timer(duration, link.close);
    onDispose(timer.cancel);
  }
}

// 使用
@riverpod
Future<List<Todo>> todos(Ref ref) async {
  // 缓存 5 分钟
  ref.cacheFor(const Duration(minutes: 5));

  return fetchTodos();
}
```

## Family (参数化 Provider)

Family 允许你创建接受参数的 provider，每个参数组合会创建独立的 provider 实例。

### 基本用法

```dart
// 代码生成 - 直接添加参数（推荐）
@riverpod
Future<User> user(Ref ref, int userId) async {
  final response = await http.get(Uri.parse('api/user/$userId'));
  return User.fromJson(jsonDecode(response.body));
}

// 使用
final user = ref.watch(userProvider(123));
final anotherUser = ref.watch(userProvider(456));

// 手动定义
final userProvider = FutureProvider.family<User, int>((ref, userId) async {
  final response = await http.get(Uri.parse('api/user/$userId'));
  return User.fromJson(jsonDecode(response.body));
});
```

### 多个参数

```dart
// 使用命名参数
@riverpod
Future<List<Product>> products(
  Ref ref, {
  required String category,
  required int page,
  int pageSize = 20,
}) async {
  final response = await http.get(
    Uri.parse('api/products?category=$category&page=$page&size=$pageSize'),
  );
  return (jsonDecode(response.body) as List)
      .map((json) => Product.fromJson(json))
      .toList();
}

// 使用
final products = ref.watch(productsProvider(category: 'electronics', page: 1));
```

### 使用自定义类作为参数

```dart
// 定义参数类（必须实现 == 和 hashCode）
class ProductFilter {
  const ProductFilter({
    required this.category,
    required this.minPrice,
    required this.maxPrice,
  });

  final String category;
  final double minPrice;
  final double maxPrice;

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is ProductFilter &&
          runtimeType == other.runtimeType &&
          category == other.category &&
          minPrice == other.minPrice &&
          maxPrice == other.maxPrice;

  @override
  int get hashCode => Object.hash(category, minPrice, maxPrice);
}

// 使用自定义类
@riverpod
Future<List<Product>> filteredProducts(Ref ref, ProductFilter filter) async {
  final response = await http.get(
    Uri.parse(
      'api/products?category=${filter.category}'
      '&min=${filter.minPrice}&max=${filter.maxPrice}',
    ),
  );
  return (jsonDecode(response.body) as List)
      .map((json) => Product.fromJson(json))
      .toList();
}

// 使用
final filter = ProductFilter(
  category: 'electronics',
  minPrice: 100,
  maxPrice: 1000,
);
final products = ref.watch(filteredProductsProvider(filter));
```

### Family 与 Notifier

```dart
@riverpod
class TodoNotifier extends _$TodoNotifier {
  @override
  Future<Todo> build(String todoId) async {
    // todoId 可通过 this.todoId 访问
    return fetchTodo(todoId);
  }

  Future<void> toggle() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final todo = state.value!;
      final updated = await updateTodo(
      todoId, // 使用参数
      completed: !todo.completed,
      );
      return updated;
    });
  }
}

// 使用
final todo = ref.watch(todoNotifierProvider('todo-123'));
ref.read(todoNotifierProvider('todo-123').notifier).toggle();
```

## 测试

### 单元测试 Provider

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  test('counter increments', () {
    // 创建测试容器
    final container = ProviderContainer();

    // 添加监听器以确保 provider 被正确更新
    addTearDown(container.dispose);

    // 读取初始值
    expect(container.read(counterProvider), 0);

    // 调用方法
    container.read(counterProvider.notifier).increment();

    // 验证状态
    expect(container.read(counterProvider), 1);
  });

  test('async provider', () async {
    final container = ProviderContainer();
    addTearDown(container.dispose);

    // 等待异步 provider 完成
    final user = await container.read(userProvider.future);

    expect(user.name, 'John');
  });
}
```

### Mock Provider（覆盖）

```dart
void main() {
  test('mock user provider', () async {
    final container = ProviderContainer(
      overrides: [
        // 覆盖 provider
        userProvider.overrideWith((ref) async {
          return User(id: 1, name: 'Test User');
        }),
      ],
    );
    addTearDown(container.dispose);

    final user = await container.read(userProvider.future);
    expect(user.name, 'Test User');
  });

  test('mock notifier', () {
    final container = ProviderContainer(
      overrides: [
        // 覆盖 notifier
        todoListProvider.overrideWith(() => MockTodoListNotifier()),
      ],
    );
    addTearDown(container.dispose);

    // 测试 mock notifier
    final todos = container.read(todoListProvider);
    expect(todos, isEmpty);
  });
}

// Mock Notifier
class MockTodoListNotifier extends TodoListNotifier {
  @override
  Future<List<Todo>> build() async {
    return [];
  }
}
```

### Widget 测试

```dart
void main() {
  testWidgets('displays user name', (tester) async {
    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          userProvider.overrideWith((ref) async {
            return User(id: 1, name: 'John');
          }),
        ],
        child: const MaterialApp(home: UserWidget()),
      ),
    );

    // 等待异步操作完成
    await tester.pumpAndSettle();

    // 验证 UI
    expect(find.text('Hello John'), findsOneWidget);
  });

  testWidgets('handles loading state', (tester) async {
    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          userProvider.overrideWith((ref) async {
            await Future.delayed(const Duration(seconds: 1));
            return User(id: 1, name: 'John');
          }),
        ],
        child: const MaterialApp(home: UserWidget()),
      ),
    );

    // 验证加载状态
    expect(find.byType(CircularProgressIndicator), findsOneWidget);

    // 等待加载完成
    await tester.pumpAndSettle();

    // 验证数据显示
    expect(find.text('Hello John'), findsOneWidget);
  });
}
```

### 测试 Family Provider

```dart
void main() {
  test('family provider with different parameters', () async {
    final container = ProviderContainer();
    addTearDown(container.dispose);

    // 测试不同参数
    final user1 = await container.read(userProvider(1).future);
    final user2 = await container.read(userProvider(2).future);

    expect(user1.id, 1);
    expect(user2.id, 2);
  });

  test('mock family provider', () async {
    final container = ProviderContainer(
      overrides: [
        userProvider.overrideWith((ref, userId) async {
          return User(id: userId, name: 'Test User $userId');
        }),
      ],
    );
    addTearDown(container.dispose);

    final user = await container.read(userProvider(123).future);
    expect(user.name, 'Test User 123');
  });
}
```

## 最佳实践

### DO ✅ 推荐做法

#### 1. 在 build 中使用 ref.watch
```dart
@override
Widget build(BuildContext context, WidgetRef ref) {
  // ✅ 正确：在 build 中监听状态
  final value = ref.watch(provider);
  return Text('$value');
}
```

#### 2. 在回调中使用 ref.read
```dart
ElevatedButton(
// ✅ 正确：在回调中读取和修改状态
onPressed: () => ref.read(provider.notifier).doSomething(),
child: const Text('Click'),
)
```

#### 3. 使用 select 优化重建
```dart
@override
Widget build(BuildContext context, WidgetRef ref) {
  // ✅ 正确：只在 name 变化时重建
  final name = ref.watch(userProvider.select((u) => u.name));
  return Text(name);
}
```

#### 4. 使用 ref.listen 处理副作用
```dart
@override
Widget build(BuildContext context, WidgetRef ref) {
  // ✅ 正确：监听状态变化并显示 SnackBar
  ref.listen<AsyncValue<void>>(submitProvider, (previous, next) {
    if (next.hasError) {
      ScaffoldMessenger.of(
        context,
      ).showSnackBar(SnackBar(content: Text('Error: ${next.error}')));
    }
  });

  return const MyForm();
}
```

#### 5. Provider 应该是顶级声明
```dart
// ✅ 正确：顶级声明
final myProvider = Provider<String>((ref) => 'Hello');

// ✅ 也可以：静态成员
class MyClass {
  static final myProvider = Provider<String>((ref) => 'Hello');
}
```

### DON'T ❌ 避免做法

#### 1. 不要在 build 中使用 ref.read
```dart
@override
Widget build(BuildContext context, WidgetRef ref) {
  // ❌ 错误：不会响应状态变化
  final value = ref.read(provider);
  return Text('$value');
}
```

#### 2. 不要在回调中使用 ref.watch
```dart
ElevatedButton(
// ❌ 错误：会导致不必要的重建
onPressed: () {
  final value = ref.watch(provider);
  print(value);
},
child: const Text('Click'),
)
```

#### 3. 不要在 initState 中使用 ref
```dart
class _MyWidgetState extends ConsumerState<MyWidget> {
  @override
  void initState() {
    super.initState();
    // ❌ 错误：可能导致问题
    ref.read(provider);
  }
}

// ✅ 正确：使用 ref.listen 或在 build 中处理
@override
Widget build(BuildContext context) {
  ref.listen(provider, (previous, next) {
    // 首次构建时会触发
  });
  return Container();
}
```

#### 4. 不要将 Provider 用于临时 UI 状态
```dart
// ❌ 错误：不应该用 Provider 管理表单输入
final textFieldProvider = StateProvider<String>((ref) => '');

// ✅ 正确：使用 TextEditingController 或 StatefulWidget
class MyForm extends StatefulWidget {
  // ...
}
```

#### 5. 不要在 Provider 中执行副作用
```dart
// ❌ 错误：在初始化时执行副作用
final myProvider = Provider<String>((ref) {
  print('Initializing...'); // 副作用
  showDialog(...); // 副作用
  return 'Hello';
});

// ✅ 正确：使用 ref.listen 或 Mutations
```

### 性能优化技巧

#### 1. 使用 select 减少重建
```dart
// 只在 name 变化时重建
final name = ref.watch(userProvider.select((user) => user.name));

// 使用多个 select
final firstName = ref.watch(userProvider.select((user) => user.firstName));
final lastName = ref.watch(userProvider.select((user) => user.lastName));
```

#### 2. 使用 Consumer 局部重建
```dart
Column(
children: [
const Text('Static content'), // 不会重建
Consumer(
builder: (context, ref, child) {
  final count = ref.watch(counterProvider);
  return Text('Count: $count'); // 只有这里重建
},
),
],
)
```

#### 3. 合理使用 autoDispose
```dart
// 对于一次性数据，启用 autoDispose
@riverpod
Future<User> user(Ref ref, int userId) async {
  return fetchUser(userId);
}

// 对于全局配置，禁用 autoDispose
@Riverpod(keepAlive: true)
AppConfig appConfig(Ref ref) {
  return AppConfig();
}
```

## 从 v2 迁移到 v3

### 主要变化

#### 1. Ref 类型统一
```dart
// v2
ProviderRef<String> ref
AutoDisposeProviderRef<String> ref

// v3 - 统一为 Ref
Ref ref
```

#### 2. 代码生成默认启用 autoDispose
```dart
// v2 - 默认不启用
@riverpod
String example(Ref ref) => 'Hello';

// v3 - 默认启用，需要显式禁用
@Riverpod(keepAlive: true)
String example(Ref ref) => 'Hello';
```

#### 3. 自动重试功能
```dart
// v3 新增：失败时自动重试
// 全局禁用
ProviderScope(
retry: (retryCount, error) => null,
child: MyApp(),
)

// 单个 provider 禁用
@Riverpod(retry: (retryCount, error) => null)
Future<String> example(Ref ref) async {
  return fetchData();
}
```

#### 4. StateProvider 移动到新导入
```dart
// v3 - 需要新导入
import 'package:flutter_riverpod/flutter_riverpod.dart' hide StateProvider;
import 'package:flutter_riverpod/legacy.dart' show StateProvider;
```

#### 5. Provider 使用 == 过滤更新
```dart
// v3 - 所有 provider 使用 == 过滤
// 如需自定义，覆盖 updateShouldNotify
@override
bool updateShouldNotify(T previous, T next) {
  return previous != next;
}
```

### 迁移步骤

1. **更新依赖**
```yaml
dependencies:
  flutter_riverpod: ^3.0.0
  riverpod_annotation: ^4.0.0

dev_dependencies:
  riverpod_generator: ^4.0.0
  riverpod_lint: ^3.0.0
```

2. **运行代码生成**
```bash
dart run build_runner build --delete-conflicting-outputs
```

3. **替换类型引用**
- `AutoDisposeNotifier` → `Notifier`
- `AutoDisposeAsyncNotifier` → `AsyncNotifier`
- `FamilyNotifier` → `Notifier`（参数直接在 build 中）

4. **测试应用**
- 检查自动重试行为
- 验证 autoDispose 逻辑
- 测试异步状态处理

## 参考文件

详细文档位于 `references/` 目录：

### 核心文档
- **`getting_started.md`** - 安装和入门指南
- **`concepts.md`** - Provider、Consumer、Ref 等核心概念详解（18 页）
- **`guides.md`** - 实战教程：构建随机笑话生成器应用

### 高级特性
- **`advanced.md`** - Mutations、离线持久化、自动重试（3 页）
- **`codegen.md`** - 代码生成使用指南和最佳实践
- **`hooks.md`** - Flutter Hooks 集成指南

### 迁移和最佳实践
- **`migration.md`** - 版本迁移指南（0.14→1.0, 0.13→0.14, 2.0→3.0）（6 页）
- **`best_practices.md`** - DO/DON'T 推荐和避免的做法
- **`other.md`** - 其他主题：防抖、取消网络请求等

## 常见问题

### 1. 如何在 StatefulWidget 中使用 Riverpod？
使用 `ConsumerStatefulWidget` 和 `ConsumerState`:
```dart
class MyWidget extends ConsumerStatefulWidget {
  const MyWidget({super.key});

  @override
  ConsumerState<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends ConsumerState<MyWidget> {
  @override
  Widget build(BuildContext context) {
    final value = ref.watch(myProvider);
    return Text('$value');
  }
}
```

### 2. 如何处理下拉刷新？
```dart
RefreshIndicator(
onRefresh: () => ref.refresh(myProvider.future),
child: ListView(...),
)
```

### 3. 如何在 Provider 之间共享状态？
```dart
@riverpod
Future<User> user(Ref ref) async {
  return fetchUser();
}

@riverpod
Future<List<Post>> userPosts(Ref ref) async {
  // 依赖其他 provider
  final user = await ref.watch(userProvider.future);
  return fetchPosts(user.id);
}
```

### 4. 如何实现分页加载？
```dart
@riverpod
class PostList extends _$PostList {
  @override
  Future<List<Post>> build() async {
    return fetchPosts(page: 1);
  }

  Future<void> loadMore() async {
    final currentPosts = state.value ?? [];
    final nextPage = (currentPosts.length / 20).ceil() + 1;

    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final newPosts = await fetchPosts(page: nextPage);
      return [...currentPosts, ...newPosts];
    });
  }
}
```

### 5. 如何实现搜索防抖？
```dart
@riverpod
class SearchQuery extends _$SearchQuery {
  @override
  String build() => '';

  void update(String query) {
    state = query;
  }
}

@riverpod
Future<List<Result>> searchResults(Ref ref) async {
  final query = ref.watch(searchQueryProvider);

  // 防抖：等待 500ms
  await Future.delayed(const Duration(milliseconds: 500));

  // 检查是否仍然存活
  if (!ref.mounted) return [];

  return performSearch(query);
}
```

## 实用代码片段

### 1. 全局加载状态
```dart
@riverpod
class LoadingState extends _$LoadingState {
  @override
  bool build() => false;

  void show() => state = true;

  void hide() => state = false;
}

// 使用
ref.read(loadingStateProvider.notifier).show();
await performOperation();
ref.read(loadingStateProvider.notifier).hide();
```

### 2. 错误处理
```dart
@riverpod
Future<Data> data(Ref ref) async {
  try {
    return await fetchData();
  } on NetworkException catch (e) {
    throw Exception('Network error: ${e.message}');
  } on AuthException {
    // 导航到登录页
    throw Exception('Please login');
  }
}
```

### 3. 乐观更新
```dart
Future<void> toggleLike(String postId) async {
  final previousState = state;

  // 立即更新 UI
  state = AsyncData(
    state.value!.map((post) {
      if (post.id == postId) {
        return post.copyWith(liked: !post.liked);
      }
      return post;
    }).toList(),
  );

  try {
    await api.toggleLike(postId);
  } catch (e) {
    // 失败时回滚
    state = previousState;
    rethrow;
  }
}
```

## 资源链接

### 官方资源
- **官网**: https://riverpod.dev/
- **GitHub**: https://github.com/rrousselGit/riverpod
- **pub.dev**: https://pub.dev/packages/flutter_riverpod
- **Discord 社区**: https://discord.gg/Bbumvej

### 相关包
- **riverpod_lint**: https://pub.dev/packages/riverpod_lint - Lint 规则和重构
- **riverpod_generator**: https://pub.dev/packages/riverpod_generator - 代码生成
- **flutter_hooks**: https://pub.dev/packages/flutter_hooks - Hooks 集成
- **hooks_riverpod**: https://pub.dev/packages/hooks_riverpod - Riverpod + Hooks

### 学习资源
- **官方文档**: https://riverpod.dev/docs/introduction/getting_started
- **示例应用**: https://github.com/rrousselGit/riverpod/tree/master/examples
- **视频教程**: https://www.youtube.com/c/RemiRoussel

---

**版本**: 3.1.0
**最后更新**: 2024-03
**维护者**: Remi Rousselet (@rrousselGit)
