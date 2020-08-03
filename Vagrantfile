# coding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :

linux_os = "ubuntu/bionic64"   # Ubuntu 18.04
bridge_if = "en0: Wi-Fi (Wireless)"

vm_spec = [
  { name: "master", cpu: 2, memory: 2048,
    box: linux_os,
    private_ip: "172.16.2.15",
    public_ip: "192.168.1.85",
    storage: [], playbook: "install.yaml",
    comment: "jenkins master" },
  { name: "agent1", cpu: 1, memory: 1024,
    box: linux_os,
    private_ip: "172.16.2.16",
    storage: [], playbook: "install.yaml",
    comment: "jenkins agent-1" },
  { name: "agent2", cpu: 1, memory: 1024,
    box: linux_os,
    private_ip: "172.16.2.17",
    storage: [], playbook: "install.yaml",
    comment: "jenkins agent-2" },
]


Vagrant.configure("2") do |config|
  vm_spec.each do |spec|
    config.vm.define spec[:name] do |v|
      v.vm.box = spec[:box]
      v.vm.hostname = spec[:name]
      v.vm.network :private_network,ip: spec[:private_ip]
      if !( spec[:public_ip].nil? )
        v.vm.network :public_network,ip:  spec[:public_ip], bridge: bridge_if
      end	     
      v.vm.provider "virtualbox" do |vbox|
        vbox.gui = false
        vbox.cpus = spec[:cpu]
        vbox.memory = spec[:memory]
        i = 1
        spec[:storage].each do |vol|
          vdisk = "vdisks/sd-" + spec[:name] + "-" + i.to_s + ".vdi"
          if not File.exist?(vdisk) then
            if i == 1 then
              vbox.customize [
                'storagectl', :id,
                '--name', 'SATA Controller',
                '--add', 'sata',
                '--controller', 'IntelAHCI']
            end            
            vbox.customize [
              'createmedium', 'disk',
              '--filename', vdisk,
              '--format', 'VDI',
              '--size', vol * 1024 ]
          end
          vbox.customize [
            'storageattach', :id,
            '--storagectl', 'SATA Controller',
            '--port', i,
            '--device', 0,
            '--type', 'hdd',
            '--medium', vdisk]
          i = i + 1
        end
      end
      v.vm.synced_folder ".", "/vagrant", owner: "vagrant",
                            group: "vagrant", mount_options: ["dmode=700", "fmode=700"]

      v.vm.provision "ansible_local" do |ansible|
        ansible.playbook       = "playbook/" + spec[:playbook]
        ansible.install_mode   = "pip"
        ansible.version        = "2.9.6"    
        ansible.verbose        = false
        ansible.install        = true
        ansible.limit          = spec[:name] 
        ansible.inventory_path = "hosts"
      end
    
      # v.vm.provision "shell", inline: <<-SHELL
      #   apt-get update
      #   apt-get install -y apache2
      # SHELL
    end
  end
end
