# OpenAPI SDK для Java

Данный проект представляет собой инструментарий на языке Java для работы с OpenAPI Тинькофф Инвестиции, который можно
использовать для создания торговых роботов.

## Начало работы

Для сборки библиотеки понадобится Gradle версии не ниже 5, а также JDK версии не ниже 11. Затем в терминале перейдите
в директорию проекта и выполните следующую команду
```bash
gradlew build
```
Или с помощью docker
```
docker run --rm -u gradle -v "$PWD":/home/gradle/project -w /home/gradle/project gradle:jdk11 gradle build
```
После успешной сборки в поддиректории `sdk/build/libs` появится jar-файл, который можно подключить к любому другому
Java-проекту (или Java-совместимому, например, на таких языках, как Kotlin и Scala).

### Где взять токен аутентификации?

В разделе инвестиций вашего [личного кабинета tinkoff](https://www.tinkoff.ru/invest/). Далее:


* Перейдите в настройки
* Проверьте, что функция "Подтверждение сделок кодом" отключена
* Выпустите токен для торговли на бирже и режима "песочницы" (sandbox)
* Скопируйте токен и сохраните, токен отображается только один раз, просмотреть его позже не получится, тем не менее вы
  можете выпускать неограниченное количество токенов

## Документация

Для проекта можно сгенерировать javadoc-документацию с помощью команды
```bash
gradlew javadoc
```
Или с помощью docker
```
docker run --rm -u gradle -v "$PWD":/home/gradle/project -w /home/gradle/project gradle:jdk11 gradle javadoc
```
Единственной зависимостью в проекте явлется библиотека Jackson для работы с JSON.

Документацию непосредственно по OpenAPI можно найти по [ссылке](https://api-invest.tinkoff.ru/ru.tinkoff.invest.openapi/docs/).

### А если вкратце?

Для непосредственного взаимодействия с OpenAPI нужно создать подключение.

```java
import ru.tinkoff.invest.ru.tinkoff.invest.openapi.wrapper.impl.ConnectionFactory;

var token = "super_token"; // токен авторизации
var connection = ConnectionFactory.connect(token, logger).join(); // содание подключения происходит асинхронно
// Для работы в "песочнице" используйте connectSandbox
var context = connection.context();

// Вся работа происходит через объект контекста, все запросы асинхронны
var portfolio = context.getPortfolio().join(); // получить текущий портфель
```
Для написания собственной торговой стратегии реализуйте интерфейс `Strategy`. Затем запустите исполнение стратегии через
`StrategyExecutor`.

```java
import ru.tinkoff.invest.ru.tinkoff.invest.openapi.automata.Strategy;
import ru.tinkoff.invest.ru.tinkoff.invest.openapi.automata.StrategyExecutor;

final var myStrategy = new Strategy() { /*...*/ };
final var strategyExecutor = new StrategyExecutor(context, strategy, logger);
strategyExecutor.run();
```

### А пример готового робота есть?

В качестве примера готовой простой стратегии исследуйте устройство класса `SimpleStopLossStrategy`. Его использование
продемонстрировано в подпроекте _example_. После сборки в поддиректории `example/build/libs` появится jar-файл, который
запускает робота. При желании можно запустить робота в Docker-контейнере - есть соответствующий `Dockerfile`. После
сборки проекта постройте docker-образ и запустите его.
```bash
docker build --tag=ru.tinkoff.invest.openapi-example .
docker run -ti --mount source=openapi_volume,target=/app/logs \
    -e "token=<auth_token>" \
    -e "ticker=<ticker>" \
    -e "interval=<candle_interval>" \
    -e "max_volume=<your_money>" \
    -e "use_sandbox=<true_or_false>" \
    ru.tinkoff.invest.openapi-example
```
В подключённой директории `openapi_volume` будет файл с подробным логом работы.

Краткое описание стратегии:
* При старте происходит закупка заданного инструмента на сумму не превыщающую заданный в параметрах потолок. Наличие уже
закупленного актива игнорируется.
* Происходит слежение за ценами посредством изучения информации приходящей в свечах с заданным в параметрах интервалом.
За текущую цену берётся средняя цена свечи.
* В процессе слежения за ценой имитируются механизмы наподобие стоп-лоссов и тейк-профитов.
* Если инструмент не торгуется, то робот бездействует.

**ПРИВЕДЁННЫЙ В КАЧЕСТВЕ ПРИМЕРА РОБОТ ОЧЕНЬ ПРОСТ - НЕ РЕКОМЕНДУЕТСЯ ИСПОЛЬЗОВАТЬ ЕГО В РЕАЛЬНЫХ ТОРГАХ!** В параметрах
запуска можно указать включение режима "песочницы".

## У меня есть вопрос

[Основной репозиторий с документацией](https://github.com/TinkoffCreditSystems/invest-ru.tinkoff.invest.openapi/) - в нем вы можете задать вопрос в Issues и получать информацию о релизах в Releases.

Если возникают вопросы по данному SDK, нашёлся баг или есть предложения по улучшению, то можно  задать его в Issues, либо писать на почту:

* Мельникову Никите (n.v.melnikov@tinkoff.ru)
* Иванову Владимиру (v.ivanov8@tinkoff.ru)
