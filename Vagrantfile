nodes = [
#  { :hostname => 'ansible', :ip => '192.168.0.40', :box => 'xenial64' },
  { :hostname => 'mon1', :ip => '192.168.0.41', :box => 'xenial64' },
  { :hostname => 'mon2', :ip => '192.168.0.42', :box => 'xenial64' },
  { :hostname => 'mon3', :ip => '192.168.0.43', :box => 'xenial64' },
  { :hostname => 'osd1', :ip => '192.168.0.51', :box => 'xenial64', :ram => 1024, :osd => 'yes' },
  { :hostname => 'osd2', :ip => '192.168.0.52', :box => 'xenial64', :ram => 1024, :osd => 'yes' },
  { :hostname => 'osd3', :ip => '192.168.0.53', :box => 'xenial64', :ram => 1024, :osd => 'yes' }
]

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  # config.ssh.private_key_path = "~/.ssh/id_rsa"
  config.ssh.forward_agent = true

  check_guest_additions = false
  functional_vboxsf = false

  nodes.each do |node|
    config.vm.define node[:hostname] do |nodeconfig|
      nodeconfig.vm.box = "bento/ubuntu-16.04"
      nodeconfig.vm.hostname = node[:hostname]
      nodeconfig.vm.network :private_network, ip: node[:ip]

      memory = node[:ram] ? node[:ram] : 512;
      nodeconfig.vm.provider :virtualbox do |vb|
        vb.name = node[:hostname]
        vb.customize [
          "modifyvm", :id,
          "--memory", memory.to_s,
        ]
        if node[:osd] == "yes"
          unless File.exist?("disk_osd-#{node[:hostname]}.vdi")
            vb.customize [ "createhd", "--filename", "disk_osd-#{node[:hostname]}", "--size", "10000" ]
          end
          vb.customize [ "storageattach", :id, "--storagectl", "SATA Controller", "--port", 3, "--device", 0, "--type", "hdd",
          "--medium", "disk_osd-#{node[:hostname]}.vdi" ]
        end
      end

      config.hostmanager.enabled = true
      config.hostmanager.manage_guest = true

      if node[:hostname] == "osd3"
        nodeconfig.vm.provision "ansible" do |ansible|
          ansible.playbook = "ansible/playbook.yml"
          ansible.inventory_path = "ansible/inventory"
          ansible.verbose = true
          ansible.limit = "all"
        end
      end
    end
  end
end
