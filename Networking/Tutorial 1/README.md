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

## Connectivity

In this environment, there is no DHCP (Dynamic Host Configuration Protocol) present. DHCP is the protocol that allows operating systems to receive IP addresses from an existing network in order to apply it to   the network card that exists on the machine. Since this is the case, we want to set our own IP addresses for each of the Pi's.

- Connect the RPi's to the network switch using the twisted-pair copper cables ending in RJ45 connectors.

- Open a terminal and type in `ip addr`

  - Here you should see a list of network devices. You should see devices like `eth0` (network card 1). __Note down the name of the device you're using.__

  - Challenge: Find out what the `lo` device is.

- Run the command
```
sudo systemctl disable dhcpcd.service
sudo systemctl stop dhcpcd.service
```

  - Raspbian, the operating system that is on these Pi's, has a service that automatically runs on boot called dhcpcd. This service is used to attempt to get an IP address from a DHCP server. This needs to be disabled

- Set up the IP addresses for each Pi manually
    - Navigate to /etc/network/interfaces