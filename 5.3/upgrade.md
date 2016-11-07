
# 升级指南

- [从 5.2 升级到 5.3](#upgrade-5.3.0)

<a name="upgrade-5.3.0"></a>
## 从 5.2 升级到 5.3

#### 注：预计升级升级时间：2-3小时

> {note} 我们尽量罗列出每一个不兼容的变更。但因为其中一些不兼容变更只存在于框架很不起眼的地方，事实上只有一小部分会真正影响到你的应用程序。

### PHP & HHVM

Laravel 5.3要求PHP 5.6.4及以上版本，官方将不再支持HHVM，因为其不包含PHP 5.6+新提供的语言特性。

### 弃用的功能

所有罗列在[Laravel 5.2](https://laravel.com/docs/5.3/upgrade#5.2-deprecations)升级指南中的废弃功能都已从框架中移除，你需要查看这个列表以确定不再使用这些废弃功能。

### 应用服务提供者

你可以将 `EventServiceProvider`，`RouteSerivceProvider` 和 `AuthServiceProvider` 类的 `boot` 方法里的参数移除。任何给定参数的调用都可以被转化为等效的 [facade](/laravel/5.3/facades)。举个例子，你可以使用 `Event` 门面， 来代替 `$dispatcher` 上的方法。同样，也可以使用 `Route` 门面，来替代 `$router` 参数上的方法，用 `Gate` 门面，来代替 `$gate`参数的方法。

> {note} 当把方法调用转化为门面调用时，请先在服务提供者顶部引入对应的门面类。

### 数组

#### Key / Value 顺序更改

Arr类上的 `first`、`last`、`contains` 方法现在将"value"作为第一个参数传递给给定闭包，例如：

    Arr::first($array, function ($value, $key) {
        return ! is_null($value);
    });

在Laravel之前版本中，`$key` 是第一个参数，但是由于大多数使用案例只对 `$value` 感兴趣，所以我们将其放到第一个。你可以在应用中进行一次 "全局搜索" 以验证是否你在应用中通过旧的方式使用了这个函数。

### Artisan

##### The `make:console` 命令

`make:console` 命令现在被重命名为 `make:command`。

### 认证

#### 认证脚手架

Laravel框架提供的默认的两个认证控制器现在被分割成更小的四个，这一更改让认证控制器变得更加清爽、责任更加明确。升级应用认证控制器到最新的最简单方法就是从[GitHub](https://github.com/laravel/laravel/tree/master/app/Http/Controllers/Auth)上将四个控制器代码拷贝过来复制到项目中。

你还要确保在路由文件 `routes/web.php` 中调用了 `Auth::routes()` 方法，该方法在底层已经为新控制器注册了适当的路由。

这些新控制器拷贝到应用后，需要重新实现之前在认证控制器中实现的方法和业务。例如，如果你在自定义用于认证的guard，需要重写控制器的 `guard` 方法，你可以检查每个认证控制器的trait以判断要重写哪些方法。

> {tip} 如果你没有自定义认证控制器，只需要将代码从Github拷到本地项目，并确保在路由文件 `routes/web.php` 中调用了 `Auth::routes` 方法。

#### 密码重置邮件

密码重置邮件现在使用新的通知功能（Laravel Notifications），如果你想要在发送密码重置链接的时候自定义通知发送，需要重写 `Illuminate\Auth\Passwords\CanResetPassword` trait上的 `sendPasswordResetNotification` 方法。

`User` 模型**必须**使用新的 `Illuminate\Notifications\Notifiable` trait以便邮件重置链接可以正常发送：

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;
    }

> {note} 不要忘了在配置文件 `config/app.php` 的 `providers` 数组中注册服务提供者 `Illuminate\Notifications\NotificationServiceProvider`。

#### 使用 POST 登出

`Auth::routes` 方法现在为 `/logout` 注册了一个 `POST` 路由取代之前的 `GET` 路由。这可以防止其它应用让用户从应用中退出。想要升级的话，需要将原来的退出请求转化为 `POST` 请求方式，或者为 `/logout`  URI自定义 `GET` 路由：

    Route::get('/logout', 'Auth\LoginController@logout');

### 授权

#### 使用类名调用策略方法

有时候一些策略方法只接收当前认证用户而不是授权模型实例，最常见的场景就是授权 `create` 动作。例如，如果你在创建一篇博客，你可能希望检查当前用户是否被授权创建文章。

当定义一个不接收模型实例的策略方法时，比如 `create` 方法，对应类名就不再需要以第二个参数的方式传入，只需要传入认证用户实例即可：

    /**
     * Determine if the given user can create posts.
     *
     * @param  \App\User  $user
     * @return bool
     */
    public function create(User $user)
    {
        //
    }

#### `AuthorizesResources` Trait

`AuthorizesResources` trait已经和 `AuthorizesRequests` trait合并到一起，你需要从 `app/Http/Controllers/Controller.php` 中移除 `AuthorizesResources` trait。

### Blade

#### 自定义指令

在之前版本的Laravel中，我们使用 `directive` 方法注册自定义的Blade指令，传递给指令回调的 `$expression` 参数包含了最外层的括号。在Laravel 5.3中，这些最外层的括号将不再包含在传递给指令回调的表达式中，请查看 [Blade](/laravel/5.3/blade#extending-blade) 文档确保自定义的Blade指令还能正常工作。

### 广播

#### 服务提供者

Laravel 5.3 对 [事件广播](/laravel/{{version}}/broadcasting) 进行了显著的优化，需要添加的新的 `BroadcastServiceProvider`（[从GitHub下载文件](https://raw.githubusercontent.com/laravel/laravel/develop/app/Providers/BroadcastServiceProvider.php)）到 `app/Providers` 目录，然后将这个新的服务提供者注册到配置文件 `config/app.php` 的 `providers` 数组中。

### 缓存

#### 扩展闭包绑定 & `$this`

使用闭包调用 `Cache::extend` 方法时，`$this` 会被绑定到 `CacheManager` 实例，从而允许你在扩展闭包中调用其提供的方法：

    Cache::extend('memcached', function ($app, $config) {
        try {
            return $this->createMemcachedDriver($config);
        } catch (Exception $e) {
            return $this->createNullDriver($config);
        }
    });

### Cashier

如果你在使用Cashier，需要升级 `laravel/cashier` 扩展包到 `~7.0` 版本，这一版本的Cashier只修改了一些内置方法以便兼容于Laravel 5.3，并没有什么重大更新。

### 集合

#### Key / Value 顺序调整

集合方法 `first`, `last` 和 `contains` 都将"value"作为第一个参数传递给相应的回调闭包，例如：

    $collection->first(function ($value, $key) {
        return ! is_null($value);
    });

在Laravel之前的版本中，`$key` 是第一个参数，由于大部分使用案例只对 `$value` 感兴趣，所以将其调整为第一个，你需要在应用中对这些方法做一个"全局搜索"，以验证 `$value` 是否按照期望的方式以第一个参数传入闭包。

#### 集合 `where` 默认使用非严格比较 (loose)

集合的 `where` 现在默认使用非严格比较而不是之前的严格比较，如果你想要进行严格比较，可以使用 `whereStrict` 方法。

`where` 方法也不再接收第三个参数标识"严格"，你需要基于应用的需求区分调用 `where` 或 `whereStrict`。

### 控制器

<a name="5.3-session-in-constructors"></a>
#### 构造函数中的Session

在Laravel以前的版本中，你可以在控制器构造函数中获取session变量或者认证后的用户实例。框架从未打算具有如此明显的特性。在Laravel 5.3中，你在控制器构造函数中不再能够直接获取到session变量或认证后的用户实例，因为中间件还未启动。

仍然有替代方案，那就是在控制器构造函数中使用Closure来直接定义中间件。请注意，在使用这个方案的时候，确保你所使用的Laravel版本高于 `5.3.4`：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Support\Facades\Auth;
    use App\Http\Controllers\Controller;

    class ProjectController extends Controller
    {
        /**
         * All of the current user's projects.
         */
        protected $projects;

        /**
         * Create a new controller instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware(function ($request, $next) {
                $this->projects = Auth::user()->projects;

                return $next($request);
            });
        }
    }

当然，你可以在控制器的其他方法中依靠 `Illuminate\Http\Request` 类获取到session以及认证用户信息。

    /**
     * Show all of the projects for the current user.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return Response
     */
    public function index(Request $request)
    {
        $projects = $request->user()->projects;

        $value = $request->session()->get('key');

        //
    }

### 数据库

#### 集合

[查询构建器](/laravel/{{version}}/queries) 现在返回 `Illuminate\Support\Collection` 实例而不是原生数组，以便保持和Eloquent返回结果类型一致。

如果你不想要迁移查询构建器结果到 `Collection` 实例，可以在查询构建器的 `get` 方法后调用 `all` 方法，这将会返回原生的PHP数组结果，从而保证向后兼容：

    $users = DB::table('users')->get()->all();

#### Eloquent的 `$morphClass` 属性

可以在Eloquent模型上定义的 `$morphClass` 属性已经被移除，以便定义一个"morph map"（变形映射），定义变形映射可以支持渴求式加载并且解决使用多态关联关系引起的额外bugs，如果你之前使用了 `$morphClass` 属性，需要使用如下语法将其迁移到 `morphMap`：

```php
Relation::morphMap([
    'YourCustomMorphName' => YourModel::class,
]);
```

例如，如果你之前定义了如下 `$morphClass`：

```php
class User extends Model
{
    protected $morphClass = 'user'
}
```

现在则需要在 `AppServiceProvider` 的 `boot` 方法中定义如下 `morphMap`：

```php
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::morphMap([
    'user' => User::class,
]);
```

#### Eloquent的 `save` 方法

如果模型在上一次获取或保存之后没有被改变过，调用Eloquent的 `save` 方法会返回 `false`。

#### Eloquent 作用域

Eloquent作用域现在会以第一个作用域约束的布尔值为准，例如，如果你的作用域以 `orWhere` 开头，则将不再被转化为正常的 `where`。如果你现在的代码中使用了这个特性（在循环中添加了多个 `orWhere` 约束），需要验证第一个条件是否是正常的 `where` 以避免布尔逻辑问题。

如果你的作用域约束都是以 `where` 开头则不需要做任何调整，你可以通过 `toSql` 方法查看当前的SQL语句：

    User::where('foo', 'bar')->toSql();

#### Join 闭包

`JoinClause` 类被重写以便统一查询构建器的语法，`on` 方法上可选的 `$where` 参数已被移除，要添加"where"条件需要显式使用 [查询构建器](/laravel/{{version}}/queries#where-clauses) 提供的 `where` 方法：

    $query->join('table', function ($join) {
        $join->on('foo', 'bar')->where('bar', 'baz');
    });

`on` 方法已经带有验证，不再包含无效的值。如果想依赖这个特性 (例如： `$join->on('foo', 'in', DB::raw('("bar")'))`)， 你应该用适当的where方法重写条件：

    $join->whereIn('foo', ['bar']);

`$bindings` 属性也被移除，要直接操作join绑定可以使用 `addBinding` 方法：

    $query->join(DB::raw('('.$subquery->toSql().') table'), function ($join) use ($subquery) {
        $join->addBinding($subquery->getBindings(), 'join');
    });

### 加密

#### Mcrypt 加密已被移除

在2015年六月的Laravel 5.1.0版本中，Mcrypt加密就已经被废弃，在Laravel 5.3中该功能被完全移除，以便于统一使用基于OpenSSL的新的加密实现，该实现从Laravel 5.1.0开始就已经作为默认的加密实现方式。

如果你还在配置文件 `config/app.php` 中使用基于 `cipher` 的Mcrypt加密，需要将cipher更新到 `AES-256-CBC` 并且将key设置为32位随机字符串（这可以通过Artisan命令 `php artisan key:generate` 生成）。

如果你在使用Mcrypt加密保存加密数据到数据库，则需要安装包含Mcrypt加密实现的 `laravel/legacy-encrypter` [扩展包](https://github.com/laravel/legacy-encrypter)，你应该通过该扩展包解密数据然后通过新的OpenSSL加密方式对数据进行重新加密。例如，你可能在 [自定义Artisan命令](/laravel/{{version}}/artisan) 中这么做：

    $legacy = new McryptEncrypter($encryptionKey);

    foreach ($records as $record) {
        $record->encrypted = encrypt(
            $legacy->decrypt($record->encrypted)
        );

        $record->save();
    }

### 异常 Handler

#### 构造函数

异常处理器基类现在需要传递一个 `Illuminate\Container\Container` 实例到构造函数，这只有当你在 `app/Exception/Handler.php` 中定义了自定义的 `__construct` 方法时才会对应用产生影响，如果你这么做了，需要传递一个容器实例到 `parent::__construct` 方法：

    parent::__construct(app());

#### Unauthenticated 方法

你应该在 `App\Exceptions\Handler` 类中添加 `unauthenticated` 方法，这个方法将会把认证异常转为HTTP返回：

    /**
     * Convert an authentication exception into an unauthenticated response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Auth\AuthenticationException  $exception
     * @return \Illuminate\Http\Response
     */
    protected function unauthenticated($request, AuthenticationException $exception)
    {
        if ($request->expectsJson()) {
            return response()->json(['error' => 'Unauthenticated.'], 401);
        }

        return redirect()->guest('login');
    }

### 中间件

#### `can` 中间件命名空间修改

HTTP Kernel里 `$routeMiddleware` 属性中的 `can` 中间件需要作如下修改：

    'can' => \Illuminate\Auth\Middleware\Authorize::class,

#### `can` 中间件认证异常

如果用户没有认证的话 `can` 中间件会抛出 `Illuminate\Auth\AuthenticationException` 异常实例，如果你手动捕获了其它异常类型，需要修改为捕获这个异常，在大多数案例中，这一修改对应用不会造成影响。

#### 绑定替代中间件

路由模型绑定现在通过中间件来完成，所有应用都需要在 `app/Http/Kernel.php` 文件的 `web` 中间件组中添加 `Illuminate\Routing\Middleware\SubstituteBindings`：

    \Illuminate\Routing\Middleware\SubstituteBindings::class,

你还需要在HTTP Kernel的 `$routeMiddleware` 属性中注册路由中间件用于绑定替代：

    'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,

这个路由中间件注册后，还要将其添加到 `api` 中间件组：

    'api' => [
        'throttle:60,1',
        'bindings',
    ],

### 通知（Notifications）

#### 安装

Laravel 5.3内置了一个新的、基于驱动的通知系统，你需要在配置文件 `config/app.php` 的 `providers` 数组中注册服务提供者 `Illuminate\Notifications\NotificationServiceProvider`。

你还需要在配置文件 `config/app.php` 的 `aliases` 数组中注册门面 `Illuminate\Support\Facades\Notification`。

最后，你可以在 `User` 模型或其它你希望接收通知的模型中使用 `Illuminate\Notifications\Notifiable` trait。

### 分页

#### 自定义

与之前版本的Laravel相比，Laravel 5.3中自定义分页生成的HTML变得更加简单，你只需要定义一个简单的Blade模板，而不需要定义一个“Presenter”类。自定义分页视图最简单的方式就是通过 `vendor:publish` 命令将它们导出到 `resources/views/vendor` 目录：

    php artisan vendor:publish --tag=laravel-pagination

这个命令会将视图放置到 `resources/views/vendor/pagination` 目录，该目录下的 `default.blade.php` 文件会对应到默认的分页视图。只需要编辑这个文件就可以自定义分页HTML。

阅读完整的 [分页文档](/laravel/{{version}}/pagination) 了解更多实现细节。

### 队列

#### 配置

在队列配置中，所有配置项 `expire` 需要重命名为 `retry_after`，类似的，Beanstalk配置的配置项 `ttr` 也需要重命名为 `retry_after`。这一命名更改让配置项的意义更加明确。

#### 闭包

队列闭包不再支持，如果你在应用中将闭包添加到队列，需要将闭包转化为一个类，然后将类实例添加到队列。

#### 集合序列化

`Illuminate\Queue\SerializesModels` trait现在适当实例化了 `Illuminate\Database\Eloquent\Collection`。对绝大多数应用来说这不算重大更新，但是，如果你的应用绝对依赖于不是通过队列任务从数据库重新获取的集合，那么就要验证这一改动对应用没有负面影响。

#### 后台 Workers

在使用Artisan命令 `queue:work` 的时候不再需要指定 `--daemon` 选项，运行 `php artisan queue:work` 命令的时候自动判断在后台运行。如果你想要处理单个任务，可以在命令后加上 `--once` 选项：

    // Start a daemon queue worker...
    php artisan queue:work

    // Process a single job...
    php artisan queue:work --once

#### 事件数据修改

多个队列任务事件如 `JobProcessing` 和 `JobProcessed` 将不再包含 `$data` 属性，你需要更新应用调用 `$event->job->payload()` 来获取对应数据。

#### 数据库驱动修改

如果你使用 `database` 驱动来储存你的队列任务，则需要删除`jobs_queue_reserved_reserved_at_index` 索引，然后从 `jobs` 表中删除 `reserved` 列。当使用 `database` 驱动的时候，这个列已经不再需要。做完这些修改以后，需要给 `queue` 列和 `reserved_at` 列添加一个复合索引。

下面是一个迁移的例子，可以用于执行必要的修改：

    public function up()
    {
        Schema::table('jobs', function (Blueprint $table) {
            $table->dropIndex('jobs_queue_reserved_reserved_at_index');
            $table->dropColumn('reserved');
            $table->index(['queue', 'reserved_at']);
        });

        Schema::table('failed_jobs', function (Blueprint $table) {
            $table->longText('exception')->after('payload');
        });
    }

    public function down()
    {
        Schema::table('jobs', function (Blueprint $table) {
            $table->tinyInteger('reserved')->unsigned();
            $table->index(['queue', 'reserved', 'reserved_at']);
            $table->dropIndex('jobs_queue_reserved_at_index');
        });

        Schema::table('failed_jobs', function (Blueprint $table) {
            $table->dropColumn('exception');
        });
    }

#### 过程控制扩展

如果你的应用使用了 `--timeout` 选项给队列处理器，需要确保 [pcntl 扩展](http://php.net/manual/en/pcntl.installation.php) 已经安装。

#### 在传统风格队列任务上序列化模型

通常，Laravel的队列任务通过传递任务实例到 `Queue::push` 方法添加到队列，不过，一些应用会通过如下这种传统的方式添加任务到队列：

    Queue::push('ClassName@method');

如果你在使用这种语法，Eloquent模型将不再会被自动序列化然后通过队列重新获取，如果你想要Eloquent模型继续被队列自动序列化，需要在任务类上使用 `Illuminate\Queue\SerializesModels` trait并使用新的 `push` 方法将任务推送到队列：

    Queue::push(new ClassName);

### 路由

#### 资源路由参数默认是单数

之前版本的Laravel中，使用 `Route::resource` 注册的路由参数并没有“单数化”，这可能会在注册路由模型绑定的时候引起一些非预期的行为，例如，给定如下 `Route::resource` 调用：

    Route::resource('photos', 'PhotoController');

`show` 路由的URI将会定义如下：

    /photos/{photos}

在Laravel 5.3中，所有资源路由参数默认都是单数化的，因此，同样调用 `Route::resource` 将会注册如下URI：

    /photos/{photo}

如果你想要继续保持之前版本的行为而不是自动单数化资源路由参数，可以在 `AppServiceProvider` 中这样调用 `singularResourceParameters` 方法：

    use Illuminate\Support\Facades\Route;

    Route::singularResourceParameters(false);

#### 资源路由名称不再受路由前缀影响

当使用 `Route::resource` 的时候，URL前缀将不再影响分配给路由的路由名称，因为这种行为首先违反了使用路由名称的整个目的。

如果你的应用程序在 `Route::group` 中使用 `Route::resource`，并且使用了 `prefix` 选项，你应该检查所有的 `route` 辅助函数和 `UrlGenerator:: route` 调用，以验证不再将此URI前缀附加到路由名称。

如果此更改导致你有两个相同名称的路由，有两种方式来处理。第一种，在调用 `Route::resource` 时，使用 `names` 选项来为给定的路由指定一个自定义名称。 有关更多信息，请参阅 [资源路由文档](/laravel/5.3/controllers#resource-controllers)。 或者，您可以在路由组上添加 `as` 选项：

    Route::group(['as' => 'admin.', 'prefix' => 'admin'], function () {
        //
    });

### 验证

#### 表单请求异常

如果一个表单请求验证失败，Laravel现在会抛出一个 `Illuminate\Validation\ValidationException` 实例而不是 `HttpException` 实例，如果你手动捕获了表单请求的 `HttpException` 实例，需要修改 `catch` 区块代码为捕获 `ValidationException`。

#### 原生可空的检查

当验证数组(arrays)、布尔值(booleans)、整型(integers)、数字(numerics)、字符串(strings)时，`null` 不会被当作有效值，除非在约束条件中设置包含 `nullable`：

    Validate::make($request->all(), [
        'field' => 'nullable|max:5',
    ]);
