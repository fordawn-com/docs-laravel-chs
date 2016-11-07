# Installation

- [安装](#installation)
    - [服务器要求](#server-requirements)
    - [Installing Laravel](#installing-laravel)
    - [Configuration](#configuration)

<a name="installation"></a>
## 安装

<a name="server-requirements"></a>
### 服务器要求

Laravel 框架有对服务器有少量要求，当然，Laravel Homestead 已经满足所有这些要求，所以我们强烈推荐使用 Homestead 作为 Laravel 本地开发环境（Mac的话还可以使用Valet作为本地开发环境）。

不过，如果你没有使用 Homestead，那么需要保证开发环境满足以下要求：

<div class="content-list" markdown="1">
- PHP版本 >= 5.6.4
- PHP扩展：OpenSSL
- PHP扩展：PDO
- PHP扩展：Mbstring
- PHP扩展：Tokenizer
- PHP扩展：XML
</div>

<a name="installing-laravel"></a>
### 安装 Laravel

Laravel 使用 [Composer](http://getcomposer.org) 管理依赖，因此，使用 Laravel 之前，确保机器上已经安装了Composer。

#### 通过 Laravel 安装器

首先，通过 Composer 安装 Laravel 安装器：

    composer global require "laravel/installer"

确保 `$HOME/.composer/vendor/bin`（或你系统中的等效目录）在环境变量中，以便系统可以找到 `laravel` 命令。

安装完成后，使用 `laravel new` 命令会当前目录下创建一个新的 Laravel 应用，例如，`laravel new blog` 将会创建一个名为 `blog` 的新应用，且包含所有  Laravel 依赖：

    laravel new blog

#### 通过 Composer Create-Project

或者，你也可以在终端中通过 `create-project` 命令来安装 Laravel 应用：

    composer create-project --prefer-dist laravel/laravel blog

#### 本地开发服务器

如果你在本地安装了 PHP，并且希望使用PHP的内置开发服务器来为应用程序提供服务，你可以使用 `serve` Artisan 命令。这个命令将在 `http://localhost:8000` 启动一个开发服务器：

    php artisan serve

当然，通过 [Homestead](/laravel/{{version}}/homestead) 和 [Valet](/laravel/{{version}}/valet) 可以获得更强大的本地开发选项。

<a name="configuration"></a>
### 配置

#### Public 目录

安装完 Laravel 后，需要将HTTP服务器的web根目录指向 `public` 目录，该目录下的 `index.php` 文件将作为前端控制器，所有HTTP请求都会通过该文件进入应用。

#### 配置文件

Laravel框架的所有配置文件都存放在 `config` 目录下，所有的配置项都有注释，所以你可以轻松遍览这些配置文件以便熟悉所有配置项。

#### 目录权限

安装完 Laravel 后，需要配置一些目录的读写权限 `storage` 和 `bootstrap/cache` 目录应该是可写的，如果你使用 [Homestead](/laravel/{{version}}/homestead) 虚拟机做为开发环境，这些权限已经设置好了。

#### 应用密钥

接下来要做的事情就是将应用密钥（APP_KEY）设置为一个随机字符串，如果你是通过 Composer 或者 Laravel 安装器安装的话，该 key 的值已经通过 `php artisan key:generate` 命令生成好了。

通常，此字符串应为32个字符长。 该键可以在 `.env` 环境文件中设置。 如果你没有将 `.env.example` 文件重命名为 `.env`，现在立即这样做。 **如果未设置应用程序密钥，你的用户会话和其他加密数据将不安全！**

#### 附加配置
Laravel 几乎不再需要其它任何配置就可以正常使用了，但是，你最好再看看 `config/app.php` 文件和它的注释，其中包含了一些基于应用可能需要进行改变的配置，比如 `timezone` 和 `locale`。

你可能还想要配置 Laravel 的一些其它组件，例如：

<div class="content-list" markdown="1">
- [缓存](/laravel/{{version}}/cache#configuration)
- [数据库](/laravel/{{version}}/database#configuration)
- [会话](/laravel/{{version}}/session#configuration)
</div>

安装完成后，就应该开始 [配置本地环境](/laravel/{{version}}/configuration#environment-configuration)。
