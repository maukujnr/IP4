Vagrant.configure("2") do |config|
    # Set the Vagrant box to use
    config.vm.box = "geerlingguy/ubuntu2004"
  
    # Forward SSH agent for Ansible to use
    config.ssh.forward_agent = false
  
    # Disable SSH key and certificate validation
    config.ssh.insert_key = false
    config.ssh.verify_host_key = :never
  
    # Provision the VM with Ansible
    config.vm.provision "ansible" do |ansible|
      # Specify the ansible playbook file with config tasks for this VM such as software installations
      ansible.playbook = "yolo-playbook.yml"
  
      # Disable SSH key and certificate validation for Ansible
      ansible.extra_vars = { ansible_ssh_common_args: '-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null' }
  
      # Define any other Ansible-related settings here
      # ansible.verbose = "v"
    end
  end
  