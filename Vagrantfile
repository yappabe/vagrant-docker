
Vagrant.require_version ">= 1.7.0"

$vm_gui = false
$vm_memory = 2048
$vm_cpus = 8

$docker_version = "17.05.0-ce"
$vm_ip_address = "172.17.8.101"
$docker_net = "172.16.0.0/12"

def vm_gui
  $vb_gui.nil? ? $vm_gui : $vb_gui
end

def vm_memory
  $vb_memory.nil? ? $vm_memory : $vb_memory
end

def vm_cpus
  $vb_cpus.nil? ? $vm_cpus : $vb_cpus
end

# A dummy plugin for Barge to set hostname and network correctly at the very first `vagrant up`
module VagrantPlugins
  module GuestLinux
    class Plugin < Vagrant.plugin("2")
      guest_capability("linux", "change_host_name") { Cap::ChangeHostName }
      guest_capability("linux", "configure_networks") { Cap::ConfigureNetworks }
    end
  end
end

Vagrant.configure("2") do |config|

  config.vm.box = "ailispaw/barge"
  config.vm.network :private_network, ip: "#{$vm_ip_address}"
  config.vm.synced_folder ENV['HOME'], ENV['HOME'], id: "home", :nfs => true, :mount_options => ['noatime,soft,nolock,vers=3,udp,proto=udp,udp,rsize=8192,wsize=8192,namlen=255,timeo=10,retrans=3,nfsvers=3,actimeo=1']
  config.vm.guest = :linux

  config.vm.provider :virtualbox do |vb|
    vb.check_guest_additions = false
    vb.functional_vboxsf     = false
    vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
    vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    vb.gui = vm_gui
    vb.memory = vm_memory
    vb.cpus = vm_cpus
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
    vb.customize ["modifyvm", :id, "--nictype1", "virtio"]
  end

  config.vm.network "forwarded_port", guest: 2375, host: 2375, auto_correct: true
  config.vm.network "forwarded_port", guest: 9000, host: 9000, auto_correct: true

  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  # Adjusting datetime before provisioning.
  config.vm.provision :shell, run: "always" do |sh|
    sh.inline = "sntp -4sSc pool.ntp.org; date"
  end

  config.vm.provision "file", source: "./docker", destination: '/tmp/docker'
  config.vm.provision "shell", inline: "mv /tmp/docker /etc/default/docker", privileged: true
  config.vm.provision "shell", inline: "mkdir -p /home/bargee/cronjobs", privileged: false
  config.vm.provision "file", source: "./cronjobs/date.sh", destination: '/home/bargee/cronjobs/date.sh'
  config.vm.provision "file", source: "./crontab", destination: '/home/bargee/crontab'
  config.vm.provision "shell", inline: "cd /home/bargee/cronjobs; chmod 755 *.sh", privileged: true
  config.vm.provision "shell", inline: "cd /home/bargee; cat crontab | crontab -;  crontab -l", privileged: true
  config.vm.provision "shell", inline: "/etc/init.d/docker restart #{$docker_version}", privileged: true

  config.vm.provision "docker" do |d|
    d.pull_images "ailispaw/dnsdock:1.16.4"
    d.run "dnsdock",
      image: "ailispaw/dnsdock:1.16.4",
      args: "-v /var/run/docker.sock:/var/run/docker.sock -p 0.0.0.0:53:53/udp",
      restart: "always",
      daemonize: true
  end

  # http://portainer.io
  config.vm.provision "docker" do |d|
    d.pull_images "portainer/portainer"
    d.run "portainer/portainer",
      image: "portainer/portainer",
      args: "-v /var/run/docker.sock:/var/run/docker.sock -p 9000:9000  --privileged",
      restart: "always",
      daemonize: true
  end

  config.vm.provision :shell do |sh|
    sh.inline = <<-EOT
      echo "nameserver 127.0.0.1" > /etc/resolv.conf.head
      dhcpcd -x eth0 && dhcpcd eth0
    EOT
  end

  if Vagrant.has_plugin?("vagrant-triggers") then
    config.trigger.after [:up, :resume] do
      info "Setup route to vm ip."
      run <<-EOT
        sh -c "sudo route -n add -net #{$docker_net} #{$vm_ip_address}"
      EOT
    end

    config.trigger.after [:destroy, :suspend, :halt] do
      info "Remove route to vm ip."
      run <<-EOT
        sh -c "sudo route -n delete -net #{$docker_net} #{$vm_ip_address}"
      EOT
    end
  end

end
