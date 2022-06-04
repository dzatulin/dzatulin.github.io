---
title: "Ansible CD pipeline with docker"
date: 2022-05-17
categories:
  - Blog
tags:
  - CI/CD
---
Доброго времени суток, хочу показать вам автоматизацию CD с помощью Ansible для деплоя docker контейнера. В роли CI я использую TeamCity который через плагин вызывает Ansible.
Ansible позволяет нам одновременно управлять несколькими узлами или хостами, пример описания хостов:
```
[docker-test]
dockertest-01 ansible_host=10.10.10.1 ansible_user=dev
dockertest-02 ansible_host=10.10.10.2 ansible_user=dev
``` 
Таким образом мы можем развернуть приложение в контейнере на N-количестве серверов. 

Пример Ansible Playbook:
```
---
- hosts: docker-test
  vars: 
      image_name: <<dockerhub-name>>/<< project_name >>
  tasks:
  - name: "INFO: Start deploy"
    slack:
      token: "<< slack_token >>"
      msg: ':partydeploy: Start deploy *<< project_name >>* on *<< inventory_hostname >>* environment:*<< env >>*'
      channel: '<< slack_channel >>'
  - name: "INFO: Check docker version"
    slack:
      token: "<< slack_token >>"
      msg: ':loading: Check docker version on *<< inventory_hostname >>*'
      channel: '<< slack_channel >>'
  - name: "Get docker version"
    shell: "docker -v | cut -d ' ' -f 3 | cut -d ',' -f 1"
    register: version
  - name: Send Docker version
    slack:
      token: "<< slack_token >>"
      msg: ":white_check_mark: Docker version: << version.stdout >> on *<< inventory_hostname >>*"
      channel: '<< slack_channel >>'
  - name: Ensure docker deamon is running
    service:
      name: docker
      state: started
  - name: "INFO: Pull project image"
    slack:
      token: "<< slack_token >>"
      msg: ':loading: pull *<< image_name >>* environment:*<< env >>* image on *<< inventory_hostname >>*'
      channel: '<< slack_channel >>'
  - name: Run deploy script and notification 
    block:
      - name: Pull an actual project image
        docker_image:
          name: "<<image_name>>"
          tag: "<<env>>"
          source: pull
          state: present
          force_source: Yes
        become: yes
        become_method: sudo
      - name: "Get last docker pul"
        shell: "docker images | awk '{print $1,$2,$4,$5,$7}' | awk 'NR==2'"
        register: docker_pull
        become: yes
        become_method: sudo
      - name: Send last docker build
        slack:
          token: "<< slack_token >>"
          msg: ":white_check_mark: Last Pull: <<docker_pull.stdout>> on *<< inventory_hostname >>*"
          channel: '<< slack_channel >>'
      - name: Run app service
        docker_container:
          name: "<< project_name >>"
          image: "<< image_name >>:<< env >>"
          state: started
          recreate: yes
          ports:
          - "<<service_port>>:80"
          log_driver: gelf
          log_options:
            gelf-address: "udp://elk.example.com:224455"
            tag: "<< project_name >>"
        become: yes
        become_method: sudo
      - name: add container to inventory
        add_host:
          name: "<< project_name >>"
          ansible_connection: docker
        changed_when: false
      - name: "INFO: Check if container exist"
        slack:
          token: "<< slack_token >>"
          msg: ':loading: Check containers exist and running on *<< inventory_hostname >>*'
          channel: '<< slack_channel >>'
      - name: Get info on container
        docker_container_info:
          name: "<< project_name >>"
        become: yes
        become_method: sudo
        register: result_service
      - name: Send container status
        slack:
          token: "<< slack_token >>"
          msg: ":docker_run: Conatiner *<< project_name >>* status: *<< 'exists' if result_service.exists else 'does not exist' >>* and *<< result_service.container['State']['Status'] >>* on *<< inventory_hostname >>*"
          channel: '<< slack_channel >>'
    rescue:
      - name: Print when deploy ended with an error
        slack:
          token: "<< slack_token >>"
          msg: ":x: Deploy failed see <https://teamcity.example.com/viewLog.html?buildId=<< buildid >>&buildTypeId=<< buildtype >>&tab=buildLog|TeamCity build log> on *<< inventory_hostname >>*"
          channel: '<< slack_channel >>'
        failed_when: result_service.container.State.Status != "running"
  handlers:
  - name: Restart supervisorctl
    supervisorctl:
      name: all
      state: restarted
```
Такие параметры как:

- project_name     - имя проекта(репозитория)
- env              - ветка
- slack_token      - токен для оповещения в SLACK
- slack_channel    - слак канал 
- buildid          - id сбоки
