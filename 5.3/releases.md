# 发行说明

- [支持政策](#support-policy)
- [Laravel 5.3](#laravel-5.3)

<a name="support-policy"></a>
## 支持政策

对于 LTS 版本，比如 Laravel 5.1，我们将会提供为期两年的 bug 修复和三年的安全修复支持。LTS 版本将会提供最长时间的支持和维护。对于其他通用版本，只提供六个月的 bug 修复和一年的安全修复支持，比如 Laravel 5.3。

<a name="laravel-5.3"></a>
## Laravel 5.3

Laravel 5.3 在 5.2 的基础上继续进行优化，提供了大量新功能和新特性：基于驱动的通知系统；通过Laravel Echo提供强大的实时支持；通过Laravel Passport实现无痛的OAuth2服务器；通过Laravel Scout实现全文模型搜索；在Laravel Elixir中支持Webpack；“可邮寄”的对象；明确分离web和api路由；基于闭包的控制台命令；存储上传文件的辅助函数；支持POPO和单动作控制器；以及优化前端脚手架；等等等等。

### 通知（Notifications）

> {video} Laracasts上有关于此特性的免费[视频教程](https://laracasts.com/series/whats-new-in-laravel-5-3/episodes/9)。

Laravel Notifications为我们提供了简单、优雅的API用于在不同的发行渠道中发送通知，例如邮件、SMS、Slack等等。例如，你可以定义一个单据已经支付的通知，然后通过邮件和SMS发送这个通知，你可以使用一个很简单的来实现：

    $user->notify(new InvoicePaid($invoice));

[Laravel社区](http://laravel-notification-channels.com)已经为通知系统编写了各种各样的驱动，包括对iOS和Android通知的支持，要学习更多关于通知系统的细节，查看其 [相应文档](/laravel/5.3/notifications)。

### WebSockets / 事件广播（Event Broadcasting）

事件广播在之前版本的Laravel中已经存在，Laravel 5.3 通过为已私有和已存在的 WebSocket 频道添加频道级认证对此功能进行了极大的优化和提升：

    /*
     * Authenticate the channel subscription...
     */
    Broadcast::channel('orders.*', function ($user, $orderId) {
        return $user->placedOrder($orderId);
    });

Laravel Echo，通过NPM安装的全新的JavaScript包，将和Laravel 5.3一起发布，用于为订阅频道以及在客户端JavaScript应用中监听服务器端事件提供了简单、优美的API，Echo包含对[Pusher](https://pusher.com)和[Socket.io](http://socket.io)的支持：

    Echo.channel('orders.' + orderId)
        .listen('ShippingStatusUpdated', (e) => {
            console.log(e.description);
        });

为了订阅到传统频道，Laravel Echo还使得订阅到提供谁在监听给定频道信息的已存在频道变得简单：

    Echo.join('chat.' + roomId)
        .here((users) => {
            //
        })
        .joining((user) => {
            console.log(user.name);
        })
        .leaving((user) => {
            console.log(user.name);
        });

要学习更多关于Echo和事件广播的内容，请参考其[对应文档](/laravel/5.3/broadcasting)。

### Laravel Passport (OAuth2 Server)

> {video} Laracasts的[免费视频](https://laracasts.com/series/whats-new-in-laravel-5-3/episodes/13)。

Laravle 5.3 使用[Laravel Passport](/laravel/{{version}}/passport)让API认证变得简单。Laravel Passport可以在几分钟内为应用提供一个完整的Oauth2服务器实现，Passport基于 Alex Bilbie维护的 [League OAuth2 server](https://github.com/thephpleague/oauth2-server) 实现。

Passport使得通过OAuth2授权码获取访问令牌（access token）变得轻松，你还可以允许用户通过Web UI创建“个人访问令牌”。为了让你更快上手，Passport内置了一个[Vue组件](https://vuejs.org)，该组件提供了OAuth2后台界面功能，允许用户创建客户端、撤销访问令牌，以及更多其他功能：

    <passport-clients></passport-clients>
    <passport-authorized-clients></passport-authorized-clients>
    <passport-personal-access-tokens></passport-personal-access-tokens>

如果你不想使用Vue组件，欢迎提供你自己的用于管理客户端和访问令牌的前端后台。Passport提供了一个简单的JSON API，你可以在前端使用任何JavaScript框架与之集成。

当然，Passport还让定义可能在应用消费你的API期间被请求的访问令牌域变得简单：

    Passport::tokensCan([
        'place-orders' => 'Place new orders',
        'check-status' => 'Check order status',
    ]);

此外，Passport还包含了用于验证访问令牌认证请求包含必要令牌域的中间件：

    Route::get('/orders/{order}/status', function (Order $order) {
        // Access token has "check-status" scope...
    })->middleware('scope:check-status');

最后，Passport还支持从JavaScript应用访问你的API，而不必担心访问令牌传输，Passport通过加密JWT cookies和同步CSRF令牌来实现这一功能，从而让开发者专注于业务开发。想要了解更多Passport的信息，请查看[相关文档](/docs/5.3/passport)。

### 搜索 (Laravel Scout)

Laravel Scout 提供了一个简单的、基于驱动的针对 [Eloquent模型](/laravel/5.3/eloquent) 的全文搜索解决方案。通过模型观察者，Scout会自动同步更新Eloquent记录的搜索索引，目前，Scout使用 [Algolia](https://www.algolia.com/) 驱动，不过，编写自己的驱动很简单，你可以通过自己的搜索实现扩展Scout。

你可以简单通过添加 `Searchable` trait到模型让模型变得可搜索：

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;
    }

trait被添加到模型之后，当保存模型实例的时候其信息将会被同步到搜索索引：

    $order = new Order;

    // ...

    $order->save();

模型被索引之后，就可以通过模型进行全文搜索了，甚至还可以对搜索结果进行分页：

    return Order::search('Star Trek')->get();

    return Order::search('Star Trek')->where('user_id', 1)->paginate();

当然，Scout还有很多其他特性，具体请查看 [相关文档](/laravel/5.3/scout) 。

### 可邮寄对象（Mailable Objects）

> {video} Laracasts上有关于该特性的免费视 [视频教程](https://laracasts.com/series/whats-new-in-laravel-5-3/episodes/6) 。

Laravel 5.3支持可邮寄对象，这些对象可以以一个简单对象的形式表示邮件信息，而不再需要在闭包中自定义邮件信息，例如，你可以定义一个简单的邮寄对象用作欢迎邮件：

    class WelcomeMessage extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.welcome');
        }
    }

可邮寄对象被创建以后，你可以使用一个简单、优雅的API将其发送给用户。可邮寄对象可以在浏览代码的同时了解邮件信息：

    Mail::to($user)->send(new WelcomeMessage);

当然，你还可以标记可邮寄对象为“队列化”，这样这封邮件就会在后台通过队列任务发送：

    class WelcomeMessage extends Mailable implements ShouldQueue
    {
        //
    }

更多可邮寄对象细节请查看其 [对应文档](/laravel/5.3/mail) 。

### 存储上传文件

> {video} Laracasts上有关于该特性的免费 [视频教程](https://laracasts.com/series/whats-new-in-laravel-5-3/episodes/12) 。

在web应用中，最常见的任务之一就是保存用户上传文件了，比如头像、照片、文档等等。Laravel 5.3通过在上传文件实例上使用新的 `store` 方法让这一工作变得简单。只需要简单调用 `store` 方法并传入文件保存路径即可：

    /**
     * Update the avatar for the user.
     *
     * @param  Request  $request
     * @return Response
     */
    public function update(Request $request)
    {
        $path = $request->file('avatar')->store('avatars', 's3');

        return $path;
    }

更多细节请查看其 [完整文档](/laravel/{{version}}/filesystem#file-uploads) 。

### Webpack & Laravel Elixir

Laravel Elixir 6.0和Laravel 5.3一起发布，新版本将捆绑支持Webpack和Rollup JavaScript模块。默认情况下，Laravel 5.3 的 `gulpfile.js` 文件现在已经使用Webpack来编译JavaScript。[Elixir文档](/laravel/5.3/elixir) 包含所有信息：

    elixir(mix => {
        mix.sass('app.scss')
           .webpack('app.js');
    });

### 前端架构

> {video} Laracasts有关于本特性的免费 [视频教程](https://laracasts.com/series/whats-new-in-laravel-5-3/episodes/4) 。

Laravel 5.3 提供了一个更加现代的前端架构。这主要会影响 `make:auth` 命令生成的认证脚手架。不再从CDN中加载前端资源，所有依赖都被定义在默认的 `package.json` 文件中。

此外，支持单文件的 [Vue组件](https://vuejs.org) 现在已经开箱支持， `resources/assets/js/components` 目录下包含了一个简单的示例组件 `Example.vue` ，新的 `resources/assets/js/app.js` 文件将会启动被配置你的JavaScript库以及Vue组件。

这种架构对开始开发现代的、强大的JavaScript应用提供了更好的指导，而不需要要求应用使用任何给定JavaScript或者CSS框架。关于如何进行现代Laravel前端开发，请查看新的 [前端架构文档](/laravel/5.3/frontend) 。

### 路由文件

默认情况下，新安装的Laravel 5.3应用在新的顶级目录 `routes` 下包含两个HTTP路由文件。`web` 和 `api` 路由文件在如何分割Web界面和API路由方面提供了指导。`api` 路由文件中的路由会通过 `RouteServiceProvider` 自动添加 `api` 前缀。

### 闭包控制台命令

除了通过命令类定义之外，现在Artisan命令还可以在 `app/Console/Kernel.php` 文件的 `commands` 方法中以简单闭包的方式定义。在新安装的Laravel 5.3应用中， `commands` 方法会加载 `routes/console.php` 文件，从而允许你基于闭包、以路由风格定义控制台命令：

    Artisan::command('build {project}', function ($project) {
        $this->info('Building project...');
    });

更多详情请查看完整的 [Artisan文档](/laravel/5.3/artisan#closure-commands)。

### `$loop` 变量

> {video} Laracasts中有关于此特性的免费 [视频教程](https://laracasts.com/series/whats-new-in-laravel-5-3/episodes/7)。

当我们在Blade模板中循环遍历的时候，`$loop` 变量将会在循环中生效。通过该变量可以访问很多有用的信息，比如当前循环索引值，以及当前循环是第一个还是最后一个迭代：

    @foreach ($users as $user)
        @if ($loop->first)
            This is the first iteration.
        @endif

        @if ($loop->last)
            This is the last iteration.
        @endif

        <p>This is user {{ $user->id }}</p>
    @endforeach

更多详情请查看完整的 [Blade文档](/docs/5.3/blade#the-loop-variable)。
