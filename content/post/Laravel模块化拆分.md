---
date: 2017-11-27
title: "Laravel模块化开发"
tags:
    - Laravel
    - PHP
categories:
    - Laravel
    - PHP
gitment: true
---

1. Laravel项目的基本目录结构

    使用composer初始化一个laravel项目
  
    ```shell
    #composer create-project laravel/laravel MyLaravelProject
    ```
    
    初始化完成后观察目录结构基本如下:
    
    ```shell
      .
      ├── app
      │    ├── Console
      │    │    └── Kernel.php
      │    ├── Exceptions
      │    │    └── Handler.php
      │    ├── Http
      │    │    ├── Controllers
      │    │    │    ├── Auth
      │    │    │    └── Controller.php
      │    │    ├── Kernel.php
      │    │    └── Middleware
      │    │        ├── EncryptCookies.php
      │    │        ├── RedirectIfAuthenticated.php
      │    │        ├── TrimStrings.php
      │    │        ├── TrustProxies.php
      │    │        └── VerifyCsrfToken.php
      │    ├── Providers
      │    │    ├── AppServiceProvider.php
      │    │    ├── AuthServiceProvider.php
      │    │    ├── BroadcastServiceProvider.php
      │    │    ├── EventServiceProvider.php
      │    │    └── RouteServiceProvider.php
      │    └── User.php
      ├── artisan
      ├── bootstrap
      │    ├── app.php
      │    └── cache
      │        ├── packages.php
      │        └── services.php
      ├── composer.json
      ├── composer.lock
      ├── config
      │    ├── app.php
      │    ├── auth.php
      │    ├── broadcasting.php
      │    ├── cache.php
      │    ├── database.php
      │    ├── filesystems.php
      │    ├── mail.php
      │    ├── queue.php
      │    ├── services.php
      │    ├── session.php
      │    └── view.php
      ├── database
      │    ├── factories
      │    ├── migrations
      │    └── seeds
      ├── package.json
      ├── phpunit.xml
      ├── public
      │    ├── css
      │    ├── favicon.ico
      │    ├── index.php
      │    ├── js
      │    ├── robots.txt
      │    └── web.config
      ├── readme.md
      ├── resources
      │    ├── assets
      │    ├── lang
      │    │    └── en
      │    │        ├── auth.php
      │    │        ├── pagination.php
      │    │        ├── passwords.php
      │    │        └── validation.php
      │    └── views
      │        └── welcome.blade.php
      ├── routes
      │    ├── api.php
      │    ├── channels.php
      │    ├── console.php
      │    └── web.php
      ├── server.php
      ├── storage
      ├── tests
      └── vendor
    ```
    
    按照框架的约定,通常在开发中
    
    * 数据模型(models)会直接置于app目录下;
    * 控制器(controllers)会置于app/Http/Controllers目录下;
    * 路由配置(routers)会直接写在routes/api.php和web.php中
    * 页面视图(views)会置于resource/views目录下
    * 常量配置值(config)会置于config目录下
    
    实际开发中,一般都会按照框架的这些约定进行编码,这就导致了一些问题
    
    * 随着项目功能的增加,数据模型层中的类越来越多,相互之间的关联越来越多,也就越来越多的会出现数据模型的关联查询,这就存在一种隐患,如果想迁移部分或所有的数据模型层,从现有数据库到其他数据库,这些关联查询,君有可能发生严重的异常;
    * 控制器类功能过于集中,无论是页面跳转的控制,还是数据接口的控制,都写在同一个类中,这导致控制器类的高度复杂,后期很难进行维护,这也不符合单一职责原则的要求;
    * 路由配置过于集中,项目越胖大,涉及的路由地址越多,router配置文件就越复杂,难以维护,许多格式类似的路由,很难进行查询
    * 页面视图文件集中在同一个目录下,命名冲突时有发生,只能通过不断的新建目录来分散文件
    * 常量配置基本同上面,过于集中,debug过程中,需要频繁的在很多文件的目录中进行切换,效率大打折扣 

