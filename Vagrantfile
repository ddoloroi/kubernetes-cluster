# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

################################################################################

API_VERSION=2
CURRENT_DIR="#{File.dirname(__FILE__)}"

################################################################################

yaml_config = YAML.load_file('config.yml')
vms         = yaml_config["vms"]
provision   = yaml_config["provision"]

################################################################################

module OS
    def OS.windows?
        (/cygwin|mswin|mingw|bccwin|wince|emx/ =~ RUBY_PLATFORM) != nil
    end

    def OS.mac?
        (/darwin/ =~ RUBY_PLATFORM) != nil
    end

    def OS.unix?
        !OS.windows?
    end

    def OS.linux?
        OS.unix? and not OS.mac?
    end
end

Vagrant.configure(API_VERSION) do |config|
  vms.each do |vm_host, vm_conf|

    config.vm.define vm_host, autostart: true do |conf|
      conf.ssh.forward_agent = true
      conf.ssh.insert_key = false

      conf.vm.box = vm_conf["box"]
      if !(vm_conf["box_url"].nil?)
        conf.vm.box_url = vm_conf["box_url"]
      end

      if !(vm_conf["box_download_checksum"].nil?)
        conf.vm.box_download_checksum = vm_conf["box_download_checksum"]
        conf.vm.box_download_checksum_type = "sha1"
        conf.vm.box_check_update = false
      end

      conf.vm.hostname = "#{vm_host}"

      if !(vm_conf["ip"].nil?)
        conf.vm.network "private_network", ip: vm_conf["ip"]
      end

      if !(vm_conf["shared"].nil?)
        vm_conf["shared"].each do |shared|
          shared_src = shared.match(':').pre_match
          shared_dst = shared.match(':').post_match
          Dir.mkdir(shared_src) unless File.exists?(shared_src)

          if (OS.linux? or OS.mac?)
            conf.vm.synced_folder "#{shared_src}", "#{shared_dst}", type: "nfs", nfs_udp: false, mount_options: ['rw', 'vers=3', 'tcp', 'fsc', 'nolock', 'noacl', 'async']
            conf.nfs.map_uid = Process.uid
            conf.nfs.map_gid = Process.gid
          else
            conf.vm.synced_folder "#{shared_src}", "#{shared_dst}", :mount_options => ["dmode=777", "fmode=666"]
          end
        end
      end
      conf.vm.synced_folder File.join(CURRENT_DIR, "provision"), "/srv/provision", :mount_options => ["dmode=777", "fmode=666"]

      conf.vm.provider "virtualbox" do |vbox|
        vbox.memory = vm_conf["memory"] || 512
        vbox.cpus   = vm_conf["cpus"] || 1
      end

      if !(provision["ansible"].nil?)
        conf.vm.provision :ansible do |ansible|
          ansible.limit = "#{vm_host}"
          ansible.playbook = "#{provision['ansible']['playbook']}"
          if !(provision['ansible']['tags'].nil?)
            ansible.tags = provision['ansible']['tags']
          end
          ansible.extra_vars = {
            kube_hosts: vms,
            kube_ip_public: vm_conf["ip"]
          }
          ansible.groups = {
            vm_conf['role'] => "#{vm_host}"
          }
        end
      end

    end
  end
end

################################################################################
