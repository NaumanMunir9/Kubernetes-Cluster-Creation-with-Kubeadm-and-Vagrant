Vagrant.configure('2') do |config|
  config.vm.define 'node-a' do |node|
    node.vm.box = 'bento/ubuntu-22.04'
    node.vm.provider 'virtualbox' do |v|
      v.memory = 2048
    end
    node.vm.network :private_network, ip: "192.168.56.101"
  end
  config.vm.define 'node-b' do |node|
    node.vm.box = 'bento/ubuntu-22.04'
    node.vm.provider 'virtualbox' do |v|
      v.memory = 2048
    end
    node.vm.network :private_network, ip: "192.168.56.102"
  end
end
