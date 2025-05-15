using Telegram.Bot;
using Telegram.Bot.Types;
using Telegram.Bot.Polling;
using Telegram.Bot.Types.Enums;
using System.Collections.Generic;

class Program
{
    static Dictionary<long, List<string>> diary = new();

    static async Task Main()
    {
        var botClient = new TelegramBotClient("TOKEN");

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

        var chatId = message.Chat.Id;
        var text = messageText.Trim();

        if (!diary.ContainsKey(chatId))
            diary[chatId] = new List<string>();

        if (text.StartsWith("/add "))
        {
            string entry = text.Substring(5);
            diary[chatId].Add(entry);
            await botClient.SendTextMessageAsync(chatId, "Запись добавлена!", cancellationToken: cancellationToken);
        }
        else if (text == "/list")
        {
            if (diary[chatId].Count == 0)
            {
                await botClient.SendTextMessageAsync(chatId, "У вас нет записей.", cancellationToken: cancellationToken);
            }
            else
            {
                string allEntries = string.Join("\n— ", diary[chatId]);
                await botClient.SendTextMessageAsync(chatId, $"Ваши записи:\n— {allEntries}", cancellationToken: cancellationToken);
            }
        }
        else if (text == "/clear")
        {
            diary[chatId].Clear();
            await botClient.SendTextMessageAsync(chatId, "Все записи удалены.", cancellationToken: cancellationToken);
        }
        else
        {
            string response = text switch
            {
                "/start" => "Привет! Я бот-дневник. Используйте команды: /add, /list, /clear",
                "/help" => "Команды:\n/add <запись> — добавить запись\n/list — показать записи\n/clear — очистить записи",
                "/info" => "Этот бот написан на C# и сохраняет ваши личные записи.",
                _ => "Неизвестная команда. Введите /help"
            };

            await botClient.SendTextMessageAsync(chatId, response, cancellationToken: cancellationToken);
        }
    }

    static Task HandleErrorAsync(ITelegramBotClient botClient, Exception exception, CancellationToken cancellationToken)
    {
        Console.WriteLine($"Ошибка: {exception.Message}");
        return Task.CompletedTask;
    }
}
