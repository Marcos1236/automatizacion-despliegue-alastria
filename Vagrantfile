Vagrant.configure("2") do |config|
  # Usamos la imagen oficial de Ubuntu para VirtualBox
  config.vm.box = "ubuntu/jammy64"

  # Creamos una red privada para que Ansible se conecte siempre a esta IP
  config.vm.network "private_network", ip: "192.168.56.10"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
    # Esto ayuda a evitar problemas de red en algunos kernels de Fedora
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end
end