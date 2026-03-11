# Riverpod-V3 - 其他主题

**页数:** 1

---

## 如何防抖/取消网络请求

**URL:** https://riverpod.dev/docs/how_to/cancel

**内容:**
- 如何防抖/取消网络请求
- 应用程序
- 取消请求
- 防抖请求
- 进阶：同时实现两者

随着应用程序复杂性的增长，同时进行多个网络请求是很常见的。例如，用户可能在搜索框中输入并为每次按键触发新请求。如果用户输入速度很快，应用程序可能同时有许多请求在进行中。

或者，用户可能触发一个请求，然后在请求完成之前导航到不同的页面。在这种情况下，应用程序可能有一个不再需要的请求在进行中。

为了在这些情况下优化性能，你可以使用几种技术：

在 Riverpod 中，这两种技术可以以类似的方式实现。关键是使用 ref.onDispose 结合"自动销毁"或 ref.watch 来实现所需的行为。

为了展示这一点，我们将创建一个包含两个页面的简单应用程序：

然后我们将实现以下行为：

首先，让我们创建应用程序，不进行任何防抖或取消。我们不会在这里使用任何花哨的东西，只使用一个简单的 FloatingActionButton 和 Navigator.push 来打开详情页面。

首先，让我们从定义主屏幕开始。像往常一样，不要忘记在应用程序的根部指定 ProviderScope。

然后，让我们定义详情页面。要获取活动并实现下拉刷新，请参阅实现下拉刷新案例研究。

现在我们有了一个可工作的应用程序，让我们实现取消逻辑。

为此，我们将使用 ref.onDispose 在用户离开页面时取消请求。为了使其工作，重要的是启用 providers 的自动销毁。

取消请求所需的确切代码将取决于 HTTP 客户端。在此示例中，我们将使用 package:http，但相同的原理适用于其他客户端。

这里的关键是当用户离开时会调用 ref.onDispose。这是因为我们的 provider 不再被使用，因此由于自动销毁而被销毁。因此，我们可以使用此回调来取消请求。使用 package:http 时，可以通过关闭 HTTP 客户端来完成此操作。

现在我们已经实现了取消，让我们实现防抖。目前，如果用户连续多次刷新活动，我们将为每次刷新发送一个请求。

从技术上讲，现在我们已经实现了取消，这不是问题。如果用户连续多次刷新活动，当发出新请求时，先前的请求将被取消。

但是，这并不理想。我们仍在发送多个请求，浪费带宽和服务器资源。我们可以做的是延迟请求，直到用户停止刷新活动一段固定的时间。

这里的逻辑与取消逻辑非常相似。我们将再次使用 ref.onDispose。但是，这里的想法是，我们不是关闭 HTTP 客户端，而是依赖 onDispose 在请求开始之前中止请求。然后我们将任意等待 500ms 再发送请求。然后，如果用户在 500ms 过去之前再次刷新活动，将调用 onDispose，中止请求。

要中止请求，一个常见的做法是主动抛出异常。在 provider 被销毁后在 providers 内部抛出异常是安全的。异常将自然地被 Riverpod 捕获并被忽略。

我们现在知道如何防抖和取消请求。但目前，如果我们想做另一个请求，我们需要在多个地方复制粘贴相同的逻辑。这并不理想。

但是，我们可以更进一步，实现一个可重用的实用程序来同时执行两者。

这里的想法是在 Ref 上实现一个扩展方法，该方法将在单个方法中处理取消和防抖。

然后我们可以在 providers 中使用此扩展方法，如下所示：

**示例:**

示例 1 (dart):
```dart
void main() => runApp(const ProviderScope(child: MyApp()));

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      routes: {'/detail-page': (_) => const DetailPageView()},
      home: const ActivityView(),
    );
  }
}

class ActivityView extends ConsumerWidget {
  const ActivityView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home screen')),
      body: const Center(
        child: Text('Click the button to open the detail page'),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => Navigator.of(context).pushNamed('/detail-page'),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

示例 2 (dart):
```dart
class Activity {
  Activity({
    required this.activity,
    required this.type,
    required this.participants,
    required this.price,
  });
  factory Activity.fromJson(Map<Object?, Object?> json) {
    return Activity(
      activity: json['activity']! as String,
      type: json['type']! as String,
      participants: json['participants']! as int,
      price: json['price']! as double,
    );
  }
  final String activity;
  final String type;
  final int participants;
  final double price;
}

final activityProvider = FutureProvider.autoDispose<Activity>((ref) async {
  final response = await http.get(
    Uri.https('www.boredapi.com', '/api/activity'),
  );
  final json = jsonDecode(response.body) as Map;
  return Activity.fromJson(json);
});

class DetailPageView extends ConsumerWidget {
  const DetailPageView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final activity = ref.watch(activityProvider);
    return Scaffold(
      appBar: AppBar(title: const Text('Detail page')),
      body: RefreshIndicator(
        onRefresh: () => ref.refresh(activityProvider.future),
        child: ListView(
          children: [
            switch (activity) {
              AsyncValue(:final value?) => Text(value.activity),
              AsyncValue(:final error?) => Text('Error: $error'),
              _ => const Center(child: CircularProgressIndicator()),
            },
          ],
        ),
      ),
    );
  }
}
```

示例 3 (dart):
```dart
@freezedsealed
class Activity with _$Activity {
  factory Activity({
    required String activity,
    required String type,
    required int participants,
    required double price,
  }) = _Activity;
  factory Activity.fromJson(Map<String, dynamic> json) =>
      _$ActivityFromJson(json);
}

@riverpod
Future<Activity> activity(Ref ref) async {
  final response = await http.get(
    Uri.https('www.boredapi.com', '/api/activity'),
  );
  final json = jsonDecode(response.body) as Map;
  return Activity.fromJson(Map.from(json));
}

class DetailPageView extends ConsumerWidget {
  const DetailPageView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final activity = ref.watch(activityProvider);
    return Scaffold(
      appBar: AppBar(title: const Text('Detail page')),
      body: RefreshIndicator(
        onRefresh: () => ref.refresh(activityProvider.future),
        child: ListView(
          children: [
            switch (activity) {
              AsyncValue(:final value?) => Text(value.activity),
              AsyncValue(:final error?) => Text('Error: $error'),
              _ => const Center(child: CircularProgressIndicator()),
            },
          ],
        ),
      ),
    );
  }
}
```

示例 4 (dart):
```dart
final activityProvider = FutureProvider.autoDispose<Activity>((ref) async {
  // 我们使用 package:http 创建一个 HTTP 客户端
  final client = http.Client();
  // 在销毁时，我们关闭客户端。
  // 这将取消客户端可能有的任何待处理请求。
  ref.onDispose(client.close);
  // 我们现在使用客户端发出请求，而不是"get"函数。
  final response = await client.get(
    Uri.https('www.boredapi.com', '/api/activity'),
  );
  // 其余代码与之前相同
  final json = jsonDecode(response.body) as Map;
  return Activity.fromJson(Map.from(json));
});
```

---
