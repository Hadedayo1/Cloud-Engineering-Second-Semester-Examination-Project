# Cloud-Engineering-Second-Semester-Examination-Project
Vagrant.configure("2") do |config|
  config.vm.define "Master" do |master|
    master.vm.box = "ubuntu/bionic64"
    # Additional configurations for Master server
  end

  config.vm.define "Slave" do |slave|
    slave.vm.box = "ubuntu/bionic64"
    # Additional configurations for Slave server
  end
end

 #BASHSCRIPT
 #!/bin/bash

# Update apt repositories
sudo apt update

# Install Apache, MySQL, PHP
sudo apt install -y apache2 mysql-server php libapache2-mod-php php-mysql

# Clone PHP application from GitHub
git clone https://github.com/laravel/laravel /var/www/html/laravel

# Configure Apache web server
sudo chown -R www-data:www-data /var/www/html/laravel
sudo chmod -R 755 /var/www/html/laravel

# Restart Apache
sudo systemctl restart apache2

# MySQL configuration 
# ...


YAML
- name: Deploy PHP application on Slave node
  hosts: Slave
  become: yes
  tasks:
    - name: Copy and execute bash script
      copy:
        src: deploy_lamp.sh
        dest: /tmp/deploy_lamp.sh
        mode: 0755
      register: deploy_output

    - name: Execute the bash script
      shell: /tmp/deploy_lamp.sh
      register: shell_output

    - name: Verify application accessibility
      uri:
        url: http://{{ ansible_host }}/laravel/public
        return_content: yes
      register: result

- name: Configure cron job
  hosts: Slave
  tasks:
    - name: Add cron job to check uptime
      cron:
        name: Check server uptime
        job: "/usr/bin/uptime >> /var/log/uptime.log"
        minute: 0
        hour: 0

