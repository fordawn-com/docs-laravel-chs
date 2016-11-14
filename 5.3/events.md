# 事件

- [简介](#introduction)
- [注册事件 & 监听器](#registering-events-and-listeners)
    - [生成事件 & 监听器类](#generating-events-and-listeners)
    - [手动注册事件](#manually-registering-events)
- [定义事件](#defining-events)
- [定义监听器](#defining-listeners)
- [事件监听器队列](#queued-event-listeners)
    - [手动访问队列](#manually-accessing-the-queue)
- [触发事件](#firing-events)
- [事件订阅者](#event-subscribers)
    - [编写事件订阅者](#writing-event-subscribers)
    - [注册事件订阅者](#registering-event-subscribers)

<a name="introduction"></a>
## 简介

Laravel 事件提供了简单的观察者模式实现，允许你订阅和监听应用中的事件。事件类通常存放在 `app/Events` 目录，监听器存放在 `app/Listeners`。如果你在应用中没有看到这些目录，不要担心，因为它们会在你使用Artisan命令生成事件和监听器的时候创建。

事件为应用功能模块解耦提供了行之有效的办法，因为单个事件可以有多个监听器而这些监听器之间并不相互依赖。例如，你可能想要在每次订单发送时给用户发送一个Slack通知，有了事件之后，你大可不必将订单处理代码和Slack通知代码耦合在一起，而只需要简单触发一个可以被监听器接收并处理为Slack通知的 `OrderShipped` 事件即可。

<a name="registering-events-and-listeners"></a>
## 注册事件 & 监听器

Laravel 自带的 `EventServiceProvider` 为事件监听器注册提供了方便之所。其中的 `listen` 属性包含了事件（键）和对应监听器（值）数组。如果应用需要，你可以添加多个事件到该数组。例如，让我们添加一个 `OrderShipped` 事件：

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'App\Events\OrderShipped' => [
            'App\Listeners\SendShipmentNotification',
        ],
    ];

<a name="generating-events-and-listeners"></a>
### 生成事件 & 监听器类

当然，手动为每个事件和监听器创建文件是很笨重的，取而代之地，我们可见简单添加监听器和事件到 `EventServiceProvider`  然后使用 `event:generate` 命令。该命令将会生成罗列在 `EventServiceProvider` 中的所有事件和监听器。当然，已存在的事件和监听器不会被重复创建：

    php artisan event:generate

<a name="manually-registering-events"></a>
### 手动注册事件

通常，我们需要通过 `EventServiceProvider` 的 `$listen` 数组注册事件，此外，你还可以在 `EventServiceProvider` 的 `boot` 方法中手动注册基于闭包的事件：

    /**
     * Register any other events for your application.
     *
     * @return void
     */
    public function boot()
    {
        parent::boot();

        Event::listen('event.name', function ($foo, $bar) {
            //
        });
    }

#### 通配符事件监听器

你甚至还可以使用通配符 `*` 来注册监听器，这样就可以通过同一个监听器捕获多个事件。通配符监听器接收整个事件数据数组作为参数：

    Event::listen('event.*', function (array $data) {
        //
    });

<a name="defining-events"></a>
## 定义事件

事件类是一个处理与事件相关的简单数据容器，例如，假设我们生成的 `OrderShipped` 事件接收一个 [Eloquent ORM](/laravel/{{version}}/eloquent) 对象：

    <?php

    namespace App\Events;

    use App\Order;
    use App\Events\Event;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Event
    {
        use SerializesModels;

        public $order;

        /**
         * Create a new event instance.
         *
         * @param  Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }
    }

正如你所看到的，该事件类不包含任何特定逻辑，只是一个存放被购买的 `Order` 对象的容器，如果事件对象被序列化的话，事件使用的 `SerializesModels` trait 将会使用 PHP 的 `serialize` 函数序列化所有 Eloquent 模型。

<a name="defining-listeners"></a>
## 定义监听器

接下来，让我们看看我们的示例事件的监听器，事件监听器在 `handle` 方法中接收事件实例，`event:generate` 命令将会自动在 `handle` 方法中导入合适的事件类和类型提示事件。在 `handle` 方法内，你可以执行任何需要的逻辑以响应事件：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;

    class SendShipmentNotification
    {
        /**
         * Create the event listener.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Handle the event.
         *
         * @param  OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            // Access the order using $event->order...
        }
    }

> {tip} 你的事件监听器还可以在构造器中类型提示任何需要的依赖，所有事件监听器通过 [服务容器](/laravel/{{version}}/container) 解析，所以依赖会自动注入。

#### 停止事件继续往下传播

有时候，你希望停止事件被传播到其它监听器，你可以通过从监听器的 `handle` 方法中返回 `false` 来实现。

<a name="queued-event-listeners"></a>
## 事件监听器队列

如果监听器将要执行耗时任务比如发送邮件或者发送HTTP请求，那么将监听器放到队列是一个不错的选择。在队列化监听器之前，确保已经 [配置好队列](/laravel/{{version}}/queues) 并且在服务器或本地环境启动一个队列监听器。

要指定某个监听器需要放到队列，只需要让监听器类实现 `ShouldQueue` 接口即可，通过 Artisan 命令 `event:generate`  生成的监听器类已经将接口导入当前命名空间，所以你可以直接拿来使用：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        //
    }

就是这么简单！当这个监听器被调用的时候，将会使用 Laravel 的 [队列系统](/laravel/{{version}}/queues) 通过事件分发器自动推送到队列。如果通过队列执行监听器的时候没有抛出任何异常，队列任务会在执行完成后被自动删除。

<a name="manually-accessing-the-queue"></a>
### 手动访问队列

如果你需要手动访问底层队列任务的 `delete` 和 `release` 方法，在生成的监听器中默认导入的 `Illuminate\Queue\InteractsWithQueue` trait 为此提供了访问这两个方法的权限：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        public function handle(OrderShipped $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="firing-events"></a>
## 触发事件

要触发一个事件，可以传递事件实例到辅助函数 `event`，这个辅助函数会分发事件到所有注册的监听器。由于辅助函数 `event` 全局有效，所以可以在应用的任何地方调用它：

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  int  $orderId
         * @return Response
         */
        public function ship($orderId)
        {
            $order = Order::findOrFail($orderId);

            // Order shipment logic...

            event(new OrderShipped($order));
        }
    }

> {tip} 测试的时候，只需要断言特定事件被触发，无需真正触发监听器，Laravel内置的 [测试函数](/laravel/{{version}}/mocking#mocking-events) 让这一实现不在话下。

<a name="event-subscribers"></a>
## 事件订阅者

<a name="writing-event-subscribers"></a>
### 编写事件订阅者

事件订阅者是指那些在类本身中订阅多个事件的类，通过事件订阅者你可以在单个类中定义多个事件处理器。订阅者需要定义一个 `subscribe` 方法，该方法中传入一个事件分发器实例。你可以在给定的分发器中调用 `listen` 方法注册事件监听器：

    <?php

    namespace App\Listeners;

    class UserEventSubscriber
    {
        /**
         * Handle user login events.
         */
        public function onUserLogin($event) {}

        /**
         * Handle user logout events.
         */
        public function onUserLogout($event) {}

        /**
         * Register the listeners for the subscriber.
         *
         * @param  Illuminate\Events\Dispatcher  $events
         */
        public function subscribe($events)
        {
            $events->listen(
                'Illuminate\Auth\Events\Login',
                'App\Listeners\UserEventSubscriber@onUserLogin'
            );

            $events->listen(
                'Illuminate\Auth\Events\Logout',
                'App\Listeners\UserEventSubscriber@onUserLogout'
            );
        }

    }

<a name="registering-event-subscribers"></a>
### 注册事件订阅者

编写好订阅者之后，就可以通过事件分发器对订阅者注册，你可以使用 `EventServiceProvider` 提供的 `$subcribe` 属性来注册订阅者。例如，让我们添加 `UserEventSubscriber`：

    <?php

    namespace App\Providers;

    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * The event listener mappings for the application.
         *
         * @var array
         */
        protected $listen = [
            //
        ];

        /**
         * The subscriber classes to register.
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventSubscriber',
        ];
    }
