package com.skobelev;

import org.telegram.telegrambots.bots.TelegramLongPollingBot;
import org.telegram.telegrambots.meta.TelegramBotsApi;
import org.telegram.telegrambots.meta.api.methods.send.SendMessage;
import org.telegram.telegrambots.meta.api.objects.Update;
import org.telegram.telegrambots.meta.exceptions.TelegramApiException;
import org.telegram.telegrambots.updatesreceivers.DefaultBotSession;

import java.time.Instant;
import java.time.temporal.ChronoUnit;
import java.util.Locale;
import java.util.concurrent.ConcurrentHashMap;

public class Main {
    private static final String TOKEN = "";
    // Map
    // key      value
    // hello -> привет
    // задача -> issue
    // username -> timer
    // userId -> timer
    // timer -> userId
    //
    // userId -> timer
    private static final ConcurrentHashMap<PomodoroBot.Timer, Long> userTimers = new ConcurrentHashMap();

    public static void main(String[] args) throws TelegramApiException {
        TelegramBotsApi telegramBotsApi = new TelegramBotsApi(DefaultBotSession.class);
        PomodoroBot bot = new PomodoroBot();
        telegramBotsApi.registerBot(bot);
        new Thread(() -> {
            try {
                bot.checkTimer();
            } catch (InterruptedException e) {
                System.out.println("Уппс");
            }
        }).run();
    }

    static class PomodoroBot extends TelegramLongPollingBot {

        enum TimerType {
            WORK,
            BREAK
        }

        static record Timer(Instant time, TimerType timerType){};

        @Override
        public String getBotUsername() {
            return "Pomodoro bot";
        }

        @Override
        public String getBotToken() {
            return TOKEN;
        }

        @Override
        public void onUpdateReceived(Update update) {
            if (update.hasMessage() && update.getMessage().hasText()) {
                Long chatId = update.getMessage().getChatId();
                if (update.getMessage().getText().equals("/start")) {
                    sendMsg("""
                            Pomodoro - сделай свое время более эффективным.
                            Задай мне время работы и отдыха через пробел. Например, '1 1'.
                            PS Я работаю пока в минутах
                            """, chatId.toString());
                } else {
                    var args = update.getMessage().getText().split(" ");
                    if (args.length >= 1) {
                        var workTime = Instant.now().plus(Long.parseLong(args[0]), ChronoUnit.MINUTES);
                        userTimers.put(new Timer(workTime, TimerType.WORK), chatId);
                        sendMsg("Давай работай!", chatId.toString());
                        if (args.length >= 2) {
                            var breakTime = workTime.plus(Long.parseLong(args[1]), ChronoUnit.MINUTES);
                            userTimers.put(new Timer(breakTime, TimerType.BREAK), chatId);
                        }
                    }
                }
            }
        }

        public void checkTimer() throws InterruptedException {
            while(true) {
                System.out.println("Количество таймеров пользователей " + userTimers.size());
                userTimers.forEach((timer, userId) -> {
                    System.out.printf("Проверка userId = %d, server_time = %s, user_timer = %s\n",
                            userId, Instant.now().toString(), timer.time.toString());
                    if (Instant.now().isAfter(timer.time)) {
                        userTimers.remove(timer);
                        switch (timer.timerType) {
                            case WORK -> sendMsg("Пора отдыхать", userId.toString());
                            case BREAK -> sendMsg("Таймер завершил свою работу", userId.toString());
                        }
                    }
                });
                Thread.sleep(1000);
            }
        }

        private void sendMsg(String text, String chatId) {
            SendMessage msg = new SendMessage();
            // пользователь чата
            msg.setChatId(chatId);
            msg.setText(text);

            try {
                execute(msg);
            } catch (TelegramApiException e) {
                System.out.println("Уппс");
            }
        }
    }

    static class EchoBot extends TelegramLongPollingBot {

        @Override
        public String getBotUsername() {
            return "Попугай bot";
        }

        @Override
        public String getBotToken() {
            return TOKEN;
        }

        /**
         * Обработка входящих сообщений
         */
        @Override
        public void onUpdateReceived(Update update) {
            int userCount = 0;
            if (update.hasMessage() && update.getMessage().hasText()) {
                if (update.getMessage().getText().equals("/start")) {
                    userCount+=1;
                    System.out.println("Новый пользователь " + userCount);
                    // приветсвие
                    sendMsg("Я попугай бот",
                            update.getMessage().getChatId().toString());
                } else {
                    System.out.println("Обработка сообщений");
                    sendMsg(update.getMessage().getText().toUpperCase(Locale.ROOT),
                            update.getMessage().getChatId().toString());
                }
            }
        }

        private void sendMsg(String text, String chatId) {
            SendMessage msg = new SendMessage();
            // пользователь чата
            msg.setChatId(chatId);
            msg.setText(text);

            try {
                execute(msg);
            } catch (TelegramApiException e) {
                System.out.println("Уппс");
            }
        }
    }
}
