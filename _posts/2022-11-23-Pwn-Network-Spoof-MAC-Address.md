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
<li><p>If you are caught doing some suspecious stuff in a network, your MAC address might be banned, or even to identify you.</p></li>
<li><p>MAC spoofing is essential in MITM (Man In The Middle) attack to redirect network traffic.</p></li>
</ul>

### [](#header-3) Final product
The final product we aim to code is a python script that allow us to show the current MAC address provided a network interface, change it to some random MAC or change it to a MAC of our choosing.

First let's check our current MAC address using `ifconfig` command in linux.
<p align="center">
  <img src="https://raw.githubusercontent.com/Ahmed-Z/the-blog/gh-pages/assets/ini-ifconfig.PNG" style="width:600px;"><br>
</p>
We can acheive the same result using our script but in a much cleaner way.
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
We can also set a MAC address of our choosing. This can be usefull in some situations. 
<p align="center">
  <img src="https://raw.githubusercontent.com/Ahmed-Z/the-blog/gh-pages/assets/fake-mac.PNG" style="width:600px;"><br>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/Ahmed-Z/the-blog/gh-pages/assets/ifconfig-fake.PNG" style="width:600px;"><br>
</p>

### [](#header-3) Let's code
Coding your own MAC spoofer is easy and straightforward.<br>
First, let's define some functions to check if the MAC address format is correct or not and if the network interface is up.

```python
def check_mac(mac):
    pattern = re.compile("\A\w\w:\w\w:\w\w:\w\w:\w\w:\w\w\Z")
    if pattern.match(mac) != None:
        return True
    return False
```
```python
def check_interface(ifname):
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
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

```python
def generate_random_mac():
    char = "abcdef" + string.digits
    return random.choice(char) + random.choice("26ae") + ":" + "".join(random.choice(char) + random.choice(char) + ":"  for _ in range(5))[:-1]
```
`getHwAddr()` function return the MAC address provided an interface name.
```python
def getHwAddr(ifname):
    info = check_interface(ifname)
    if info:
        return ':'.join('%02x' % b for b in info[18:24])
    return f"Interface {ifname} does not exist."
```
Changing the MAC address in linux, is as simple as executing 3 commands. We can automate this process using the `subprocess` library of python.


```python
def change_mac(interface,new_mac):
    subprocess.check_output(["sudo","ifconfig",interface,"down"])
    subprocess.check_output(["sudo","ifconfig",interface,"hw","ether",new_mac])
    subprocess.check_output(["sudo","ifconfig",interface,"up"])
```
Finally, in the `main()` function we handle the user input using the module `argparse`.
```python
def main():
    parser = argparse.ArgumentParser()
    parser.add_argument('--change', action='store_true')
    parser.add_argument('--show', type=str, required=False)
    parser.add_argument('--interface', type=str, required=False)
    parser.add_argument('--fake', type=str, required=False)
    parser.add_argument('--random', action='store_true')
    args = parser.parse_args()
    if args.show:
        print(getHwAddr(args.show))
    if args.change:
        if not args.interface:
            print("Please specify an interface.")
            exit()
        if not (args.fake or args.random):
            print("Please specify --random or --fake <fake_mac>.")
            exit()
        if not args.interface:
            print("You must select an interface first.")
            exit()
        if not check_interface(args.interface):
            print(f"Interface {args.interface} does not exist.")
            exit()
        if args.random:
            fake_mac = generate_random_mac()
            change_mac(args.interface, fake_mac)
            print("Current MAC address: ",fake_mac)
            exit()
        if args.fake:
            if not check_mac(args.fake):
                print("MAC address is not valid.")
                exit()
            change_mac(args.interface, args.fake)
            print("Current MAC address: ",args.fake)
            exit()    
        
if __name__ == "__main__":
    main()
```

### [](#header-3) Conclusion
Understanding the MAC address and its importance to uniquely identify devices on a network is fundamental knowledge for every hacker/pentester. The ARP protocol is used to associate a logical address with a physical or MAC address. In an upcoming article, we will be conducting MITM attack by spoofing the ARP protocol and we will encounter MAC addresses again.<br>
Full code in github [here](https://github.com/Ahmed-Z/mac-changer).<br>
HAPPY HACKING.
