# Hyperf 3.1 开发技能

Hyperf 3.1 框架开发助手，专注于控制器、模型、命令行工具的快速开发。

## 功能特性

- **快速生成代码** - 提供控制器、模型、命令行工具的生成指导
- **开发最佳实践** - 包含依赖注入、路由配置、验证器等核心概念
- **官方文档集成** - 完整的 Hyperf 3.1 官方中文文档

## 使用场景

当你需要进行以下操作时，这个技能会自动触发：

- 创建 Hyperf 控制器
- 创建 Hyperf 模型
- 创建 Hyperf 命令行工具
- 实现服务层
- 配置路由
- 使用依赖注入
- 处理验证

## 触发关键词

- "创建 Hyperf 控制器"
- "生成模型"
- "创建命令"
- "Hyperf 开发"

## 快速开始

### 创建控制器

```bash
php bin/hyperf.php gen:controller UserController
```

### 创建模型

```bash
php bin/hyperf.php gen:model User
```

### 创建命令

```bash
php bin/hyperf.php gen:command ImportDataCommand
```

## 文档结构

```
hyperf-skill/
├── SKILL.md              # 技能主文档
├── README.md             # 本文件
└── references/
    └── zh-cn/            # Hyperf 官方中文文档
        ├── controller.md
        ├── db/
        │   └── model.md
        ├── command.md
        ├── router.md
        ├── validation.md
        └── ...
```

## 版本信息

- **Hyperf 版本**: 3.1
- **PHP 版本**: 7.2+

## 参考资源

- [Hyperf 官方文档](https://hyperf.wiki/)
- [Hyperf GitHub](https://github.com/hyperf/hyperf)
