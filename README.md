Code server in Vagrant
=======================
Host [code server](https://github.com/cdr/code-server) inside [vagrant Ubuntu](https://app.vagrantup.com/bento/boxes/ubuntu-18.04).

How to use it
-------------
1. Install Vagrant and VirtualBox
2. [Optional] Run `vagrant plugin install vagrant-vbguest` 
3. [Optional] Depending on your Host capability, you may want to adjust the settings for memory and CPU resource.
4. [Optional] Set `VSCODE_PASS` (code-server password) and `MKCERT_PATH` (`MKCERT_PATH=$(mkcert -CAROOT)` if `mkcert` is installed on host) in env if needed.
5. Run `vagrant up`

Notes
-----
- Defaults to 6GB of RAM, and 4 CPU cores.
- Install `vagrant-vbguest` for ensuring the guest addition version is same as the Virtualbox host version to avoid folder mount issue.
- Go 1.14, php and node is installed.
- OpenVPN, Squid, GVM and NVM is installed.