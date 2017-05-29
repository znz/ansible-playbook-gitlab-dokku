# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = '2'

PRIVATE_NETWORK = '10.0.0'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  if Vagrant.has_plugin?('vagrant-cachier')
    config.cache.scope = :box
    config.cache.synced_folder_opts = {
      owner: '_apt',
      group: '_apt',
      mount_options: ['dmode=777', 'fmode=666']
    }
  end

  config.vm.define 'gitlab0' do |vm|
    vm.vm.box = 'bento/ubuntu-16.04'
    vm.vm.hostname = 'gitlab0'
    vm.vm.network :private_network, ip: "#{PRIVATE_NETWORK}.10"
    vm.vm.provider 'virtualbox' do |vb|
      vb.memory = ENV['GITLAB_MEMORY'] || 4096
    end
  end

  config.vm.define 'gitlab-runner1' do |vm|
    vm.vm.box = 'bento/ubuntu-16.04'
    vm.vm.hostname = 'gitlab-runner1'
    vm.vm.network :private_network, ip: "#{PRIVATE_NETWORK}.11"
    vm.vm.provider 'virtualbox' do |vb|
      vb.memory = ENV['GITLAB_RUNNER_MEMORY'] || 1024
    end
  end

  config.vm.define 'gitlab-runner2' do |vm|
    vm.vm.box = 'bento/ubuntu-16.04'
    vm.vm.hostname = 'gitlab-runner2'
    vm.vm.network :private_network, ip: "#{PRIVATE_NETWORK}.12"
    vm.vm.provider 'virtualbox' do |vb|
      vb.memory = ENV['GITLAB_RUNNER_MEMORY'] || 1024
    end
  end

  config.vm.define 'dokku0' do |vm|
    vm.vm.box = 'bento/ubuntu-16.04'
    vm.vm.hostname = 'dokku0'
    vm.vm.network :private_network, ip: "#{PRIVATE_NETWORK}.9"
    vm.vm.provider 'virtualbox' do |vb|
      vb.memory = ENV['DOKKU_MEMORY'] || 1024
    end
  end

  config.vm.provider 'virtualbox' do |vb|
    vb.cpus = 2
    vb.customize ['modifyvm', :id, '--nictype1', 'virtio']
    vb.customize [
      'modifyvm', :id,
      '--hwvirtex', 'on',
      '--nestedpaging', 'on',
      '--largepages', 'on',
      '--ioapic', 'on',
      '--pae', 'on',
      '--paravirtprovider', 'kvm',
    ]
  end

  config.vm.provision 'ansible' do |ansible|
    #ansible.inventory_path = 'provision/hosts'
    ansible.playbook = 'provision/playbook.yml'
    ansible.inventory_path = 'provision/inventory/vagrant'
    ansible.verbose = ENV['ANSIBLE_VERBOSE'] if ENV['ANSIBLE_VERBOSE']
    ansible.tags = ENV['ANSIBLE_TAGS'] if ENV['ANSIBLE_TAGS']

    ansible.galaxy_role_file = 'provision/requirements.yml'
    unless ENV['ANSIBLE_GALAXY_WITH_FORCE']
      # without --force
      ansible.galaxy_command = 'ansible-galaxy install --role-file=%{role_file} --roles-path=%{roles_path}'
    end

    ansible.extra_vars = {
      gitlab_runner_registration_token: ENV['GITLAB_RUNNER_REGISTRATION_TOKEN'],
      postfix_relay_smtp_server: ENV['SMTP_SERVER'],
      postfix_relay_smtp_user: ENV['SMTP_USER'],
      postfix_relay_smtp_pass: ENV['SMTP_PASS'],
    }
  end
end
