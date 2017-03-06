# Roll your own Adblocking VPN
This is a how to guide to creating your own VPN server that also blocks malicious domains to enhance your security and privacy while browsing.

## How does this work?
Quite simply, this guide will set you up with a Linux server that runs OpenVPN, with Dnsmasq, with a modified `hosts` file that routes offending sites to `0.0.0.0`.

## Prerequisites
- You will need a Debian/CentOS/Ubuntu server to run your OpenVPN server on.
  - If you don't have one, you can get a low cost VPS from a provider like [Bandwagon Host](https://bandwagonhost.com/aff.php?aff=575&pid=12)
  - Disclaimer: Wherever you get a server from, be sure you're obeying their TOS. I'm not responsible for anything you do from following this guide.

## Instructions
1. Get OpenVPN installed on your server. For this, we will use [Nyr](https://github.com/Nyr)'s fantastic [OpenVPN installer script](https://github.com/Nyr/openvpn-install)
  - `wget https://git.io/vpn -O openvpn-install.sh && bash openvpn-install.sh`
    - Follow the instructions to get it set up, it should take about 1 minute
    - It will generate an `.ovpn` file which you will use to connect to the VPN with from your client. We'll need this later on, so feel free to `scp` it to your client machine.
1. Now we're going to overwrite our `hosts` file to route malicious domains to `0.0.0.0` by using [StevenBlack](https://github.com/StevenBlack)'s amazing [hosts](https://github.com/StevenBlack/hosts) project.
  - `wget https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts -O /etc/hosts`
1. Install [Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html)
  - `sudo apt-get install dnsmasq`
1. We need to edit the dnsmasq config file to do a few things:
  - `sudo vim /etc/dnsmasq.conf`
    - Enable `domain-needed` and `bogus-priv`
    - Add in some alternative DNS servers (if you don't like the one provided by your host). For this example, we'll add Google DNS
    ```
    server=8.8.8.8
    server=8.8.4.4
    ```
    - Tell dnsmasq to listen on both localhost and to the subnet that OpenVPN created
    ```
    listen-address=127.0.0.1
    listen-address=10.8.0.1
    ```
1. Edit the OpenVPN config file to resolve dhcp through dnsmasq
  - `vim /etc/openvpn/server.conf`
    - Add `push "dhcp-option DNS 10.8.0.1"`
    - Delete any other lines about `"dhcp-option"`
1. Create a crontab entry that updates your hosts file every night at midnight:
  - `crontab -e`
    - Add the following line `0 0 * * * wget https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts -O /etc/hosts && service openvpn restart`
1. Restart the services
  - `sudo service dnsmasq restart && sudo service openvpn restart`
1. At this point, we have an OpenVPN server routing traffic through Dnsmasq, which is checking our hosts file for malicious hosts, and falling back to a DNS provider for non-malicious hosts. Using the `.ovpn` file from earlier, you can now connect to the VPN from your client.
  - Mac: [Tunnelblick](https://tunnelblick.net/)
  - iOS: [OpenVPN Connect](https://itunes.apple.com/us/app/openvpn-connect/id590379981)
  - Android: [OpenVPN Connect](https://play.google.com/store/apps/details?id=net.openvpn.openvpn)

## License
These instructions are licensed under an [MIT License](LICENSE)
