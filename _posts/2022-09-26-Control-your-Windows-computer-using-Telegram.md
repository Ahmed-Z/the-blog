---
title: Remote control your Windows computer using Telegram
published: true
---

If you work in an office, you are probably told to always lock your computer before you go AFK as a measure of security. In my case, my colleagues will not hesitate to prank whoever forgets his screen unlocked as a punishment.
I am a very caring person, so things like these should not happen to me. I must find a way to lock my screen from a distance and preferably via my smartphone.<br>
To do that, we are going to use python to build our own Telegram bot to control our Windows PC.
[python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot) is an awesome interface for the Telegram Bot API that enables us to do so much cool stuff using `telegram.ext` submodule.

### [](#header-3)Final product

The final result looks like the image below: the `/start` command pops up the several buttons that you can tap to perform several tasks such as checking the screen status and locking it. At first, that was the main purpose of the bot, but well.. things escalated quickly.
This bot is also capable of taking screenshots, paste the clipboard, list and kill running processes, navigate file system as well execute shell commands.

<p align="center">
  <img src="https://github.com/Ahmed-Z/the-blog/raw/gh-pages/assets/telegram-final-product.png" style="height:600px;">
</p>

### [](#header-3)Step 1: Get the API token

The first step is to create a bot using BotFather of telegram. This is necessary to get the access token and chat id that we will use later.
The process is pretty simple: search for BotFather and run the following commands: `/start` , `/newbot` choose a name for your bot and finally you will get the required token. Note that the chat id and the token are separated by `:` .

<p align="center">
  <img src="https://github.com/Ahmed-Z/the-blog/raw/gh-pages/assets/telegram-access-token.jpg" style="height:400px;">
</p>

### [](#header-3)Step2: Get coding

Now we get to the fun part. First, we need to import all the necessary libraries

```python
from telegram.ext import *
from telegram import KeyboardButton, ReplyKeyboardMarkup
from mss import mss # To to capture screenshots
import tempfile
import os
import psutil # To list and kill running processes
import ctypes # To lock the screen
import webbrowser # To to open URLs in the webbrowser of the computer from the phone
import pyperclip # To access the clipboard
import subprocess # To execute shell commands
import json
```

Now let's define our telegram bot class. Of course, this can be done without using a class, but I like to keep things organized. I stored the chat id and the access token in `auth.json` file so I can access it in the `__init__` method.

```python
class telegramBOT:

    def __init__(self):
        f = open('auth.json')
        auth = json.load(f)
        self.TOKEN = auth["TOKEN"]
        self.CHAT_ID = auth["CHAT_ID"]
```

Next we define start_command() method that will handle /start command typed by the user in the mobile app. A menu appears containing multiple buttons to command the bot.

```python
def start_command(self, update, context):
        buttons = [[KeyboardButton("⚠ Screen status")], [KeyboardButton("🔒 Lock screen")], [KeyboardButton("📸 Take screenshot")],
                   [KeyboardButton("✂ Paste clipboard")], [KeyboardButton(
                       "📄 List process")], [KeyboardButton("💤 Sleep")],
                   [KeyboardButton("💡 More commands")]]
        context.bot.send_message(
            chat_id=self.CHAT_ID, text="I will do what you command.", reply_markup=ReplyKeyboardMarkup(buttons))
```

We implement now handle_message() method. When the user taps a button or type a command, this method will handle it and execute the appropriate task.

```python
def handle_message(self, update, input_text):

        usr_msg = input_text.split()

        if input_text == 'screen status':
            for proc in psutil.process_iter():
                if (proc.name() == "LogonUI.exe"):
                    return 'Screen is Locked'
            return 'Screen is Unlocked'

        if input_text == 'lock screen':
            try:
                ctypes.windll.user32.LockWorkStation()
                return "Screen locked successfully"
            except:
                return "Error while locking screen"
```

We can check the status of the screen by searching for the `logonUI.exe` in the running processes. If it exists, then the screen is locked. `ctypes` enable us to lock the screen without any problem.

I think taking screenshots is a useful feature to have. Here we define `take_screenshot()` method to handle this task.

```python
if input_text == "take screenshot":
            update.message.bot.send_photo(
                chat_id=self.CHAT_ID, photo=open(self.take_screenshot(), 'rb'))
            return None
```

```python
def take_screenshot(self):
        TEMPDIR = tempfile.gettempdir()
        os.chdir(TEMPDIR)
        with mss() as sct:
            sct.shot(mon=-1)
        return os.path.join(TEMPDIR, 'monitor-0.png')
```

Getting the clipboard content is pretty straightforward with the `pyperclip` module.

```python
if input_text == "paste clipboard":
            return pyperclip.paste()
```

Here we handle the part of process listing and killing. To kill a process a user must manually type the keyword `kill` proceeded by the name of the process (capitalization matters).

