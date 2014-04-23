# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  
  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "precise64"
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"
  config.vm.hostname="elasticsearch"

  config.vm.network :forwarded_port, host: 8888, guest: 8080

  config.vm.provider :virtualbox do |vb|
    # Use VBoxManage to customize the VM. For example to change memory:
    vb.customize ["modifyvm", :id, "--memory", "512"]
  end

  config.vm.provision :shell do |shell|
        shell.inline = %Q{
          which apt-get > /dev/null 2>&1 && apt-get update --quiet --yes || true;
          which yum > /dev/null 2>&1 && yum update -y || true;
        }
  end 

  # Install latest Chef on the machine
  #
  config.vm.provision :shell do |shell|
   
    shell.inline = %Q{
      which apt-get > /dev/null 2>&1 && apt-get install curl --quiet --yes || true;
      which yum > /dev/null 2>&1 && yum install curl -y || true;
      test -d "/opt/chef" && chef-solo -v | grep 11 || curl -# -L http://www.opscode.com/chef/install.sh | sudo bash -s --;
      /opt/chef/embedded/bin/gem list pry | grep pry || /opt/chef/embedded/bin/gem install pry --no-ri --no-rdoc;
    }
   
  end

  config.vm.provision :chef_solo do |chef|
    chef.data_bags_path    = './tmp/data_bags'
    chef.cookbooks_path = "./tmp/cookbooks"
    chef.run_list =  %w| apt build-essential vim java monit elasticsearch elasticsearch::plugins elasticsearch::proxy elasticsearch::aws elasticsearch::data elasticsearch::monit |
    
    chef.json = {
                  :java => {
                    :install_flavor => "openjdk",
                    :jdk_version    => "7"
                  },

                  :elasticsearch => {
                    :cluster => { :name => "elasticsearch_vagrant" },

                    :plugins => {
                      'karmi/elasticsearch-paramedic' => {
                        :url => 'https://github.com/karmi/elasticsearch-paramedic/archive/master.zip'
                      }
                    },

                    :limits => {
                      :nofile  => 1024,
                      :memlock => 512
                    },

                    :logging => {
                      :discovery => 'TRACE',
                      'index.indexing.slowlog' => 'INFO, index_indexing_slow_log_file'
                    },

                    :nginx => {
                      :user  =>  'www-data',
                      :users => [{ username: 'admin', password: 'password' }]
                    },
                    # For testing flat attributes:
                    "index.search.slowlog.threshold.query.trace" => "1ms",
                    # For testing deep attributes:
                    :discovery => { :zen => { :ping => { :timeout => "9s" } } },
                    # For testing custom configuration
                    :custom_config => {
                      'threadpool.index.type' => 'fixed',
                      'threadpool.index.size' => '2'
                    }
                  }
                }   
    
  end

end