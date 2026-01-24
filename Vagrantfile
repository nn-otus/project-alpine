# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # Актуальный и легкий Alpine Linux 3.19 (на 24.01.2026)
  config.vm.box = "generic/alpine319"

  # --- ГЛОБАЛЬНЫЕ НАСТРОЙКИ ---
  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.linked_clone = false
    vb.cpus = 1
    vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
    vb.customize ["modifyvm", :id, "--vram", "8"]
  end

  # --- ЗОНА 1: РОУТЕРЫ (Debian) ---
  (1..2).each do |i|
    config.vm.define "router-#{i}" do |node|
      node.vm.box = "debian/bookworm64" # Более стабильный образ
      node.vm.hostname = "router-#{i}"
      node.vm.network "private_network", ip: "192.168.56.1#{i}"
      node.vm.network "private_network", ip: "10.0.10.1#{i}"
      node.vm.network "private_network", ip: "10.0.20.1#{i}"
      node.vm.network "private_network", ip: "10.0.30.1#{i}"

      node.vm.provider "virtualbox" do |vb|
        vb.memory = 1024 # Увеличим до 1024 для Debian
      end

      node.vm.provision "shell", inline: <<-SHELL
        # Подавляем интерактивные окна для Debian
        export DEBIAN_FRONTEND=noninteractive

        apt-get update
        # Устанавливаем пакеты с флагом -y и без вопросов
        apt-get install -y python3 iptables iptables-persistent

        echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
        sysctl -p
      SHELL
    end
  end


  # --- ЗОНА 2: FRONTEND ---
  (1..2).each do |i|
    config.vm.define "front-#{i}" do |node|
      node.vm.hostname = "front-#{i}"
      node.vm.network "private_network", ip: "10.0.10.2#{i}"

      node.vm.provider "virtualbox" do |vb|
        vb.memory = 256
      end

      node.vm.provision "shell", inline: <<-SHELL
        apk update
        apk add python3 ethtool
        ip route del default
        ip route add default via 10.0.10.1
      SHELL
    end
  end

  # --- ЗОНА 3: BACKEND ---
  (1..2).each do |i|
    config.vm.define "back-#{i}" do |node|
      node.vm.hostname = "back-#{i}"
      node.vm.network "private_network", ip: "10.0.20.2#{i}"

      node.vm.provider "virtualbox" do |vb|
        vb.memory = 512
      end

      node.vm.provision "shell", inline: <<-SHELL
        apk update
        apk add python3 ethtool
        ip route del default
        ip route add default via 10.0.20.1
      SHELL
    end
  end

  # --- ЗОНА 4: DATABASE ---
  (1..2).each do |i|
    config.vm.define "db-#{i}" do |node|
      node.vm.box = "debian/bookworm64"
      node.vm.hostname = "db-#{i}"
      node.vm.network "private_network", ip: "10.0.30.3#{i}"

      node.vm.provider "virtualbox" do |vb|
        vb.memory = 1024
        vb.customize ["modifyvm", :id, "--cpuexecutioncap", "80"]
      end

      node.vm.provision "shell", inline: <<-SHELL
        export DEBIAN_FRONTEND=noninteractive
        apt-get update
        apt-get install -y python3 ethtool

        # Устанавливаем шлюз роутера как ПРИОРИТЕТНЫЙ (metric 100)
        # Шлюз Vagrant (eth0) обычно имеет метрику 0 или 100+,
        # поэтому мы явно ставим приоритет.
        ip route add default via 10.0.30.1 metric 100 || true
      SHELL
    end
  end
end
