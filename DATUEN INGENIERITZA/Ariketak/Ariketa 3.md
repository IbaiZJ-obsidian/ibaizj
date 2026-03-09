
![[7 Datuen Ingeniaritza - 2.3.1 Datu masiboentzako azpiegitura oinarriak - Oinarrizko teknologiak - Kontenedoreak.pdf#page=19]]


```
---

- name: Instalatu Docker LXC kontenedorean

  hosts: webserver

  become: yes

  tasks:

    - name: Instalatu dependentziak

      apt:

        name: [ca-certificates, curl, gnupg]

        update_cache: yes

  

    - name: Deskargatu Docker instalazio scripta

      get_url:

        url: https://get.docker.com

        dest: /tmp/get-docker.sh

        mode: '0755'

  

    - name: Instalatu Docker

      command: sh /tmp/get-docker.sh

  

    - name: Hasi Docker zerbitzua

      systemd:

        name: docker

        enabled: yes

        state: started
```