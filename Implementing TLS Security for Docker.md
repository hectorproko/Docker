

# 

The project involves configuring Docker for secure communication between the client and the daemon in both client mode and daemon mode TLS. The aim is to establish a secure environment for production use, even when all traffic is confined within trusted internal networks.

To achieve this, the client and the daemon will be secured using certificates signed by a trusted Certificate Authority (CA). The client will be configured to only connect to Docker daemons that present certificates signed by the trusted CA, while the daemon will only accept connections from clients that provide certificates from the same trusted CA.

![Example Image](images/docker-tls-diagram.png)

*The CA we're building here is to help demonstrate how to configure Docker, we're not attempting to build something production-grade.*

I'll be using Vagrant to create the test environment

<details>
<summary>Vagrantfile</summary>

```json
Vagrant.configure("2") do |config|
  
    config.vm.box = "ubuntu/bionic64" # Choose your desired base box
    
    # Shared folder configuration
    config.vm.synced_folder "./shared", "/home/vagrant/shared"

    # Create node1
    config.vm.define "node1" do |node1|
      node1.vm.hostname = "node1"
      node1.vm.network "private_network", ip: "10.0.0.10"
      # Passing the script to execute commands
      node1.vm.provision "file", source: "node1.sh", destination: "~/node1.sh"
      # Provisioning script for Docker client and host entries
      node1.vm.provision "shell", inline: <<-SHELL
        # Install Docker client
        apt-get update
        apt-get install -y docker.io
        
        # Add entries in hosts file for name resolution
        echo "10.0.0.10 node1" >> /etc/hosts
        echo "10.0.0.11 node2" >> /etc/hosts
        echo "10.0.0.12 node3" >> /etc/hosts
        sed -i '/127\.0\.2\.1/d' /etc/hosts
      SHELL
    end
    
    # Create node2
    config.vm.define "node2" do |node2|
      node2.vm.hostname = "node2"
      node2.vm.network "private_network", ip: "10.0.0.11"
      # Passing the script to execute commands
      node2.vm.provision "file", source: "node2.sh", destination: "~/node2.sh"
      # Provisioning script for CA and host entries
      node2.vm.provision "shell", inline: <<-SHELL
        # Install CA services or perform other configuration as needed
        chmod +x /home/vagrant/node2.sh

        # Add entries in hosts file for name resolution
        echo "10.0.0.10 node1" >> /etc/hosts
        echo "10.0.0.11 node2" >> /etc/hosts
        echo "10.0.0.12 node3" >> /etc/hosts
        sed -i '/127\.0\.2\.1/d' /etc/hosts
      SHELL
    end
    
    # Create node3
    config.vm.define "node3" do |node3|
      node3.vm.hostname = "node3"
      node3.vm.network "private_network", ip: "10.0.0.12"
      # Passing the script to execute commands
      node3.vm.provision "file", source: "node3.sh", destination: "~/node3.sh"
      # Provisioning script for Docker daemon and host entries
      node3.vm.provision "shell", inline: <<-SHELL
        # Install Docker daemon or perform other configuration as needed
        apt-get update
        apt-get install -y docker.io
        # Add entries in hosts file for name resolution
        echo "10.0.0.10 node1" >> /etc/hosts
        echo "10.0.0.11 node2" >> /etc/hosts
        echo "10.0.0.12 node3" >> /etc/hosts
        sed -i '/127\.0\.2\.1/d' /etc/hosts
      SHELL
    end

  end
```

</details>

*All nodes have access to a shared folder called `shared` that I'll use to distribute the keys*




