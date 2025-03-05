#### Слайд
- **Redis Cluster** и **Redis Sentinel** — два подхода к обеспечению отказоустойчивости и масштабируемости Redis.
- **Redis Cluster**: Автоматическое распределение данных между узлами, поддержка горизонтального масштабирования.
- **Redis Sentinel**: Система мониторинга и автоматического переключения на резервный узел (failover) для обеспечения высокой доступности.

#### Слайд  Redis Cluster: Основные концепции
- **Распределение данных**: Данные автоматически распределяются между узлами кластера с использованием хэш-слотов.
- **Отказоустойчивость**: Кластер может продолжать работу даже при выходе из строя одного или нескольких узлов.
- **Горизонтальное масштабирование**: Возможность добавления новых узлов для увеличения производительности.

#### Слайд  Redis Cluster: Пример кода на PHP
- Используем библиотеку `predis` для работы с Redis Cluster.
```php
require 'vendor/autoload.php';

use Predis\Client;

$nodes = [
    'tcp://127.0.0.1:7000',
    'tcp://127.0.0.1:7001',
    'tcp://127.0.0.1:7002',
];

$options = [
    'cluster' => 'redis',
];

$redis = new Client($nodes, $options);

// Пример работы с кластером
$redis->set('key1', 'value1');
$value = $redis->get('key1');
echo $value; // Выведет: value1
```

#### Слайд  Redis Sentinel: Основные концепции

- **Мониторинг**: Sentinel отслеживает состояние мастер-узла и реплик.
- **Автоматический failover**: Если мастер-узел становится недоступным, Sentinel автоматически переключает на одну из реплик.
- **Конфигурация**: Sentinel требует настройки конфигурационных файлов для мониторинга узлов.

#### Слайд Redis Sentinel: Пример кода на PHP

- Используем библиотеку `predis` для работы с Redis Sentinel.
```php
require 'vendor/autoload.php';

use Predis\Client;

$sentinels = [
    'tcp://127.0.0.1:26379',
    'tcp://127.0.0.1:26380',
    'tcp://127.0.0.1:26381',
];

$options = [
    'replication' => 'sentinel',
    'service' => 'mymaster', // Имя мастера, указанное в конфигурации Sentinel
];

$redis = new Client($sentinels, $options);

// Пример работы с Sentinel
$redis->set('key2', 'value2');
$value = $redis->get('key2');
echo $value; // Выведет: value2
```

#### Слайд  Отличия Redis Cluster и Redis Sentinel


(вставить картинку)

- **Redis Cluster**:
    - Автоматическое распределение данных между узлами.
    - Подходит для горизонтального масштабирования.
    - Требует настройки кластера и управления хэш-слотами.
- **Redis Sentinel**:
    - Обеспечивает высокую доступность через автоматический failover.
    - Подходит для сценариев с одним мастер-узлом и несколькими репликами.
    - Требует настройки Sentinel и конфигурации мониторинга.

#### Слайд Redis Cluster и Sentinel в PHP
- **Redis Cluster**:
    - Используйте библиотеку `predis` или `phpredis` для работы с кластером.
    - Убедитесь, что все узлы кластера доступны и правильно настроены.
    - Используйте хэш-слоты для равномерного распределения данных.
- **Redis Sentinel**:
    - Настройте несколько экземпляров Sentinel для повышения отказоустойчивости.
    - Используйте библиотеку `predis` для автоматического переключения на резервный узел.
    - Регулярно тестируйте failover, чтобы убедиться в корректной работе Sentinel.




#### Слайд Redis и PHP — типовые сценарии использования.

