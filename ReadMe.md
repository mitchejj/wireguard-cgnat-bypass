# Project: Honeydew (Dr. Bunsen Honeydew)

## Objective:

An ansiable automated method to generate the required files to by-pass a CG-NAT, whereas the clients should be able to access the home network and route internet traffic through the home gateway.

## Prerequisite:

* VPS
* vaultr, DigitalOcean, linode
* Ubuntu 19.10
* home server -- a node on your home network
* Raspberry Pi 4
* Ubuntu 19.10
* ansiable

I'm choosing Ubuntu 19.10 here for the sake of simplicity. I had initial tried this using a free instance of the Google Compute Engine. Nearly everything worked, expect accessing any `https` resource on my laptop.

## Source:

* ansible.cfg -- tells ansiable where to find the vault password
* wireguard.yml -- the playbook
* valut_pass
* vars
* vault.yml -- storage for the keys
* example_vault.yml -- an example vault
* wireguard.yml -- variables specific to the setup
* tasks
* install.yml -- basic packages for this to work
* configs.yml -- task which generates our files
* templates
* wg-vps.j2
* wg-home.j2
* wg-iphone.j2
* wg-ipad.j2
* wg-laptop.j2
## Assumptions:

`vars/wireguard.yml`

I choose to have my home network use a `10.0.0.0/24` subnet. Therefore, for the sake of simplicity I want my Wireguard network to use the `10.0.1.0/24` subnet. I'm also choosing to use a different port than the standard Wireguard tutorials. Also my VPS is using `eth0` as its network interface. Any changes from my defaults, can be made in `vars/wireguard.yml`. Also, an endpoint IP address **must** be provided. Also, I assume the home router can/will provide DNS.

`vars/vault.yml` and `vault_pass`

I wanted a system that would allow me to add clients as need be, I also waned a system where I could keep everything under revision control. So I am using `ansiable-vault` to keep my keys secure. 

I provided a sample vault in the project, to decrypt it:

ansible-vault decrypt vars/example_vault.yml

I am storing the vault password in `vault_pass`, of course don't keep the actual password file under source control. Edit `ansible.cfg` and choose a more secure location for the file.

To encrypt the vault:

ansible-vault encrypt vars/example_vault.yml

## Method:

Generate private and public keys for the servers and copy the resulting keys `vars/vault.yml`. A quick and dirty method to complete this would be something like:

# clear && wg genkey | tee privatekey | wg pubkey > publickey && cat *key

...

gImSti1vM4o6sf7quuP++s4GFcaR39wlKkCdjnhjHm0=
4amEjJRdtlvfXiO+Srqz3v4YJxRQsjJ1Gh0/+3txqGs=

Where the first line of the output would give you the *privatekey* and second line would give you the *publickey*, each server and client will need a unique keypair. Once completed could encrypt the vault.

Next, edit `vars/wireguard.yml` to reflect your home network, desired Wireguard network, and address of your VPS address. See Assumptions.

Now that the templates are set up accordingly run

ansiable-playbook --ask-become-pass wireguard.yml

The generated files will be created in `/etc/wireguard` only the root user can view and edit them.

Now what? Copy `wg-server-vps.conf` to your VPS server, `wg-server-home.conf` to your home server

For iPhone & iPad, the best way to to get configuration to your device is via qr code:

sudo qrencode -t ansiutf8 < /etc/wireguard/wg-iphone.conf

## Special Thanks:

[Config to bypass CGNAT using a VPS](https://github.com/smbm/wireguard-cgnat-bypass)

## To-Do:

Just a small list of things I would to do at some point in time

- [ ] actually use Ansiable as intended
- [ ] deploy to home and VPS servers, start/enable/restart services
- [ ] make the variable nesting in `vars/wireguard.yml` slightly more logical
- [ ] also allowing to use just one template file to generate multiple clients
- [ ] make naming more generic, iphone --> phone
- [ ] automated key generation and insertion

## Questions:

Fill a bug, submit a pull request... who wants to do that!

- Reddit: \_mitchejj\_
- Twitter: @mitchejj
