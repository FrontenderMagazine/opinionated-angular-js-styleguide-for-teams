
# Руководство по стилю кода в AngularJS

> Репозиторий этого руководства по стилю кода на [GitHub][1]. Все будущие 
обновления автора будут публиковаться там на английском языке!

После прочтения [руководства по стилю Angular кода от Google][2], мне 
показалось что оно немного незаконченно и подталкивает к использованию 
[Closure библиотеки][3]. Они также
[заявляют][4] *«Мы не думаем что это руководство подходит для всех проектов 
использующих AngularJS, и мы бы хотели, чтобы наше сообщество разработчиков 
придумало более общий стиль, который подойдет как к большим, так и к маленьким 
проектам»*.

Я написал это руководство по стилю кода, сборке и структурированию Angular 
приложения, основываясь на моем опыте с Angular, [нескольких презентациях][5] 
и работе в командах.

### Определение модуля

После объявления Angular модуля, ссылку на него можно хранить в переменной 
либо использовать геттер синтаксис. 
Используйте геттер синтаксис всегда ([рекомендация Angular][6]).

###### Плохо:

    var app = angular.module('app', []);
    app.controller();
    app.factory();

###### Хорошо:

    angular
      .module('app', [])
      .controller()
      .factory();

### Функции методов модуля

Модули в Angular имеют большое количество методов, таких как `controller`, 
`factory`, `directive`, `service` и другие. Существует множество способов 
передачи зависимостей и форматирования кода. Я рекомендую определять 
именованную функцию и передавать ее в соответствующий метод модуля. Это 
поможет отслеживать ее в стеке, в отличии от анонимной функции (можно 
именовать анонимную функцию, но мой метод намного чище).

###### Плохо:

    var app = angular.module('app', []);
    app.controller('MyCtrl', function () {
    });

###### Хорошо:

    function MainCtrl () {
    
    }
    angular
      .module('app', [])
      .controller('MainCtrl', MainCtrl);

Объявляйте модуль один раз, используя `angular.module('app', [])` сеттер, а 
затем используйте `angular.module('app')` геттер везде (например в других 
файлах).

Чтобы избежать загрязнения глобального пространства имен, оборачивайте все 
ваши функции во время *сборки/объединения* внутри [IIFE][7], что будет 
выглядеть как-то так:

###### Отлично:

    (function () {
      angular.module('app', []);
      
      // MainCtrl.js
      function MainCtrl () {
      
      }
      
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);
    
    // AnotherCtrl.js
    function AnotherCtrl () {
    
    }
  
    angular
      .module('app')
      .controller('AnotherCtrl', AnotherCtrl);
    
    // и так далее…
    
    })();

### Контроллеры

Контроллеры — это классы которые могут использовать `controllerAs` синтаксис 
или общий `controller` синтаксис. Используйте `controllerAs` синтаксис всегда, 
так как это дает нам ссылку на экземпляр контроллера.

##### controllerAs DOM связи

###### Плохо:

    <div ng-controller="MainCtrl">
      {{ someObject }}
    </div>

###### Хорошо:

    <div ng-controller="MainCtrl as main">
      {{ main.someObject }}
    </div>

Aтрибут `ng-controller` связывает вид с конкретным контроллером, что лишает 
нас возможности использовать другой контроллер с тем же видом. Для решения 
этой проблемы используйте роуты, сообщая каждому `route` какой контроллер и 
вид использовать.

###### Отлично:

    <!-- main.html -->
    <div>
      {{ main.someObject }}
    </div>
    <!-- main.html -->
    
    <script>
    // …
    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        controller: 'MainCtrl',
        controllerAs: 'main'
      });
    }
    angular
      .module('app')
      .config(config);
    // …
    </script>

Использование ссылки `main`, позволяет избежать обращения к `$parent` для 
доступа к родительским контроллерам из дочернего контроллера. Это поможет 
избежать таких вещей как вызов `$parent.$parent`.

##### controllerAs и ключевое слово this

`controllerAs` синтаксис использует внутри контроллеров ключевое слово `this`, 
вместо `$scope`. Используя `controllerAs`, контроллер, фактически 
*привязывается* к `$scope`.

###### Плохо:

    function MainCtrl ($scope) {
      $scope.someObject = {};
      $scope.doSomething = function () {
      
      };
    }
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);

