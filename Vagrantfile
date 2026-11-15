pubkey = File.read(File.expand_path(ENV.fetch("VAGRANT_PUBLIC_KEY", "~/.ssh/id_ed25519.pub"))).strip

Vagrant.configure("2") do |config|

  config.vm.box = "generic/debian12"

  config.vm.provision "shell", env: { "PUBKEY" => pubkey }, inline: <<-SHELL
    grep -qxF "$PUBKEY" ~vagrant/.ssh/authorized_keys || echo "$PUBKEY" >> ~vagrant/.ssh/authorized_keys
  SHELL

  config.vm.define "k8s-master" do |master|
    master.vm.hostname = "k8s-master"
    master.vm.network "private_network", ip: "192.168.56.10"

    master.vm.provider "libvirt" do |vm|
      vm.memory = 2048
      vm.cpus = 2
    end
  end

  config.vm.define "k8s-worker-01" do |worker|
    worker.vm.hostname = "k8s-worker-01"
    worker.vm.network "private_network", ip: "192.168.56.11"

    worker.vm.provider "libvirt" do |vm|
      vm.memory = 2048
      vm.cpus = 2
    end
  end

  config.vm.define "k8s-worker-02" do |worker|
    worker.vm.hostname = "k8s-worker-02"
    worker.vm.network "private_network", ip: "192.168.56.12"

    worker.vm.provider "libvirt" do |vm|
      vm.memory = 2048
      vm.cpus = 2
    end
  end

end
