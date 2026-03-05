---
name: hyperf-skill
description: Hyperf 3.1 framework development assistant. Use when the user needs to: (1) Create Hyperf controllers, (2) Create Hyperf models, (3) Create Hyperf commands, (4) Implement services, (5) Configure routes, (6) Use dependency injection, (7) Handle validation, or any other Hyperf 3.1 development tasks. Triggers on phrases like "创建 Hyperf 控制器", "生成模型", "创建命令", "Hyperf 开发".
---

# Hyperf 3.1 开发指南

Hyperf 3.1 框架开发助手，专注于控制器、模型、命令行工具的快速开发。

## 常用命令

```bash
# 生成控制器
php bin/hyperf.php gen:controller UserController

# 生成模型
php bin/hyperf.php gen:model User

# 生成命令
php bin/hyperf.php gen:command ImportCommand

# 生成中间件
php bin/hyperf.php gen:middleware AuthMiddleware

# 生成请求验证类
php bin/hyperf.php gen:request UserRequest

# 启动服务
php bin/hyperf.php start
```

## 快速示例

### 控制器

```php
<?php
declare(strict_types=1);

namespace App\Controller;

use Hyperf\HttpServer\Annotation\Controller;
use Hyperf\HttpServer\Annotation\GetMapping;

#[Controller(prefix: "/api/users")]
class UserController extends AbstractController
{
    #[GetMapping("")]
    public function index()
    {
        return $this->response->json(['code' => 0, 'data' => []]);
    }
}
```

### 模型

```php
<?php
declare(strict_types=1);

namespace App\Model;

use Hyperf\DbConnection\Model\Model;

class User extends Model
{
    protected ?string $table = 'users';
    protected array $fillable = ['name', 'email'];
}
```

### 命令

```php
<?php
declare(strict_types=1);

namespace App\Command;

use Hyperf\Command\Command as HyperfCommand;
use Hyperf\Command\Annotation\Command;

#[Command]
class ImportDataCommand extends HyperfCommand
{
    protected ?string $name = 'import:data';

    public function handle()
    {
        $this->info('Processing...');
    }
}
```

### 服务类

```php
<?php
declare(strict_types=1);

namespace App\Service;

use App\Model\User;

class UserService
{
    public function create(array $data): User
    {
        return User::create($data);
    }
}
```

## 官方文档参考

详细文档请参考 `references/zh-cn/` 目录下的官方文档：

- **控制器**: [references/zh-cn/controller.md](references/zh-cn/controller.md)
- **模型**: [references/zh-cn/db/model.md](references/zh-cn/db/model.md)
- **命令行**: [references/zh-cn/command.md](references/zh-cn/command.md)
- **路由**: [references/zh-cn/router.md](references/zh-cn/router.md)
- **验证器**: [references/zh-cn/validation.md](references/zh-cn/validation.md)
- **依赖注入**: [references/zh-cn/di.md](references/zh-cn/di.md)
