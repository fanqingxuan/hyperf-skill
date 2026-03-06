---
name: laravel12
description: "Laravel 12 framework development assistant. Use when the user needs to: (1) Create Laravel controllers, (2) Create Laravel models, (3) Create Laravel commands, (4) Create middleware, (5) Create facades, (6) Implement services, (7) Configure routes, (8) Use dependency injection, (9) Handle validation, (10) Work with Eloquent ORM, or any other Laravel 12 development tasks. Triggers on phrases like \"创建 Laravel 控制器\", \"生成模型\", \"创建命令\", \"创建中间件\", \"创建门面\", \"生成中间件\", \"生成门面\", \"Laravel 开发\", \"Laravel 12\"."
---

# Laravel 12 开发指南

Laravel 12 框架开发助手，专注于控制器、模型、命令行工具、服务层的快速开发。

## 常用 Artisan 命令

```bash
# 生成控制器
php artisan make:controller UserController

# 生成资源控制器（RESTful）
php artisan make:controller UserController --resource

# 生成 API 控制器
php artisan make:controller API/UserController --api

# 生成模型
php artisan make:model User

# 生成模型 + 迁移文件
php artisan make:model User -m

# 生成模型 + 迁移 + 工厂 + 控制器
php artisan make:model User -mcf

# 生成命令
php artisan make:command ImportDataCommand

# 生成中间件
php artisan make:middleware CheckUserRole

# 生成服务提供者（用于注册门面）
php artisan make:provider PaymentServiceProvider

# 生成请求验证类
php artisan make:request StoreUserRequest

# 生成迁移文件
php artisan make:migration create_users_table

# 执行迁移
php artisan migrate

# 回滚迁移
php artisan migrate:rollback

# 生成 Seeder
php artisan make:seeder UserSeeder

# 执行 Seeder
php artisan db:seed

# 清除缓存
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear

# 启动开发服务器
php artisan serve
```

## 快速示例

### 控制器

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;

class UserController extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function index(): JsonResponse
    {
        $users = User::all();

        return response()->json([
            'success' => true,
            'data' => $users
        ]);
    }

    /**
     * Store a newly created resource in storage.
     */
    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
        ]);

        $user = User::create($validated);

        return response()->json([
            'success' => true,
            'data' => $user
        ], 201);
    }

    /**
     * Display the specified resource.
     */
    public function show(User $user): JsonResponse
    {
        return response()->json([
            'success' => true,
            'data' => $user
        ]);
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(Request $request, User $user): JsonResponse
    {
        $validated = $request->validate([
            'name' => 'sometimes|string|max:255',
            'email' => 'sometimes|email|unique:users,email,' . $user->id,
        ]);

        $user->update($validated);

        return response()->json([
            'success' => true,
            'data' => $user
        ]);
    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy(User $user): JsonResponse
    {
        $user->delete();

        return response()->json([
            'success' => true,
            'message' => 'User deleted successfully'
        ]);
    }
}
```

### 模型（Eloquent ORM）

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class User extends Model
{
    use HasFactory, SoftDeletes;

    /**
     * The table associated with the model.
     */
    protected $table = 'users';

    /**
     * The attributes that are mass assignable.
     */
    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    /**
     * The attributes that should be hidden for serialization.
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * The attributes that should be cast.
     */
    protected $casts = [
        'email_verified_at' => 'datetime',
        'password' => 'hashed',
    ];

    /**
     * Get the posts for the user.
     */
    public function posts()
    {
        return $this->hasMany(Post::class);
    }

    /**
     * Get the user's profile.
     */
    public function profile()
    {
        return $this->hasOne(Profile::class);
    }
}
```

### 命令（Artisan Command）

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use App\Models\User;

class ImportDataCommand extends Command
{
    /**
     * The name and signature of the console command.
     */
    protected $signature = 'import:data {file} {--force}';

    /**
     * The console command description.
     */
    protected $description = 'Import data from file';

    /**
     * Execute the console command.
     */
    public function handle(): int
    {
        $file = $this->argument('file');
        $force = $this->option('force');

        $this->info("Processing file: {$file}");

        if ($force) {
            $this->warn('Force mode enabled');
        }

        // 处理逻辑
        $bar = $this->output->createProgressBar(100);
        $bar->start();

        for ($i = 0; $i < 100; $i++) {
            // 处理数据
            $bar->advance();
        }

        $bar->finish();
        $this->newLine();

        $this->info('Import completed successfully!');

        return Command::SUCCESS;
    }
}
```

### 中间件（Middleware）

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckUserRole
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next, string $role): Response
    {
        if (!$request->user() || !$request->user()->hasRole($role)) {
            abort(403, 'Unauthorized action.');
        }

        return $next($request);
    }
}
```

**注册中间件**（在 `app/Http/Kernel.php` 或 `bootstrap/app.php`）：

```php
// 全局中间件
protected $middleware = [
    \App\Http\Middleware\CheckUserRole::class,
];

// 路由中间件
protected $middlewareAliases = [
    'role' => \App\Http\Middleware\CheckUserRole::class,
];
```

**使用中间件**：

```php
// 在路由中使用
Route::get('/admin', function () {
    // ...
})->middleware('role:admin');

// 在控制器构造函数中使用
public function __construct()
{
    $this->middleware('role:admin');
}
```

### 门面（Facade）

**创建服务类**：

```php
<?php

namespace App\Services;

class PaymentService
{
    public function process(array $data): array
    {
        // 处理支付逻辑
        return [
            'status' => 'success',
            'transaction_id' => uniqid(),
        ];
    }

    public function refund(string $transactionId): bool
    {
        // 退款逻辑
        return true;
    }
}
```

**创建门面类**：

```php
<?php

namespace App\Facades;

use Illuminate\Support\Facades\Facade;

class Payment extends Facade
{
    /**
     * Get the registered name of the component.
     */
    protected static function getFacadeAccessor(): string
    {
        return 'payment';
    }
}
```

**注册服务提供者**（在 `app/Providers/AppServiceProvider.php`）：

```php
<?php

namespace App\Providers;

use App\Services\PaymentService;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton('payment', function ($app) {
            return new PaymentService();
        });
    }
}
```

**使用门面**：

```php
use App\Facades\Payment;

// 调用门面方法
$result = Payment::process([
    'amount' => 100,
    'currency' => 'USD',
]);

// 退款
Payment::refund($transactionId);
```

### 服务类

```php
<?php

namespace App\Services;

use App\Models\User;
use Illuminate\Support\Facades\Hash;

class UserService
{
    /**
     * Create a new user.
     */
    public function create(array $data): User
    {
        return User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => Hash::make($data['password']),
        ]);
    }

    /**
     * Update user information.
     */
    public function update(User $user, array $data): User
    {
        $user->update($data);

        return $user->fresh();
    }

    /**
     * Delete a user.
     */
    public function delete(User $user): bool
    {
        return $user->delete();
    }

    /**
     * Find user by email.
     */
    public function findByEmail(string $email): ?User
    {
        return User::where('email', $email)->first();
    }
}
```

### 请求验证类

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreUserRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     */
    public function authorize(): bool
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     */
    public function rules(): array
    {
        return [
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users,email',
            'password' => 'required|string|min:8|confirmed',
        ];
    }

    /**
     * Get custom messages for validator errors.
     */
    public function messages(): array
    {
        return [
            'name.required' => '用户名不能为空',
            'email.required' => '邮箱不能为空',
            'email.email' => '邮箱格式不正确',
            'email.unique' => '该邮箱已被注册',
            'password.required' => '密码不能为空',
            'password.min' => '密码至少需要8个字符',
            'password.confirmed' => '两次密码输入不一致',
        ];
    }
}
```

### 路由定义

```php
<?php

