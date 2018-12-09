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
  config.vm.box = "gusztavvargadr/w16s-de"

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
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.boot_timeout = 180

  config.vm.provider "virtualbox" do |vb|
    vb.gui    = true
    vb.memory = "4096"
  end
  
  config.vm.provider "hyperv" do |hv|
    hv.maxmemory                        = 4096
    # hv.differencing_disk                = false
    hv.linked_clone                     = true
    hv.enable_virtualization_extensions = true
    hv.vm_integration_services          = {
      guest_service_interface: true
    }
  end

  # config.vm.provider "vmware_desktop" do |vmw|
  #   vmw.gui             = true
  #   vmw.vmx["memsize"]  = "4096"
  #   vmw.vmx["numvcpus"] = "2"
  # end

  config.vm.provision "shell", inline: <<-SHELL
    If ([string]::IsNullOrEmpty((Get-Command -Name 'choco' -ErrorAction 'SilentlyContinue'))) {
      Set-ExecutionPolicy Bypass -Scope Process -Force
      Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
    }
    If ([string]::IsNullOrEmpty((Get-Command -Name 'code' -ErrorAction 'SilentlyContinue'))) {
      choco install visualstudiocode vscode-powershell -y
    }
    If ([string]::IsNullOrEmpty((Get-Command -Name 'docker' -ErrorAction 'SilentlyContinue'))) {
      choco install docker-desktop -y
    }
    If ((docker images) -notmatch 'Microsoft/NanoServer') {
      docker pull microsoft/nanoserver
    }
    If ([string]::IsNullOrEmpty($env:Servers)) {
      1..3 | ForEach-Object -Process { docker run -d -it --name="server0$_" microsoft/nanoserver }

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
