# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
    
fdir = File.dirname(File.expand_path(__FILE__))
conf = YAML::load_file(File.join(fdir, 'config.yml'))

Vagrant.configure(2) do |config|

  conf[:plugins].each do |plugin, pconf|
    if Vagrant.has_plugin?(pconf[:name])
      pconf[:settings].each do |setting, value|
        config.send(plugin).send(setting, value)
      end
    end
  end unless conf[:plugins].nil?

  conf[:vms].each.with_index do |vm, i|
  
    # merge the default vm config and the specific vm config
    # :provisioners refers to an array, so deep merge
    vconf = conf[:defaults].merge(vm) do |key, old, new|
      case key
      when :provisioners
        old + new
      else
        new
      end
    end

    config.vm.define vconf[:name] do |v|
      config.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--natdnspassdomain1", "off"]
        v.check_guest_additions = false
      end

      v.vm.box = vconf[:box]
      v.vm.hostname = vconf[:name]
      v.vm.network "private_network", ip: "#{conf[:private_network][:prefix]}.#{i + 2}" unless conf[:private_network].nil?

      v.vm.provider :virtualbox do |vb|
        vb.memory = vconf[:memory] unless vconf[:memory].nil?
        vb.cpus = vconf[:cpus] unless vconf[:cpus].nil?
      end

      v.vm.provider :libvirt do |lv|
        lv.memory = vconf[:memory] unless vconf[:memory].nil?
        lv.cpus = vconf[:cpus] unless vconf[:cpus].nil?
      end

      vconf[:forwarded_ports].each do |fp|
        v.vm.network "forwarded_port", guest: fp[:guest], host: fp[:host], host_ip: fp[:host_ip]
      end unless vconf[:forwarded_ports].nil?

      vconf[:synced_folders].each do |sf|
        v.vm.synced_folder sf[:src], sf[:dst], sf[:options]
      end unless vconf[:synced_folders].nil?

      vconf[:provisioners].each do |prov|
        type = prov.keys[0]
        v.vm.provision type do |_prov|
          prov[type].keys.each do |setting|
            _prov.send(setting, prov[type][setting])
          end
        end
      end unless vconf[:provisioners].nil?

    end
  end unless conf[:vms].nil?

end
