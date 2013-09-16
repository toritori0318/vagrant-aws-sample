# -*- encoding: utf-8 -*-
chef_json  = ''
aws_user   = 'ec2-user'

#
# tasks
#
desc "berks install"
task :berks_install do
  sh "rm -rf Berksfile.lock && berks install --path cookbooks"
end

# local
# desc "Local Vagrant"
namespace :local do
  vm = "local"

  desc "Local: vagrant up #{vm}"
  task :up do
    sh "vagrant up #{vm} --no-provision"
  end

  desc "Local: vagrant destroy #{vm}"
  task :destroy do
    sh "vagrant destroy #{vm}"
  end

  desc "Local: iptables無効"
  task :disable_iptables do
    [
      "./nodes/#{vm}_disable_iptables.json",
    ].each do |j|
      ENV['chef_json'] = j
      sh "vagrant provision #{vm}"
    end
  end

  desc "Local: Chef実行"
  task :provision do
    ENV['chef_json'] = build_json_name('local')
    sh "vagrant provision #{vm}"
  end
end


# AWS
# desc "AWS Vagrant"
namespace :aws do
  vm = nil

  desc "AWS: vagrant up (Launch instance)"
  task :up do
    vm = validate_vm
    ENV['chef_json'] = build_json_name(vm)
    sh "vagrant up #{vm} --provider=aws --no-provision"
  end

  desc "AWS: vagrant destroy (Terminate instance)"
  task :destroy do
    vm = validate_vm
    sh "vagrant destroy #{vm}"
  end

  desc "AWS: vagrant create-ami"
  task :create_ami do
    vm = validate_vm
    name = "vagrant-%s-%s" % [vm, Time.now.strftime("%Y%m%d%H%M%S")]
    sh "vagrant create-ami #{vm} --name '#{name}' --desc '#{name}'"
  end

  desc "AWS: Chef: レシピ実行"
  task :provision do
    vm = validate_vm
    ENV['chef_json'] = build_json_name(vm)
    sh "vagrant provision #{vm}"
  end

  desc "AWS: Chef: レシピ実行(deployのみ)"
  task :provision_deploy do
    vm = validate_vm
    ENV['chef_json'] = build_json_name(vm, true)
    sh "vagrant provision #{vm}"
  end

  desc "AWS: Chef: Packer風マルチタスク[up > init > provision > create-ami > destroy]"
  task :fakepacker => [:up, :provision, :create_ami, :destroy]
end

#
# method
#
def validate_vm
  vm = ENV['vm']
  abort ('vm=<vm_name> parameter is required.') unless vm
  return vm
end
def build_json_name vm, is_deploy_only=false
  return ENV['chef_json'] if ENV['chef_json']
  return (is_deploy_only) ? "./nodes/#{vm}_deploy.json" : "./nodes/#{vm}.json"
end