Кроме того, для создания класса контроллера можно использовать `prototype` 
объекта, но это достаточно быстро приведет к бардаку, так как каждая 
зависимость должна быть связана с конструктором объекта.

###### Хорошо и плохо:

Хорошо для наследования, плохо (излишне) для общего использования.

    function MainCtrl ($scope) {
      this.someObject = {};
      this._$scope = $scope;
    }
    MainCtrl.prototype.doSomething = function () {
      // использование this._$scope
    };
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);

Если не знаете для чего используете `prototype`, то это плохо. Если вы 
используете `prototype` для наследования из других контроллеров, тогда это 
хорошо. Для общего использования, `prototype` паттерн может быть излишним.

###### Хорошо:

    function MainCtrl () {
      this.someObject = {};
      this.doSomething = function () {
      
      };
    }
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);

Это пример объектов/функций внутри контроллера, однако не стоит писать логику 
в контроллерах…

##### Избегайте логики в контроллерах

Избегайте написания логики в контроллерах, переносите ее в фабрики и сервисы.

###### Плохо:

    function MainCtrl () {
      this.doSomething = function () {
    
      };
    }
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);

###### Хорошо:

    function MainCtrl (SomeService) {
      this.doSomething = SomeService.doSomething;
    }
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);

Это увеличивает возможность повторного использования, инкапсулирует 
функциональность и упрощает тестирование.

### Сервисы

Сервисы это синглтоны. Используйте ключевое слово `this` для публичных методов 
и свойств.

###### Хорошо:

    function SomeService () {
      this.someMethod = function () {
    
      };
    }
    angular
      .module('app')
      .service('SomeService', SomeService);

### Фабрики

Фабрики дают нам синглтон для создания методов сервиса (например, для 
коммуникации с сервером через REST).

Важно: Фабрика по факту являются паттерном и это не должно отражаться в 
названии. Все фабрики и сервисы должны именоваться сервисами.

###### Плохо:

    function AnotherService () {
    
      var someValue = '';
    
      var someMethod = function () {
    
      };
      
      return {
        someValue: someValue,
        someMethod: someMethod
      };
    
    }
    angular
      .module('app')
      .factory('AnotherService', AnotherService);

###### Хорошо:

Мы создаем объект с таким же именем внутри функции. Это упрощает 
документирование.

    function AnotherService () {
    
      var AnotherService = {};
      
      AnotherService.someValue = '';
    
      AnotherService.someMethod = function () {
    
      };
      
      return AnotherService;
    }
    angular
      .module('app')
      .factory('AnotherService', AnotherService);

Все связи с примитивами находятся в актуальном состоянии и мы можем легко 
увидеть все приватные методы и свойства.

### Директивы

Любые DOM манипуляции должны производиться только внутри директив и только там.

##### Манипуляции с DOM

Манипуляции с DOM должны выполняться внутри директивы, в методе `link`.

###### Плохо:

    // не используйте контроллер
    function MainCtrl (SomeService) {
    
      this.makeActive = function (elem) {
        elem.addClass('test');
      };
    
    }
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);

###### Хорошо:

    // используйте директивы
    function SomeDirective (SomeService) {
      return {
        restrict: 'EA',
        template: [
          '<a href="" class="myawesomebutton" ng-transclude>',
            '<i class="icon-ok-sign"></i>',
          '</a>'
        ].join(''),
        link: function ($scope, $element, $attrs) {
          // манипуляции с DOM и обработка событий
          $element.on('click', function () {
            $(this).addClass('test');
          });
        }
      };
    }
angular
  .module('app')
  .directive('SomeDirective', SomeDirective);

##### Соглашение об именовании

Чтобы предотвратить переопределение вашей директивы, Angular директивой, 
следует избегать `ng-*` префикса в имени. Так например существует множество 
пользовательских `ng-focus` директив, с которыми возникнут проблемы, если 
Angular выпустит свою директиву с таким именем. Это так же усложняет отделение 
пользовательских директив, от Angular директив.

###### Плохо:

    function ngFocus (SomeService) {
    
      return {};
    
    }
    angular
      .module('app')
      .directive('ngFocus', ngFocus);

###### Хорошо:

    function focusFire (SomeService) {
    
      return {};
    
    }
    angular
      .module('app')
      .directive('focusFire', focusFire);

