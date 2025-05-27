**1\. Введение**

Тема проекта: Telegram-бот

Цель проекта - изучить принципы создания Telegram-бота с использованием языка C#, реализовать работающее приложение, взаимодействующее с Telegram Bot API.

Функциональность:

• Обработка команд /start, /help, /info.

• Ответы на сообщения.

• Возможность расширения (например, ведение дневника, напоминания и др.

**2\. Выбор технологий**

| Компонент | Технология | Описание |
| --- | --- | --- |
| Язык программирования | C# | Современный язык с мощной экосистемой |
| Telegram API | Telegram.Bot | Популярная .NET-библиотека для работы с Bot API |
| IDE | Visual Studio / Rider | Для разработки |
| Хранение данных | JSON / SQLite / PostgreSQL | Выбирается в зависимости от проекта |
| Хостинг | Render / Railway / VPS | Развёртывание Telegram-бота |
| Документация | Markdown, Draw.io, PlantUML | Для технического описания и UML |

**3\. Исследование: как работает Telegram-бот на C#**

**3.1 Как работает Telegram Bot API**

Telegram Bot API — это HTTP-интерфейс, через который можно получать и отправлять сообщения, обрабатывать команды и взаимодействовать с пользователями. Есть два основных способа получения обновлений:

• Long Polling — бот запрашивает новые сообщения с интервалом

• Webhook — Telegram сам отправляет обновления на указанный URL

Для простоты мы будем использовать Long Polling, так как он не требует публичного сервера на первом этапе.

**3.2 Основы библиотеки Telegram.Bot для C#**

Telegram.Bot (https://github.com/TelegramBots/Telegram.Bot) — официальная .NET-библиотека для Telegram Bot API. Возможности:

• Отправка и получение сообщений.

• Обработка команд.

• Поддержка клавиатур и inline-кнопок.

• Работа с медиа и API Telegram.

**4\. Пошаговое создание Telegram-бота на C#**

**Шаг 1: Регистрация бота в Telegram**

1\. Откройте Telegram и найдите бота @BotFather (https://t.me/BotFather)

2\. Отправьте команду /newbot

3\. Укажите имя и username (например, MyTestBot)

4\. Скопируйте токен, он выглядит так:

123456789:ABCDefGhIJKLmNoPQRsTUVwxyZ

![1](https://github.com/user-attachments/assets/377fb62f-d7d0-45e8-bfb2-2983e1ac750a)

**Шаг 2: Создание проекта**

1\. Откройте Visual Studio

2\. Создайте консольное приложение:

File → New → Project → Console App (.NET Core или .NET 6/8)

3\. Установите NuGet-пакет:

dotnet add package Telegram.Bot

Или через Visual Studio:

Project → Manage NuGet Packages → Telegram.Bot

![2](https://github.com/user-attachments/assets/8eb95b0e-e4ae-484f-92e0-d3c1839027c9)

![3](https://github.com/user-attachments/assets/1085d37d-a16a-4ae9-b6ec-1e94a33cbc86)

Шаг 3: Базовая реализация бота**

Создайте файл Program.cs и вставьте следующий код:

using Telegram.Bot;

using Telegram.Bot.Types;

using Telegram.Bot.Polling;

using Telegram.Bot.Types.Enums;

class Program

{

static async Task Main()

{

var botClient = new TelegramBotClient("YOUR\_TOKEN\_HERE");

using var cts = new CancellationTokenSource();

var receiverOptions = new ReceiverOptions

{

AllowedUpdates = Array.Empty<UpdateType>() // получаем все типы обновлений

};

botClient.StartReceiving(

HandleUpdateAsync,

HandleErrorAsync,

receiverOptions,

cancellationToken: cts.Token

);

var me = await botClient.GetMeAsync();

Console.WriteLine($"Бот запущен: {me.Username}");

Console.ReadLine();

cts.Cancel();

}

static async Task HandleUpdateAsync(ITelegramBotClient botClient, Update update, CancellationToken cancellationToken)

{

if (update.Message is not { } message || message.Text is not { } messageText)

return;

Console.WriteLine($"Сообщение от {message.Chat.Id}: {messageText}");

string response = messageText switch

{

"/start" => "Привет! Я Telegram-бот на C#.",

"/help" => "Доступные команды: /start, /help, /info",

"/info" => "Этот бот написан на C# с использованием Telegram.Bot.",

\_ => $"Вы написали: {messageText}"

};

await botClient.SendTextMessageAsync(

chatId: message.Chat.Id,

text: response,

cancellationToken: cancellationToken);

}

static Task HandleErrorAsync(ITelegramBotClient botClient, Exception exception, CancellationToken cancellationToken)

{

Console.WriteLine($"Ошибка: {exception.Message}");

return Task.CompletedTask;

}

}

**Шаг 4: Запуск**

1\. Вставьте свой токен в строку:

var botClient = new TelegramBotClient("YOUR\_TOKEN\_HERE");

2\. Запустите проект: Ctrl + F5

3\. Напишите боту в Telegram команду /start

**Итог:**

На этом этапе у нас есть базовый Telegram-бот, который:

• Работает через Long Polling

• Обрабатывает команды /start, /help, /info

• Отвечает на любые текстовые сообщени

**5\. Иллюстрации и диаграммы**

**5.1. UML-диаграмма компонентов**

![4](https://github.com/user-attachments/assets/cffce8ff-e6b0-4718-a569-bf136f707903)


**5.2. Диаграмма последовательности обработки сообщений**

![5](https://github.com/user-attachments/assets/f104f9e0-991e-460e-a280-c8606a6b03a5)


**5.3. Схема логики обработки команд**

\[Старт\]

\[Получено сообщение\]

\[Это команда?\] -- нет --> \[Ответ: "Вы написали..."\]

да

\[Команда /start?\] --> Ответ: "Привет!"

\[Команда /help?\] --> Ответ: "Доступные команды..."

\[Команда /info?\] --> Ответ: "Информация о боте"

**6\. Модификация проекта: Ведение дневника**

**Цель модификации**

Добавить в Telegram-бота возможность сохранять и просматривать дневниковые записи пользователя. Пользователь сможет:

• Добавить запись: /add Сегодня был тяжёлый день

• Посмотреть все записи: /list

• Очистить записи: /clear

**Техническая реализация**

Мы используем:

• Словарь Dictionary<long, List<string>> для хранения записей в памяти

• Команды: /add, /list, /clear

Позже это можно заменить на SQLite или сохранение в файл.

**Пример кода**

Добавим в Program.cs:

static Dictionary<long, List<string>> diary = new();

static async Task HandleUpdateAsync(ITelegramBotClient botClient, Update update, CancellationToken cancellationToken)

{

if (update.Message is not { } message || message.Text is not { } messageText)

return;

var chatId = message.Chat.Id;

var text = messageText.Trim();

if (!diary.ContainsKey(chatId))

diary\[chatId\] = new List<string>();

if (text.StartsWith("/add "))

{

string entry = text.Substring(5);

diary\[chatId\].Add(entry);

await botClient.SendTextMessageAsync(chatId, "Запись добавлена!", cancellationToken: cancellationToken);

}

else if (text == "/list")

{

if (diary\[chatId\].Count == 0)

{

await botClient.SendTextMessageAsync(chatId, "У вас нет записей.", cancellationToken: cancellationToken);

}

else

{

string allEntries = string.Join("\\n— ", diary\[chatId\]);

await botClient.SendTextMessageAsync(chatId, $"Ваши записи:\\n— {allEntries}", cancellationToken: cancellationToken);

}

}

else if (text == "/clear")

{

diary\[chatId\].Clear();

await botClient.SendTextMessageAsync(chatId, "Все записи удалены.", cancellationToken: cancellationToken);

}

else

{

string response = text switch

{

"/start" => "Привет! Я бот-дневник. Используйте /add, /list, /clear",

"/help" => "Команды: /add <запись>, /list, /clear",

"/info" => "Этот бот сохраняет ваши личные записи.",

\_ => "Неизвестная команда. Введите /help"

};

await botClient.SendTextMessageAsync(chatId, response, cancellationToken: cancellationToken);

}

}

**Идеи для будущей доработки**

• Добавить фильтрацию по дате

• Сохранять данные в базу

• Напоминания на основе записей

• Экспорт в PDF или TXT

**7 и 8 пункты по подключению кода к гиту и записыванию видео**

**![](data:image/x-emf;base64,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)**
