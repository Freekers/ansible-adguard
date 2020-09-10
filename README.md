# Ansible-AdGuard

## Intended Usecase
This Anible playbook deploys a self updating [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome/) stack based on Docker, featuring:

- [Unbound](https://nlnetlabs.nl/projects/unbound/about/) as recursive DNS server instead of public upstream DNS servers
- DNS over HTTPS (DoH)
- DNS over TLS (DoT)
- IPv4 & IPv6*
- Admin interface over HTTPS
- Automatic SSL certificate for DoH & DoT, powered by [Let's Encrypt](https://letsencrypt.org/)
- Self-updating, powered by Docker & Ouroboros

*The playbook will try to detect if IPv6 connectivity is available and if so automatically configure AdGuard Home to serve DNS resolution over both IPv4 and IPv6.

## Disclaimer
Please do not set up a public DNS resolver, i.e. an AdGuard Home instance facing the internet, if you don't know what you're doing. [You risk getting in all sorts of trouble.](http://openresolverproject.org/) Most ISPs don't allow public DNS resolvers on their networks and will shut you down without notice, because it's generally [a bad idea.](https://community.infoblox.com/t5/Community-Blog/How-Dangerous-Can-An-Open-DNS-Resolver-Be-Part-I/ba-p/4017).

If all you're looking for is an adblocking DNS service, please consider using [AdGuard's own public DNS service](https://adguard.com/en/adguard-dns/overview.html#instruction) instead.

## Prerequisites

1. Your Linux server must be reachable over the internet on the following ports:
- 53 (UDP/TCP) for plain DNS resolution
- 80 (TCP) for Let's Encrypt's validation method 
- 443 (TCP) for AdGuard Home's webinterface & DoH
- 853 (TCP) for DoT

2. You _must_ own a Fully Qualified Domain Name (FQDN), such as yourdomain.com.   
This is required to generate a valid Let's Encrypt SSL Certificate used for DoH & DoT.

3. You _must_ setup an A (and AAAA record if IPv6 DNS resolution is desired) for your domain, pointing to the IP address of your Linux server.   
This is required to generate a valid Let's Encrypt SSL Certificate and used for DoH & DoT.

## Installation Instructions
1. Install Ansible using `sudo apt install ansible` on the machine that will initiate the playbook.

2. Clone repository using `git clone https://github.com/Freekers/ansible-adguard.git`

3. Edit the `hosts` file to reflect your setup, i.e. change domain etc. `playbook.yml` does NOT need to be changed!

4. Start playbook using `ansible-playbook playbook.yml --ask-become-pass`

5. After installation, it can take up to 5 minutes before your AdGuard Home instance will be accessible. This is due to Let's Encrypt's certificate creation process. AdGuard Home will _not_ start before a valid SSL certificate has been generated, so please be patient! For more information, please refer to the 'Usage Instructions' section below.

Supported distros:
- Ubuntu 18.04 & 20.04
- Debian 9 & 10

## Usage Instructions
After installation, you can access the AdGuard Home admin interface of your instance by navigating to yourdomain.com. You should automatically be redirected to the login screen of your AdGuard Home instance.   
Please remember that it can take up to 5 minutes before your AdGuard Home instance will be accessible after installation due to Let's Encrypt's certificate creation process. AdGuard Home will _not_ start before a valid SSL certificate has been generated, so please be patient!

Refer to the setup page within the AdGuard Home's Admin interface to setup your devices to use your AdGuard Home instance as DNS server.

The `docker-compose.yml` file will be located at `/opt/adguard`. You can use regular docker and docker-compose commands to stop/start/restart containers.

If needed, for manual configuration of AdGuard Home, [please refer to their official documententation.](https://github.com/AdguardTeam/AdGuardHome/wiki/Configuration)   
If needed, for manual configuration of Unbound, [please refer to their official documententation.](https://nlnetlabs.nl/documentation/unbound/)   
If needed, for manual configuration of Ouroboros, [please refer to their official documententation.](https://github.com/pyouroboros/ouroboros/wiki/Usage)


## Uninstallation Instructions
1. `sudo docker-compose -f /opt/adguard/docker-compose.yml down`
2. `rm -rf /opt/adguard`
3. Consult playbook.yml to manually review packages & firewall rules that are no longer needed (Optional)

## Acknowledgements
- [ansible-role-docker](https://github.com/geerlingguy/ansible-role-docker) Ansible role by geerlingguy
- [ansible-rc-local](https://github.com/Oefenweb/ansible-rc-local) Ansible role by Oefenweb
- [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) Docker image by AdguardTeam
- [unbound](https://github.com/obi12341/docker-unbound) Docker image by obi12341
- [docker-letsencrypt](https://github.com/linuxserver/docker-letsencrypt) Docker image by LinuxServer.io
- [ouroboros](https://github.com/pyouroboros/ouroboros) Docker image by pyouroboros
- [oisd.nl](https://credits.oisd.nl) Blocklist by [sjhgvr](https://www.reddit.com/user/sjhgvr)
- [Commonly whitelisted domains](https://github.com/anudeepND/whitelist) Whitelist by anudeepND
- [Commonly whitelisted domains](https://github.com/Freekers/whitelist) Whitelist by Freekers

## License
Unless otherwise specified, all code in this repository is released under the GNU Affero General Public License v3.0. See the [repository's `LICENSE` file](https://github.com/Freekers/ansible-adguard/blob/master/LICENSE) for details.
