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
    config.vm.box = "ubuntu/focal64"
  
    # Disable automatic box update checking. If you disable this, then
    # boxes will only be checked for updates when the user runs
    # `vagrant box outdated`. This is not recommended.
    # config.vm.box_check_update = false
  
    # Create a forwarded port mapping which allows access to a specific port
    # within the machine from a port on the host machine. In the example below,
    # accessing "localhost:8080" will access port 80 on the guest machine.
    # NOTE: This will enable public access to the opened port
    # config.vm.network "forwarded_port", guest: 8080, host: 8080
  
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
    config.vm.synced_folder "notebooks", "/opt/notebooks"
  
    # Provider-specific configuration so you can fine-tune various
    # backing providers for Vagrant. These expose provider-specific options.
    # Example for VirtualBox:
    
    config.vm.provider "virtualbox" do |vb|
      # Display the VirtualBox GUI when booting the machine
      vb.gui = false
    
      # Customize the amount of memory and CPU on the VM:
      vb.memory = 3096
      vb.cpus = 3
    end
    
    # View the documentation for the provider you are using for more
    # information on available options.

    config.vm.define "main", primary: true do |main|
      main.vm.hostname = "main"
      main.vm.network "forwarded_port", guest: 8192, host: 8192
      main.vm.network "forwarded_port", guest: 4040, host: 4040

      # Shell script to provision the main instance
      main.vm.provision "shell", inline: <<-SHELL
        # Configure OpenJDK 11
        wget -nv https://github.com/adoptium/temurin11-binaries/releases/download/jdk-11.0.14.1+1/OpenJDK11U-jdk_x64_linux_hotspot_11.0.14.1_1.tar.gz
        tar -zxpf OpenJDK11U-jdk_x64_linux_hotspot_11.0.14.1_1.tar.gz
        mv jdk-11.0.14.1+1/ /opt/
        chown -R vagrant:vagrant /opt/jdk-11.0.14.1+1/
        echo 'JAVA_HOME="/opt/jdk-11.0.14.1+1/"' >> /etc/environment
        sed -E -i 's|(PATH.*)"|\\1:/opt/jdk-11.0.14.1+1/bin"|g' /etc/environment

        # Configure Spark
        wget -nv https://archive.apache.org/dist/spark/spark-2.4.8/spark-2.4.8-bin-hadoop2.7.tgz
        tar -zxpf spark-2.4.8-bin-hadoop2.7.tgz
        mv spark-2.4.8-bin-hadoop2.7/ /opt/
        chown -R vagrant:vagrant /opt/spark-2.4.8-bin-hadoop2.7/
        echo 'SPARK_HOME="/opt/spark-2.4.8-bin-hadoop2.7/"' >> /etc/environment
        sed -E -i 's|(PATH.*)"|\\1:/opt/spark-2.4.8-bin-hadoop2.7/bin"|g' /etc/environment
        sed -E -i 's|(PATH.*)"|\\1:/opt/spark-2.4.8-bin-hadoop2.7/sbin"|g' /etc/environment

        # Configure Python 3.7
        add-apt-repository ppa:deadsnakes/ppa -y

        # Install APT packages
        apt-get update
        apt-get upgrade -y
        apt-get install -y \
          python3.7 \
          python3.7-dev \
          python3-pip
      SHELL

      # Shell script to configure Polynote
      main.vm.provision "shell", inline: <<-SHELL
        wget -nv https://github.com/polynote/polynote/releases/download/0.4.5/polynote-dist.tar.gz
        tar -zxpf polynote-dist.tar.gz
        mv polynote/ /opt/
        chown -R vagrant:vagrant /opt/polynote/
        python3.7 -m pip install -r /opt/polynote/requirements.txt --no-warn-script-location
        sed -i 's/python3/python3.7/g' /opt/polynote/polynote.py
      SHELL

      # Shell script to run Polynote every time the instance starts
      main.vm.provision "shell", privileged: "false", run: "always", inline: <<-SHELL
        cp /vagrant/config.yml /opt/polynote/
        export PATH="$PATH:$JAVA_HOME/bin:$SPARK_HOME/bin:$SPARK_HOME/sbin"
        nohup /opt/polynote/polynote.py &
      SHELL

      main.vm.post_up_message = "POLYNOTE RUNNING\nConnect to the VM by using the browser in \"http://localhost:8192\" or by running \"vagrant ssh main\""
  end
end