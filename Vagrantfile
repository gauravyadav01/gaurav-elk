# -*- mode: ruby -*-
# vi: set ft=ruby noet :
#
#
# Require YAML module
require 'yaml'


# mutable by ARGV and settings file
file    = 'servers.yml'
default_domain  = 'example.com'
default_box = 'centos/7'
default_network = '192.168.2.0/24'


# Specify minimum Vagrant version and Vagrant API version
Vagrant.require_version ">= 1.6.0"
VAGRANTFILE_API_VERSION = "2"
vagrantfiledir = File.expand_path File.dirname(__FILE__)
f = File.join(vagrantfiledir, file)

# Read YAML file with box details
begin
  settings = YAML.load_file f
  domain   = settings.fetch(:domain, default_domain)
  network  = settings.fetch(:network, default_network)
  box      = settings.fetch(:box, default_box)

  if settings[:vms].is_a?(Array)
    vms = settings[:vms]
  end
rescue
  puts "Create a servers.yaml file in current direcory"
  message = <<-EOF
  ---
  - name: coreos-01
    box: coreos-alpha
    ip: 192.168.10.2
  EOF
  puts message
  exit 1
end

roles = Hash.new { |h, k| h[k] = Array.new }
INVENTORY_PATH = "./environments/vagrant/hosts"
 vms.each_with_index do |x, i|
    name   = x[:name]
    role   = x[:role]
		fqdn   = [name, domain].join('.')

   if roles.has_key?(role)
     roles[role].push(fqdn)
   else
     roles[role].push(fqdn)
   end
 end
   #f = File.open("#{INVENTORY_PATH}", File::WRONLY|File::TRUNC|File::CREAT)
   f = File.open("#{INVENTORY_PATH}", 'w+')
   roles.keys.sort.each do |k|
     f.write("[#{k}]\n")
     roles[k].each { |v| f.write("#{v}\n") }
     f.write("\n")
   end
   f.close


# Create boxes
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  if Vagrant.has_plugin?('vagrant-cachier')
    config.cache.scope = :box
    config.cache.auto_detect = true
  else
    puts 'WARN:  Vagrant-cachier plugin not detected. Continuing unoptimized.'
  end

  default_cpu = 1
  default_memory = 512
  config.ssh.insert_key         = false
  config.vm.box_check_update    = false
  config.landrush.enabled       = true
  config.landrush.tld           = domain
  config.vm.synced_folder       ".", "/vagrant"

  # Iterate through entries in YAML file
  vms.each_with_index do |x, i|
    name   = x[:name]
    ip     = x[:ip]
    role   = x[:role]
		fqdn   = [name, domain].join('.') 
    box    = x.fetch(:box, box)
    memory = x.fetch(:memory, default_memory)
    cpu    = x.fetch(:cpu, default_cpu)
		playbook = x[:playbook]

    config.vm.define name.to_sym do |vm|
      vm.vm.hostname = "#{name}.#{domain}"
      vm.vm.box      = box
      vm.vm.network  :private_network, ip: ip
      vm.vm.provider :virtualbox do |vb|
        vb.linked_clone = true
        vb.name         = name
        vb.memory       = memory
        vb.cpus         = cpu
        vb.auto_nat_dns_proxy = false
        vb.customize    ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize    ["modifyvm", :id, "--natdnsproxy1", "on"]
        vb.customize    ["modifyvm", :id, "--nictype1", "virtio" ]
        vb.customize    ["modifyvm", :id, "--nictype2", "virtio" ]
      end
      config.vm.provision :ansible do |a|
        a.playbook = "playbook/#{playbook}.yml"
				a.inventory_path = INVENTORY_PATH
				a.galaxy_role_file = 'requirements.yml'
				a.limit = "#{fqdn}"
      end
    end
  end
end
