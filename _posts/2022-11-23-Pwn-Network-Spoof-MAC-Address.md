---
title: Pwn The Network - How to spoof your MAC address
published: true
---

### [](#header-3) What is a MAC address?
MAC stands for Media Access Control, and also known as Physical address, hardware address which uniquely identifies each device on a given network. If a LAN network has two or more devices with the same MAC address, that network will not work.

<p align="center">
  <img src="https://raw.githubusercontent.com/Ahmed-Z/the-blog/gh-pages/assets/architecture-of-mac-1.webp" style="width:600px;"><br>
  <em>Architecture of MAC</em>
</p>

### [](#header-3) Reasons to change (Spoof) the MAC address
<ul>
<li><p>Some of the free wifi connection filter the MAC addresses to enable only devices that are on the whitelist to connect to the network. You can impersonate one of the allowed MAC addresses to access the network.</p></li>
<li><p>If you are caught doing some suspicious stuff in a network, your MAC address might be banned, or even to identify you.</p></li>
<li><p>MAC spoofing is essential in MITM (Man In The Middle) attack to redirect network traffic.</p></li>
</ul>

### [](#header-3) Final product
The final product, we aim to code is a python script that allows us to show the current MAC address provided a network interface, change it to some random MAC or change it to a MAC of our choosing.

First, let's check our current MAC address using `ifconfig` command in Linux.
<p align="center">
  <img src="https://raw.githubusercontent.com/Ahmed-Z/the-blog/gh-pages/assets/ini-ifconfig.PNG" style="width:600px;"><br>
</p>
We can achieve the same result using our script, but in a much cleaner way.
<p align="center">
  <img src="https://raw.githubusercontent.com/Ahmed-Z/the-blog/gh-pages/assets/mac-show.PNG" style="width:600px;"><br>
</p>
Now let's set a random MAC address.
<p align="center">
  <img src="https://raw.githubusercontent.com/Ahmed-Z/the-blog/gh-pages/assets/random-change.PNG" style="width:600px;"><br>
</p>
We can check if we really changed the MAC address of `enp0s3` interface using the command `ifconfig`.
<p align="center">
  <img src="https://raw.githubusercontent.com/Ahmed-Z/the-blog/gh-pages/assets/ifconfig-random.PNG" style="width:600px;"><br>
</p>
We can also set a MAC address of our choosing. This can be useful in some situations. 
<p align="center">
  <img src="https://raw.githubusercontent.com/Ahmed-Z/the-blog/gh-pages/assets/fake-mac.PNG" style="width:600px;"><br>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/Ahmed-Z/the-blog/gh-pages/assets/ifconfig-fake.PNG" style="width:600px;"><br>
</p>

### [](#header-3) Let's code
This code is a Python script that can be used to change the MAC address of a network interface on a computer. The script takes several command-line arguments that determine which action it will take.<br>
The `check_mac` function is used to verify that a given MAC address is valid. It does this by using a regular expression to check if the MAC address is in the correct format. If the MAC address is in the correct format, it returns True, otherwise it returns False.<br>
The `check_interface` function is used to check if a given network interface exists on the computer. It does this by creating a socket using the `AF_INET` (Internet) and `SOCK_DGRAM` (datagram) protocols and then using the `ioctl` method to retrieve information about the given interface. If the interface exists, it returns the interface's information, otherwise it returns `False`. <br>

```python
def check_mac(mac):
    # Compile a regular expression pattern to match a MAC address
    pattern = re.compile("\A\w\w:\w\w:\w\w:\w\w:\w\w:\w\w\Z")

    # Use the `match` method to check if the pattern matches the given MAC address
    if pattern.match(mac) != None:
        # If the pattern matches, return True
        return True
    # If the pattern does not match, return False
    return False
```
```python
def check_interface(ifname):
    try:
        # Create a socket using the AF_INET (Internet) and SOCK_DGRAM (datagram) protocols
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        # Use the ioctl method to retrieve information about the given interface
        # The ioctl method takes the file descriptor of the socket, a command (0x8927)
        # and a packed binary string representing the interface name
        info = fcntl.ioctl(s.fileno(), 0x8927,  struct.pack('256s', bytes(ifname, 'utf-8')[:15]))
        return info
    except OSError:
        return False
```

Now we define the function that will generate a random MAC address.
The random MAC address must respect the **LAA** (Locally Administered Address) format. A locally administered address is assigned to a device by a network administrator, overriding the burned-in address.

<p align="center">
  <img src="https://raw.githubusercontent.com/Ahmed-Z/the-blog/gh-pages/assets/LAA-format.PNG" style="width:600px;"><br>
  <em>Ranges of group and locally administered addresses (Wikipedia)</em>
</p>

The `generate_random_mac` function is used to generate a random MAC address. It creates a string of characters that can be used in a MAC address and then uses the `random` module to randomly select characters from this string to create the MAC address.

