# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing
VAGRANTFILE_API_VERSION = '2'

PM_ROOT = File.dirname(__FILE__)
PM_CONFIG = File.expand_path(File.join(File.dirname(__FILE__), 'config.json'))
require_relative File.join(PM_ROOT, 'lib', 'ruby', 'playa_settings')
pmconf = PlayaSettings.new(PM_CONFIG)

box_url = "#{pmconf.base_url}/#{pmconf.box_name}-#{pmconf.platform}.box"

# #############################################################################
# Vagrant VM Definitions
# #############################################################################

ENV['VAGRANT_DEFAULT_PROVIDER'] = pmconf.platform

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network :private_network, ip: pmconf.ip_address

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  config.ssh.forward_agent = true

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = pmconf.box_name

  # There are two levels of caching here.
  # 1. If pmconf.box_local is set, then the file referenced by pmconf.box_url
  #    was found in the Packer build path (packer/builds/*.box) and Vagrant's
  #    vm.box_url is set to that path. To force retrieving the box
  #    from the URL again, simply remove the Packer builds directory.
  # 2. Vagrant only retrieves box images from vm.box_url if it does not have
  #    a local copy in ~/.vagrant.d/boxes/$BOX_NAME. These can be removed
  #    with the command: "vagrant box remove $BOX_NAME"
  config.vm.box_url = pmconf.box_local ? pmconf.box_local : box_url

  # Note: You'll want a decent amount of memory for your mesos master/slave
  # VM. The strict minimum, at least while the VM is provisioned, is the
  # amount necessary to compile mesos and the jenkins plugin. 2048m+ is
  # recommended.  The CPU count can be lowered, but you may run into issues
  # running the Jenkins Mesos Framework if you do so.
  config.vm.provider :virtualbox do |vb|
    vb.name = pmconf.box_name
    vb.customize ['modifyvm', :id, '--memory', pmconf.vm_ram]
    vb.customize ['modifyvm', :id, '--cpus',   pmconf.vm_cpus]
  end
  config.vm.provider :vmware_fusion do |v|
    v.vmx['memsize'] = pmconf.vm_ram
    v.vmx['numvcpus'] = pmconf.vm_cpus
  end
  config.vm.provider :vmware_workstation do |v|
    v.vmx['memsize'] = pmconf.vm_ram
    v.vmx['numvcpus'] = pmconf.vm_cpus
  end

  # Make the project root available to the guest VM.
  config.vm.synced_folder '.', '/vagrant'
  config.vm.synced_folder '~/vagrantResources', '/published'


  # Only provision if explicitly request with 'provision' or 'up --provision'
  args = Hash[ ARGV.flat_map{|s| s.scan(/--?([^=\s]+)(?:=(\S+))?/) } ]
  if args.key?('provision') &&  ! args.key?('provision-with')
    config.vm.provision "install", type: "shell" do |shell|
      shell.path = 'lib/scripts/common/mesosflexinstall'

      arg_array = ['--slave-hostname', pmconf.ip_address]

      # If mesos_release exists in the config.json file, pass the '--rel'
      # argument and a version to mesosflexinstall. Otherwise, do nothing.
      if pmconf.instance_variable_get(:@settings).include?('mesos_release')
        arg_array += ['--rel', pmconf.mesos_release]
      end

      if pmconf.instance_variable_get(:@settings).include?('slave_cpus')
        arg_array += ['--slave-cpus', pmconf.slave_cpus ]
      end

      if pmconf.instance_variable_get(:@settings).include?('slave_mem')
        arg_array += ['--slave-mem', pmconf.slave_mem ]
      end

      # Using an array for shell args requires Vagrant 1.4.0+
      # TODO: Set as array directly when Vagrant 1.3 support is dropped
      shell.args = arg_array.join(' ')
    end
  end

  # Only provision if specified provision-with
  if args.key?('provision') && args.key?('provision-with')
      config.vm.provision "slave", type: "shell" do |shell|
        if pmconf.instance_variable_get(:@settings).include?('slave_cpus')
          arg_array += ['--slave-cpus', pmconf.slave_cpus ]
        end
        shell.path = 'lib/scripts/common/mesosflexinstall'
        arg_array = ['reconfigure_slave']

        if pmconf.instance_variable_get(:@settings).include?('ip_address')
          arg_array += ['--slave-hostname', pmconf.ip_address]
        end

        if pmconf.instance_variable_get(:@settings).include?('slave_cpus')
          arg_array += ['--slave-cpus', pmconf.slave_cpus ]
        end

        if pmconf.instance_variable_get(:@settings).include?('slave_mem')
          arg_array += ['--slave-mem', pmconf.slave_mem ]
        end
        shell.args = arg_array.join(' ')
      end
  end

end
