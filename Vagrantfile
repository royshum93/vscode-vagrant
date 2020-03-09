# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "bento/ubuntu-18.04"
  config.vm.provision "file", source: "./.bash_profile", destination: ".bash_profile"
  config.vm.provision "file", source: "~/.gitconfig", destination: ".gitconfig"
  config.vm.provision "file", source: "./vscode.service", destination: "~/vscode.service"
  config.vm.provision "file", source: "./nginx/default", destination: "~/default"
  config.vm.provision "file", source: "./settings.json", destination: "~/.local/share/code-server/User/settings.json"
  if ENV['MKCERT_PATH'].respond_to?("concat")
    config.vm.provision "file", source: ENV['MKCERT_PATH']+"/rootCA.pem", destination: "~/.local/share/mkcert/rootCA.pem"
    config.vm.provision "file", source: ENV['MKCERT_PATH']+"/rootCA-key.pem", destination: "~/.local/share/mkcert/rootCA-key.pem"
  end

  config.vm.provision "docker"
  config.vm.provision "shell", privileged: false, inline: $INSTALL, env: {"VSCODE_PASS" => ENV['VSCODE_PASS']}
  config.vm.provision "shell", privileged: false, run: "always", inline: $BOOT
  config.vm.synced_folder '.', '/vagrant', mount_options: ["dmode=777,fmode=777"]
  config.vm.network "private_network", type: "dhcp"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "6144"
    vb.cpus = 4
  end
end


$INSTALL = <<SCRIPT
sudo usermod -a -G vboxsf vagrant

echo "Install Git"
export DEBIAN_FRONTEND=noninteractive
sudo apt-get update
sudo apt-get install -y git bison dialog build-essential mycli dos2unix wget unzip pgcli nginx ntp openvpn squid php
sudo locale-gen "en_US.UTF-8"
sudo systemctl disable systemd-networkd-wait-online.service
sudo systemctl mask systemd-networkd-wait-online.service
git config --global gpg.program gpg
git config --global core.editor vim
sudo sed -i '977s/^.//' /etc/squid/squid.conf
sudo sed -i '1190s/^.//' /etc/squid/squid.conf
sudo echo "acl vmhost src 172.28.128.1/32" >> /etc/squid/squid.conf
sudo echo "acl vmhostext src 10.0.2.2/32" >> /etc/squid/squid.conf
sudo echo "http_access allow vmhost" >> /etc/squid/squid.conf
sudo echo "http_access allow vmhostext" >> /etc/squid/squid.conf
sudo service squid restart

sudo dos2unix ~/.bash_profile ~/.gitconfig ~/.ssh/id_rsa.pub ~/.ssh/id_rsa

echo "Install Node.js with NVM"
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.2/install.sh | bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
nvm install node
nvm use node
npm i -g yarn

echo "Install GVM, Go1.14 and other go tools"
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
source /home/vagrant/.gvm/scripts/gvm
gvm install go1.14 -B
gvm use go1.14
sudo mkdir /go
sudo chmod -R 777 /go
mkdir /go/bin
export GOPATH=/go
curl https://raw.githubusercontent.com/golang/dep/master/install.sh | sh
go get golang.org/x/tools/gopls
curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin v1.23.0
go get -u github.com/rogpeppe/godef
go get golang.org/x/tools/cmd/godoc
go get -u github.com/FiloSottile/mkcert
mkcert -install
cd ~/.local/share/mkcert
mkcert my.dev $(ifconfig eth1 | grep 'inet ' | awk -F'[ ]+' '{ print $3 }')
mv my.dev+1.pem my.dev.pem
mv my.dev+1-key.pem my.dev-key.pem
cd ~

echo "Install Docker Compose"
sudo curl -L https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

echo "Install Code Server"
curl -s https://api.github.com/repos/cdr/code-server/releases/latest \
  | grep browser_download_url \
  | grep linux \
  | cut -d '"' -f 4 \
  | wget -qi -
tar xvzf *x86_64.tar.gz --strip 1 --wildcards */code-server
sudo mv ./code-server /usr/bin
rm *.gz
mkdir /vagrant/code
if [ ! -z "$VSCODE_PASS" ]; then
  sed -i "8s;^;Environment=\"PASSWORD=$VSCODE_PASS\";" vscode.service;
fi
code-server --install-extension ms-vscode.Go
code-server --install-extension bierner.markdown-mermaid
code-server --install-extension eamodio.gitlens
code-server --install-extension humao.rest-client
code-server --install-extension fabiospampinato.vscode-diff
code-server --install-extension jvalecillos.gpg
code-server --install-extension felixfbecker.php-debug
code-server --install-extension felixfbecker.php-intellisense
code-server --install-extension mechatroner.rainbow-csv
sudo mv ./default /etc/nginx/sites-enabled/default
sudo mv vscode.service /lib/systemd/system
sudo systemctl enable vscode.service
sudo systemctl start vscode.service
sudo systemctl status vscode.service | grep Password
(sudo crontab -l 2>/dev/null; echo "0 */3 * * * service vscode restart") | sudo crontab -
sudo service nginx restart
SCRIPT

$BOOT = <<SCRIPT
sleep 5
echo "VSCode Starts at https://"$(ifconfig eth1 | grep 'inet ' | awk -F'[ ]+' '{ print $3 }')""
sudo systemctl status vscode.service | grep -i password
SCRIPT
