
Vagrant.require_version ">= 1.7.0"

$vm_gui = false
$vm_memory = 2048
$vm_cpus = 4

def vm_gui
  $vb_gui.nil? ? $vm_gui : $vb_gui
end

def vm_memory
  $vb_memory.nil? ? $vm_memory : $vb_memory
end

def vm_cpus
  $vb_cpus.nil? ? $vm_cpus : $vb_cpus
end

Vagrant.configure("2") do |config|

  config.vm.box = "ailispaw/docker-root"
  config.vm.network :private_network, ip: "172.17.8.101"
  config.vm.synced_folder ENV['HOME'], ENV['HOME'], id: "home", :nfs => true, :mount_options => ['noatime,soft,nolock,vers=3,udp,proto=udp,udp,rsize=8192,wsize=8192,namlen=255,timeo=10,retrans=3,nfsvers=3']

  config.vm.provider :virtualbox do |vb|
    vb.check_guest_additions = false
    vb.functional_vboxsf     = false
    vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
    # vb.customize ["modifyvm", :id, "--uartmode1", serialFile]
    vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    vb.gui = vm_gui
    vb.memory = vm_memory
    vb.cpus = vm_cpus
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
    vb.customize ["modifyvm", :id, "--nictype1", "virtio"]
  end

  config.vm.network "forwarded_port", guest: 2375, host: 2375, auto_correct: true

  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  config.vm.provision "docker" do |d|
    d.run "tonistiigi/dnsdock",
      args: "--restart=always -v /var/run/docker.sock:/var/run/docker.sock  --name dnsdock -p 0.0.0.0:53:53/udp"
  end
end