2. 为什么要拆分模块化开发

    正如前文分析的结果一样,按照框架约定的方式进行开发,会为后续的项目维护带来很大的不方便和维护成本的上升
    
    因此将同一功能拆分成模块,将相关模块的代码文件集中组合在一起,独立维护,同时又对外暴露接口,允许其他模块顺利访问是最简单有效的办法
    

3. 模块化拆分方案

    * 模块拆分准备

      * 创建模块根目录,用于存放拆分后的模块
      
        * 在项目根目录,新建一个Modules目录;
        * 为模块根目录设置根namespaces,laravel框架本身是完全遵循[PSR-4](http://www.php-fig.org/psr/psr-4/)标准进行类加载的,因此需要为模块目录设置根namespaces;
        ```json
        "autoload": {
            "classmap": [
                "database"
            ],
            "psr-4": {
                "App\\": "app/",
                "Modules\\":"Modules/"
            }
        },
        ```
        
        * laravel的[PSR-4](http://www.php-fig.org/psr/psr-4/)是通过composer-autoload实现的,因此在配置好模块的根namespaces后,一定需要重建composer的autoload,使得在运行过程中,模块中的类可以被正常的autoload

        ```shell
          #composer dump-autoload -o
        ```
        
      * 创建各个功能模块的独立目录

        * 完成根namespaces的定以后就可以在Modules目录下,按照业务需要新建各个功能模块的独立目录了
        
    * 配置模块IOC注入

      * 在模块目录下新建Providers目录
      * 在Providers目录中新建模块服务注入类,继承自框架的Illuminate\Support\ServiceProvider类

      ``` php
      <?php

        namespace Modules\模块名称\Providers;

        use Illuminate\Support\ServiceProvider;

        class ModuleServiceProvider extends ServiceProvider
        {
            private $module_name = '模块名称';

            public function register()
            {
                $this->binds();
            }

            public function boot()
            {

            }

            private function binds()
            {

            }
        }
      ```
      
      * 在框架核心配置文件config/app.php的providers数组中将新增的ModuleServiceProvider添加进去

      ```php
      <?php
        ...
        'providers' => [
          ...
          Modules\模块名称\Providers\ModuleServiceProvider::class,
        ]
      ```
        
    * 拆分数据模型到模块

      * 在模块对应的目录下,新建Modules目录,用于放置当前模块相关的数据模型类;       * 注意,其中的数据模型类的namespaces一定要与先前设置的根namespace以及实际目录结构一致;
      例如:
      > 数据模型类的namespaces应该为
      > Modules\模块名称\Models;

    * 拆分控制器到模块

      * 仿照laravel的app目录下控制器的存放目录新建Http/Controllers目录用于放置控制器类
      * 同拆分数据模型类时类似,需要修改对应控制器类的namespaces
      > 控制器目录namespaces应该为
      > Modules\模块名称\Http\Controllers

    * 拆分路由配置到模块

      * 在模块目录下新建Routes目录用于放置路由配置文件,可以现在该Routers目录下新建api.php和web.php两个路由配置文件,分别用来配置api接口和web页面跳转的路由;
      * 在模块的Provider目录新建路由配置注入类,继承自框架的Illuminate\Foundation\Support\Providers\RouteServiceProvider类
      
      ```php
      <?php

        namespace Modules\模块名称\Providers;

        use Illuminate\Foundation\Support\Providers\RouteServiceProvider;
        use Route;

        class ModuleRouterServiceProvider extends RouteServiceProvider
        {
            protected $namespace = 'Modules\模块名称\Http\Controllers';

            public function boot()
            {
                parent::boot();
            }

            public function map()
            {
                $this->mapApiRoutes();

                $this->mapWebRoutes();
            }

            protected function mapWebRoutes()
            {
                Route::group([
                    'middleware' => 'web',
                    'namespace' => $this->namespace,
                ], function ($router) {
                    require base_path('Modules\模块名称/Routers/web.php');
                });
            }

            protected function mapApiRoutes()
            {
                Route::group([
                    'middleware' => 'api',
                    'namespace' => $this->namespace,
                    'prefix' => 'api',
                ], function ($router) {
                    require base_path('Modules\模块名称/Routers/api.php');
                });
            }
        }
      ```
      
      * 编辑之前创建的ModuleServiceProvider,在register方法中,调用路由注入类的register方法,将路由配置,关联到模块注入配置

      ```php
        <?php 
        ...
        public function register()
        {
            $this->app->register(ModulesRouterServiceProvider::class);
            $this->binds();
        }
      ```

    * 拆分视图文件到模块

      * 在模块目录下新建Views目录,用于取代resource/views目录,放置页面模板和blade模板引擎文件
      * 编辑之前创建的ModuleServiceProvider,boot方法中,将新增的视图目录,注入到框架的viewslocation中

      ```php
        <?php
        public function boot()
        {
            $view_location = sprintf('Modules/%s/Views', $this->module_name);
            View::addLocation(base_path($view_location));
        }
      ```

    * 拆分常量配置到模块

      * 在模块目录下新建Configs目录,用于存放模块独立的常量配置,建议每个模块只使用一个单独的常量配置
      * 由于框架的加载机制,所有常量配置必须放置在configs目录下,才能正常被读取和使用,因此,拆分到模块Configs目录下的配置,在实际运行时还是要放置到configs目录下才能正常使用,为避免不必要的麻烦和自动化构建,框架本身提供了从指定的目录发布配置文件到configs目录的功能,在之前创建的ModuleServiceProvider中,在register方法中添加相关代码

      ```php
        <?php
        public function register()
        {
            $config_path = __DIR__ . '/../Configs';
            if (function_exists('config_path')) {
                $publish_path = config_path('');
            } else {
                $publish_path = base_path('config/');
            }
            $this->publishes([$config_path => $publish_path], 'config');
        }
      ```
    * 拆分模块服务到模块

      * 在模块目录下新建Services目录,根据业务需要,在其中开发具体的业务服务的接口和实现类
      * 在模块的ModuleServiceProvider中的bind方法实现绑定实现类到接口实现DI和服务定位

    * 拆分其他组件到模块

      * 启发组件部分包括自定义异常,请求中间件,表单校验器等等,可以俺需要在模块目录下新建对应的Exceptions,Middlewares和Validators等目录

4. 模块化拆分后的基本目录结构

    * 进行完上述的拆分过程后,完全模块化;模块代码独立目录管理的结构就完成了
    * 最终呈现的目录结构大致如下

    ```shell
      .
      ├── Modules
      │    ├── 模块名称
      │    │    ├── Configs
      │    │    │    └── xxx.php
      │    │    ├── Console
      │    │    ├── Exceptions
      │    │    │    └── xxxException.php
      │    │    ├── Http
      │    │    │    ├── Controllers
      │    │    │    └── Middlewares
      │    │    ├── Models
      │    │    │    └── xxxModel.php
      │    │    ├── Providers
      │    │    │    ├── xxxRouteServiceProvider.php
      │    │    │    └── xxxServiceProvider.php
      │    │    ├── Routers
      │    │    │    ├── api.php
      │    │    │    └── web.php
      │    │    ├── Services
      │    │    │    ├── Ifaces
      │    │    │    └── Impls
      │    │    ├── Validations
      │    │    │    └── xxxValidation.php
      │    │    └── Views
    ```

5. 模块化开发后部署上线的一些注意点

    * 由于拆分了配置文件到模块目录,因此项目上线时,必须先发布配置文件到config目录下

    ```shell
      # php artisan vender:publish --provider='Modules\模块名称\Providers\ModuleServiceProvider' --force
    ```
    * 这部分流程的执行是幂等的,可以加入到正常构建流程中,每次构建都执行,并不会有额外的影响
    * 在每次执行vender:publish后,由于框架本身存在的配置缓存问题,会出现publish了新的配置,但是并没有生效,此时就需要手动去清除默认缓存起来的配置值

    ```shell
      #php artisan compile-clear
      #php artisan config:clear 
    ```

