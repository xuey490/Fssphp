<p align="center">
  <img src="logo.png" alt="FssPHP" width="420">
</p>

<h1 align="center">FssPHP</h1>

<p align="center">
  <strong>A lightweight, high-performance, secure PHP 8.3 framework</strong>
</p>

<p align="center">
  <a href="https://www.php.net/releases/8.3/"><img src="https://img.shields.io/badge/PHP-%5E8.3-777BB4?logo=php&logoColor=white" alt="PHP >= 8.3"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="MIT License"></a>
  <a href="https://symfony.com/"><img src="https://img.shields.io/badge/Symfony-7.x-000000?logo=symfony&logoColor=white" alt="Symfony 7"></a>
  <a href="https://www.workerman.net/"><img src="https://img.shields.io/badge/Workerman-5.x-orange" alt="Workerman 5"></a>
  <a href="README.md">中文文档</a> ·
  <a href="https://www.phpframe.org/">Website</a>
</p>

FssPHP is a modern PHP framework whose kernel lives in `framework/`. It combines Symfony 7 components, a Workerman long-running runtime, and pieces of the ThinkPHP / Laravel ecosystems so the **same application code** can run under PHP-FPM, Workerman, or optional Swoole. This repository is both the framework and a reference project.

## Features

- Dual runtime: PHP-FPM and Workerman (HTTP, WebSocket, SSE, Redis queue). Optional Swoole entry (`swoole.php`).
- Hybrid routing: Symfony `RouteCollection`, PHP 8 attributes (`#[Route]`, REST mappings), convention-based auto routes.
- Symfony DI with a Laravel-style facade (`App::make()`, `app()`), property injection, and compiled container cache in production.
- Dual ORM (`thinkORM` / `laravelORM`) and dual templates (Twig + ThinkTemplate).
- Multi-app layout via `config/apps.php` (namespace, URL prefix, optional domain).
- Onion middleware: CSRF, CORS, XSS, rate limit, IP block, Referer check, security headers.
- Casbin RBAC, JWT (strong `JWT_SECRET` required), tenant context, plugins, PSR-14 events.

## Requirements

- PHP **8.3+** (Composer constraint `^8.3`)
- Composer 2.x, MySQL 5.7+/8.x, Redis recommended

## Quick start

```bash
git clone https://github.com/xuey490/project.git
cd project
composer install
cp .env.example .env
```

Set a `JWT_SECRET` of at least 32 bytes. Database credentials are read from `config/database.php` (default engine: `laravelORM`).

```bash
# Built-in server
php -S localhost:8000 -t public

# Workerman
php server.php start          # foreground
php server.php start -d       # daemon
```

Open http://localhost:8000. Workerman also binds AI HTTP on `:8001` (`/api/ai/*`) and WebSocket on `:1234`. Health check: `GET /_health`.

The hot-reload helper `watch.php` is gitignored and not shipped. Use `php server.php start` or the built-in server during development.

## Architecture

```text
Request → Framework → Container / Kernel → Router → Middleware → Controller → Service → Dao
```

| `framework/` module | Role |
|---------------------|------|
| Core | Bootstrap, Kernel, Router, `App` facade |
| Container | Symfony DI wrapper, providers, prod dump |
| Attributes | Route, Auth, Role, Permission, Validate, … |
| Middleware | Dispatcher + built-in security stack |
| Basic | Base Controller / Service / Dao + CRUD traits |
| ORM / Repository | ThinkORM and Eloquent adapters |
| Plugin / Tenant / Security | Plugins, multi-tenant, Casbin / CSRF |
| Console | `php novaphp` generators and plugin commands |

Routing priority: manual > attributes > plugin auto > app-prefix auto > domain auto > default namespace.

## CLI

```bash
php novaphp make:controller User
php novaphp make:model User
php novaphp route:list
php novaphp plugin:list
```

## Docs

Full Chinese README (canonical): [README.md](README.md). Module notes live under `framework/*/readme.md`. Site: [phpframe.org](https://www.phpframe.org/). AI-generated tour: [Zread](https://zread.ai/xuey490/project).

## License

[MIT](LICENSE) © 2025 blue2004 (`xuey863toy@gmail.com`) / xuey490
