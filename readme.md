#9Новинок PHP 7.1

Наконец, в свет вышла PHP 7.1, новая минорная версия PHP, которая включает в себя ряд новых функций, изменений и исправления ошибок. В настоящей статье мы рассмотрим некоторые новые замечательные функции PHP 7.1, а именно:

1. Итерируемый псевдотип
2. Замыкания через функции обратного вызова
3. Синтаксис с квадратными скобками для списка ()
4. Поддержка ключей в списке
5. Видимость констант класса
6. Типы, допускающие значение null
7. Функции типа void
8. Захват исключений разных типов
9. Исключение «Слишком мало аргументов»

Для вашего удобства по ходу статьи я также буду ссылаться на соответствующие RFC.

Перед тем как мы начнем:
А вы хотели бы попробовать PHP 7.1,  не устраивая бардак с вашей текущей средой программирования? Тогда Docker – это ваш лучший друг! PHP 7.1 можно запросто настроить при помощи Docker, используя одно из этих образов.

Чтобы запустить контейнер 7.1:

docker run -v /Users/apple/Workplace/php/:/php -it php:7.1-rc bash

Это позволит загрузить (если на вашем локальном устройстве уже не сохранены подобные файлы) образы Docker для php:7.1-rc, после чего запустите контейнер и зайдите в него, используя всплывающие подсказки. Перенесите карты опций -v /Users/apple/Workplace/php/:/php из директории /Users/apple/Workplace/php центрального компьютера в директорию /php внутри контейнера Docker. Таким образом, все файлы внутри /Users/apple/Workplace/php будут доступны в директории контейнера /php.
Теперь заходим в директорию /php.

cd /php

Супер! Теперь мы готовы опробовать новые функции PHP 7.1.

##1. Итерируемый псевдотип

Предположим, у нас есть функция getUserNames, которая принимает перечень пользователей и возвращает их имена в виде списка.

```php
function getUserNames($users)
{
    return nikic\map(function($user){
        return $user->name;
    }, $users); 
}
```

Примечание: Здесь мы используем библиотеку nikic/iter.

Параметр $users может быть представлен массивом или объектом с реализацией  функции Traversable. До выхода PHP 7 было невозможно задать для данной функции какие-либо подсказки относительно типа, поскольку не существовало общего типа, который одновременно соответствовал бы и массиву и объекту Traversable.

Для решения данной проблемы в PHP 7.1 был реализован новый псевдотип iterable, который мы можем использовать для того, что задать значения типа для всего, что может быть итерировано с использованием foreach.

function getUserNames(iterable $users): iterable
{
    return nikic\map(function($user){
        return $user->name;
    }, $users); 
}

Теперь данная функция может принимать массивы, итерируемые объекты и генераторы. 

См. RFC

##2. Замыкания через функции обратного вызова

Данный функционал позволяет нам создавать механизмы замыкания через любую вызываемую функцию. Давайте посмотрим один пример, чтобы лучше с этим разобраться.

class MyClass
{
  public function getCallback() {
    return Closure::fromCallable([$this, 'callback']);
  }
  public function callback($value) {
    echo $value . PHP_EOL;
  }
}
$callback = (new MyClass)->getCallback();
$callback('Hello World');

Наверняка, это не самый лучший пример, чтобы объяснить, зачем нам нужна эта функция, но при помощи него можно проиллюстрировать, как именно она работает. Применение функции Closure::fromCallable имеет ряд преимуществ по сравнению с более традиционными методами.

Улучшенный механизм исправления ошибок – При работе с Closure::fromCallable ошибки отображаются в нужном месте, а не там, где мы используем вызываемую функцию. Это значительно упрощает процесс исправления багов.

Перенос объемов – Пример выше будет нормально работать, даже если callback – это частный/защищенный (private/protected) метод MyClass.

Производительность – Подобная функция также повышает производительность, позволяя избежать избыточных проверок того, что данная вызываемая функция действительно является вызываемой. 

См. RFC

##3. Синтаксис с квадратными скобками для списка ()

Синтаксис коротких массивов был одной из потрясающих функций, добавленных в PHP 5.4, что сделало работу с массивами намного проще и удобнее. Теперь синтаксис краткого перечня позволяет нам использовать квадратные скобки вместо list() для присвоения массиву переменных. 


//prior to php 7.1
list($firstName, $lastName) = ["John", "Doe"];
// php 7.1
[$firstName, $lastName] = ["John", "Doe"];

См. RFC

##4. Поддержка ключей в списке

До выхода PHP7 функция list() поддерживала только проиндексированные числами массивы, индекс которых начинается с нуля. Мы не могли использовать данную функцию с ассоциативными массивами.

Теперь новый функционал позволяет нам указывать ключи при деструктуризации массивов.  

