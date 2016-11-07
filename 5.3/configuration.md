# 配置

- [简介](#introduction)
- [访问配置值](#accessing-configuration-values)
- [环境配置](#environment-configuration)
    - [确定当前环境](#determining-the-current-environment)
- [配置缓存](#configuration-caching)
- [维护模式](#maintenance-mode)

<a name="introduction"></a>
## 简介

Laravel 的所有配置文件都存放在 `config` 目录下，每个配置项都有注释，所以您可以随意查看文件，并熟悉可用的选项。

<a name="accessing-configuration-values"></a>
## 访问配置值

您可以在应用程序的任何位置使用全局 `config` 帮助函数轻松访问您的配置值。 可以使用“dot”语法访问配置值，其中包括要访问的文件名称和选项的名称。 还可以指定默认值，如果配置选项不存在，将返回默认值：

    $value = config('app.timezone');

要在运行时设置配置值，请传递一个数组到 `config` 帮助函数：

    config(['app.timezone' => 'America/Chicago']);

<a name="environment-configuration"></a>
## 环境配置

根据应用程序运行的环境，具有不同的配置值通常是有帮助的。例如，您可能希望在本地使用与生产服务器上不同的缓存驱动程序。

Laravel 使用 Vance Lucas 开发的 PHP 库 [DotEnv](https://github.com/vlucas/phpdotenv) 来实现这一机制，在新安装的 Laravel 中，根目录下有一个 `.env.example` 文件，如果 Laravel 是通过 Composer 安装的，那么该文件已经被重命名为 `.env`。否则，话你应该手动重命名该文件。

#### 获取环境变量配置值

在应用每次接受请求时，.env 中列出的所有配置及其值都会被载入到 PHP 超全局变量  `$_ENV` 中，然后你就可以在应用中通过辅助函数 `env` 来获取这些配置值。实际上，如果你去查看 Laravel 的配置文件，就会发现很多地方已经在使用这个辅助函数了：

    'debug' => env('APP_DEBUG', false),

传递到 `env` 函数的第二个参数是默认值，如果环境变量没有被配置将会使用该默认值。

不要把 `.env` 文件提交到源码控制（svn 或 git 等）中，因为每个使用你的应用的开发者/服务器可能要求不同的环境配置。

如果你是在一个团队中进行开发，您可能希望继续在应用程序中包含一个 `.env.example` 文件。通过在示例配置文件中放置占位符值，您团队中的其他开发人员可以清楚地看到运行应用程序所需的环境变量。

<a name="determining-the-current-environment"></a>
### 判断当前应用环境

当前应用环境由 `.env` 文件中的 `APP_ENV` 变量决定，你可以通过 `App` [门面](/laravel/{{version}}/facades) 的 `environment` 方法来访问其值：

    $environment = App::environment();

你也可以向 `environment` 方法中传递参数来判断当前环境是否匹配给定值，如果需要的话你甚至可以传递多个值。如果当前环境与给定值匹配，该方法返回 `true`：

    if (App::environment('local')) {
        // The environment is local
    }

    if (App::environment('local', 'staging')) {
        // The environment is either local OR staging...
    }

<a name="configuration-caching"></a>
## 配置缓存

为了给应用加速，你可以使用 Artisan 命令 `config:cache` 将所有配置文件的配置缓存到单个文件里，这将会将所有配置选项合并到单个文件从而可以被框架快速加载。

应用一旦上线，就要运行一次 `php artisan config:cache`，但是在本地开发时，没必要经常运行该命令，因为配置值经常需要改变。

<a name="maintenance-mode"></a>
## 维护模式

当你的应用处于维护模式时，所有对应用的请求都会返回同一个自定义视图。这一机制在对应用进行升级或者维护时，使得“关闭”站点变得轻而易举。对维护模式的判断代码位于应用默认的中间件栈中，如果应用处于维护模式，则状态码为 503 的   `MaintenanceModeException` 将会被抛出。

要开启维护模式，只需执行 Artisan 命令 `down` 即可：

    php artisan down

你还可以向 `down` 命令提供 `message` 和 `retry` 选项。 `message` 值可以用于显示或记录自定义消息，而 `retry` 值将被设置为 `Retry-After` HTTP头的值：

    php artisan down --message='Upgrading Database' --retry=60

要关闭维护模式，对应的 Artisan 命令是 `up`：

    php artisan up

#### 维护模式响应模板

维护模式响应的默认模板是 `resources/views/errors/503.blade.php`。您可以根据应用程序的需要自由修改此视图。

#### 维护模式 & 队列

当你的站点处于维护模式中时，所有的 [队列任务](/laravel/{{version}}/queues) 都不会执行。一旦应用程序退出维护模式，任务将继续正常处理。

#### 维护模式的替代方案

由于维护模式命令的执行需要几秒时间，你可以考虑使用 [Envoyer](https://envoyer.io) 实现 0 秒下线作为替代方案。