use App\Http\Controllers\UserController;
use Illuminate\Support\Facades\Route;

// RESTful 资源路由
Route::resource('users', UserController::class);

// API 路由组
Route::prefix('api')->group(function () {
    Route::get('/users', [UserController::class, 'index']);
    Route::post('/users', [UserController::class, 'store']);
    Route::get('/users/{user}', [UserController::class, 'show']);
    Route::put('/users/{user}', [UserController::class, 'update']);
    Route::delete('/users/{user}', [UserController::class, 'destroy']);
});

// 中间件保护的路由
Route::middleware(['auth'])->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
});
```

### 迁移文件

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
            $table->softDeletes();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```

## 官方文档参考

详细文档请参考 `references/` 目录下的官方文档：

### 核心概念
- **安装**: [references/installation.md](references/installation.md)
- **配置**: [references/configuration.md](references/configuration.md)
- **目录结构**: [references/structure.md](references/structure.md)
- **生命周期**: [references/lifecycle.md](references/lifecycle.md)

### 基础功能
- **路由**: [references/routing.md](references/routing.md)
- **控制器**: [references/controllers.md](references/controllers.md)
- **请求**: [references/requests.md](references/requests.md)
- **响应**: [references/responses.md](references/responses.md)
- **视图**: [references/views.md](references/views.md)
- **Blade 模板**: [references/blade.md](references/blade.md)

### 数据库
- **数据库配置**: [references/database.md](references/database.md)
- **查询构造器**: [references/queries.md](references/queries.md)
- **Eloquent ORM**: [references/eloquent.md](references/eloquent.md)
- **关联关系**: [references/eloquent-relationships.md](references/eloquent-relationships.md)
- **迁移**: [references/migrations.md](references/migrations.md)
- **数据填充**: [references/seeding.md](references/seeding.md)

### 高级功能
- **验证**: [references/validation.md](references/validation.md)
- **中间件**: [references/middleware.md](references/middleware.md)
- **门面**: [references/facades.md](references/facades.md)
- **依赖注入**: [references/container.md](references/container.md)
- **服务提供者**: [references/providers.md](references/providers.md)
- **队列**: [references/queues.md](references/queues.md)
- **任务调度**: [references/scheduling.md](references/scheduling.md)
- **缓存**: [references/cache.md](references/cache.md)
- **事件**: [references/events.md](references/events.md)

### 测试
- **测试**: [references/testing.md](references/testing.md)
- **HTTP 测试**: [references/http-tests.md](references/http-tests.md)
- **数据库测试**: [references/database-testing.md](references/database-testing.md)

### Artisan 命令
- **Artisan 控制台**: [references/artisan.md](references/artisan.md)

## 开发最佳实践

1. **使用依赖注入**：通过构造函数或方法注入依赖
2. **遵循 RESTful 规范**：使用资源控制器和标准 HTTP 方法
3. **使用 Form Request**：将验证逻辑从控制器中分离
4. **服务层模式**：将业务逻辑放在服务类中
5. **中间件保护路由**：使用中间件进行权限验证和请求过滤
6. **门面简化调用**：为常用服务创建门面，提供简洁的静态调用接口
7. **使用 Eloquent 关联**：充分利用 ORM 的关联功能
8. **数据库迁移**：所有数据库变更都通过迁移文件管理
9. **使用队列**：将耗时任务放入队列异步处理
10. **缓存优化**：合理使用缓存提升性能
11. **编写测试**：为关键功能编写单元测试和功能测试
12. **遵循 PSR 规范**：代码风格遵循 PSR-12 标准
