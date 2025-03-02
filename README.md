# Домашнее задание к занятию 1 «Введение в Ansible»

## Подготовка к выполнению

1. Установите Ansible версии 2.10 или выше.
2. Создайте свой публичный репозиторий на GitHub с произвольным именем.
3. Скачайте [Playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.

```
thrsnknwldgthtsntpwr@ubnt:~/NETOLOGY/hw-ansible-01$ ls -al
total 24
drwxrwxr-x 4 thrsnknwldgthtsntpwr thrsnknwldgthtsntpwr 4096 мар  2 10:09 .
drwxrwxr-x 8 thrsnknwldgthtsntpwr thrsnknwldgthtsntpwr 4096 мар  2 10:09 ..
drwxrwxr-x 8 thrsnknwldgthtsntpwr thrsnknwldgthtsntpwr 4096 мар  2 10:10 .git
drwxrwxr-x 4 thrsnknwldgthtsntpwr thrsnknwldgthtsntpwr 4096 мар  2 10:09 playbook
-rw-rw-r-- 1 thrsnknwldgthtsntpwr thrsnknwldgthtsntpwr 5178 мар  2 10:12 README.md
thrsnknwldgthtsntpwr@ubnt:~/NETOLOGY/hw-ansible-01$ ansible --version
ansible 2.10.8
  config file = None
  configured module search path = ['/home/thrsnknwldgthtsntpwr/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.10.12 (main, Feb  4 2025, 14:57:36) [GCC 11.4.0]

```

## Основная часть

1. Попробуйте запустить playbook на окружении из `test.yml`, зафиксируйте значение, которое имеет факт `some_fact` для указанного хоста при выполнении playbook.
```
TASK [Print fact] ************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": 12
```
2. Найдите файл с переменными (group_vars), в котором задаётся найденное в первом пункте значение, и поменяйте его на `all default fact`.
3. Воспользуйтесь подготовленным (используется `docker`) или создайте собственное окружение для проведения дальнейших испытаний.
Запустил два контейнера
```
docker run -d --name centos7 centos:cento7 /bin/bash -c "sleep 1d"
docker run -d --name ubuntu ubuntu:latest /bin/bash -c "sleep 1d"
```
4. Проведите запуск playbook на окружении из `prod.yml`. Зафиксируйте полученные значения `some_fact` для каждого из `managed host`.
Были проблемы с версиями питона и ансибла. Долго перебирал подходящие, остановился на последней версии ansible 3.17, centos8 с Python3.12 и ubuntu 24.04

```
TASK [Print fact] ************************************************************************************************************************************************************************************************
task path: /home/thrsnknwldgthtsntpwr/NETOLOGY/hw-ansible-01/playbook/site.yml:8
redirecting (type: connection) ansible.builtin.docker to community.docker.docker
ok: [centos7] => {
    "msg": "el"
}
redirecting (type: connection) ansible.builtin.docker to community.docker.docker
ok: [ubuntu] => {
    "msg": "deb"
}
```
5. Добавьте факты в `group_vars` каждой из групп хостов так, чтобы для `some_fact` получились значения: для `deb` — `deb default fact`, для `el` — `el default fact`.
```
thrsnknwldgthtsntpwr@ubnt:~/NETOLOGY/hw-ansible-01$ cat playbook/group_vars/deb/examp.yml 
---
  some_fact: "deb default fact"
thrsnknwldgthtsntpwr@ubnt:~/NETOLOGY/hw-ansible-01$ cat playbook/group_vars/el/examp.yml 
---
  some_fact: "el default fact"thrsnknwldgthtsntpwr@ubnt:~/NETOLOGY/hw-ansible-01$ 
```
6.  Повторите запуск playbook на окружении `prod.yml`. Убедитесь, что выдаются корректные значения для всех хостов.
```
TASK [Print fact] ************************************************************************************************************************************************************************************************
task path: /home/thrsnknwldgthtsntpwr/NETOLOGY/hw-ansible-01/playbook/site.yml:8
redirecting (type: connection) ansible.builtin.docker to community.docker.docker
ok: [centos7] => {
    "msg": "el default fact"
}
redirecting (type: connection) ansible.builtin.docker to community.docker.docker
ok: [ubuntu] => {
    "msg": "deb default fact"
}
```
7. При помощи `ansible-vault` зашифруйте факты в `group_vars/deb` и `group_vars/el` с паролем `netology`.
```
thrsnknwldgthtsntpwr@ubnt:~/NETOLOGY/hw-ansible-01$ ansible-vault encrypt playbook/group_vars/deb/examp.yml
thrsnknwldgthtsntpwr@ubnt:~/NETOLOGY/hw-ansible-01$ ansible-vault encrypt playbook/group_vars/el/examp.yml
```
8. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь в работоспособности.
```
thrsnknwldgthtsntpwr@ubnt:~/NETOLOGY/hw-ansible-01$ ansible-playbook playbook/site.yml -i playbook/inventory/prod.yml --ask-vault-pass
Vault password: 

PLAY [Print os facts] ********************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************
[WARNING]: Platform linux on host ubuntu is using the discovered Python interpreter at /usr/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [ubuntu]
[WARNING]: Platform linux on host centos7 is using the discovered Python interpreter at /usr/local/bin/python3.12, but future installation of another Python interpreter could change the meaning of that path.
See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [centos7]

TASK [Print OS] **************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP *******************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```
9. Посмотрите при помощи `ansible-doc` список плагинов для подключения. Выберите подходящий для работы на `control node`.
```
ansible-doc -t connection -l
```
Подходящий, наверное, ansible.builtin.local

10. В `prod.yml` добавьте новую группу хостов с именем  `local`, в ней разместите localhost с необходимым типом подключения.
```
  local:
    hosts:
      localhost:
        ansible_connection: ansible.builtin.local
```
11. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь, что факты `some_fact` для каждого из хостов определены из верных `group_vars`.
Добавил каталог local для группы хостов. Создал examp.yml с содержимым
```
---
  some_fact: 'localhost default fact'
```

Результат выполнения `ansible-playbook playbook/site.yml -i playbook/inventory/prod.yml --ask-vault-pass`
```
TASK [Print fact] ************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}
ok: [localhost] => {
    "msg": "localhost default fact"
}
```
12. Заполните `README.md` ответами на вопросы. Сделайте `git push` в ветку `master`. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым `playbook` и заполненным `README.md`.
13. Предоставьте скриншоты результатов запуска команд.