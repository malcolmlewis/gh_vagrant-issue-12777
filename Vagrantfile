# Use libvirt for the provider
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

# Configure the nodes
cluster_nodes = [
   { :host => "gh-12777",
     :type => "master",
     :box => "opensuse/MicroOS.x86_64",
     :ip => "192.168.10.180",
     :cpu => 2,
     :ram => 2048
   },
]

# Configure the domain to use
varDomain = "example.com"

# Bring up the node loop
Vagrant.configure("2") do |config|

   # The 'config' loop for all nodes in the cluster
   # Set version that uses wicked, rem out to duplicate issue
   config.vm.box_version = "16.0.0.20220225"
   config.vm.synced_folder ".", "/vagrant", disabled: true
   # Setup the timezone
   config.vm.provision "shell", inline: <<-SHELL
      echo "Set timezone"
      timedatectl set-timezone America/Chicago
      SHELL
   # The main cluster node loop
   cluster_nodes.each do |node|

      # The per node 'config' loop
      config.vm.define node[:host] do |node_config|

         # Setup the node distribution and network
         node_config.vm.box = node[:box]
         node_config.vm.network "public_network", :dev => "enp0s20u4", ip: node[:ip], :netmask => "255.255.255.0"
         node_config.vm.hostname = "#{node[:host]}.#{varDomain}"
         # The node specific configuration loop based on type
         if node[:type] == "master"
            node_config.vm.provision "shell", inline: <<-SHELL
            echo "Configure Master..."
            NODE_NAME=`hostnamectl --transient`
            NODE_IP=$(grep `hostnamectl --transient` /etc/hosts | awk '{print $1}' | tail -n1)
            echo "Node name: $NODE_NAME IP"
            echo "Configuration of Master completed..."
            SHELL
         end

         # The node libvirt provisioning loop
         node_config.vm.provider :libvirt do |libvirt|
            libvirt.cpus = "#{node[:cpu]}"
            libvirt.memory = "#{node[:ram]}"
            # Add ignition iso to set root password, ssh key, ssh options
            # Don't forget to set the full path to the iso image!
            libvirt.storage :file, :device => :cdrom, :path => '/github/gh_vagrant-issue-12777/ignition.iso'
         end

      end

   end

end