Только в названии директив, первая буква должна быть строчной. Это связано со 
строгим соглашением об именах, так как Angular переводит `camelCase` в имя 
через дефис. Таким образом, `focusFire` будет `<input focus-fire>` при 
использовании в элементе.

##### Ограничения по использованию

Если вам нужна поддержка IE8, вам следует избегать использования синтаксиса 
комментариев для директив. В действительности, этот синтаксис следует избегать 
в любом случаи, так как нет никаких реальных преимущества его использования. 
Это только добавляет путаницу в отделении реальных комментариев от директив.

###### Плохо:

Это ужасно запутанно.

    <!-- directive: my-directive -->
    <div class="my-directive"></div>

###### Хорошо:

Тут объявление пользовательских элементов и атрибутов является абсолютно 
понятным.

    <my-directive></my-directive>
    <div my-directive></div>

Можно ограничить использование директивы в свойстве `restrict`. Используйте 
`E` для `элемента`, `A` для `атрибута`, `M` для `комментария` (следует 
избегать) и `C` для `имени класса` (так же следует избегать, так как это еще 
больше запутывает, хотя и работает в IE). Можно использовать несколько 
ограничений, например `restrict: 'EA'`.

### Разрешайте промисов в роуте

После создания сервисов, вероятно вы добавите их в контроллер, вызовите и 
привяжите пришедшие данные. Это делает код более запутанным. 

К счастью, используя `angular-route.js` (или например `ui-router.js`) мы можем 
использовать `resolve` свойство для разрешения промиса вида прежде, чем 
страница будет показана. Это означает, что контроллер инициализируется только 
когда все данные будут доступны.

###### Плохо:

    function MainCtrl (SomeService) {
    
      var self = this;
    
      // неразрешено
      self.something;
    
      // разрешено асинхронно
      SomeService.doSomething().then(function (response) {
        self.something = response;
      });
    
    }
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);

###### Хорошо:

    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        resolve: {
          doSomething: function (SomeService) {
            return SomeService.doSomething();
          }
        }
      });
    }
    angular
      .module('app')
      .config(config);

На этой стадии, наш сервис внутренне свяжет ответ от промиса с другим 
объектом, на который мы можем ссылаться в нашем контроллере:

###### Хорошо:

    function MainCtrl (SomeService) {
      // разрешено!
      this.something = SomeService.something;
    }
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);

Мы можем сделать еще лучше создав `resolve` свойство у нашего собственного 
контроллера и передать его роутеру. Так мы сможем избавиться от логики в 
роутере.

###### Отлично:

    // конфигурируем роутер, указывая на resolve соответствующего контроллера
    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        controller: 'MainCtrl',
        controllerAs: 'main',
        resolve: MainCtrl.resolve
      });
    }
    // контроллер как обычно
    function MainCtrl (SomeService) {
      // разрешено!
      this.something = SomeService.something;
    }
    // создаем разрешенное свойство
    MainCtrl.resolve = {
      doSomething: function (SomeService) {
        return SomeService.doSomething();
      }
    };
    
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl)
      .config(config);

##### Изменение роута и аякс спиннеры

Пока разрешаются промисы для роутов, мы можем показать пользователю какой-то 
прогресс. Angular вызовет событие `$routeChangeStart` когда мы начнем переход 
на новую страницу, и в это время мы можем показать аякс спиннер, который потом 
будет удален по событию `$routeChangeSuccess` ([документация][8]).

### Избегайте $scope.$watch

Не используйте `$scope.$watch` пока не останется другого выбора. Это менее 
производительно, нежели привязывание выражения на события вроде `ng-change`. 
Список всех поддерживаемых событий можно найти в документации.

###### Плохо:

    <input ng-model="myModel">
    <script>
      $scope.$watch('myModel', callback);
    </script>

###### Хорошо:

    <input ng-model="myModel" ng-change="callback">
    <!--
      $scope.callback = function () {
        // вперед
      };
    -->

### Структура проекта

Одна задача — один файл и это правило. Разделяйте все контроллеры, 
сервисы/фабрики, директивы в отдельные файлы. Не добавляйте все контроллеры в 
один файл, иначе вы закончите с огромным файлом в котором будет очень сложно 
ориентироваться.

###### Плохо:

    |-- app.js
    |-- controllers.js
    |-- filters.js
    |-- services.js
    |-- directives.js

