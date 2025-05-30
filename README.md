# Настройка обс для откатов как shadowplay с переключением сцен и отдельным профилем для записи/стримов

для реплея:
1. установить python (для obs 31.0.3 ставить python 3.12, 3.13 слишком новый и не работает)

в тулбаре: сервис - скрипты - настройки python - указать путь установки, где лежит python.exe (%LocalAppData%/Programs/Python/Python312)
пока просто закрываем это окно, скрипты закинем позже, т.к. после импорта профилей и сцен придется настраивать заново

2. в папку C:\Program Files\obs-studio\obs-plugins\64bit кидаем obs-nopreventsleep.dll, чтобы реплей не блокировал сон

3. устанавливаем advanced scene switcher в папку с обс [https://obsproject.com/forum/resources/advanced-scene-switcher.395/] [https://github.com/WarmUpTill/SceneSwitcher/releases/]

4. Импортируем профили, сцены и макросы:
   1) профиль - импорт - выбираем папку Record, затем Replay
   2) коллекция сцен - импорт - выбираем "коллекция сцен.json"
   3) сервия - advanced scene switcher - общие - импорт - выбираем "adv-ss.txt"

5. меняем настройки на свой вкус, не забываем про бинды и путь сохранения видео ДЛЯ ВСЕХ ПРОФИЛЕЙ (Replay и Record). Не забываем выбрать свой монитор в источнике сцены "Screen"

6. возвращаемся в сервис - скрипты, во вкладке скрипты добавляем smart_replays.py (удалить, если он уже есть), настроить, проверить, что путь к клипам совпадает с тем, что указан в настройках обс

7. для автозапуска от админа [https://obsproject.com/forum/threads/start-obs-as-administrator-on-startup-in-windows-10-with-startreplaybuffer.116313/]:
   1) пишем в поиск в винде "Планировщик задач", открываем
   2) библиотека планировщика задач - Создать задачу (или создать НОВУЮ задачу, НЕ ПРОСТУЮ)
   3) Общие: называем OBS Studio autostart, ставим галку "Выполнить с наивысшими правами"
      Триггеры: создать, Начать задачу: При входе в систему, выбираем своего пользователя, ставим внизу галку "Включено" (вроде должна изначально стоять)
      Действия: создать, Действие Запуск программы, Программа или сценарий: указываем путь к obs64.exe ("C:\Program Files\obs-studio\bin\64bit\obs64.exe"), Добавить аргументы: --startreplaybuffer --minimize-to-tray , Рабочая папка: папка с obs64.exe (C:\Program Files\obs-studio\bin\64bit)

8. настройки - общие - область уведомлений - поставить все галки

Несколько комментов по настройкам:
в профилях кодировщик nvenc av1, меняем на hevc (aka h.265) если старая видюха или если хочитите редактировать клипы в AE

аудио - все откл (добавляем источники ручками в сценах)

остальное меняй как хочешь
видео - фпс поставь сколько надо
как считать память для повтора: настройки - вывод - запись - ставим битрейт CBR, выставляем примерный желаемый битрейт (50к как референс, низкое качество для 2к60фпс в shadowplay), заходим в буфер повтора, ставим длину повтора и смотрим сколько он высчитал требуемой памяти, запоминаем это число. возвращаем битрейт на CQP, ставим кол-во памяти которое запомнили, умноженное на 2

настройка каналов аудио: расширенные настройки звука (иконка шестеренки в микшере)
активировать звуковые дорожки: настройки - вывод - запись - звуковая дорожка
переименовать звуковые дорожки: настройки - вывод - аудио




Копии постов с реддита на всякий случай

 A few points to set up replay buffer to function like Shadowplay
Guide [https://www.reddit.com/r/obs/comments/12qotws/a_few_points_to_set_up_replay_buffer_to_function/]

There are guides for how to set up OBS Replay Buffer, but I just want to compile what I did to have it in one place. It's also kind of a note for me if I had to set it up again. You might want things to work differently.

I try to periodically update the post if I change anything.

Recording:

    NVENC HEVC, CQ around 24. I switched to NVENC AV1 after I got RTX 4000 series card.

    Save files as fragmented MP4. Might have compatibility issues with some video editors.

Audio:

    right-click in audio mixer -> properties -> set mic and desktop audio to use channels 2 and 3. Check them in the recording tab. You will have 2 audio tracks, one with your microphone and another with your desktop audio.

    you can add audio filters, e.g. noise reduction for your mic etc. I personally EQ my desktop audio to counter-balance my system-wide EQ.

Start on startup:

    use this guide [https://obsproject.com/forum/threads/start-obs-as-administrator-on-startup-in-windows-10-with-startreplaybuffer.116313/] to launch OBS minimized with replay buffer on startup as admin. Basically add a task in task scheduler to run on log on with "--startreplaybuffer --minimize-to-tray" args. You can also add "--disable-shutdown-check" arg to stop getting a pop-up that OBS did not shut down correctly after a restart.

    plugin [https://github.com/Meachamp/OBS-NoPreventSleep] that stops/starts replay buffer on sleep/wake-up. Replay buffer normally prevents PC from going to sleep. I was getting really annoyed by this before I found this plugin.

Misc:

    Create 2 scenes, one with display capture and one with game capture. Use automatic scene switcher to switch to game capture scene if the active window is a game you specify, otherwise switch to display capture. Game capture performs better than display capture. For example, these are my games in automatic scene switcher - https://i.imgur.com/Rv2CKhh.png

    I checked "limit capture framerate" in-game capture source. In theory it should give slightly better performance, although it might introduce skipped frames. Try it out.

    Disable preview for better performance.

    script [https://obsproject.com/forum/resources/sound-notification-on-replay-buffer-save-windows.1453/] for playing a sound on save (not needed if you use Smart Replays)

File organization:

Shadowplay saves recordings into subfolders based on the active application.

There are multiple OBS scripts and plugins that provide this functionality.

    Smart Replays [https://obsproject.com/forum/resources/smart-replays.2039/] - the one I am currently using. It also provides other features, such as playing a sound on save, restarting replay buffer periodically etc. Requires you to have Python with Tkinter installed.

    I wrote my own plugin [https://obsproject.com/forum/resources/replay-buffer-move-to-folder.2055/] - it moves recordings into folders based on the maximized window. One advantage is that it does not have any dependencies, although it works only on Windows.

The reasons for switching to OBS from Shadowplay for me are:

    Shadowplay writes temp files to disk instead of RAM, not good for SSD health

    Shadowplay keeps turning off randomly

    No option to encode using HEVC unless you record in HDR

    More potential options (e.g. filters, more sources in a scene etc)

    The pop-up when you save a replay using Shadowplay is annoying, I prefer the sound from the OBS plugin







I was able to get the macros working with Advanced Scene Switcher. I have 2 scenes, one called Desktop and one called Game. Desktop is just a Display Capture of my primary desktop, and Game is Game Capture set for any Fullscreen Application.

Then, I created 2 macros, one called Desktop the other Game. For Game, I set it to:

    If > Window

    Window is fullscreen

    Window is focused

    Switch scene

    Program > Game > Cut

    And have both options ticked at the top.

For Desktop, I have:

    If not > Macro

    Macro condition state

    Game

    Switch scene

    Program > Desktop > Cut

    And only have the first option ticked at the top.

With that, it automatically uses my scene that has game capture whenever something is in fullscreen, and otherwise locks it to display capture.

