# Ansible-AdGuard Automated setup for online use

## Intended Usecase

This Anible playbook deploys a self updating [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome/) stack based on Docker, featuring:

- [Unbound](https://nlnetlabs.nl/projects/unbound/about/) as recursive DNS server instead of public upstream DNS servers
- DNS over HTTPS (DoH)
- DNS over TLS (DoT)
- IPv4
- Admin interface over HTTPS
- Automatic SSL certificate for DoH & DoT
- Self-updating, powered by Docker & Watchtower

## Disclaimer

Please do not set up a public DNS resolver, i.e. an AdGuard Home instance facing the internet, if you don't know what you're doing. [You risk getting in all sorts of trouble.](http://openresolverproject.org/) Most ISPs don't allow public DNS resolvers on their networks and will shut you down without notice, because it's generally [a bad idea.](https://community.infoblox.com/t5/Community-Blog/How-Dangerous-Can-An-Open-DNS-Resolver-Be-Part-I/ba-p/4017).

If all you're looking for is an adblocking DNS service, please consider using [AdGuard's own public DNS service](https://adguard.com/en/adguard-dns/overview.html#instruction) instead.

## Prerequisites

1. Your Linux server must be reachable over the internet on the following ports:

- 53 (UDP/TCP) for plain DNS resolution
- 80 (TCP) for ZeroSSL's validation method
- 443 (TCP) for AdGuard Home's webinterface & DoH
- 853 (TCP) for DoT
- 5443 (UDP/TCP) for DNSCrypt
- 784 (UDP) for DNS over QUIC

2. You _must_ own a Fully Qualified Domain Name (FQDN), such as yourdomain.com.  
   This is required to generate a valid ZeroSSL SSL Certificate used for DoH & DoT.

3. You _must_ setup an A (and AAAA record if IPv6 DNS resolution is desired) for your domain, pointing to the IP address of your Linux server.  
   This is required to generate a valid ZeroSSL SSL Certificate and used for DoH & DoT.

## Installation Instructions

1. Install Ansible using `sudo apt-add-repository -y ppa:ansible/ansible && sudo apt install ansible` on the machine that will initiate the playbook. Or run [THIS] (https://github.com/bruvv/terraform-oracle-cloud-free-adguard) Terraform script to automaticilly create a free linux ARM server with Oracle Cloud)

2. Clone repository using `git clone https://github.com/bruvv/ansible-adguard-unbound.git`

3. Install requirements `ansible-galaxy install -r requirements/requirements.yml`

4. Change all the needed stuff in `vars` folder. But in specific: `docker.yml` & `firewall.yml` & `user-management.yml`

5. Start playbook using `ansible-playbook --connection=local --inventory 127.0.0.1, ansible-playbook.yml -e "hostname=adguard.website.com emailaddress=here@email.com"`

6. After installation, it can take up to 5 minutes before your AdGuard Home instance will be accessible. This is due to ZeroSSL's certificate creation process. AdGuard Home will _not_ start before a valid SSL certificate has been generated, so please be patient! For more information, please refer to the 'Usage Instructions' section below.

Supported distros:

- Ubuntu 18.04 & 20.04 && 22.04
- Debian 9 & 10
- RockyLinux 8 & 9

You can also switch to [blocky](https://0xerr0r.github.io/blocky/) just change the docker-compose file found in: `/roles/docker/templates/docker-compose.yml.j2`.

## Usage Instructions

After installation, you can access the AdGuard Home admin interface of your instance by navigating to yourdomain.com. You should automatically be redirected to the login screen of your AdGuard Home instance.  
Please remember that it can take up to 5 minutes before your AdGuard Home instance will be accessible after installation due to ZeroSSL's certificate creation process. AdGuard Home will _not_ start before a valid SSL certificate has been generated, so please be patient!

Refer to the setup page within the AdGuard Home's Admin interface to setup your devices to use your AdGuard Home instance as DNS server.

The `docker-compose.yml` file will be located at `/srv/docker`. You can use regular docker and docker-compose commands to stop/start/restart containers.

If needed, for manual configuration of AdGuard Home, [please refer to their official documententation.](https://github.com/AdguardTeam/AdGuardHome/wiki/Configuration)  
If needed, for manual configuration of Unbound, [please refer to their official documententation.](https://nlnetlabs.nl/documentation/unbound/)  
If needed, for manual configuration of Watchtower, [please refer to their official documententation.](https://containrrr.dev/watchtower/)

## Uninstallation Instructions

1. `sudo docker-compose -f /srv/docker/docker-compose.yml down`
2. `rm -rf /srv/docker`
3. Consult playbook.yml to manually review packages & firewall rules that are no longer needed (Optional)

## Acknowledgements

- [ansible-role-docker](https://github.com/geerlingguy/ansible-role-docker) Ansible role by geerlingguy
- [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) Docker image by AdguardTeam
- [unbound](https://github.com/obi12341/docker-unbound) Docker image by obi12341
- [watchtower](https://github.com/containrrr/watchtower) Docker image by containrrr
- [oisd.nl](https://credits.oisd.nl) Blocklist by [sjhgvr](https://www.reddit.com/user/sjhgvr)

## License

Unless otherwise specified, all code in this repository is released under the GNU Affero General Public License v3.0. See the [repository's `LICENSE` file](https://github.com/bruvv/ansible-adguard-unbound/blob/main/LICENSE) for details.
