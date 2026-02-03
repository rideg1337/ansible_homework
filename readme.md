
# Ansible Homewrok

## A feladat:
Telepitenunk kell egy nginx webszervert majd azt automatikus inditasba kell rakni.

### Playbook magyarazat:


Definialjuk a hosztokat, root-a valunk es felparameterezzuk valtozoval a jinja template-et
```yml
- name: Do My Homework
  hosts: web
  become: yes
  vars:
    - my_name: "<h1>Rideg Zsolt</h1>"
```

Mivel ez Rocky Linux alatt fut, amin alapbol bevan kapcsolva egy firewalld, igy szukseges a service-t/portot engedelyeznunk, a mi esetunkben ez a http service ami defaultban a 80-as porton figyel.



```yml
- name: Firewalld enable http
    ansible.posix.firewalld:
      service: http
      permanent: true
      state: enabled
      immediate: true
```

---
Itt ha kulso gepen szeretnenk elerni a service-t akkor a 8888-as portal kell probalkoznunk mivel az mutat a VM gep 80-as portjara

#### Vagrantfile
```ruby
web.vm.network "forwarded_port", guest:80, host:8888
```
---

Feltepeitjuk az nginx-et es automatikus indulasba tesszuk, ami azt jelent hogy reboot utan automatikusan elindul

```yml
- name: Install nginx
    package:
      name: nginx
      state: present

  - name: Enable nginx permanent
    ansible.builtin.systemd:
      name: nginx
      state: started
      enabled: yes

```

Itt pedig atszerkeztjuk az nginx-t index.html-el. Megadjuk a forrast ami ezesetben```index.j2``` illetve a destination-t ahol az ```index.html``` talalhato.
Mivel atszerkeztettuk az ```index.html``` fajlt, igy szukseges ujrainditanunk a service-t amit handler segitsegevel a legegyszerubb. itt a ```notify: Restart nginx``` triggereli a handler-t ami ujrainditja a service-t

```yml
  - name: Edit nginx
    template:
      src: index.j2
      dest: /usr/share/nginx/html/index.html
    notify: Restart nginx

  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

---

### Inventory magyarazat:

```yml
# A web csoporthoz tartozo gep(ek)
[web] 
# Itt lehet web mivel az etc/hosts-ban megadtuk a nevfeloldast de lehet ip cim is
rocky9 ansible_host=web 
#SSH port a celgepen
ansible_port=22
# Melyk felhasznalo es jelszo kombinacioval csatlakozunk a celgepre 
# (celgepen leteznie kell a felhasznalonak)
ansible_user=homework
ansible_ssh_pass=homework
```

---

### Jinja template magyarazat:

Ez egy egyszeru HTML oldal, itt a lenyeg a vars-oknal talalhato.
Lenyegeben a playbook ugyanezt felmasolja csak ahol megadtuk a valtozokat ott fogja a playbook behelyettesiteni:

- ```{{ ansible_hostname }}``` - ez a hostname a celgepen (gather facts)
- ```{{ my_name }}``` - ezt definialtuk a playbookban ``` - my_name: "<h1>Rideg Zsolt</h1>"```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ ansible_hostname }} - Nginx</title>
</head>
<body>
        {{ my_name }}
</body>
</html>
```