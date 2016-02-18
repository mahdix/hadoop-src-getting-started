# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT
    echo >>>>>>>>>>>>>>>>> Setting up repositories...
    export DEBIAN_FRONTEND=noninteractive
    echo "deb http://http.debian.net/debian jessie-backports main" > /etc/apt/sources.list.d/jessie_backports.list
    echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" | tee /etc/apt/sources.list.d/webupd8team-java.list
    echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" | tee -a /etc/apt/sources.list.d/webupd8team-java.list
    apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886

    echo >>>>>>>>>>>>>>>>> Updating...
    apt-get -qq update

    echo >>>>>>>>>>>>>>>>> Installing pre-requisites for Hadoop...
    apt-get -qq install -y wget curl git htop build-essential vim-nox
    apt-get -qq install -y software-properties-common
    apt-get -qq install -y autoconf automake libtool cmake zlib1g-dev pkg-config libssl-dev

    echo >>>>>>>>>>>>>>>>> Installing JDK8...
    apt-get -qq install -y oracle-java8-installer

    echo >>>>>>>>>>>>>>>>> IInstalling Maven...
    apt-get -qq install -y maven

    echo >>>>>>>>>>>>>>>>> Installing required version of protocol-buffers...
    cd /tmp
    wget -quiet https://github.com/google/protobuf/archive/v2.5.0.tar.gz
    tar xvf v2.5.0.tar.gz
    cd protobuf-2.5.0
    ./autogen.sh
    ./configure --prefix=/usr
    sudo make
    sudo make install
    cd java
    mvn -q install

    echo >>>>>>>>>>>>>>>>> Getting Hadoop source...
    cd /home/vagrant
    git clone git://git.apache.org/hadoop.git
    
    echo >>>>>>>>>>>>>>>>> Building Hadoop Maven plugins...
    cd hadoop/hadoop-maven-plugins
    mvn -q install

    echo >>>>>>>>>>>>>>>>> Compiling Hadoop
    cd ..
    mvn -q compile

    echo >>>>>>>>>>>>>>>>> Installing Hadoop
    mvn -q install

    echo >>>>>>>>>>>>>>>>> Setting up SSH
    ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
    cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
    chmod 0600 ~/.ssh/authorized_keys

    echo >>>>>>>>>>>>>>>>> Finalizing...
    echo "export JAVA_HOME=/usr/lib/jvm/java-8-oracle" >> ~/.bashrc
    echo "export HADOOP_CLASSPATH=${JAVA_HOME}/lib/tools.jar" >> ~/.bashrc
    echo "export HADOOP_HOME=/home/vagrant/hadoop/hadoop-dist/target/hadoop-3.0.0-SNAPSHOT" >> ~/.bashrc
    echo "export HADOOP_HDFS_HOME=$HADOOP_HOME" >> ~/.bashrc
    echo "export HADOOP_PREFIX=$HADOOP_HOME" >> ~/.bashrc
    echo "PATH=$PATH:$HADOOP_HOME/bin" >> ~/.bashrc
    echo "PATH=$PATH:$HADOOP_HOME/sbin" >> ~/.bashrc

    cd /home/vagrant/hadoop
    ln -s /home/vagrant/hadoop/hadoop-dist/target/hadoop-3.0.0-SNAPSHOT app

    echo All Hadoop tools are available at /home/vagrant/hadoop/app/bin and sbin (These are added to PATH)
    echo Remember to setup JAVA_HOME in /home/vagrant/hadoop/hadoop-dist/target/hadoop-3.0.0-SNAPSHOT/etc/hadoop/hadoop-env.sh to /usr/lib/jvm/java-8-oracle
    echo Remember to setup etc/hadoop/hdfs-site.xml and etc/hadoop/core-site.xml (see https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)
    echo Remember to include dfs.namenode.name.dir and dfs.datanode.data.dir in hdfs-site.xml with values pointing to valid directories with correct permissions

    echo Provisioning done.
SCRIPT


Vagrant.configure("2") do |config|
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "3000"]
    vb.customize ["modifyvm", :id, "--cpus", "2"]
    vb.customize ["modifyvm", :id, "--usb", "off"]
  end

  config.vm.box = "ARTACK/debian-jessie"
  config.vm.hostname = "hadoop-host"

  config.vm.synced_folder "root", "/root", owner: "root", group: "root"
  config.vm.provision "shell", inline: $script
  config.vm.network "private_network", ip: "192.168.50.4"

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end
end
