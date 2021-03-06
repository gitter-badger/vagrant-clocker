# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.6.0"
VAGRANTFILE_API_VERSION = 2
DEFAULT_BOX = "base"
DEFAULT_RAM = 256
DEFAULT_CPUS = 1
COREOS_STABLE = "http://stable.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json"
require 'yaml'

clockerConfig = YAML.load_file('clockerConfig.yaml')

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  update_brooklyn_locations(clockerConfig['master']['node'], clockerConfig['slave']['hostname_template'],clockerConfig['slave']['ip_template'],clockerConfig['slave']['count'])  
  clockerConfig['master']['node'].each do |master_node|
    
    #Create MASTERs
    config.vm.define master_node['name'] do |master_node_config|
      master_node_config.vm.hostname = master_node['name']
      #Set BOX 
      if master_node.has_key?('box') then
        master_node_config.vm.box = master_node['box']
      elsif clockerConfig['master'].has_key?('box') then
        master_node_config.vm.box = clockerConfig['master']['box']
      else
         master_node_config.vm.box = DEFAULT_BOX
      end
      
      #Configure Network
      if master_node.has_key?("ip") then
        master_node_config.vm.network "private_network", ip: master_node["ip"]
      end
      
      #Configure Port forwards
      if master_node.has_key?("forwarded_ports") then
        master_node["forwarded_ports"].each do |ports|
          master_node_config.vm.network "forwarded_port", guest: ports["guest"], host: ports["host"], guest_ip: ports["guest_ip"]
        end
      end
      
      #Custom SHELL Provisioning
      if master_node.has_key?("shell")
        master_node["shell"].each do |cmd|
          master_node_config.vm.provision "shell", privileged: false, inline: cmd
        end
      end
      
      #Configure VIRTUALBOX
      master_node_config.vm.provider :virtualbox do |vb|
        vb.name = master_node["name"]
        #Configure RAM
        if master_node.has_key?('ram') then
          vb.memory = master_node['ram']
        elsif clockerConfig['master'].has_key?('ram') then
          vb.memory = clockerConfig['master']['ram']
        else
          vb.memory = DEFAULT_RAM
        end
        
        #Configure CPUs
        if master_node.has_key?('cpus') then
          vb.cpus = master_node['cpus']
        elsif clockerConfig['master'].has_key?('cpus') then
          vb.cpus = clockerConfig['master']['cpus']
        else
          vb.cpus = DEFAULT_CPUS
        end
        master_node_config.vm.post_up_message = "Node [%s] has been started but may still be installing software" %  master_node['name']
      end
      
    end
  end
  
  #Create SLAVES
  (1..clockerConfig['slave']['count']).each do |i|
    config.vm.define vm_name = clockerConfig['slave']['hostname_template'] % [i] do |slave_node_config|
      slave_node_config.vm.hostname = vm_name
      #Set BOX
      if clockerConfig['slave'].has_key?('box') then
        slave_node_config.vm.box = clockerConfig['slave']['box']
      else
         slave_node_config.vm.box = DEFAULT_BOX
      end
      slave_node_config.vm.network "private_network", ip: clockerConfig['slave']['ip_template'] % [i]
      #Custom SHELL Provisioning
      if clockerConfig['slave'].has_key?("shell")
        clockerConfig['slave']["shell"].each do |cmd|
          slave_node_config.vm.provision "shell", privileged: false, inline: cmd
        end
      end
      
      #Configure VIRTUALBOX
      slave_node_config.vm.provider :virtualbox do |vb|
        vb.name = vm_name
        #Configure RAM
        if clockerConfig['slave'].has_key?('ram') then
          vb.memory = clockerConfig['slave']['ram']
        else
          vb.memory = DEFAULT_RAM
        end
        
        #Configure CPUs
        if clockerConfig['slave'].has_key?('cpus') then
          vb.cpus = clockerConfig['slave']['cpus']
        else
          vb.cpus = DEFAULT_CPUS
        end
      end
    end 
  end
end

def update_brooklyn_locations(masters, slave_hostname_template, slave_ip_template, slave_node_count)
  saved_lines = Array.new
  File.readlines('files/.brooklyn/brooklyn.properties').each do |line|
    if !line.start_with?('brooklyn.location') then
      saved_lines.push(line)
    end
  end
  File.open("files/.brooklyn/brooklyn.properties", "w") do |file|
    file.puts saved_lines
    #Add Masters
    masters.each do |master|
      file.puts "brooklyn.location.named.%s=byon:(hosts=\"%s\")" % [master['name'], master['ip']]
      file.puts "brooklyn.location.named.%s.user=vagrant" % [master['name']]
      file.puts "brooklyn.location.named.%s.password=vagrant" % [master['name']]
    end
    #Add Slaves
    cluster_name = slave_hostname_template % ["-Cluster"]
    file.puts "brooklyn.location.named.%s=byon:(hosts=\"%s\")" % [cluster_name , slave_ip_template % ["{1-%d}" % slave_node_count]]
    file.puts "brooklyn.location.named.%s.user=vagrant" % [cluster_name]
    file.puts "brooklyn.location.named.%s.password=vagrant" % [cluster_name]
    (1..slave_node_count).each do |i|
      host = slave_hostname_template % [i]
      file.puts "brooklyn.location.named.%s=byon:(hosts=\"%s\")" % [host, slave_ip_template % [i]]
      file.puts "brooklyn.location.named.%s.user=vagrant" % [host]
      file.puts "brooklyn.location.named.%s.password=vagrant" % [host]
    end
  end
end