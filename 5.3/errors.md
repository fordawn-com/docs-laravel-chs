# 错误 & 日志

- [简介](#introduction)
- [配置](#configuration)
    - [错误详细信息](#error-detail)
    - [日志存储](#log-storage)
    - [日志错误级别](#log-severity-levels)
    - [自定义 Monolog 配置](#custom-monolog-configuration)
- [异常处理器](#the-exception-handler)
    - [report 方法](#report-method)
    - [render 方法](#render-method)
- [HTTP 异常](#http-exceptions)
    - [自定义 HTTP 错误页面](#custom-http-error-pages)
- [日志](#logging)

<a name="introduction"></a>
## 简介

Laravel 默认已经为我们配置好了错误和异常处理，我们在 `App\Exceptions\Handler` 类中触发异常并将响应返回给用户。本教程我们将深入探讨这个类。

此外，Laravel 还集成了 [Monolog](https://github.com/Seldaek/monolog) 日志库以便提供各种功能强大的日志处理器，默认情况下，Laravel 已经为我们配置了一些处理器，我们可以选择单个日志文件，也可以选择记录错误信息到系统日志。

<a name="configuration"></a>
## 配置

<a name="error-detail"></a>
### 错误详细信息

配置文件 `config/app.php` 中的 `debug` 配置项控制浏览器显示的错误详情数量。默认情况下，该配置项通过 `.env` 文件中的环境变量 `APP_DEBUG` 进行设置。

对本地开发而言，你应该设置环境变量 `APP_DEBUG` 值为 `true`。在生产环境，该值应该被设置为 `false`。如果在生产环境被设置为 `true`，就有可能将一些敏感的配置值暴露给终端用户。

<a name="log-storage"></a>
### 日志存储

默认情况下，Laravel 支持日志方法 `single`, `daily`, `syslog` 和 `errorlog`。如果你想要日志文件按日生成而不是生成单个文件，应该在配置文件 `config/app.php` 中设置 `log` 值如下：

    'log' => 'daily'

#### 日志文件最大生命周期

使用 `daily` 日志模式的时候，Laravel 默认最多为我们保留最近5天的日志，如果你想要修改这个时间，需要添加一个配置 `log_max_files` 到 `app` 配置文件：

    'log_max_files' => 30

<a name="log-severity-levels"></a>
### 日志错误级别

使用 Monolog 的时候，日志消息可能有不同的错误级别，默认情况下，Laravel 将所有日志写到 storage 目录，但是在生产环境中，你可能想要配置最低错误级别，这可以通过在配置文件 `app.php` 中通过添加配置项 `log_level` 来实现。

该配置项被配置后，Laravel 会记录所有错误级别大于等于这个指定级别的日志，例如，默认 `log_level` 是 `error`，则将会记录 **error**、**critical**、**alert** 以及 **emergency** 级别的日志信息：

    'log_level' => env('APP_LOG_LEVEL', 'error'),

> {tip} Monolog支持以下错误级别 —— `debug`、`info`、`notice`、`warning`、`error`、`critical`、`alert`、`emergency`。

<a name="custom-monolog-configuration"></a>
### 自定义 Monolog 配置

如果你想要在应用中完全控制 Monolog 的配置，可以使用 `configureMonologUsing` 方法。你需要在 `bootstrap/app.php` 文件返回 `$app` 变量之前调用该方法：

    $app->configureMonologUsing(function($monolog) {
        $monolog->pushHandler(...);
    });

    return $app;

<a name="the-exception-handler"></a>
## The 异常处理器

所有异常都由类 `App\Exceptions\Handler` 处理，该类包含两个方法：`report` 和 `render`。下面我们详细阐述这两个方法。

<a name="report-method"></a>
### report 方法



`report` 方法用于记录异常并将其发送给外部服务如 [Bugsnag](https://bugsnag.com) 或 [Sentry](https://github.com/getsentry/sentry-laravel)，默认情况下，`report` 方法只是将异常传递给异常被记录的基类，当然你也可以按自己的需要记录异常并进行相关处理。

例如，如果你需要以不同方式报告不同类型的异常，可使用 PHP 的 `instanceof` 比较操作符：

    /**
     * Report or log an exception.
     *
     * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
     *
     * @param  \Exception  $exception
     * @return void
     */
    public function report(Exception $exception)
    {
        if ($exception instanceof CustomException) {
            //
        }

        return parent::report($exception);
    }

#### 通过类型忽略异常

异常处理器的 `$dontReport` 属性包含一个不会被记录的异常类型数组，默认情况下，404错误异常不会被写到日志文件，如果需要的话你可以添加其他异常类型到这个数组：

    /**
     * A list of the exception types that should not be reported.
     *
     * @var array
     */
    protected $dontReport = [
        \Illuminate\Auth\AuthenticationException::class,
        \Illuminate\Auth\Access\AuthorizationException::class,
        \Symfony\Component\HttpKernel\Exception\HttpException::class,
        \Illuminate\Database\Eloquent\ModelNotFoundException::class,
        \Illuminate\Validation\ValidationException::class,
    ];

<a name="render-method"></a>
### render 方法

`render` 方法负责将给定异常转化为发送给浏览器的 HTTP 响应，默认情况下，异常被传递给为你生成响应的基类。当然，你也可以按照自己的需要检查异常类型或者返回自定义响应：

    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof CustomException) {
            return response()->view('errors.custom', [], 500);
        }

        return parent::render($request, $exception);
    }

<a name="http-exceptions"></a>
## HTTP 异常

有些异常描述来自服务器的 HTTP 错误码，例如，这可能是一个“页面未找到”错误（404），“认证失败错误”（401）亦或是程序出错造成的 500 错误，为了在应用中生成这样的响应，可以使用 `abort` 方法：

    abort(404);

`abort` 方法会立即引发一个会被异常处理器渲染的异常，此外，你还可以像这样提供响应描述：

    abort(403, 'Unauthorized action.');

<a name="custom-http-error-pages"></a>
### 自定义HTTP错误页面

在 Laravel 中，返回不同 HTTP 状态码的错误页面很简单，例如，如果你想要自定义404错误页面，创建一个 `resources/views/errors/404.blade.php` 文件，该视图文件用于渲染程序返回的所有404错误。需要注意的是，该目录下的视图命名应该和相应的HTTP状态码相匹配。由 `abort` 函数引发的 `HttpException` 实例将作为 `$exception` 变量传递给视图。

<a name="logging"></a>
## 日志

Laravel 基于强大的 [Monolog](http://github.com/seldaek/monolog) 库提供了简单的日志抽象层，默认情况下，Laravel被配置为在 `storage/logs` 目录下每天为应用生成日志文件，你可以使用 `Log` [门面](/laravel/{{version}}/facades) 记录日志信息到日志中：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Support\Facades\Log;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            Log::info('Showing user profile for user: '.$id);

            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

该日志记录器提供了 [RFC 5424](http://tools.ietf.org/html/rfc5424) 中定义的八种日志级别：**emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** 和 **debug**。

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

#### 上下文信息

上下文数据也会以数组形式传递给日志方法，然后和日志消息一起被格式化和显示：

    Log::info('User failed to login.', ['id' => $user->id]);

#### 访问底层 Monolog 实例

Monolog有多个可用于日志的处理器，如果需要的话，你可以访问Laravel使用的底层Monolog实例：

    $monolog = Log::getMonolog();
