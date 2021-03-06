# -*- mode: ruby -*-
# vi: set ft=ruby :
# We set the last octet in IPV4 address here
 nodes = {
 'rh-controller' => [1, 11],
 'rh-compute' => [1, 31],
 'rh-block' => [1, 41],
 'rh-object1' => [1, 51],
 'rh-object2' => [1, 52],
}

# Create the folder structure now so we can also create the extra volumes.
# Whilst Vagrant will create the first folder based on the 'groups' setting below
# using this path to store the extra volumes causes an initial failure if the 
# following two folders do not exist when vagrant.configure runs 
dir = "#{ENV['HOME']}/VirtualBox VMs/ovuc"
unless File.directory?( dir )
    Dir.mkdir dir
end # unless

dir = "#{ENV['HOME']}/VirtualBox VMs/ovuc/additional-disks"
unless File.directory?( dir )
    Dir.mkdir dir
end # unless

Vagrant.configure("2") do |config| 
  # config.vm.box = "bento/ubuntu-16.04"    # Change these two lines to switch between an Ubuntu or RedHat release
  config.vm.box = "bento/centos-7.3"      # Change these two lines to switch between an Ubuntu or RedHat release
  config.vm.usable_port_range = 2800..2900
  config.vm.boot_timeout = 360
  config.vm.synced_folder ".", "/vagrant", disabled: true
  nodes.each do |prefix, (count, ip_start)|
    count.times do |i|
      hostname = "%s" % [prefix, (i+1)]
      config.vm.define "#{hostname}" do |box|
        box.vm.hostname = "#{hostname}.ovuc.local"

        if prefix == "rh-controller" or prefix == "rh-compute"
          box.vm.network :private_network, ip: "10.0.0.#{ip_start+i}", :netmask => "255.255.255.0" # Management Network
          box.vm.network :private_network, ip: "203.0.113.#{ip_start+i}", :netmask => "255.255.255.0" # Provider Network
        else
          box.vm.network :private_network, ip: "10.0.0.#{ip_start+i}", :netmask => "255.255.255.0" # Management Network
        end #prefix

        box.vm.provider :virtualbox do |vbox|
          # Defaults
          vbox.linked_clone = true
          vbox.name = "#{hostname}"
          vbox.customize ["modifyvm", :id, "--groups", "/ovuc"]
          vbox.customize ["modifyvm", :id, "--memory", 4096]
          vbox.customize ["modifyvm", :id, "--cpus", 1]
          vbox.customize ["modifyvm", :id, "--pae", "on"]
          vbox.customize ["modifyvm", :id, "--paravirtprovider", "kvm"]
          vbox.customize ["modifyvm", :id, "--nictype1", "virtio"]
          vbox.customize ["modifyvm", :id, "--nictype2", "virtio"]
          vbox.customize ["modifyvm", :id, "--nictype3", "virtio"]
          vbox.customize ["modifyvm", :id, "--nictype4", "virtio"]
          
          if prefix == "rh-block"
            vbox.customize ["modifyvm", :id, "--memory", 2048]
            #vbox.customize ["modifyvm", :id, "--nicpromisc3", "allow-all"]

            dir = "#{ENV['HOME']}/VirtualBox VMs/ovuc/additional-disks"
            unless File.directory?( dir )
                Dir.mkdir dir
            end # unless
            file_to_disk = "#{dir}/#{hostname}-sdb.vmdk"
            unless File.exists?( file_to_disk )
              vbox.customize ['createhd', '--filename', file_to_disk, '--size', 20 * 1024]
            end # unless
            vbox.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
          end # block

          if prefix == "rh-object1" or prefix == "rh-object2"
            vbox.customize ["modifyvm", :id, "--memory", 2048]
          
            dir = "#{ENV['HOME']}/VirtualBox VMs/ovuc/additional-disks"
            unless File.directory?( dir )
                Dir.mkdir dir
            end # unless
            file_to_disk = "#{dir}/#{hostname}-sdb.vmdk"
            unless File.exists?( file_to_disk )
              vbox.customize ['createhd', '--filename', file_to_disk, '--size', 8 * 1024]
            end # unless
            vbox.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
            file_to_disk = "#{dir}/#{hostname}-sdc.vmdk"
            unless File.exists?( file_to_disk )
              vbox.customize ['createhd', '--filename', file_to_disk, '--size', 8 * 1024]
            end # unless
            vbox.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
            file_to_disk = "#{dir}/#{hostname}-sdd.vmdk"
            unless File.exists?( file_to_disk )
              vbox.customize ['createhd', '--filename', file_to_disk, '--size', 8 * 1024]
            end # unless
            vbox.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 3, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
          end # object
        end # box.vm virtualbox

        if prefix == "rh-object2" # only run once the object02 VM has been brought on line
          config.vm.provision "ansible" do |ansible|
            # Disable default limit to connect to all the machines
            ansible.limit = "all"
            ansible.playbook = "vagrant_site.yml"
            ansible.groups = {
              "controller" => ["rh-controller"],
              "compute" => ["rh-compute"],
              "block" => ["rh-block"],
              "object" => ["rh-object1", "rh-object2"],
              "ntp-server" => ["rh-controller"],
              "ntp-client" => ["rh-compute", "rh-block", "rh-object1", "rh-object2"],
              "keystone" => ["rh-controller"],
              "glance" => ["rh-controller"],
              "general" => ["rh-controller", "rh-compute", "rh-block", "rh-object1", "rh-object2"]
            }
          end # do ansible
        end # if prefix == rh-object2
      end # config.vm.define
    end # count.times
  end # nodes.each
end # Vagrant.configure("2")  