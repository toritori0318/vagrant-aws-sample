# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'json'

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # install chef
  config.omnibus.chef_version = "11.4.0"

  # local vagrant server
  config.vm.define :local do |local|
    local.vm.box = "centos"
    local.vm.network :forwarded_port, guest: 80, host: 8080
    local.vm.network :private_network, ip: "192.168.33.10"
  end

  # aws dev
  config.vm.define :aws_srv_dev do |aws_srv_dev|
    aws_provider_setting(
      aws_srv_dev,
      {
        'tags' => {
          'Name'         => 'vagrant-srv-dev',
          'environments' => 'development',
        },
        #'ami' => 'ami-xxxxxxxxxxxxx', # custom-base-ami
      }
    )
  end

  # production
  # aws production wap
  config.vm.define :aws_srv_prd_wap do |aws_srv_prd_wap|
    aws_provider_setting(
      aws_srv_prd_wap,
      {
        'tags' => {
          'Name'         => 'vagrant-srv-prd-wap',
          'environments' => 'production',
        },
        #'ami' => 'ami-xxxxxxxxxxxxx', # custom-base-ami
        'instance_type' => 'c1.xlarge',
        'security_groups' => ['base', 'production-wap']
      }
    )
  end
  # aws production redis
  config.vm.define :aws_srv_prd_redis do |aws_srv_prd_redis|
    aws_provider_setting(
      aws_srv_prd_redis,
      {
        'tags' => {
          'Name'         => 'vagrant-srv-prd-redis',
          'environments' => 'production',
        },
        #'ami' => 'ami-xxxxxxxxxxxxx', # custom-base-ami
        'instance_type'   => 'm2.xlarge', # medium
        'security_groups' => ['base', 'production-redis']
      }
    )
  end

  # provisioning chef-solo
  config.vm.provision :chef_solo do |chef|
    # .chef/knife.rbと同じ内容
    chef.cookbooks_path = ["./cookbooks", "./site-cookbooks"]
    chef.roles_path = "./roles"
    chef.data_bags_path = "./data_bags"
    chef.encrypted_data_bag_secret_key_path = '.data_bag_key'
    # chef jsonファイル読み込み
    begin
      fname = ENV["chef_json"]
      nodes_json = (fname) ? JSON.parse(File.read(fname)) : {'recipes'=>{}}
      # node.jsonの情報を登録
      nodes_json["recipes"].each do |run|
        case run
        when /^recipe\[(\w+)\]/
          chef.add_recipe($1)
        when /^role\[(\w+)\]/
          chef.add_role($1)
        else
          chef.add_recipe(run)
        end
      end
      # delete recipes key
      nodes_json.delete("recipes")
      # add attrubute
      chef.json = nodes_json
    rescue => ex
      $stderr.puts "#{ex} (#{ex.class})"
    end
  end

end

# aws setting
def aws_provider_setting vagrant_aws, setting
  tags              = setting['tags'] || {}
  ami               = setting['ami'] || 'ami-39b23d38'
  keypair_name      = setting['keypair_name'] || 'awskeypair'
  instance_type     = setting['instance_type'] || 't1.micro'
  region            = setting['region'] || 'ap-northeast-1'
  availability_zone = setting['availability_zone'] || 'ap-northeast-1a'
  security_groups   = setting['security_groups'] || ['provisioning']
  private_key_path  = setting['private_key_path'] || '/path/to/keypair_name'
  ssh_username      = ENV["CHEF_USER"] || setting['ssh_username'] || 'ec2-user'

  # aws provider
  vagrant_aws.vm.provider :aws do |aws, override|
    aws.tags              = tags
    aws.ami               = ami
    aws.access_key_id     = ENV["AWS_ACCESS_KEY_ID"]
    aws.secret_access_key = ENV["AWS_SECRET_ACCESS_KEY"]
    aws.keypair_name      = keypair_name
    aws.instance_type     = instance_type
    aws.region            = region
    aws.availability_zone = availability_zone
    aws.security_groups   = security_groups
    # requiretty対策
    commands = [
      "#!/bin/bash",
      "echo 'Defaults:ec2-user !requiretty' > /etc/sudoers.d/999-vagrant-cloud-init-requiretty && cpsod 440 /etc/sudoers.d/999-vagrant-cloud-init-requiretty",
    ]
    aws.user_data = commands.join("\n")
    # override
    override.ssh.username = ssh_username
    override.ssh.private_key_path = private_key_path
  end

  vagrant_aws.vm.box = "dummy"
  vagrant_aws.vm.box_url = "https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box"
end
