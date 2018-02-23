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
      memory = 0
      if /darwin/ =~ RUBY_PLATFORM
        max_memory = `system_profiler SPHardwareDataType`[/Memory: (\d+) GB/, 1].to_i
        memory = max_memory * 1000 / 8
      end
      vb.memory = ENV['DOKKU_MEMORY'] || [memory, 1024].max
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

    vars = {
      gitlab_runner_registration_token: ENV['GITLAB_RUNNER_REGISTRATION_TOKEN'],
      nadoka: [],
      postfix_relay_smtp_server: ENV['SMTP_SERVER'],
      postfix_relay_smtp_user: ENV['SMTP_USER'],
      postfix_relay_smtp_pass: ENV['SMTP_PASS'],
    }
    if ENV.key?('NADOKA_SERVICE_NAME')
      vars[:nadoka] << {
        service_name: ENV['NADOKA_SERVICE_NAME'],
        irc_host: ENV['NADOKA_IRC_HOST'],
        irc_port: ENV['NADOKA_IRC_PORT'],
        irc_pass: ENV['NADOKA_IRC_PASS'],
        irc_ssl_params: '{}',
        irc_nick: 'User',
        channel_info: ENV['NADOKA_CHANNEL_INFO'],
        more_config: <<-'RUBY'.sub('CHANNELS', ENV['NADOKA_DOCKER_WATCH_CHANNELS']),
          if system('docker', 'ps', %i[out err]=>File::NULL)
            require 'open3'
            BotConfig << {
              :name => :WatchBot,
              :channels => %w"CHANNELS",
              :command => proc {
                out, status = Open3.capture2e({'LANG'=>'C'}, 'docker', 'ps', '--format', "{\{.ID\}}\\t{\{.Names\}}\\t{\{.CreatedAt\}}\\t{\{.Command\}}")
                out = out.split(/\n/)
                unless status.success?
                  out << status.inspect
                end
                out
              },
              :min_interval => 60,
            }
          end
        RUBY
      }
    end
    ansible.extra_vars = vars
  end
end
