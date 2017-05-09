# Raspberry Pi Netwokring Practical

In this exercise, the outcomes are to:

- Connect the devices to each other using IPv4 network
    - Learn about and run connectivity tests
- Setting device hostnames via the hosts file
- Testing SSH connectivity among the devices and setting up keyless-auth
- Copying files from one device to another using SCP
- Connect the devices to a central gateway in order to access the internet
    - Setting the gateway and DNS entries

---

# Table of Contents
- [Connectivity](#connectivity)
  - [Establishing Connectivity](#establishing-connectivity)
  - [Testing Connectivity](#testing-connectivity)
- [Access](#access)
  - [Hostnames](#hostnames)

---

## Connectivity

In this environment, there is no DHCP (Dynamic Host Configuration Protocol) present. DHCP is the protocol that allows operating systems to receive IP addresses from an existing network in order to apply it to   the network card that exists on the machine. Since this is the case, we want to set our own IP addresses for each of the Pi's.

### Establishing connectivity

- __Connect the RPi's to the network switch using the twisted-pair copper cables ending in RJ45 connectors__

- __Open a terminal and type in `ip addr`__

  - Here you should see a list of network devices. You should see devices like `eth0` (network card 1). *Note down the name of the device you're using.*

  - **Challenge: Find out what the `lo` device is.**

- __Run the command__
  ```
  sudo systemctl disable dhcpcd.service
  sudo systemctl stop dhcpcd.service
  ```
  *Raspbian, the operating system that is on these Pi's, has a service that automatically runs on boot called dhcpcd. This service is used to attempt to get an IP address from a DHCP server. This needs to be disabled so that any static ip addres that is set doesn't get overwritten.*

- __Set up the IP addresses for each Pi manually__

  Open the file `/etc/network/interfaces` in the editor of your choice. In this file you need to make an entry for the network device and the configuration you want to give it.

  Firstly, look for any lines that already mention the interface that you want to use (in this case `eth0`) and delete it. Create a new block of lines that look like this:

  ```
  auto <interface name>
  iface <interface name> inet static
  address <ip address for this machine>
  netmask 255.255.255.0
  ```
- __Restart the Raspberry Pi.__

Repeat these processes for every one of your Pi's, giving them each a different IP address.

### Testing connectivity

Most \*nix distributions provide good connectivity tools out-of-the box. In this case, since there are no funny port restrictions on the network, we'll be using the simple `ping` command to test whether each of your Pi's are reachable.

- __Execute the command `ping -c 3 <ip address of Pi>`__

  If you see soemthing like below, then your connectivity is good.

  ```
  PING 192.168.142.122 (192.168.142.122) 56(84) bytes of data.
  64 bytes from 192.168.142.122: icmp_seq=1 ttl=64 time=1.49 ms
  64 bytes from 192.168.142.122: icmp_seq=2 ttl=64 time=0.685 ms
  64 bytes from 192.168.142.122: icmp_seq=3 ttl=64 time=0.770 ms
  ```

## Access

### Hostnames

If you've ever typed in a website name in a web-browser, you've made use of a protocol called DNS, or Domain Name System. What this does is it translates hostnames (e.g. www.google.com) into IP addresses. The illustration below should show you what I mean:
```
root@raspberrypi:~# ping www.google.com
PING www.google.com (216.58.223.36) 56(84) bytes of data.
64 bytes from www.google.com (216.58.223.36): icmp_seq=1 ttl=54 time=56.5 ms
```
As you can see, after running the command `ping www.google.com`, the IP address `216.58.223.36` was used.

Typically, this is set through a DNS server that runs somewhere on a local network (like on your router or modem at home), or through various DNS servers that are used for the internet (which is out of scope for this practical).

Hostnames can also be statically assinged. This is useful when you want to manage a small to medium amount of devices and you don't want to be running a DNS service. In order to do this we follow these steps:

#### Tutorial

- Open the file `/etc/hosts` with the editor of your choice

- Add new lines for each Pi that you want to have a hostname for in the format
 `<ip address>tab<hostname>`

- Use the `ping` command on on the Pi to see if the hostnames work

Repeat this for all the Pi's in your network so that each Pi can have a hostname for each other Pi
