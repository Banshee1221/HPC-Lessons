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
  - [SSH](#ssh)
    - [SSH Setup](#ssh-setup)
    - [Passwordless SSH](#passwordless-ssh)
  - [Copying Files](#copying-files)
- [Internet](#internet)
  - [Gateway](#gateway)
  - [DNS](#dns)

---

## Connectivity

In this environment, there is no DHCP (Dynamic Host Configuration Protocol) present. DHCP is the protocol that allows operating systems to receive IP addresses from an existing network in order to apply it to   the network card that exists on the machine. Since this is the case, we want to set our own IP addresses for each of the Pi's.

### Establishing connectivity

- Connect the RPi's to the network switch using the twisted-pair copper cables ending in RJ45 connectors

- Open a terminal and type in `ip addr`

  - Here you should see a list of network devices. You should see devices like `eth0` (network card 1). Do not use the `lo` device. *Note down the name of the device you're using.*

  - **Challenge: Find out what the `lo` device is.**

- Run the command
  ```
  sudo systemctl disable dhcpcd.service
  sudo systemctl stop dhcpcd.service
  ```
  *Raspbian, the operating system that is on these Pi's, has a service that automatically runs on boot called dhcpcd. This service is used to attempt to get an IP address from a DHCP server. This needs to be disabled so that any static ip addres that is set doesn't get overwritten.*

- Set up the IP addresses for each Pi manually

  Open the file `/etc/network/interfaces` in the editor of your choice. In this file you need to make an entry for the network device and the configuration you want to give it.

  Firstly, look for any lines that already mention the interface that you want to use (in this case `eth0`) and delete it. Create a new block of lines that look like this:

  ```
  auto <interface name>
  iface <interface name> inet static
  address <ip address for this machine>
  netmask 255.255.255.0
  ```
  Make sure that the ip adresses are consistent for the first three dots. I.e. have the IPs for each Pi have the same x.x.x.<number>. Only the last dot's number should change. Do not use the 127.0.0.<number> IP since that is considered local only. <Number> can be any number from 1 - 254. Do not have conflicts.

- __Challenge: What is netmask?__

- Restart the Raspberry Pi.

Repeat these processes for every one of your Pi's, giving them each a different IP address.

### Testing connectivity

Most \*nix distributions provide good connectivity tools out-of-the box. In this case, since there are no funny port restrictions on the network, we'll be using the simple `ping` command to test whether each of your Pi's are reachable.

- Execute the command `ping -c 3 <ip address of Pi>`

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

- Open the file `/etc/hosts` as root with the editor of your choice

- Add new lines for each Pi in your network to give them a hostname in the format
 `<ip address>tab<hostname>`

- Use the `ping` command on on the Pi to see if the hostnames work

Repeat this for all the Pi's in your network so that each Pi can have a hostname for each other Pi

### SSH

SSH (Secure Shell) is a way to remotely access a Linux terminal from another terminal to execute commands. SSH requires a service to be running in the background that handles the connections.

#### SSH Setup

- The Raspberry Pi has an SSH service installed already, but it is not activated by default. Activate the service by running
  ```
  sudo systemctl enable ssh
  sudo systemctl start ssh
  ```
  (Do this for all of the Pi's)

- Test conenctivity by SSH'ing into each of the Pi's by running `ssh <useraccount>@<ip or hostname>`

#### Passwordless SSH

Managing large amounts of servers using SSH can become cumbersome if you have to type in the password every server all the time. Certain services also demand that you don't have to type the password in. To get around this, we set up a cryptographic link between two machines.

- On the first machine, type in `ssh-keygen` and press enter for all the prompts. You'll see something like this:
  ```
  root@raspberrypi:~# ssh-keygen
  Generating public/private rsa key pair.
  Enter file in which to save the key (/root/.ssh/id_rsa):
  Enter passphrase (empty for no passphrase):
  Enter same passphrase again:
  Your identification has been saved in /root/.ssh/id_rsa.
  Your public key has been saved in /root/.ssh/id_rsa.pub.
  The key fingerprint is:
  e3:5f:a7:bd:b6:72:bd:84:a7:22:71:62:43:55:94:4c root@raspberrypi
  The key's randomart image is:
  +---[RSA 2048]----+
  |            =E.  |
  |           . o   |
  |          .      |
  |         .       |
  |        S        |
  |       . * .  .  |
  |        o = ...+ |
  |         o o.+* .|
  |          o o=++.|
  +-----------------+
  ```
- Now type `ssh-copy-id <username>@<ip or hostname>`, similar to the way that you would ssh to a server.

- Test the connectivity by ssh'ing to the server that you copies your ID to.

If all went well, you should be able to SSH into the server without having to type in the password. Do this from the head Pi to all the node Pi's.

### Copying Files

Sometimes files need to be transfered between machines. We use the `scp` command to do that. This command makes use of the SSH service to transfer files securely across the network.

- Make sure you're in your home directory of the head Pi: `cd ~`

- Create some empty file in your home directory: `touch empty-file.txt`

- Copy the new `empty-file.txt` to one or both of the node Pi's: `scp empty-file.txt <username>@<ip or hostname>`

## Internet

### Gateway

Since the Pi's were not configured with DHCP, they did not get any information about which machine will be providing them with internet access. As per the example before, your home router acts as the DHCP server, but it also provides DNS services and acts as the **gateway** for the devices that connect to it.

The gateway needs to be manually set on the machines:

- __Edit the network interfaces file__

  Open the file `/etc/network/interfaces` again. Go to the line where you added the manual entry for the ip address and netmask. Under that, in the same block, add the IP address of the gateway with the `gateway` section. Ask your tutor for the IP address.

  ```
  auto <interface name>
  iface <interface name> inet static
  address <ip address for this machine>
  netmask 255.255.255.0
  gateway <ip address of gateway>
  ```

- Reboot the Pi

- Test whether you can ping Google's DNS server at 8.8.8.8

Repeat the steps for each Pi.

### DNS

Again, DNS didn't get automatically set due to there being no DHCP server on the network. To set DNS manually (ask your tutor for the IP of the DNS server):

- Edit the `/etc/resolv.conf` file. Empty the file and add an entry `nameserver <ip address of DNS server>`

- Run `chattr +i /etc/resolv.conf`. This makes the file unchangeable, even by root.

- **Challenge: Why did you run the previous command?**

Repeat the steps on all the Pi's.

---
# DISCLAIMER!!! SOME OF THESE STEPS ARE NOT THE SAME ON EVERY LINUX DISTRIBUTION. THIS IS SPECIFICALLY MADE FOR THE RASPBIAN JESSE DISTRIBUTION
