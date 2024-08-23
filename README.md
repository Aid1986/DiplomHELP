# DiplomHELP

Тема:
Не получается осуществить удаленное подключение по SSH через tf к созданным виртуальным машинам.

Код машинки:

resource "yandex_compute_instance" "bastion" {
  name        = "bastion"
  hostname    = "bastion"
  platform_id = "standard-v2"
  zone        = "ru-central1-d"

  resources {
    cores = 2
    memory = 2
    core_fraction = 5
  }

  boot_disk {
    initialize_params {
      image_id = "fd830gae25ve4glajdsj"
      size = 10
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-public.id
    nat = true
    security_group_ids = [yandex_vpc_security_group.private-group.id, yandex_vpc_security_group.bastion-group.id]
  }

  metadata = {
    user-data = file("./meta.yml")
  }

  scheduling_policy {
    preemptible = true
  }
}

meta.yml-файл, откуда берутся данные для машинки:

users:
- name: admin
  groups: sudo
  shell: /bin/bash
  sudo: ['ALL=(ALL) NOPASSWD:ALL']
  ssh-authorized-keys:
    - ssh-ed25519 AAAAC3...uB702qLtRGf andrey@IT-SUPERROOT

output-данные после создания для inventory.ini

resource "local_file" "ansible-inventory" {
  content  = <<-EOT
    [bastion]
    ${yandex_compute_instance.bastion.fqdn} public_ip=${yandex_compute_instance.bastion.network_interface.0.nat_ip_address} 
    
    [all:vars]
    ansible_ssh_common_args='-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o ProxyCommand="ssh -o StrictHostKeyChecking=no -p 22 -W %h:%p -q admin@${yandex_compute_instance.bastion.network_interface.0.nat_ip_address}"'
    EOT
  filename = "../ansible/inventory.ini"
}

P.S:
image_id - Ubuntu, но пробовал разные версии и даже Debian, проблема не изменилась.
Публичный IP машинки получают, пингуются.
Ключи делал по документации и разными способами, не помогло.

В чем может быть проблема?
