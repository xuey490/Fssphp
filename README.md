<p align="center">
  <img src="logo.png" alt="FssPHP" width="420">
</p>

<h1 align="center">FssPHP</h1>

<p align="center">
  <strong>轻量、高性能、安全的 PHP 8.3 框架</strong><br>
  <em>A lightweight, high-performance, secure PHP framework</em>
</p>

<p align="center">
  <a href="https://www.php.net/releases/8.3/"><img src="https://img.shields.io/badge/PHP-%5E8.3-777BB4?logo=php&logoColor=white" alt="PHP >= 8.3"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="MIT License"></a>
  <a href="https://symfony.com/"><img src="https://img.shields.io/badge/Symfony-7.x-000000?logo=symfony&logoColor=white" alt="Symfony 7"></a>
  <a href="https://www.workerman.net/"><img src="https://img.shields.io/badge/Workerman-5.x-orange" alt="Workerman 5"></a>
  <a href="https://getcomposer.org/"><img src="https://img.shields.io/badge/Composer-ready-885630?logo=composer&logoColor=white" alt="Composer"></a>
  <a href="https://zread.ai/xuey490/project"><img src="https://img.shields.io/badge/Ask_Zread-_.svg?style=flat&color=00b0aa&labelColor=000000" alt="Ask Zread"></a>
</p>

<p align="center">
  <a href="https://www.phpframe.org/">官网</a> ·
  <a href="#快速开始">快速开始</a> ·
  <a href="#framework-架构">架构</a> ·
  <a href="English.md">English</a> ·
  <a href="https://zread.ai/xuey490/project">文档分析</a>
</p>

---

FssPHP 以 `framework/` 为内核，组合 Symfony 7 组件、Workerman 常驻进程、ThinkPHP / Laravel 生态能力，形成一套可在 **PHP-FPM** 与 **Workerman**（以及可选 **Swoole**）之间共用同一套业务代码的现代 PHP 框架。本仓库既是框架本体，也是最佳实践示例项目。

## 目录

