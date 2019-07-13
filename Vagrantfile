# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "gusztavvargadr/docker-windows"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  config.vm.hostname = "pwshop"
  config.vm.box_version = "1809.0.1906-enterprise-windows-server-1809-standard"

  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
    vb.memory = "4096"
  end
  config.vm.provider "hyperv" do |h|
    h.maxmemory                      = 4096
    differencing_disk                = false
    linked_clone                     = true
    enable_virtualization_extensions = true
    h.vm_integration_services        = {
      guest_service_interface: true
    }
  end

  config.vm.provision "shell", inline: <<-SHELL
    If ([string]::IsNullOrEmpty((Get-Command -Name 'code' -ErrorAction 'SilentlyContinue'))) {
      choco install visualstudiocode googlechrome vscode-powershell -y
    }
    If ((docker images) -notmatch 'nanoserver') {
      docker pull mcr.microsoft.com/windows/nanoserver:1809
    }
    If ([string]::IsNullOrEmpty($env:Servers)) {
      1..3 | ForEach-Object -Process { docker run -d -it --name="server0$_" mcr.microsoft.com/windows/nanoserver:1809 }

      $Server01 = docker ps --no-trunc -qf "name=server01"
      $Server02 = docker ps --no-trunc -qf "name=server02"
      $Server03 = docker ps --no-trunc -qf "name=server03"
      $Servers  = ($Server01, $Server02, $Server03) -join ','

      [System.Environment]::SetEnvironmentVariable('Server01', $Server01, [System.EnvironmentVariableTarget]::User)
      [System.Environment]::SetEnvironmentVariable('Server02', $Server02, [System.EnvironmentVariableTarget]::User)
      [System.Environment]::SetEnvironmentVariable('Server03', $Server03, [System.EnvironmentVariableTarget]::User)
      [System.Environment]::SetEnvironmentVariable('Servers',  $Servers,  [System.EnvironmentVariableTarget]::User)
    } Else {
      ForEach ($Server in ($env:Servers -Split ',')) {
        docker start $Server
      }
    }
  SHELL
end