((обобщающая картинка)

---

#### Слайд Кэширование (Cache)
- **Пример кода**:
  ```php
  $redis = new Redis();
  $redis->connect('127.0.0.1', 6379);

  $key = 'user:1:profile';
  if (!$redis->exists($key)) {
      $data = fetchUserProfileFromDatabase(1); // Запрос к базе данных
      $redis->set($key, json_encode($data), 3600); // Кэшируем на 1 час
  }
  $data = json_decode($redis->get($key), true);
  ```
- **Применение**: Ускорение доступа к часто запрашиваемым данным.

---

#### Слайд Хранение сессий (Session Storage)

- **Пример кода**:
  ```php
  ini_set('session.save_handler', 'redis');
  ini_set('session.save_path', 'tcp://127.0.0.1:6379');

  session_start();

  // Пример работы с сессией
  $_SESSION['user_id'] = 1;
  echo $_SESSION['user_id']; // Выведет: 1
  ```
- **Применение**: Управление сессиями пользователей в распределенных приложениях, например, в микросервисной архитектуре.

---

#### Слайд Распределенные блокировки (Distributed Lock)

- **Пример кода**:
  ```php
  $redis = new Redis();
  $redis->connect('127.0.0.1', 6379);

  $lockKey = 'resource:lock';
  $lockValue = uniqid();
  $lockTimeout = 10; // Время жизни блокировки в секундах

  if ($redis->set($lockKey, $lockValue, ['nx', 'ex' => $lockTimeout])) {
      // Блокировка получена
      try {
          // Выполнение критической операции
      } finally {
          // Освобождение блокировки
          if ($redis->get($lockKey) === $lockValue) {
              $redis->del($lockKey);
          }
      }
  } else {
      echo "Не удалось получить блокировку";
  }
  ```
- **Применение**: Обеспечение атомарности операций, например, при обработке платежей.

---

#### Слайд Ограничение запросов (Rate Limiter)

- **Пример кода**:
  ```php
  $redis = new Redis();
  $redis->connect('127.0.0.1', 6379);

  $userId // ID пользователя 
  $key = "rate_limit:$userIp";
  $limit = 100; // Максимальное количество запросов
  $window = 60; // Временное окно в секундах

  $current = $redis->incr($key);
  if ($current === 1) {
      $redis->expire($key, $window);
  }

  if ($current > $limit) {
      http_response_code(429);
      echo "Слишком много запросов";
      exit;
  }
  ```
- **Применение**: Защита API от перегрузки.

---

#### Слайд Лидерборды (Leaderboard)

- **Пример кода**:
  ```php
  $redis = new Redis();
  $redis->connect('127.0.0.1', 6379);

  $leaderboardKey = 'game:leaderboard';
  $redis->zAdd($leaderboardKey, 100, 'player1');
  $redis->zAdd($leaderboardKey, 200, 'player2');

  // Получение топ-3 игроков
  $topPlayers = $redis->zRevRange($leaderboardKey, 0, 2, true);
  print_r($topPlayers);
  ```
- **Применение**: Рейтинги игроков в онлайн-играх или соревнованиях.

---

#### Слайд Очереди сообщений на основе Streams (Message Queue with Streams)

- **Пример кода**:
  ```php
  $redis = new Redis();
  $redis->connect('127.0.0.1', 6379);

  $streamKey = 'task_stream';

  // Добавление задачи в поток
  $taskId = $redis->xAdd($streamKey, '*', [
      'task' => 'send_email',
      'to' => 'user@example.com',
      'subject' => 'Welcome'
  ]);

  // Чтение задач из потока
  $tasks = $redis->xRead(['task_stream' => '0'], 1); // Чтение с начала потока
  foreach ($tasks as $stream => $messages) {
      foreach ($messages as $messageId => $messageData) {
          echo "Обработка задачи: " . $messageData['task'] . "\n";
          // Обработка задачи
          $redis->xAck($streamKey, 'consumer_group', [$messageId]); // Подтверждение обработки
      }
  }
  ```
- **Применение**: Обработка фоновых задач с возможностью масштабирования, например, отправка email, обработка данных или выполнение длительных операций.

---

#### Слайд Группы потребителей (Consumer Groups) в Streams

- **Пример кода**:
  ```php
  $redis = new Redis();
  $redis->connect('127.0.0.1', 6379);

  $streamKey = 'task_stream';
  $groupName = 'email_workers';
  $consumerName = 'worker1';

  // Создание группы потребителей (если она еще не существует)
  try {
      $redis->xGroup('CREATE', $streamKey, $groupName, '0', true);
  } catch (Exception $e) {
      // Группа уже существует
  }

  // Чтение задач из потока в рамках группы потребителей
  $tasks = $redis->xReadGroup($groupName, $consumerName, [$streamKey => '>'], 1);
  foreach ($tasks as $stream => $messages) {
      foreach ($messages as $messageId => $messageData) {
          echo "Обработка задачи: " . $messageData['task'] . "\n";
          // Обработка задачи
          $redis->xAck($streamKey, $groupName, [$messageId]); // Подтверждение обработки
      }
  }
  ```
- **Применение**: Распределенная обработка задач консьюмер группы.

---

#### Слайд Pub/Sub (Издатель/Подписчик)

- **Пример кода**:
  ```php
  $redis = new Redis();
  $redis->connect('127.0.0.1', 6379);

  // Подписчик
  $redis->subscribe(['notifications'], function ($redis, $channel, $message) {
      echo "Получено уведомление: $message\n";
  });

  // Издатель
  $redis->publish('notifications', 'Новое сообщение в чате');
  ```
- **Применение**: Чат-приложения, системы уведомлений в реальном времени, обновления статусов.

---

#### Слайд Сравнение Pub/Sub и Streams
- **Pub/Sub**:
    - Легковесный механизм для широковещательных сообщений.
    - Сообщения не сохраняются, подписчики получают только текущие сообщения.
    - Подходит для сценариев, где не требуется сохранение истории сообщений.
- **Streams**:
    - Более мощный механизм для работы с очередями сообщений.
    - Сообщения сохраняются, поддерживаются группы потребителей и отслеживание прогресса.
    - Подходит для сценариев, где требуется надежная обработка задач и масштабируемость.

---


#### Слайд Транзакции в Redis
- **Описание**: Транзакции в Redis позволяют выполнять несколько команд атомарно, то есть либо все команды выполняются успешно, либо ни одна из них не выполняется.
- **Пример задачи**: Атомарное выполнение нескольких операций, например, перевод средств между счетами.
- **Пример кода**:
  ```php
  $redis = new Redis();
  $redis->connect('127.0.0.1', 6379);

  // Начало транзакции
  $redis->multi();

  // Команды в транзакции
  $redis->decrBy('account:1', 100); // Снимаем 100 с первого счета
  $redis->incrBy('account:2', 100); // Добавляем 100 на второй счет

  // Выполнение транзакции
  $result = $redis->exec();

  if ($result === false) {
      echo "Транзакция не выполнена";
  } else {
      echo "Транзакция выполнена успешно";
  }
  ```
- **Применение**: Финансовые операции, где важно обеспечить атомарность, например, переводы между счетами.

---

#### Слайд Особенности транзакций в Redis
- **Атомарность**: Все команды в транзакции выполняются как единое целое.
- **Оптимистическая блокировка**: Redis использует механизм WATCH для отслеживания изменений ключей. Если ключи изменены другой командой, транзакция не выполняется.
- **Пример с WATCH**:
  ```php
  $redis = new Redis();
  $redis->connect('127.0.0.1', 6379);

  $key = 'account:1';
  $redis->watch($key);

  $balance = $redis->get($key);
  if ($balance >= 100) {
      $redis->multi();
      $redis->decrBy($key, 100);
      $redis->incrBy('account:2', 100);
      $result = $redis->exec();
      if ($result === false) {
          echo "Транзакция не выполнена из-за изменения ключа";
      }
  } else {
      $redis->unwatch();
      echo "Недостаточно средств";
  }
  ```
- **Применение**: Обеспечение согласованности данных при параллельном доступе.

---

#### Слайд Пайплайны в Redis
- **Описание**: Пайплайны позволяют отправлять несколько команд на сервер Redis без ожидания ответа на каждую команду, что значительно увеличивает производительность.
- **Пример задачи**: Массовая вставка данных или выполнение множества операций.
- **Пример кода**:
  ```php
  $redis = new Redis();
  $redis->connect('127.0.0.1', 6379);

  // Начало пайплайна
  $redis->pipeline();

  // Добавление команд в пайплайн
  for ($i = 1; $i <= 1000; $i++) {
      $redis->set("key:$i", "value:$i");
  }

  // Выполнение пайплайна
  $results = $redis->exec();

  echo "Добавлено 1000 ключей";
  ```
- **Применение**: Массовая вставка данных, выполнение множества операций за один запрос.

---

#### Слайд 20: Сравнение транзакций и пайплайнов
- **Транзакции**:
    - Атомарное выполнение команд.
    - Подходит для операций, где важна согласованность данных.
    - Медленнее из-за необходимости отслеживания изменений.
- **Пайплайны**:
    - Высокая производительность за счет отправки множества команд за один запрос.
    - Не обеспечивает атомарность.
    - Подходит для операций, где важна скорость, а не атомарность.

---

#### Слайд Работа с Redis JSON в PHP

- Установка PHP-клиента для Redis:
  ```bash
  composer require predis/predis
  ```
- Пример подключения к Redis в PHP:
  ```php
  $client = new Predis\Client();
  ```
- **Redis JSON** позволяет хранить и манипулировать JSON-документами.
- Пример сохранения JSON-документа:
  ```php
  $client->jsonSet('user:1', '$', json_encode(['name' => 'John', 'age' => 30]));
  ```
- Пример получения данных:
  ```php
  $user = $client->jsonGet('user:1');
  print_r(json_decode($user, true));
  ```
- Пример обновления данных:
  ```php
  $client->jsonSet('user:1', '$.age', 31);
  ```

---

#### Слайд Практическое применение Redis JSON
- **Кейс 1: Хранение профилей пользователей**
    - Хранение сложных структур данных (например, профилей пользователей) в виде JSON.
  ```php
  $client->jsonSet('user:2', '$', json_encode([
      'name' => 'Alice',
      'email' => 'alice@example.com',
      'preferences' => ['theme' => 'dark', 'notifications' => true]
  ]));
  ```

- **Кейс 2: Обновление отдельных полей**
    - Обновление только определенных полей без необходимости перезаписи всего документа.
  ```php
  $client->jsonSet('user:2', '$.preferences.theme', 'light');
  ```

---

#### Слайд Введение в Redis Search
- **Redis Search** — мощный инструмент для полнотекстового поиска и индексации.
- Создание индекса:
  ```php
  $client->rawCommand('FT.CREATE', 'userIndex', 'ON', 'JSON', 'PREFIX', '1', 'user:', 'SCHEMA', '$.name', 'AS', 'name', 'TEXT');
  ```
- Пример поиска по индексу:
  ```php
  $result = $client->rawCommand('FT.SEARCH', 'userIndex', 'John');
  print_r($result);
  ```

---

#### Слайд Практическое применение Redis Search
- **Кейс 1: Поиск пользователей по имени**
    - Индексация и поиск по полю `name`:
  ```php
  $client->rawCommand('FT.CREATE', 'userIndex', 'ON', 'JSON', 'PREFIX', '1', 'user:', 'SCHEMA', '$.name', 'AS', 'name', 'TEXT');
  $result = $client->rawCommand('FT.SEARCH', 'userIndex', 'John');
  ```

- **Кейс Поиск с фильтрацией по возрасту**
    - Индексация и поиск с использованием числовых фильтров:
  ```php
  $client->rawCommand('FT.CREATE', 'userIndex', 'ON', 'JSON', 'PREFIX', '1', 'user:', 'SCHEMA', '$.name', 'AS', 'name', 'TEXT', '$.age', 'AS', 'age', 'NUMERIC');
  $result = $client->rawCommand('FT.SEARCH', 'userIndex', '@age:[30 40]');
  ```

---

#### Слайд Комбинирование Redis JSON и Redis Search
- **Кейс: Поиск и обновление данных**
    - Поиск пользователей по имени и обновление их данных:
  ```php
  $result = $client->rawCommand('FT.SEARCH', 'userIndex', 'John');
  $userId = $result[1]; // Получаем ID пользователя
  $client->jsonSet($userId, '$.age', 32);
  ```

---

