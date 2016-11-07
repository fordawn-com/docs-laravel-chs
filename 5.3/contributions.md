# Contribution Guide

- [错误报告](#bug-reports)
- [核心开发讨论](#core-development-discussion)
- [哪个分支？](#which-branch)
- [安全漏洞](#security-vulnerabilities)
- [编码风格](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)

<a name="bug-reports"></a>
## 错误报告

为了鼓励促进更加有效积极的合作，Laravel 强烈鼓励使用 GitHub 的 pull requests，而不是仅仅报告错误，“错误报告”也可以通过一个包含失败测试的 pull requests 的方式提交。

然而，如果你以文件的方式提交缺陷报告，你的问题应该包含一个标题和对该问题的明确说明，还要包含尽可能多的相关信息以及论证该问题的代码示例，错误报告的目的是为了让你自己和 - 其他人 - 更方便的重现错误并对其进行修复。

记住，错误报告被创建是为了其他人遇到同样问题的时候能够和你一起合作解决它，不要寄期望于错误会自动解决或有人跳出来修复它，创建错误报告是为了帮你自己和别人走上修复问题之路。

Laravel 源码通过 Github 进行管理，每一个 Laravel 项目都有其对应的代码库：

- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Cashier for Braintree](https://github.com/laravel/cashier-braintree)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Website](https://github.com/laravel/laravel.com)
- [Laravel Art](https://github.com/laravel/art)

<a name="core-development-discussion"></a>
## 核心开发讨论

你可以在 [issue board](https://github.com/laravel/internals/issues) 上提出新功能或者改进已有功能，如果是新功能的话请至少实现部分代码以便完成新功能开发。

你也可以在 [LaraChat](http://larachat.co) 的 Slack 小组的 `#internals` 频道讨论关于 Laravel 的 bugs、新特性、以及现有功能的实现等。Taylor Otwell，Laravel 的主要维护者，通常在工作日的上午8点到下午5点（西六区或美国芝加哥时间）在线，其它时间也可能偶尔在线。

<a name="which-branch"></a>
## 哪个分支？

**所有**的 bug 修复应该被提交到最新的稳定分支，或者 LTS 分支(5.1)，**永远不要**把 bug 修复提交到 master 分支，除非它们能够修复下个发行版本中的特性。

当前版本中**完全向后兼容**的**次要**特性也可以提交到最新的稳定分支。

**重要**的新特性总是要被提交到 `master` 分支，包括下个发行版本。

如果你不确定是重要特性还是次要特性，请在 Slack 小组 [LaraChat](http://larachat.co) 的 `#internals` 频道咨询 Taylor Otwell。

<a name="security-vulnerabilities"></a>
## 安全漏洞

如果你在 Laravel 中发现安全漏洞，请发送邮件到 <a href="mailto:taylor@laravel.com">taylor@laravel.com</a>，所有的安全漏洞将会被及时解决。

<a name="coding-style"></a>
## 编码风格

Laravel遵循 [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) 编码标准和 [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md) 自动载入标准。

<a name="phpdoc"></a>
### PHPDoc

下面是一个有效的Laravel文档区块示例，注意到 `@param` 属性后面有两个空格，然后是参数类型，接着又是两个空格，最后是参数名称：

    /**
     * Register a binding with the container.
     *
     * @param  string|array  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        //
    }

<a name="styleci"></a>
### StyleCI

如果你的代码格式不是很完美，不必担心，[StyleCI](https://styleci.io/) 会在提交代码时自动为我们修正代码风格以保持和Laravel仓库代码一致，从而让我们更加专注于代码内容而非风格。