- [特性](#特性)
- [技术栈](#技术栈)
- [运行要求](#运行要求)
- [快速开始](#快速开始)
- [目录结构](#目录结构)
- [framework 架构](#framework-架构)
- [核心用法](#核心用法)
- [运行模式](#运行模式)
- [命令行](#命令行)
- [配置要点](#配置要点)
- [开发工具](#开发工具)
- [文档](#文档)
- [鸣谢](#鸣谢)
- [许可证](#许可证)

## 特性

**性能与运行时**

- 同一套控制器 / 服务代码，可在 PHP-FPM、Workerman HTTP、可选 Swoole 之间切换
- Workerman 同时提供 HTTP（默认 `8000`）、AI HTTP（`8001`）、WebSocket（`1234`）、SSE、Redis 队列消费者
- 生产环境编译 Symfony DI 容器与路由缓存；路由命中走 PSR-16 缓存

**路由与请求处理**

- 三种路由并存：`config/routes.php` 手动定义、PHP 8 Attribute、约定式自动路由
- 优先级：手动路由 > 属性路由 > 插件自动路由 > 应用前缀自动路由
- 洋葱模型中间件：全局栈 → 应用栈 → 路由 / 注解中间件
- 内置 CSRF、CORS、XSS 过滤、Referer 校验、IP 黑名单、限流、安全响应头、方法伪装

**开发体验**

- Symfony DI + Laravel 风格门面：`App::make()` / `app()`
- 属性注入 `#[Autowire]` / `#[Inject]` / `#[Context]`
- Controller → Service → DAO 三层，基类提供 REST CRUD Trait
- 双 ORM：ThinkORM 与 Laravel Eloquent，通过 `config/database.php` 的 `engine` 切换
- 双模板：Twig 与 ThinkTemplate
- 多应用：`config/apps.php` 配置独立命名空间、URL 前缀、可选域名绑定
- 插件系统：安装 / 启用 / 市场 / 迁移，CLI 一键生成脚手架

**安全与基础设施**

- Casbin RBAC、lcobucci/jwt（强制强密钥）、phpseclib
- 多租户上下文（Session / JWT）与模型 Scope
- PSR-14 事件、Monolog 日志、think-validate 校验
- Redis 连接池 / MySQL 连接池（Workerman 进程内）

## 技术栈

| 类别 | 技术 |
|------|------|
| 语言 | PHP `^8.3`，强制 `declare(strict_types=1)` |
| HTTP 运行时 | Workerman `^5.2` / PHP-FPM / 可选 Swoole |
| HTTP 抽象 | Symfony HttpFoundation / HttpKernel `^7.4` |
| 路由 | Symfony Routing + PHP 8 Attributes |
| DI | Symfony DependencyInjection（PSR-11）+ `Framework\Core\App` 门面 |
| ORM | ThinkORM `^4.0` **或** Illuminate Database / Eloquent `^12.58` |
| 模板 | Twig `^3.x` + ThinkTemplate |
| 缓存 | Symfony Cache + ThinkCache + Predis |
| 安全 | Casbin、lcobucci/jwt、HTMLPurifier、CSRF Token |
| 日志 / 校验 / 事件 | Monolog、think-validate、PSR-14 |
| 其它 | Guzzle、Intervention Image、Ramsey UUID、Symfony Translation |

## 运行要求

- PHP **8.3+**（`composer.json` 强制 `^8.3`）
- Composer 2.x
- MySQL 5.7+ / 8.x
- Redis（Session、缓存、队列推荐）
- Linux / macOS 推荐；Windows 下 Workerman 的 AI HTTP 与队列需拆分为 `server-ai.php`、`queue.php`

## 快速开始

```bash
git clone https://github.com/xuey490/project.git
cd project
composer install
cp .env.example .env
```

编辑 `.env`，设置 **长度 ≥ 32 字节** 的 `JWT_SECRET`（未设置或密钥过弱时 JWT 组件会拒绝启动）：

```bash
php -r "echo 'base64:'.base64_encode(random_bytes(48)), PHP_EOL;"
```

数据库账号、库名、主机写在 **`config/database.php`**（当前默认 ORM 为 `laravelORM`，连接默认为 `127.0.0.1` / `fssoa` / `root`）。`.env` 里的 `DB_*`、`ORM_DRIVER` **尚未接到该文件**，改环境变量不会改变实际连接。导入 `database/sql/` 下的初始化脚本，或按项目迁移流程执行 `database/migrate.php`。

### PHP 内置服务器（开发）

```bash
php -S localhost:8000 -t public
```

浏览器打开 <http://localhost:8000>。

### Workerman（推荐，生产 / 高性能）

```bash
php server.php start          # 前台
php server.php start -d       # 守护进程
php server.php status
php server.php reload
php server.php stop
```

默认服务：

| 服务 | 地址 |
|------|------|
| HTTP | `http://0.0.0.0:8000` |
| AI HTTP | `http://0.0.0.0:8001`（`/api/ai/*`） |
| WebSocket | `ws://0.0.0.0:1234` |
| 健康检查 | `GET /_health` |

> 热重载脚本 `watch.php` 已被忽略、不随仓库分发。开发期请用 `php server.php start` 或 PHP 内置服务器。

### PHP-FPM / Nginx / Apache

将 Web 根目录指向 `public/`，其余请求转发到 `public/index.php`（仓库已提供 `public/.htaccess`）。

## 目录结构

```text
project/
├── app/                    # 业务代码（多应用：default / admin / system / test）
│   ├── Controllers/        # 默认应用控制器
│   ├── admin/              # 管理后台
│   ├── system/             # 系统 API
│   ├── Dao/                # 数据访问层（自动注册为 DI 服务）
│   ├── Middlewares/        # 应用中间件
│   ├── Providers/          # 应用服务提供者（自动发现）
│   └── function.php
├── config/                 # 配置中心
│   ├── app.php / apps.php  # 应用与多应用
│   ├── routes.php          # 手动路由
│   ├── services.php        # Symfony DI 定义
│   ├── database.php        # 双 ORM 与连接
│   ├── middleware.php      # 中间件参数
│   └── plugin/             # 插件配置
├── framework/              # 框架内核（见下一节）
├── plugins/                # 插件（blog、bbs 示例）
├── public/index.php        # FPM / 内置服务器入口
├── resource/               # 视图与翻译
├── database/               # SQL 与迁移
├── storage/                # 缓存、日志、视图编译
├── tests/Unit/             # PHPUnit
├── server.php              # Workerman 入口
├── server-ai.php           # Windows 独立 AI Worker
├── queue.php               # 独立队列消费者
├── swoole.php              # 可选 Swoole 入口
└── novaphp                 # CLI
```

## framework 架构

`framework/` 是框架本体，按职责拆成独立模块。启动链路：

```text
public/index.php 或 server.php
        │
        ▼
Framework::getInstance()
        ├── 注册多应用 PSR-4
        ├── Container::init()     # .env + services.php，prod 编译缓存
        ├── Kernel::boot()        # 时区、异常处理、App 门面
        ├── PluginManager         # 加载已启用插件
        ├── loadAllRoutes()       # 手动 → Attribute → 缓存
        └── MiddlewareDispatcher  # 洋葱调度
                │
                ▼
        Router::match() → 控制器 → Response
```

### 模块一览

| 目录 | 职责 |
|------|------|
| **Core** | `Framework` 单例、`Kernel`、`Router`、`AttributeRouteLoader`、`App` 门面 |
| **Container** | Symfony DI 封装：`.env`、Provider `register`/`boot`、生产环境 `PhpDumper` |
| **Attributes** | `#[Route]`、`GetMapping`/`PostMapping` 等 REST 映射，以及 `Auth`、`Role`、`Permission`、`Middlewares`、`Validate`、`Cache`、`Log` |
| **Middleware** | 洋葱调度器 + CSRF / CORS / XSS / 限流 / IP 封禁 / Referer / 安全头 等内置中间件 |
| **Basic** | `BaseController` / `BaseService` / `BaseDao` / `BaseJsonResponse`，CRUD Trait，双 ORM 基模型与租户 Scope |
| **DI** | `Injectable` Trait，`#[Autowire]` / `#[Inject]` / `#[Context]` 属性注入 |
| **ORM** | ThinkORM / Eloquent 适配工厂 |
| **Repository** | 与 DAO 并行的仓储层，`ThinkStrategy` / `EloquentStrategy` 屏蔽语法差异 |
| **Event** | PSR-14 `Dispatcher` + `#[EventListener]` 扫描 |
| **Plugin** | 插件发现、清单、缓存、市场、迁移 |
| **Providers** | 框架级服务提供者（Request、Session、Cache、JWT、Twig、ThinkTemplate、Validate…） |
| **Security** | Casbin RBAC、CSRF Token |
| **Tenant** | 请求级租户上下文（Session / JWT） |
| **Cache / Session / Validation / View / Translation** | 缓存、会话、校验、双模板、i18n |
| **Http** | Workerman SSE（`SseResponse` / `SseEmitter`） |
| **Pool / Queue / Schema** | 连接池、Redis 队列、模型 Schema 预热 |
| **Console** | `novaphp` 脚手架与插件命令 |
| **Utils** | JWT 工厂、验证码、Cookie、JSON、日期等 |
| **helpers.php** | `app()`、`config()`、`view()`、`cache_*`、`EventDispatch()` 等全局函数 |

应用层约定：

```text
Controller  →  Service（事务 / 业务）  →  Dao / Repository（ORM）
```

控制器继承 `BaseController`，通过 `$serviceClass` 自动解析服务，不必手写构造函数。

## 核心用法

### 属性路由

```php
<?php

declare(strict_types=1);

namespace App\Controllers;

use Framework\Attributes\Auth;
use Framework\Attributes\Role;
use Framework\Attributes\Route;
use Framework\Basic\BaseController;
use Symfony\Component\HttpFoundation\JsonResponse;

#[Route(prefix: '/api/v1/users')]
class UserController extends BaseController
{
    protected string $serviceClass = \App\Services\UserService::class;

    #[Route(path: '/profile', methods: ['GET'], name: 'users.profile')]
    #[Auth]
    public function profile(): JsonResponse
    {
        return $this->success($this->service->current());
    }

    #[Route(path: '/{id}', methods: ['DELETE'], requirements: ['id' => '\d+'])]
    #[Auth]
    #[Role(roles: ['admin'])]
    public function destroy(int $id): JsonResponse
    {
        $this->service->delete($id);
        return $this->success(['id' => $id]);
    }
}
```

也可在 `config/routes.php` 用 Symfony `RouteCollection` 手动注册，或依赖 `/Controller/Action` 自动推断。

### 依赖注入

```php
use Framework\Core\App;

$users = App::make(UserService::class);
$redis = app('redis');
```

Service / DAO 属性上使用 `#[Autowire]` 即可，基类构造时会调用 `inject()`。

### 获取服务与配置

| 助手函数 | 作用 |
|----------|------|
| `app($id)` | 解析容器服务 |
| `config('app.debug')` | 点分配置 |
| `view('index.twig', $data)` | Twig 渲染 |
| `ThinkView('index', $data)` | ThinkTemplate 渲染 |
| `cache_get` / `cache_set` | Symfony Cache |
| `EventDispatch($event)` | 派发 PSR-14 事件 |
| `WebCsrfField()` | 表单 CSRF 隐藏域 |

### JSON 响应

`BaseController` / `BaseJsonResponse` 提供统一信封：`success()`、`fail()`、`error()`、`unauthorized()`、`csrfExpired()`。

### 多应用

在 `config/apps.php` 增加一项即可，无需改 `composer.json` 或 `dump-autoload`：

```php
'system' => [
    'dir'       => BASE_PATH . '/app/system/Controllers',
    'namespace' => 'App\System\Controllers',
    'prefix'    => 'system',   // /system/controller/action
    'domain'    => '',         // 可选：system.example.com
],
```

路由优先级：定义路由 > 插件自动路由 > 前缀自动路由 > 域名绑定自动路由 > 默认命名空间。

## 运行模式

| 模式 | 命令 | 说明 |
|------|------|------|
| 内置服务器 | `php -S localhost:8000 -t public` | 快速开发 |
| FPM | Web 根目录 → `public/` | Apache / Nginx |
| Workerman | `php server.php start` | HTTP + WS + 队列；Linux 同时监听 8000/8001 |
| Workerman AI | `php server-ai.php start` | Windows 独立 AI 进程 |
| 队列 | `php queue.php start` | 独立 Redis LIST 消费者 |
| Swoole | `swoole-cli swoole.php start` | 可选；勿与 Workerman 抢同一端口 |

Workerman 进程启动时会初始化 MySQL / Redis 连接池，并对模型做 Schema 预热。

## 命令行

```bash
php novaphp make:controller User
php novaphp make:model User
php novaphp make:middleware Auth
php novaphp make:event UserLoggedIn
php novaphp make:listener LogUserLogin
php novaphp make:plugin blog

php novaphp route:list
php novaphp plugin:list
php novaphp plugin:enable blog
php novaphp plugin:market search keyword
```

## 配置要点

从 `.env.example` 复制后按环境修改。常用变量：

| 变量 | 说明 |
|------|------|
| `APP_ENV` / `APP_DEBUG` | 环境与调试 |
| `APP_URL` | 对外根地址 |
| `ORM_DRIVER` / `DB_*` | 出现在 `.env.example` 中，**当前未写入** `config/database.php`；ORM 引擎与连接请直接改该文件（`engine`、`hostname`、`database`、`username`、`password`） |
| `SESSION_STORAGE` | `redis` / `file` 等 |
| `CACHE_DRIVER` | 缓存驱动 |
| `JWT_SECRET` | **必填**，≥ 32 字节随机密钥 |
| `JWT_SINGLE_DEVICE_LOGIN` | 单设备登录 |
| `COOKIE_SECRET` | Cookie 签名 |
| `CORS_ALLOWED_ORIGINS` | 跨域白名单，禁止 `*` |
| `WORKERMAN_HTTP_PORT` / `WORKERMAN_AI_PORT` | Workerman 端口 |

认证、租户、Casbin 等应用中间件默认写在 `config/middlewares.php` 中并注释掉，按需取消注释即可接入全局栈。

## 开发工具

```bash
php vendor/bin/phpunit                      # tests/Unit
php vendor/bin/php-cs-fixer fix             # PSR-12 + Symfony
php vendor/bin/phpstan analyze -c phpstan.neon   # Level 6，当前分析 app/
vendor/bin/composer-unused
```

代码约定：PHP 8.3 强类型、短数组 `[]`、类 PascalCase、方法 camelCase。新增控制器 / 服务 / DAO 分别继承 `BaseController`、`BaseService`、`BaseDao`。完整规范见 [AGENTS.md](AGENTS.md)。

## 文档

| 文档 | 内容 |
|------|------|
| [官网](https://www.phpframe.org/) | 项目主页 |
| [English.md](English.md) | English README |
| [docs/tech-stack-report.md](docs/tech-stack-report.md) | 技术栈分析 |
| [docs/plugin-system.md](docs/plugin-system.md) | 插件系统 |
| [docs/permission-system.md](docs/permission-system.md) | 权限系统 |
| [framework/Core/readme.md](framework/Core/readme.md) | `App` 门面与容器 |
| [framework/Attributes/readme.md](framework/Attributes/readme.md) | 注解路由 |
| [framework/Basic/readme.md](framework/Basic/readme.md) | 控制器 / Service / DAO |
| [framework/Repository/readme.md](framework/Repository/readme.md) | 双 ORM 仓储 |
| [framework/Tenant/readme.md](framework/Tenant/readme.md) | 多租户 |
| [Zread](https://zread.ai/xuey490/project) | AI 生成的仓库导读 |

代码质量扫描（历史报告）：

[![Code quality](quality.png)](https://gitee.com/fsscms/FssPhp)

## 鸣谢

站在巨人的肩膀上：

- [Workerman](https://www.workerman.net/) — 高性能 PHP 应用容器
- [Symfony](https://symfony.com/) — HTTP、路由、DI、缓存等基础设施
- [ThinkPHP](https://thinkphp.cn/) — ORM、模板、校验等中文生态组件
- [Laravel Illuminate](https://laravel.com/) — Eloquent 与 Database 组件

## 许可证

[MIT](LICENSE) © 2025 [blue2004](mailto:xuey863toy@gmail.com) / xuey490

欢迎 Issue 与 Pull Request。安全相关披露请见 [SECURITY.md](SECURITY.md)。