```python
if input_text == "list process":
            try:
                proc_list = []
                for proc in psutil.process_iter():
                    if proc.name() not in proc_list:
                        proc_list.append(proc.name())
                processes = "\n".join(proc_list)
            except (psutil.NoSuchProcess, psutil.AccessDenied, psutil.ZombieProcess):
                pass
            return processes

        if usr_msg[0] == 'kill':
            proc_list = []
            for proc in psutil.process_iter():
                p = proc_list.append([proc.name(), str(proc.pid)])
            try:
                for p in proc_list:
                    if p[0] == usr_msg[1]:
                        psutil.Process(int(p[1])).terminate()
                return 'Process terminated successfully'
            except:
                return 'Error occured while killing the process'
```

We can also open URLs using the command `url` proceeded by the URL, navigate the file system using `cd <path>` and download files from our computer with `download <name_of_the_file>`.

```python
 if usr_msg[0] == 'url':
            try:
                webbrowser.open(usr_msg[1])
                return 'Link opened successfully'
            except:
                return 'Error occured while opening link'

        if usr_msg[0] == "cd":
            if usr_msg[1]:
                try:
                    os.chdir(usr_msg[1])
                except:
                    return "Directory not found !"
                res = os.getcwd()
                if res:
                    return res

        if usr_msg[0] == "download":
            if usr_msg[1]:
                if os.path.exists(usr_msg[1]):
                    try:
                        document = open(usr_msg[1], 'rb')
                        update.message.bot.send_document(
                            self.CHAT_ID, document)
                    except:
                        return "Something went wrong !"
```

The subprocess module enables us to run shell commands on our computer. We just need to type `cmd` followed by the command we want to execute and the its output will be received.

```python
if usr_msg[0] == "cmd":
            res = subprocess.Popen(
                usr_msg[1:], shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, stdin=subprocess.DEVNULL)
            stdout = res.stdout.read().decode("utf-8", 'ignore').strip()
            stderr = res.stderr.read().decode("utf-8", 'ignore').strip()
            if stdout:
                return (stdout)
            elif stderr:
                return (stderr)
            else:
                return ''
```

We can put the computer to sleep, but we will lose our control on it.

```python
if input_text == "sleep":
            try:
                os.system("rundll32.exe powrprof.dll,SetSuspendState 0,1,0")
                return "Windows was put to sleep"
            except:
                return "Cannot put Windows to sleep"
```

We define the `send_response()` method to further process the data sent to the user. The limit of characters we can send with send_message method is 4096 characters, so we need to split larger amount of data into chunks and send them separately.
We also limit the use of this bot to only one account username (your account) for security reasons. So you need to change `YOUR_USERNAME` to your account username.
```python
def send_response(self, update, context):
        user_message = update.message.text
        # Please modify this
        if update.message.chat["username"] != "YOUR_USERNAME":
            print("[!] " + update.message.chat["username"] +
                  ' tried to use this bot')
            context.bot.send_message(
                chat_id=self.CHAT_ID, text="Nothing to see here.")
        else:
            user_message = user_message.encode(
                'ascii', 'ignore').decode('ascii').strip(' ')
            user_message = user_message[0].lower() + user_message[1:]
            response = self.handle_message(update, user_message)
            if response:
                if (len(response) > 4096):
                    for i in range(0, len(response), 4096):
                        context.bot.send_message(
                            chat_id=self.CHAT_ID, text=response[i:4096+i])
                else:
                    context.bot.send_message(
                        chat_id=self.CHAT_ID, text=response)
```

It is always nice to have readable error when something goes wrong.

```python
def error(self, update, context):
        print(f"Update {update} caused error {context.error}")
```

And finally the main method `start_bot()` where we add our handlers and start our bot.

```python
def start_bot(self):
        updater = Updater(self.TOKEN, use_context=True)
        dp = updater.dispatcher
        dp.add_handler(CommandHandler("start", self.start_command))
        dp.add_handler(MessageHandler(Filters.text, self.send_response))
        dp.add_error_handler(self.error)
        updater.start_polling()
        print("[+] BOT has started")
        updater.idle()
```

To start the bot we instantiate the TelegramBot class and call the `start_bot()` method.

```python
bot = TelegramBot()
bot.start_bot()
```

The entire code is available on github [here](https://github.com/Ahmed-Z/Telegram-Remote-Desktop).

### [](#header-3)Final step

Once you run this python script on the computer you want to control, you need to search for the bot in the telegram app by the name you defined in the first step.
You can always add more features to this bot. Remember, you are only limited by your imagination (and your coding skills lol). <br> python-telgram-bot documentation [here](https://docs.python-telegram-bot.org/en/stable/telegram.ext.html).

HAPPY HACKING.
