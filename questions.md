## NodeJS fundamentals

<details>
  <summary>Что такое NodeJS? На каком движке работает? Длячего используется библиотека libuv?</summary>

  Node.js представляет среду выполнения кода на JavaScript, которая построена на основе движка JavaScript Chrome V8, который позволяет транслировать вызовы на языке JavaScript в машинный код. Node.js прежде всего предназначен для создания серверных приложений на языке JavaScript.

  libuv - библиотека на C, реализующая цикл событий Node.js и все асинхронные операции платформы.

  Библиотека LibUV отвечает за две принципиально важные вещи:

  - кроссплатформенные операции ввода-вывода — работа с файлами, работа с сетью(как работать с Windows, Linux);
  - поддержка основного событийного цикла Node.JS.

  Когда мы запускаем какой то скрипт, то он запускается в режиме цикла. Этот цикл чередует выполнение JavaScript, который обеспечивается виртуальной машиной V8, с ожиданием различных событий ввода-вывода, срабатывания таймеров, за которые так же отвечает библиотека LibUV.
  И этот цикл будет продолжаться до тех пор, пока возможно появление каких то новых событий, ввода-вывода или таймеров которые нужно будет обработать.

  ![](https://i.stack.imgur.com/GwLqV.png)

  Источник: [imnotgenius.com](http://imnotgenius.com/21-sobytijnyj-tsikl-biblioteka-libuv/)

</details>

<details>
  <summary>Что такое fs module? async/sync fc function</summary>
  Модуль fs(File system) предоставляет множество очень полезных функций для доступа к файловой системе и взаимодействия с ней.
  Одна особенность модуля fs заключается в том, что все методы по умолчанию асинхронны, но они также могут работать синхронно, добавив Sync.

```javascript
fs.rename("before.json", "after.json", (err) => {
  if (err) {
    return console.error(err);
  }

  //done
});
// <--------------------------
const fs = require("fs");

try {
  fs.renameSync("before.json", "after.json");
  //done
} catch (err) {
  console.error(err);
}
```

Ключевое отличие здесь в том, что выполнение вашего скрипта будет заблокировано во втором примере до тех пор, пока операция с файлом не завершится успешно.

Источник: [nodejs.dev](https://nodejs.dev/learn/the-nodejs-fs-module)

</details>

<details>
  <summary>Что такое Streams? Types of streams. Streams vs fs</summary>
  Потоки - одна из фундаментальных концепций, лежащих в основе приложений Node.js.
  Они позволяют эффективно обрабатывать чтение / запись файлов, сетевое взаимодействие или любой вид сквозного обмена информацией.
  Например, обычный способ, когда вы говорите программе прочитать файл, файл считывается в память от начала до конца, а затем вы его обрабатываете.
  Используя потоки, вы читаете его по частям, обрабатывая его содержимое, не сохраняя его в памяти.
  
  Потоки в основном предоставляют два основных преимущества по сравнению с другими методами обработки данных:

- Эффективность памяти: вам не нужно загружать большие объемы данных в память, прежде чем вы сможете их обработать;
- Экономия времени: для начала обработки данных требуется гораздо меньше времени, поскольку вы можете начать обработку, как только они у вас появятся, а не ждать, пока будет доступна вся полезная нагрузка данных.

```javascript
const http = require("http");
const fs = require("fs");

const server = http.createServer(function (req, res) {
  fs.readFile(__dirname + "/data.txt", (err, data) => {
    res.end(data);
  });
  // vs
  const stream = fs.createReadStream(__dirname + "/data.txt");
  stream.pipe(res);
});
server.listen(3000);
```

Вместо того, чтобы ждать, пока файл будет полностью прочитан, мы начинаем его потоковую передачу HTTP-клиенту, как только у нас есть фрагмент данных, готовый к отправке.
Источник: [nodejs.dev](https://nodejs.dev/learn/nodejs-streams)

</details>

<details>
  <summary>Что такое Event Loop? Phases</summary>
  Код JavaScript для Node.js выполняется в одном потоке. Одновременно происходит только одно событие.

Обзор фаз:

- таймеры: в этой фазе выполняются коллбэки, заданные двумя функцияями  setTimeout() и setInterval();
- Panding callbacks: выполняет колбэки ввода/вывода отложенные до следующей итерации цикла, за исключением событий close, таймеров и setImmediate();
- ожидание, подготовка: используется только внутри node.js, c ними мы ничего делать не можем;
- опрос(poll): получение новых events ввода/вывода и вызываются колбэки ввода/вывода. Node.js может блокироваться на этом этапе;
- проверка(check): коллбэки, вызванные setImmediate(), вызываются на этом этапе;
- коллбэки события close: например, socket.on('close', ...);
  Между каждой итерацией цикла событий Node.js проверяет, ожидается ли завершение каких-либо асинхронных операций ввода/вывода или таймеров, и завершает работу, если их нет.

![](https://habrastorage.org/r/w780/getpro/habr/post_images/0fd/b3b/2e9/0fdb3b2e986359c1aa073bfc34b5e36f.png)

Источник: [medium.com](https://medium.com/devschacht/event-loop-timers-and-nexttick-18579cd122e0)

</details>

<details>
  <summary>EventEmitter и утечки памяти</summary>
  «EventEmitter» представляет собой основной объект реализующий работу с событиями в Node.JS. Большое количество других встроенных объектов, которые генерируют события в Node.JS, ему наследуют.

![image](https://user-images.githubusercontent.com/62482805/138591485-953099a8-e609-41fd-8d15-09f8d4ecb1b3.png)

![image](https://user-images.githubusercontent.com/62482805/138592107-d41b2aeb-714d-4b55-83c4-01c864709b2d.png)

Источник: [medium.com](https://medium.com/devschacht/event-loop-timers-and-nexttick-18579cd122e0)

</details>

## Clean Code

<details>
  <summary>Linters</summary>
</details>

<details>
  <summary>Git Hooks. Husky</summary>
  pre-commit, pre-push
</details>

<details>
  <summary>How to keep clean code? How to enforce clean code on a project?</summary>

1. Придерживаться соглашения нэйминга переменных:

- Для имен классов используйте UpperCamelCase с заглавной первой буквой.
- При именовании функций использовать CamelCase
- Для констант и переменных с более чем одним именем в объявлении использовать подчеркивание для разделения имен.

2. Разделяйте свои функции:

- Простой способ упорядочить ваш код - создать функцию для каждой отдельной задачи. Если функция делает больше, чем следует из ее названия, вам следует подумать о разделении функциональности и создании другой функции.
- Использование меньших функциональных блоков делает ваш код аккуратным. Кроме того, у вас может быть функция maininit (), которая хранит структуру приложения. Это упрощает повторное использование функций без дублирования кода.

3. Правильное комментирование:

- Комментарии проясняют сложные сегменты и объясняют высокоуровневые механизмы в вашем коде. Они описывают функциональность конкретного блока кода, тем самым давая другим программистам возможность лучше понять ваш код.
- Комментируя, воздержитесь от ненужных комментариев, которые повторяют тривиальные вещи.

4. Деструктуризация:

- позволяет разбивать сложные структуры данных, такие как объекты или массивы, на более простые части. Он обеспечивает удобный способ доступа к элементам массива и свойствам объекта.

5. Do Not Repeat Yourself
6. Использовать prettier
7. Используйте осмысленные имена переменных.
8. Порядок импорта модулей

Источник: [blog.logrocket.com](https://blog.logrocket.com/12-tips-for-writing-clean-and-scalable-javascript-3ffe30abfe20/)

</details>

## HTTP

<details>
  <summary>Что такое HTTP? HTTP методы. PUT vs POST</summary>
  HTTP — это просто название самого популярного протокола для общения в сети, и браузеры в основном выбирают HTTP при общении с серверами. HTTP-обмен подразумевает, что клиент (наш браузер) отправляет запрос, а сервер присылает ответ.

Работа браузера в основном состоит из:

- Разрешение DNS
- HTTP-обмен
- Рендеринг
- Сброс и повтор

Методы: GET, POST, PUT, PATCH, DELETE.

  <table>
    <thead>
      <tr>
        <th>
          <strong>PUT</strong>
        </th>
        <th>
          <strong>POST</strong>
        </th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>RFC-2616 четко упоминает, что метод PUT запрашивает прикрепленный объект (в теле запроса), который должен быть сохранен на сервере, на котором размещен предоставленный Request-URI.<br><br>Если Request-URI относится к уже существующему ресурсу - произойдет операция обновления, в противном случае должна произойти операция создания, если Request-URI является допустимым URI ресурса (при условии, что клиенту разрешено определять идентификатор ресурса)..<pre>PUT /questions/{question-id}</pre></td>
        <td>>Метод POST используется для запроса, чтобы исходный сервер принял объект, прикрепленный в запросе, в качестве нового подчиненного ресурса, идентифицированного Request-URI в строке запроса.<br><br>По сути, это означает, что URI запроса POST должен иметь URI коллекции.<pre>POST /questions</pre></td>
      </tr>
      <tr>
        <td>Метод PUT идемпотентен. Поэтому, если мы повторим запрос несколько раз, это должно быть эквивалентно вызову одного запроса.</td>
        <td >POST НЕ идемпотентен. Итак, если мы повторим запрос N раз, у нас будет N ресурсов с N разными URI, созданными на сервере.</td>
      </tr>
      <tr>
        <td>Используйте PUT, когда мы хотим изменить отдельный ресурс, который уже является частью коллекции ресурсов.<br><br>PUT полностью заменяет ресурс. Используйте PATCH, если запрос обновляет часть ресурса.</td>
        <td>Используйте POST, если вы хотите добавить дочерний ресурс в коллекцию ресурсов.</td>
      </tr>
      <tr>
        <td>Хотя PUT идемпотентен, мы не должны кэшировать его ответ.</td>
        <td>Ответы на этот метод не кэшируются, если ответ не включает соответствующие поля заголовка Cache-Control или Expires.<br><br>Однако ответ 303 (см. Прочее) может использоваться для указания пользовательскому агенту получить кэшируемый ресурс.</td>
      </tr>
      <tr>
        <td>Как правило, на практике используйте PUT для операций UPDATE.</td>
        <td>Всегда используйте POST для операций CREATE.</td>
      </tr>
    </tbody>
  </table>
</details>

<details>
  <summary>Что такое WS protocol? Цель использования? Как происходит процесс установки соединения по WS? Альтернативы WS?</summary>
  WebSocket - постоянное соединение между клиентом и сервером, пользуясь которыми клиент и сервер могут отправлять данные друг другу в любое время.
  Один из наиболее часто используемых приёмов для создании иллюзии того, что сервер самостоятельно отправляет данные клиенту, называется «длинный опрос» (long polling). С использованием этой технологии клиент открывает HTTP-соединение с сервером, который держит его открытым до тех пор, пока не будет отправлен ответ. В результате, когда у сервера появляются данные для клиента, он их ему отправляет.
  У таких технологий одна и та же проблема: дополнительная нагрузка на систему, которую создаёт использование HTTP, что делает всё это неподходящим для организации работы приложений, где требуется высокая скорость отклика. 
</details>

## Express

<details>
  <summary>Что такое Express? Для какой цели он используется? В чём преимущество использования Express над нативными модулями nodejs?</summary>

  Express - самый популярный веб-фреймворк для Node.
  Он предоставляет следующие механизмы:
  - Написание обработчиков для запросов с различными HTTP-методами в разных URL-адресах (маршрутах).
  - Интеграцию с механизмами рендеринга «view», для генерации ответов, вставляя данные в шаблоны.
  - Установка общих параметров веб-приложения, такие как порт для подключения, и расположение шаблонов, которые используются для отображения ответа.
  - «middlewares» для дополнительной обработки запроса в любой момент в конвейере обработки запросов.
</details>

<details>
  <summary>Express routing. Как происходит сопоставление path и зарегестрированного в приложении обработчика?</summary>

  Route - это часть кода Express, которая связывает команду HTTP (GET, POST, PUT, DELETE и т. Д.), Путь / шаблон URL-адреса и функцию, которая вызывается для обработки этого шаблона.
  Ниже приводятся примеры путей маршрутов на основе строк.

  Данный путь маршрута сопоставляет запросы с корневым маршрутом, /.
  ```javascript
  app.get('/', function (req, res) {
    res.send('root');
  });
  ```
  Примеры путей маршрутов на основе регулярных выражений.

  Данный путь маршрута сопоставляет любой элемент с “a” в имени маршрута.
  ```javascript
  app.get(/a/, function(req, res) {
    res.send('/a/');
  });
  ```
  Источник: [expressjs.com](https://expressjs.com/ru/guide/routing.html)
</details>

<details>
  <summary>Что такое middleware?</summary>

  Функции промежуточной обработки (middleware) - это функции, имеющие доступ к объекту запроса (req), объекту ответа (res) и к следующей функции промежуточной обработки в цикле “запрос-ответ” приложения. Следующая функция промежуточной обработки, как правило, обозначается переменной next.
  ![image](https://user-images.githubusercontent.com/62482805/138703995-3f6a48b8-fb50-451e-bfc2-39a0ce76ceee.png)

  Источник: [expressjs.com](https://expressjs.com/ru/guide/writing-middleware.html#:~:text=%D0%9E%D0%B1%D0%B7%D0%BE%D1%80,%D0%BA%D0%B0%D0%BA%20%D0%BF%D1%80%D0%B0%D0%B2%D0%B8%D0%BB%D0%BE%2C%20%D0%BE%D0%B1%D0%BE%D0%B7%D0%BD%D0%B0%D1%87%D0%B0%D0%B5%D1%82%D1%81%D1%8F%20%D0%BF%D0%B5%D1%80%D0%B5%D0%BC%D0%B5%D0%BD%D0%BD%D0%BE%D0%B9%20next%20.)
</details>

<details>
  <summary>Как происходит обработка ошибок в express? Как создать кастомный обработчик ошибок?</summary>

  Можно обработывать синхронные и асинхронные ошибки.

  Синхронные:
  ```javascript
  app.post('/testing', (req, res) => {
    throw new Error('Something broke! ')
  })
  ```
  Такие ошибки можно перехватить с помощью обработчика ошибок Express.
  Вот что делает стандартный обработчик ошибок Express:
  - Устанавливает код состояния HTTP-ответа в значение 500.
  - Отправляет сущности, выполнившей запрос, текстовый ответ.
  - Логирует текстовый ответ в консоль.

  Для обработки асинхронных ошибок нужно отправить ошибку обработчику ошибок Express через аргумент next:
  ```javascript
  app.post('/testing', async (req, res, next) => {
    return next(new Error('Something broke again! '))
  })
  // or
  app.get('/', function (req, res, next) {
    fs.readFile('/file-does-not-exist', function (err, data) {
      if (err) {
        next(err) // Pass errors to Express.
      } else {
        res.send(data)
      }
    })
  })
  ```
  Обработчики ошибок Express принимают 4 аргумента:
  - error
  - req
  - res
  - next

  Размещать их нужно после промежуточных обработчиков и маршрутов.
  Если создать собственный обработчик ошибок, то Express прекратит использование стандартного обработчика. Для того чтобы обработать ошибку, нужно сформировать ответ для фронтенд-приложения, которое обратилось к конечной точке, в которой возникла ошибка. Это означает, что нужно выполнить следующие действия:
  - Сформировать и отправить подходящий код состояния ответа.
  - Сформировать и отправить подходящий ответ.

  ```javascript
  app.use((error, req, res, next) => {
    // Ошибка, выдаваемая в ответ на неправильно сформированный запрос
    res.status(400)
    res.json(/* ... */)
  })  
  ```
  Источник: [habr.com](https://habr.com/ru/company/ruvds/blog/476290/)
</details>

<details>
  <summary>Валидация. Для чего она нужна? На каком этапе должна проходить валидация? Как обрабатывать ошибки валидации?</summary>
</details>

<details>
  <summary>Трехуровневая архитектура. Из каких слоёв она состоит? Какие задачи решает каждый слой?</summary>

  Трехуровневая архитектура - это архитектура веб-приложений, которая широко используется во всем мире. В основном он содержит 3 уровня: клиент, сервер и база данных.

  Это архитектура приложения основана на типичной модели MVC. 
  
  Клиентский уровень (View) будет написан на Javascript, HTML и CSS с использованием ReactJS в качестве фреймворка. Этот уровень архитектуры - это то, с чем пользователь будет взаимодействовать, чтобы получить доступ к функциям нашего приложения.

  Уровень бизнес-логики (Controller) например написан с использованием NodeJs и ExpressJS, и этот уровень представляет собой сервер приложений, который будет действовать как мост для связи для уровня клиента и уровня базы данных. Этот уровень будет обслуживать HTML-страницы на устройстве пользователя и принимать HTTP-запросы от пользователя с соответствующим ответом.

  Уровнь базы данных (Model). Здесь мы хранятся все важные данные, необходимые нашему приложению для работы.

  Источник: [ichi.pro](https://ichi.pro/ru/polnoe-rukovodstvo-po-sozdaniu-horoso-strukturirovannoj-trehurovnevoj-arhitektury-stek-mern-es6-222181300500485)

</details>

<details>
  <summary>Что за Dependency injection паттерн? Для чего служит DI патерн и как его использовать в Express?</summary>

  В разработке программного обеспечения, внедрение зависимостей это такая техника, где посредством одного объекта (или статического метода) предоставляются зависимости другого объекта. Зависимость — это объект, который может быть использован (как сервис).

  Существует три основных типа внедрения зависимостей:
  - constructor injection: все зависимости передаются через конструктор класса.
  - setter injection: разработчик добавляет setter-метод, с помощью которого инжектор внедряет зависимость
  - interface injection: зависимость предоставляет инжектору метод, с помощью которого инжектор передаст зависимость. Разработчики должны реализовать интерфейс, предоставляющий setter-метод, который принимает зависимости.

  Инверсия управления — концепция, лежащая в основе внедрения зависимости
  Это означает, что класс не должен конфигурировать свои зависимости статистически, а должен быть сконфигурирован другим классом извне.

  Преимущества использования внедрения зависимостей:
  - Помогает в модульном тестировании
  - Количество шаблонного кода сокращается, поскольку инициализация зависимостей выполняется компонентом инжектора;
  - Расширение приложения становится еще проще;
  - Помогает уменьшить связность кода, что важно при разработке приложений.

  Источник: [medium.com](https://xufocoder.medium.com/a-quick-intro-to-dependency-injection-what-it-is-and-when-to-use-it-de1367295ba8)
</details>

## Relational DataBase (RB)

<details>
  <summary>Для чего используют RD?</summary>

  Реляционная база данных – это набор данных с предопределенными связями между ними. 
  Эти данные организованны в виде набора таблиц, каждая строка содержит запись с уникальным идентификатором, известным как ключ, и каждый столбец содержит атрибуты данных. В таблицах хранится информация об объектах, представленных в базе данных.

  К основным преимуществам реляционных баз данных можно отнести следующее:
  - Категоризация данных. Администраторы баз данных могут легко классифицировать и хранить данные в реляционной базе данных, которые затем можно запрашивать и фильтровать для извлечения информации для отчетов. Реляционные базы данных также легко расширяются и не зависят от физической организации. После создания исходной базы данных можно добавить новую категорию данных без изменения существующих приложений.
  - Точность. Данные хранятся только один раз, что исключает дедупликацию данных в процедурах хранения.
  - Сотрудничество. Несколько пользователей могут получить доступ к одной и той же базе данных.
  - Безопасность. Прямой доступ к данным в таблицах в РСУБД может быть ограничен определенными пользователями.

  Cloud-based relational databases - Amazon Relational Database Service, Google Cloud SQL, IBM DB2 on Cloud, SQL Azure and Oracle Cloud.

  Источник: [techtarget.com](https://searchdatamanagement.techtarget.com/definition/relational-database)
</details>

<details>
  <summary>Что такое RDBMS и какая цель использования?</summary>
  
  RDBMS (Relational Database Management System) – это система для управления базами данных (далее – БД), базирующаяся на реляционной модели.
  
  Таблица 

  В RDBMS данные хранятся в объектах, которые называются таблицами. Таблица – это набор связанных по смыслу данных, состоящий из столбцов и рядов.

  Поле, Запись, Колонка, NULL.

  Источник: [proselyte.net](https://proselyte.net/tutorials/sql/rdbms-basic-concepts/)
</details>

<details>
  <summary>Что такое индекс и для чего он используется? Как задать индекс?</summary>

  Констрейнт INDEX используется для того, чтобы крайне быстро добавлять и получать данные из таблицы. 

  Правильное использование данного элемента способно крайне повысить производительность при работе с большими базами данных (далее – БД).

  Источник: [proselyte.net](https://proselyte.net/tutorials/sql/rdbms-basic-concepts/constraint-index/)
</details>

<details>
  <summary>Отошение в rd. Many-to-many и промежуточная таблица. Для чего необходима</summary>

  Типы отношений:
  - ManyToOne
  - OneToMany
  - ManyToMany
  - OneToOne

  ManyToMany использует промежуточную таблицу и нужен, когда обе стороны отношений могут иметь множество с другой стороны (например, “ученики” и “предметы”: каждый ученик во многих предметах, и каждый класс имеет множество учеников).
  Для реализации связи ManyToMany нам нужен некий посредник между двумя рассматриваемыми таблицами. Он должен хранить два внешних ключа, первый из которых ссылается на первую таблицу, а второй — на вторую.
  Связь OneToOne базируется на том, что столбец может хранить только уникальное значение, накладывается ограничение unique (либо primary key, может принимать null).
  Можно сказать, что отношение один к одному — это разделение одной и той же таблицы на две.

  Источник: [habr.com](https://habr.com/ru/post/488054/)
</details>

<details>
  <summary>Что такое внешний ключ? Как задать внешний ключ?</summary>
  
  Внешние ключи позволяют установить связи между таблицами. Внешний ключ устанавливается для столбцов из зависимой, подчиненной таблицы, и указывает на один из столбцов из главной таблицы. Как правило, внешний ключ указывает на первичный ключ из связанной главной таблицы.
  
  Определим две таблицы и свяжем их посредством внешнего ключа:
  ```javascript
  CREATE TABLE Customers
  (
      Id INT PRIMARY KEY AUTO_INCREMENT,
      Age INT, 
      FirstName VARCHAR(20) NOT NULL,
      LastName VARCHAR(20) NOT NULL,
      Phone VARCHAR(20) NOT NULL UNIQUE
  );
  
  CREATE TABLE Orders
  (
      Id INT PRIMARY KEY AUTO_INCREMENT,
      CustomerId INT,
      CreatedAt Date,
      FOREIGN KEY (CustomerId)  REFERENCES Customers (Id)
  ); 
  ```

  Источник: [metanit.com](https://metanit.com/sql/mysql/2.5.php)
</details>

<details>
  <summary>Что такое Transaction? ACID принцип</summary>

  Transaction — это осуществление одного или нескольких изменений базы данных.

  Транзакция включает в себя два результата: либо Успех, либо Неудача.

  Например, если вы создаете, обновляете или удаляете запись из таблицы, вы выполняете в этой таблице транзакцию. 
  Практически вы собираете множество SQL-запросов в группу, и они будут выполняться вместе как часть транзакции.

  Транзакции имеют следующие четыре стандартных свойства, обычно обозначаемых аббревиатурой ACID.
  - Атомарность(Atomicity) – обеспечивает, чтобы все операции входящие в единицу работы были завершены успешно. В противном случае транзакция прерывается в момент сбоя, и все предыдущие операции возвращаются в прежнее состояние.
  - Согласованность(Consistency) — обеспечивает, чтобы база данных надлежащим образом изменяла состояние при успешной транзакции.
  - Изолированность(Isolation) — позволяет транзакциям работать независимо друг от друга и прозрачно.
  - Долговечность(Durability) — гарантирует, что результат совершенной транзакции сохранится в случае сбоя системы.

  Журнал транзакций SQL - это файл, содержащий журналы, которые были созданы в процессе регистрации транзакций, произошедших в базе данных.

  Источник: [devacademy.ru](https://devacademy.ru/recipe/chto-takoe-sql-tranzakciya)
</details>

<details>
  <summary>Что такое Consistency? Нормализация БД</summary>

  Согласованность(Consistency) — обеспечивает, чтобы база данных надлежащим образом изменяла состояние при успешной транзакции.

  Нормальная форма базы данных – это набор правил и критериев, которым должна отвечать база данных.

  Нормализация данных борется с избыточностью.

  Избыточность данных – это когда одни и те же данные хранятся в базе в нескольких местах, именно это и приводит к аномалиям.

  Дело в том, что избыточность данных создает предпосылки для появления различных аномалий, снижает производительность, и делает управление данными не гибким и не очень удобным. 

  Нормализация нужна для:
  - Устранения аномалий
  - Повышения производительности
  - Повышения удобства управления данными

  Нужно выносить повторяющиеся поля в отдельные таблицы.

  Источник: [info-comp.ru](https://info-comp.ru/database-normalization)
</details>

<details>
  <summary>Что такое Connection pool и для чего он используется?</summary>

  Пул соединений — это метод повышения производительности приложения, когда N соединений с базой данных открываются и управляются в пуле. 
  Когда база данных посылается запрос, из пула выбиратся свобное подключение (или создается новое, если сводобных нет и не превышен лимит). Это позволяет снизить издержки на создание новых подключений.

  Источник: [metanit.com](https://metanit.com/web/nodejs/8.5.php)
</details>

<details>
  <summary>Что такое ORM? Преимущества использования ORM</summary>

  Object-Relational Mapping, объектно-реляционное отображение — прослойка между базой данных и кодом который пишет программист, которая позволяет созданые в программе объекты складывать/получать в/из бд.

  Библиотеки ORM часто содержат гораздо больше важных функций, таких как:
  - построители запросов
  - сценарии миграции
  - инструмент CLI для генерации шаблонного кода
  - функция заполнения для предварительного заполнения таблиц тестовыми данными

  Источник: [metanit.com](https://metanit.com/web/nodejs/8.5.php)
</details>

<details>
  <summary>SQL vs NoSQL</summary>

  SQL и NoSQL, реляционные и нереляционные базы данных. Различия между ними заключаются в том, как они спроектированы, какие типы данных поддерживают, как хранят информацию.

  Плюсы выбора SQL-базы:
  - Необходимость соответствия базы данных требованиям ACID (Atomicity, Consistency, Isolation, Durability — атомарность, непротиворечивость, изолированность, долговечность). Это позволяет уменьшить вероятность неожиданного поведения системы и обеспечить целостность базы данных. Достигается подобное путём жёсткого определения того, как именно транзакции взаимодействуют с базой данных. Это отличается от подхода, используемого в NoSQL-базах, которые ставят во главу угла гибкость и скорость, а не 100% целостность данных.
  - Данные, с которыми вы работаете, структурированы, при этом структура не подвержена частым изменением. Если ваша организация не находится в стадии экспоненциального роста, вероятно, не найдётся убедительных причин использовать БД, которая позволяет достаточно вольно обращаться с типами данных и нацелена на обработку огромных объёмов информации.

  Если есть подозрения, что база данных может стать узким местом некоего проекта, основанного на работе с большими объёмами информации, стоит посмотреть в сторону NoSQL-баз, которые позволяют то, чего не умеют реляционные БД.

  Вот возможности, которые стали причиной популярности таких NoSQL баз данных, как MongoDB, CouchDB, Cassandra, HBase:
  - Хранение больших объёмов неструктурированной информации. База данных NoSQL не накладывает ограничений на типы хранимых данных. Более того, при необходимости в процессе работы можно добавлять новые типы данных.
  - Использование облачных вычислений и хранилищ. Облачные хранилища — отличное решение, но они требуют, чтобы данные можно было легко распределить между несколькими серверами для обеспечения масштабирования. Использование, для тестирования и разработки, локального оборудования, а затем перенос системы в облако, где она и работает — это именно то, для чего созданы NoSQL базы данных.
  - Быстрая разработка. Если вы разрабатываете систему, используя agile-методы, применение реляционной БД способно замедлить работу. NoSQL базы данных не нуждаются в том же объёме подготовительных действий, которые обычно нужны для реляционных баз.

  Источник: [habr.com](https://habr.com/ru/company/ruvds/blog/324936/)
</details>

## Error handling and logging

<details>
  <summary>Error Handling в express</summary>

  Express поставляется с обработчиком ошибок по умолчанию, поэтому вам не нужно писать собственный, чтобы начать работу.

  Для ошибок, возвращаемых асинхронными функциями, вызванными обработчиками маршрутов и промежуточным программным обеспечением, вы должны передать их в функцию next (), где Express их перехватит и обработает. Например:

  ```javascript
  app.get('/', function (req, res, next) {
    fs.readFile('/file-does-not-exist', function (err, data) {
      if (err) {
        next(err) // Pass errors to Express.
      } else {
        res.send(data)
      }
    })
  })
  ```
  Начиная с Express 5, обработчики маршрутов и промежуточное ПО, возвращающие Promise, будут вызывать next (value) автоматически, когда они отклоняют или выдают ошибку. Например:

  ```javascript
  app.get('/user/:id', async function (req, res, next) {
    var user = await getUserById(req.params.id)
    res.send(user)
  })
  ```

  Источник: [expressjs.com](https://expressjs.com/en/guide/error-handling.html)
</details>

<details>
  <summary>Как создать кастомную ошибку?</summary>

  Источник: [scoutapm.com](https://scoutapm.com/blog/express-error-handling)
</details>

<details>
  <summary>В чем разница между Error и Exception?</summary>

  - JavaScript выдает Error
  - Разработчики создают Exception

  При использовании блоков try catch или try catch finally вы будете иметь дело как с JavaScript Exception , так и с Error . С точки зрения кода разница не имеет никакого влияния.

  За кулисами браузеры используют один и тот же window.Error constructor . Exception -это экземпляр Error с name и message , которые содержат "Exception".

  По условию, существует разница между Error и Exception . Error указывает на явное нарушение. A TypeError или ReferenceError означает, что вы не следуете спецификациям языка.

  Exception генерируется, когда вы получаете доступ к ответу XMLHttpRequest до его завершения. Error -это крик "you broke the law", а Exception -это подушка "Almost there!" на плече. Надеюсь, аналогия поможет!

  Источник: [scoutapm.com](https://scoutapm.com/blog/express-error-handling)
</details>

## Authorization 

<details>
  <summary>Authorization vs Authentication</summary>

  Для начала система запрашивает логин, пользователь его указывает, система распознает его как существующий — это <ins>**идентификация**</ins>.

  После этого Google просит ввести пароль, пользователь его вводит, и система соглашается, что пользователь, похоже, действительно настоящий, раз пароль совпал, — это <ins>**аутентификация**</ins>.

  Скорее всего, Google дополнительно спросит еще и одноразовый код из SMS или приложения. Если пользователь и его правильно введет, то система окончательно согласится с тем, что он настоящий владелец аккаунта, — это <ins>**двухфакторная аутентификация**</ins>.

  После этого система предоставит пользователю право читать письма в его почтовом ящике и все в таком духе — <ins>**это авторизация**</ins>.

  Популярные методы аутентификации:
  - Аутентификация на основе пароля - это простой метод аутентификации, для которого требуется пароль для подтверждения личности пользователя.
  - Аутентификация без пароля - это когда пользователь проверяется с помощью OTP или волшебной ссылки, доставленной на зарегистрированный адрес электронной почты или номер телефона.
  - 2FA / MFA требует более одного уровня безопасности, например дополнительного PIN-кода или секретного вопроса, для идентификации пользователя и предоставления доступа к системе.
  - Социальная аутентификация проверяет и аутентифицирует пользователей с существующими учетными данными из социальных сетей.
  
  Популярные методы авторизации:
  - Контроль доступа на основе ролей (RBAC) может быть реализован для управления привилегиями между системой и пользователем.
  - Веб-токен JSON (JWT) - это открытый стандарт для безопасной передачи данных между сторонами, и пользователи авторизуются с помощью пары открытого / закрытого ключей.
  
  Источник: [kaspersky.ru](https://www.kaspersky.ru/blog/identification-authentication-authorization-difference/29123/)
</details>

## Testing

<details>
  <summary>Какие типы тестирования существуют? Какая цель тестирования приложений? Пирамида тестирования</summary>

  Тестирование бывает:
  - Блочное (Unit testing) — тестирование одного модуля в изоляции.
  - Интеграционное (Integration Testing) — тестирование группы взаимодействующих модулей.
  - Системное (System Testing) — тестирование системы в целом.

  Цели:
  - Предоставление информации о качестве ПО конечному заказчику;
  - Повышение качества ПО;
  - Предотвращение появления дефектов.

  «Пирамида тестов» — метафора, которая означает группировку тестов программного обеспечения по разным уровням детализации. 

  Оригинальная пирамида тестов Майка Кона состоит из трёх уровней (снизу вверх):
  - Юнит-тесты.
  - Сервисные тесты.
  - Тесты пользовательского интерфейса
  
  Из этой пирамиды главное запомнить два принципа:
  - Писать тесты разной детализации.
  - Чем выше уровень, тем меньше тестов.

  Источник: [habr.com](https://habr.com/ru/post/358950/)
</details>

<details>
  <summary>patern AAA</summary>

  Шаблон AAA (подготовка-действие-подтверждение) стал почти стандартом в отрасли. В нем предлагается разделить метод тестирования на три части: подготовка, действие и подтверждение. Каждый из них несет ответственность только за ту часть, в честь которой они названы.
  
  Источник: [medium.com](https://medium.com/@pjbgf/title-testing-code-ocd-and-the-aaa-pattern-df453975ab80)
</details>
