# -*- mode: ruby -*-
# vim: set ft=ruby :


MACHINES = {
  :otuslinux => {
    :box_name => "centos/7",
    :ip_addr => '192.168.11.101',
    :disks => {
      :sata1 => {
        :dfile => './sata1.vdi',
        :size => 250,
        :port => 1
      },
      :sata2 => {
        :dfile => './sata2.vdi',
        :size => 250,
        :port => 2
      },
      :sata3 => {
        :dfile => './sata3.vdi',
        :size => 250,
        :port => 3
      },
      :sata4 => {
        :dfile => './sata4.vdi',
        :size => 250,
        :port => 4
      },
      :sata5 => {
        :dfile => './sata5.vdi',
        :size => 250,
        :port => 5
      }
    }
  }
}


Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s
      box.vm.network "private_network", ip: boxconfig[:ip_addr]

      box.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "1024"]
        needsController = false

        boxconfig[:disks].each do |dname, dconf|
          unless File.exist?(dconf[:dfile])
            vb.customize ['createhd', '--filename', dconf[:dfile], '--size', dconf[:size]]
            needsController = true
          end
        end

        if needsController == true
          vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
          boxconfig[:disks].each do |dname, dconf|
            vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
          end
        end
      end

      box.vm.provision "shell", inline: <<-SHELL
        mkdir -p ~root/.ssh
        cp ~vagrant/.ssh/auth* ~root/.ssh
        yum install -y mdadm

        # Create a new RAID5
        mdadm --create --verbose /dev/md0 -l 5 -n 5 /dev/sd{b,c,d,e,f}

        # Mark one drive as faulty
        mdadm /dev/md0 --fail /dev/sde
        sleep 5
        mdadm /dev/md0 --remove /dev/sde

        # Re-assemble the array with removed drive
        mdadm --stop /dev/md0
        mdadm --assemble --run --force --update=resync /dev/md0 /dev/sd{b,c,d,f,e}

        # Generate a config file for RAID5 array
        echo "DEVICE partitions" > /etc/mdadm.conf
        mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm.conf

        # Create GPT and 5 partitions
        parted -s /dev/md0 mklabel gpt
        parted /dev/md0 mkpart primary ext4 0% 20%
        parted /dev/md0 mkpart primary ext4 20% 40%
        parted /dev/md0 mkpart primary ext4 40% 60%
        parted /dev/md0 mkpart primary ext4 60% 80%
        parted /dev/md0 mkpart primary ext4 80% 100%

        # Format partitions with ext4 file system
        for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done

        # Mount partitions
        mkdir -p /raid/part{1,2,3,4,5}
        for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done

      SHELL

    end
  end
end