---
title: Меню панели платформы
description: Важный элемент графического интерфейса пользователя, потому, что спомощью него осуществляется основаня навигация по проекту.
extends: _layouts.documentation.ru
section: main
---

Меню панели платформы - это важный элемент графического интерфейса пользователя, потому, что с помощью его осуществляется основная навигация по проекту.


Для того, что бы добавить новый элемент в меню, нужно сообщить об этом нашему приложению `Dashboard`.
Для этого нужно вызвать метод в свойствах меню и передать аргументы: 

* Название меню, к которому необходимо прикрепить элемент
* Объект меню, содержащий название, ссылки и т.п.

## Пример добавления

Регистрация меню по умолчанию происходит в директории `app/Orchid/Composers`, но можно указывать прямо в `ServiceProvider`.
Например, изменим `app/Providers/AppServiceProvider.php` в такой вид:
	
```php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\View;
use Orchid\Platform\Dashboard;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @param Dashboard $dashboard
     *
     * @return void
     */
    public function boot(Dashboard $dashboard)
    {
        $dashboard->menu->add(Menu::MAIN,
            ItemMenu::label('Idea')
                ->icon('icon-bubbles')
                ->url('https://orchid.software')
        );
    }
}
```

## Месторасположение

Разберём пример, изначально мы создаём объект `ItemMenu`, устанавливая различные параметры, после чего добавляем элемент к конкретному месту `Menu::MAIN`. Таких мест несколько:

- **Menu::MAIN** - Меню отображаемое на каждой странице с левой стороны.
- **Menu::SYSTEMS** - Меню формирующую навигацию по системной странице.
- **Menu::PROFILE** - Меню отображаемое при нажатии на профиль.
- **Название собственных меню** - Делает подменю и прикрепляет элементы.

## Параметры


Базовое использование:

```php
use Orchid\Platform\ItemMenu;

ItemMenu::label('Example');
```


> **Примечание.** Для каждого элемента при создании генерируется уникальный ключ, который не может повторяться, но его можно изменить вручную с помощью метода `slug`.

### Настройка ссылок

Указание ссылки:

 ```php
ItemMenu::label('Example')->url('https://orchid.software/');
```
 
Указание ссылки через маршрут:
 ```php
ItemMenu::label('Example')->route('route.idea');
```

Для определения активности ссылки используется пакет [dwightwatson/active](https://github.com/dwightwatson/active).
Активность ссылок, при использовании `route` и `url`, устанавливается автоматически,
но допустимо изменение с помощью явного указания:

```php
ItemMenu::label('Example')
    ->route('route.idea')
    ->active('route.idea*');
    
ItemMenu::label('Example')
    ->route('route.idea')
    ->active([
        'route.idea',
        'route.other'
    ]);
    
ItemMenu::label('Example')
    ->url('/pages/contact')
    ->active('not:pages/contact');
```

### Права доступа

Вполне ожидаема ситуация, когда некоторые ссылки должны отсутствовать
в зависимости от наличия прав или других обстоятельств, для этого:

 ```php
ItemMenu::label('Example')->permission('platform.idea');
```

или любая другая проверка возвращающая булево значение:

 ```php
ItemMenu::label('Example')->canSee(true);
```

#### Внешний вид


Для элемента меню можно указать графическую иконку с помощью:

```php
ItemMenu::label('Example')->icon('icon-heart');
```

Так же, возможно объединение в визуальную группу с помощью установки заголовка для первого элемента:

```php
ItemMenu::label('Example')->title('Analytics');
```

### Порядок отображения

Сортировка устанавливается через задание порядкового номера:
 ```php
ItemMenu::label('Second')->sort(5);
ItemMenu::label('First')->sort(4);
```

### Уведомления

Пункты меню имеют возможность уведомлять пользователя о каких либо событиях в виде числового значения, для этого:

```php
ItemMenu::label('Comments')
    ->icon('icon-bubbles')
    ->route('platform.systems.comments')
    ->badge(function () {
        return 10;
    });
```

## Вложенное меню

Для возможности создания вложенного меню необходимо добавить основной пункт и указать его уникальное имя с помощью метода `slug`, после этого можно добавлять остальные элементы к новому пункту.


```php
$dashboard->menu
    ->add(Menu::MAIN,
        ItemMenu::label('My menu')
            ->slug('Idea')
	    ->childs()
    )
    ->add('Idea',
        ItemMenu::label('Sub element')
    );
```

Когда дочерние элементы имеют разные правила отображения, что бы не перечислять их все и в родительском, можно воспользоваться методом 'hiddenEmpty'. Он спрячет пункт меню, когда подпункты будут недоступны:

```php
$parent = ItemMenu::label('Dropdown menu')
    ->slug('parent-hidden-menu')
    ->childs()
    ->hideEmpty();

$dashboard->menu->add(Menu::MAIN, $parent)
    ->add('parent-hidden-menu', ItemMenu::label('Sub element item 1')->canSee(false))
    ->add('parent-hidden-menu', ItemMenu::label('Sub element item 2')->canSee(false));
```