$user = [ "first_name" => "John", "last_name" => "Doe"];
["first_name" => $firstName, "last_name" => $lastName]= $user;

В качестве интересного побочного эффекта теперь мы можем присваивать только необходимые ключи из массива.  

["last_name" => $lastName] = $user;

Также можно деструктурировать многомерные массивы, используя списки внутри списка.

$users = [
    ["name" => "John Doe", "age" => 29],
    ["name" => "Sam", "age" => 36]
];
[["name" => $name1, "age" => $age1],["name" => $name2, "age" => $age2]] = $users;

См. RFC

##5. Видимость констант класса

Константы класса помогают нам задать значимое имя для значения, которое используется в данном классе. Эти константы всегда были публичными, и доступ к ним можно было получить из внешнего класса. Но ведь мы далеко не всегда хотим выставлять их на обозрение внешнему миру, не так ли?  
В версии PHP7.1 мы можем приписать константам класса модификаторы видимости точно так же, как мы делаем с методами и свойствами.

class PostValidator
{
    protected const MAX_LENGTH = 100;
    public function validateTitle($title)
    {
        if(strlen($title) > self::MAX_LENGTH) {
            throw new \Exception("Title is too long");
        }
        return true;
    }  
}

Здесь константа MAX_LENGHT является внутренней константой  класса PostValidator, и мы не хотим, чтобы это значение было доступно для внешнего мира.

См. RFC

##6. Типы, допускающие значение null

Этот функционал позволяет нам задать подсказки относительно типа функции, позволяя при этом использовать значения null в качестве параметра или возврата. Мы можем задать параметру или результату тип Null, поставив знак вопроса (?) перед подсказкой типа.


function sayHello(?string $name) {
    echo "Hello " . $name . PHP_EOL;
}
sayHello(null); // Hello
sayHello("John"); //Hello John

Точно также это работает и для возврата.

class Cookie
{
    protected $jar;
    public function get(string $key) : ?string
    {
        if(isset($this->jar[$key])) {
            return $this->jar[$key];
        }
        return null;
    }
}

См. RFC

##7. Функции типа void

Начиная с версии PHP 7, мы можем задавать тип значения, которое функция должна возвращать на выходе. Иногда бывают случаи, когда необходимо указать, что функция не должна возвращать никакого значения вообще.  

class Cookie
{
    protected $jar;
    public function set(string $key, $value) : void
    {
        $this->jar[$key] = $value;
    }
}

Когда для возврата задано значение void, функция либо должна упускать оператор возврата вообще, либо может иметь пустой оператор возврата. Функция не должна возвращать никаких значений.

См. RFC

##8. Захват исключений разных типов 

При работе с исключениями иногда нам может потребоваться обрабатывать несколько исключений одним и тем же способом. Обычно для работы с разными типами исключений нам пришлось бы создать несколько отдельных блоков захвата. 

try {
  // somthing
} catch(MissingParameterException $e) {
  throw new \InvalidArgumentException($e->getMessage());
} catch(IllegalOptionException $e) {
  throw new \InvalidArgumentException($e->getMessage());
}

В версии PHP 7.1 мы можем работать с множественными исключениями в рамках одного общего блока захвата.

try {
  // something
} catch (MissingParameterException | IllegalOptionException $e) {
  throw new \InvalidArgumentException($e->getMessage());
}

Множественные исключения разделяются при помощи одного символа конвейеризации. Это позволит избежать дублирования одного и того же кода для работы с различными типами исключений.

Read RFC

##9.  Исключение «Слишком мало аргументов»

Как и в PHP 7, мы можем вызвать функцию с меньшим количеством аргументов, чем фактически требуется. В таком случае среда только выдаст предупреждение об отсутствующих аргументах, но продолжит выполнение функции. Иногда это может привести к странному или неожиданному поведению функции.  

В версии PHP 7.1 вызов функции без необходимых параметров приведет к срабатыванию исключения ошибки ArgumentCountError.

function sayHello($name) {
    echo "Hello " . $name;
}
sayHello();
// Fatal error: Uncaught ArgumentCountError: Too few arguments to function sayHello(), 0 passed in...

Срабатывание исключения поможет нам немедленно решать подобные проблемы, избегая неожиданного поведения функции. Однако, данное изменение является несовместимым с более ранними версиями и может повредить вашим текущим приложениями. Поэтому рекомендуем убедиться, что с вашими приложениями все в порядке, перед тем как переходить на более новую версию.

См. RFC

##Итоги

В рамках данной статьи мы обсудили некоторые важные функции, добавленные в версию PHP 7.1. Для того чтобы узнать больше о новых функциях и изменениях PHP 7.1, рекомендуем ознакомиться с официальным руководством по переходу с более ранних версий. Я пропустил какую-нибудь важную функцию? Напишите в комментариях ниже :)