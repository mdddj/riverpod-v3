# Riverpod v3 Skill

Flutter 响应式状态管理框架完整指南

## 简介

这是一个关于 Riverpod v3 的完整技能包，包含：

- 📚 完整的核心概念和 API 文档
- 💡 实战代码示例和最佳实践
- 🔧 测试指南和性能优化技巧
- 🚀 从 v2 到 v3 的迁移指南
- 📖 10 个详细的参考文档

## 内容结构

```
riverpod-v3/
├── SKILL.md              # 主文档（1483 行）
├── references/           # 详细参考文档
│   ├── getting_started.md    # 快速开始
│   ├── concepts.md           # 核心概念（18 页）
│   ├── advanced.md           # 高级特性（3 页）
│   ├── best_practices.md     # 最佳实践
│   ├── codegen.md            # 代码生成
│   ├── migration.md          # 迁移指南（6 页）
│   ├── guides.md             # 实战教程
│   ├── hooks.md              # Hooks 集成
│   ├── other.md              # 其他主题
│   └── index.md              # 索引
├── assets/               # 资源文件
└── scripts/              # 脚本文件
```

## 核心特性

- ✅ 响应式状态管理
- ✅ 类型安全
- ✅ 可测试性
- ✅ 代码生成支持
- ✅ 自动销毁
- ✅ Family（参数化 Provider）
- ✅ 自动重试
- ✅ Mutations（实验性）
- ✅ 离线持久化（实验性）

## 快速开始

```bash
# 安装依赖
flutter pub add flutter_riverpod riverpod_annotation
flutter pub add dev:riverpod_generator dev:build_runner

# 运行代码生成
dart run build_runner watch -d
```

## 使用方式

### 作为 Claude Code Skill

将此目录放在 Claude Code 的 skills 目录中：

```bash
~/.claude/skills/riverpod-v3/
```

### 作为学习资源

直接阅读 `SKILL.md` 获取完整指南，或浏览 `references/` 目录查看详细文档。

## 版本信息

- **Riverpod 版本**: 3.1.0
- **文档版本**: 1.0.0
- **最后更新**: 2024-03

## 资源链接

- [官网](https://riverpod.dev/)
- [GitHub](https://github.com/rrousselGit/riverpod)
- [pub.dev](https://pub.dev/packages/flutter_riverpod)

## 许可证

本文档基于 Riverpod 官方文档整理和增强，遵循 MIT 许可证。

---

**维护者**: Remi Rousselet (@rrousselGit)  
**文档整理**: AI Assistant