Придерживайтесь соглашения об именовании файлов. Не выдумывайте забавные имена 
для файлов, так как вы потом просто забудете их.

###### Хорошо:

    |-- app.js
    |-- controllers/
    |   |-- MainCtrl.js
    |   |-- AnotherCtrl.js
    |-- filters/
    |   |-- MainFilter.js
    |   |-- AnotherFilter.js
    |-- services/
    |   |-- MainService.js
    |   |-- AnotherService.js
    |-- directives/
    |   |-- MainDirective.js
    |   |-- AnotherDirective.js

В зависимости от размера вашего проекта, возможно будет лучше разбить код на 
куски в зависимости от функциональности.

###### Хорошо:

    |-- app.js
    |-- dashboard/
    |   |-- DashboardService.js
    |   |-- DashboardCtrl.js
    |-- login/
    |   |-- LoginService.js
    |   |-- LoginCtrl.js
    |-- inbox/
    |   |-- InboxService.js
    |   |-- InboxCtrl.js

### Соглашение об именовании и конфликты

Angular предоставляет нам множество объектов, таких как `$scope` и 
`$rootScope` с префиксом `$`. Это подсказывает что они публичные и могут быть 
использованы. Нам так же доступны методы вроде `$$listeners`, которые есть у 
объекта, но считаются приватными.

Избегайте использования `$` или `$$` при создании своих 
сервисов/директив/провайдеров или фабрик.

###### Плохо:

Здесь мы создаем `$$SomeService` как определение, не как имя функции.

    function SomeService () {
    
    }
    angular
      .module('app')
      .factory('$$SomeService', SomeService);

###### Good:

Здесь мы создаем `SomeService` как определение, *и* имя функции для 
трассировка стека.

    function SomeService () {
    
    }
    angular
      .module('app')
      .factory('SomeService', SomeService);

### Минификация и аннотации

##### Порядок аннотации

Считается хорошей практикой передавать Angular зависимости в правильном 
порядке. Вначале следует передавать Angular провайдеры, затем пользовательские 
провайдеры.

###### Плохо:

    // случайный порядок зависимостей
    function SomeCtrl (MyService, $scope, AnotherService, $rootScope) {
    
    }

###### Хорошо:

    // упорядоченные Angular -> пользовательские
    function SomeCtrl ($scope, $rootScope, MyService, AnotherService) {
    
    }

##### Автоматизируйте минификацию

Используйте `ng-annotate` для аннотации автоматической вставки зависимостей, 
т.к. `ng-min` [устарел][9]. Вы можете найти `ng-annotate` [здесь][10].

Используя объявление функций вне модуля, следует использовать `@ngInject`, 
чтобы `ng-annotate` мог вставить необходимые зависимости. Этот метод 
использует `$inject`, который быстрее нежели объявление массива зависимостей.

Объявление массивов зависимостей вручную, занимает слишком много времени.

###### Плохо:

    function SomeService ($scope) {
    
    }
    // ручное объявление отнимает много времени
    SomeService.$inject = ['$scope'];
    angular
      .module('app')
      .factory('SomeService', SomeService);

###### Хорошо:

Указываем где нужно вставить зависимости, используя ключевое слово `@ngInject`:

    /**
     * @ngInject
     */
    function SomeService ($scope) {
    
    }
    angular
      .module('app')
      .factory('SomeService', SomeService);

Будет преобразовано в:

    /**
     * @ngInject
     */
    function SomeService ($scope) {
    
    }
    // сгенерировано автоматически
    SomeService.$inject = ['$scope'];
    angular
      .module('app')
      .factory('SomeService', SomeService);

 [1]: http://github.com/toddmotto/angularjs-styleguide
 [2]: http://google-styleguide.googlecode.com/svn/trunk/angularjs-google-style.html
 [3]: https://developers.google.com/closure/library/
 [4]: http://blog.angularjs.org/2014/02/an-angularjs-style-guide-and-best.html
 [5]: http://speakerdeck.com/toddmotto
 [6]: http://docs.angularjs.org/guide/module
 [7]: https://en.wikipedia.org/wiki/Immediately-invoked_function_expression
 [8]: https://docs.angularjs.org/api/ngRoute/service/%24route
 [9]: https://github.com/btford/ngmin
 [10]: https://github.com/olov/ng-annotate