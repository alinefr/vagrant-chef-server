# -*- mode: ruby -*-
# vi: set ft=ruby
# Following https://learn.chef.io/modules/manage-a-node-chef-server/ubuntu/virtualbox/set-up-your-chef-server#/

$chef_server = <<CHEF_SERVER
chef_binary=/opt/opscode/bin/private-chef-ctl
packages=(curl ntp)
apt-get update
for pkg in ${packages[@]}; do
  dpkg -i $pkg | grep -q ^ii || apt-get install -y $pkg
done
service ntp stop; ntpdate -s 1.br.pool.ntp.org; service ntp start
if ! test -f "$chef_binary"; then
  wget -O- https://omnitruck.chef.io/install.sh | sudo bash -s -- -c stable -P chef-server &&
  for cmd in reconfigure restart; do chef-server-ctl $cmd; done &&
  until (curl -D - http://localhost:8000/_status) | grep "200 OK"; do sleep 15s; done
  while (curl http://localhost:8000/_status) | grep "fail"; do sleep 15s; done
  chef-server-ctl user-create admin Aline Admin aline@alinefreitas.com.br insecurepassword --filename admin.pem
  chef-server-ctl org-create alinefreitas "Aline Freitas" --association_user admin --filename alinefreitas-validator.pem
  mkdir -p /vagrant/secrets
  cp -f /home/vagrant/admin.pem /vagrant/secrets
fi
CHEF_SERVER

$node = <<NODE
apt-get update
apt-get install -y ntp
service ntp stop; ntpdate -s 1.br.pool.ntp.org; service ntp start
echo "10.1.1.33 chef-server.test" | tee -a /etc/hosts
NODE

def set_hostname(server)
  server.vm.provision "shell", inline: "hostname #{server.vm.hostname}"
end

Vagrant.configure("2") do |config|
  config.vm.define "chef-server" do |cs|
    cs.vm.box = "ubuntu/xenial64"
    cs.vm.provider 'virtualbox' do |v|
      v.memory = 2048
    end
    cs.vm.hostname = 'chef-server.test'
    cs.vm.network 'private_network', ip: '10.1.1.33'
    cs.vm.provision "shell", inline: $chef_server
    set_hostname(cs)
  end

  config.vm.define "node" do |node|
    node.vm.box = "ubuntu/xenial64"
    node.vm.hostname = 'node1-ubuntu'
    node.vm.network 'private_network', ip: '10.1.1.34'
    node.vm.provision "shell", inline: $node
    set_hostname(node)
  end
end