```python
def generate_random_mac():
    # Create a string of characters that can be used in a MAC address
    char = "abcdef" + string.digits
    return random.choice(char) + random.choice("26ae") + ":" + "".join(random.choice(char) + random.choice(char) + ":"  for _ in range(5))[:-1]
```
he `getHwAddr` function is used to retrieve the MAC address of a given network interface. It uses the `check_interface` function to retrieve the information about the given interface and then returns the MAC address as a string in the correct format.
```python
def getHwAddr(ifname):
    info = check_interface(ifname)
    if info:
        return ':'.join('%02x' % b for b in info[18:24])
    return f"Interface {ifname} does not exist."
```

The `change_mac` function is used to change the MAC address of a given network interface. It does this by using the `subprocess` module to run the `ifconfig` command with the `down`, `hw`, `ether`, and `up` options to disable the interface, change its MAC address, and then enable it again.


```python
def change_mac(interface,new_mac):
    # Use the `check_output` method of the `subprocess` module to run the `ifconfig` command
    # with the `down` option to disable the given interface
    subprocess.check_output(["sudo","ifconfig",interface,"down"])

    # Use the `check_output` method of the `subprocess` module to run the `ifconfig` command
    # with the `hw` and `ether` options to change the MAC address of the given interface
    subprocess.check_output(["sudo","ifconfig",interface,"hw","ether",new_mac])

    # Use the `check_output` method of the `subprocess` module to run the `ifconfig` command
    # with the `up` option to enable the given interface
    subprocess.check_output(["sudo","ifconfig",interface,"up"])
```
The `main` function is the main entry point of the script. It uses the `argparse` module to parse the command-line arguments that are passed to the script. Based on the arguments that are passed, it either prints the current MAC address of a given interface, changes the MAC address of a given interface to a specified value, or generates and sets a random MAC address for a given interface.
```python
def main():
    # Create an ArgumentParser object to parse command-line arguments
    parser = argparse.ArgumentParser()

    # Add a `--change` option to the ArgumentParser
    # If this option is specified, the `args.change` variable will be set to `True`
    parser.add_argument('--change', action='store_true')

    # Add a `--show` option to the ArgumentParser
    # This option takes a string argument that represents the name of an interface
    # If this option is specified, the `args.show` variable will be set to the specified string
    parser.add_argument('--show', type=str, required=False)

    # Add an `--interface` option to the ArgumentParser
    # This option takes a string argument that represents the name of an interface
    # If this option is specified, the `args.interface` variable will be set to the specified string
    parser.add_argument('--interface', type=str, required=False)

    # Add a `--fake` option to the ArgumentParser
    # This option takes a string argument that represents a fake MAC address
    # If this option is specified, the `args.fake` variable will be set to the specified string
    parser.add_argument('--fake', type=str, required=False)

    # Add a `--random` option to the ArgumentParser
    # If this option is specified, the `args.random` variable will be set to `True`
    parser.add_argument('--random', action='store_true')

    # Parse the command-line arguments using the ArgumentParser
    args = parser.parse_args()

    # If the `--show` option was specified, retrieve the MAC address of the specified interface
    # and print it to the console
    if args.show:
        print(getHwAddr(args.show))

    # If the `--change` option was specified, change the MAC address of the specified interface
    if args.change:
        # If the `--interface` option was not specified, print an error message and exit
        if not args.interface:
            print("Please specify an interface.")
            exit()

        # If the `--fake` or `--random` option was not specified, print an error message and exit
        if not (args.fake or args.random):
            print("Please specify --random or --fake <fake_mac>.")
            exit()

        # If the `--interface` option was not specified, print an error message and exit
        if not args.interface:
            print("You must select an interface first.")
            exit()

        # Check if the specified interface exists
        if not check_interface(args.interface):
            # If the interface does not exist, print an error message and exit
            print(f"Interface {args.interface} does not exist.")
            exit()

        # If the `--random` option was specified, generate a random MAC address
        if args.random:
            # Generate a random MAC address
            fake_mac = generate_random_mac()

            # Change the MAC address of the specified interface to the random MAC address
            change_mac(args.interface, fake_mac)

            # Print the current MAC address to the console
            print("Current MAC address: ",fake_mac)

        
if __name__ == "__main__":
    main()
```

### [](#header-3) Conclusion
Understanding the MAC address and its importance to uniquely identify devices on a network is fundamental knowledge for every hacker/pentester. The ARP protocol is used to associate a logical address with a physical or MAC address. In an upcoming article, we will be conducting a MITM attack by spoofing the ARP protocol and we will encounter MAC addresses again.<br>
Full code in github [here](https://github.com/Ahmed-Z/mac-changer).<br>
HAPPY HACKING.
