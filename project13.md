## Continued from Project 12

## Step 1: Introducing Dynamic Assignments to your structure

### Create a new branch branch in complete=ansible-automation repo and call it dynamic-assignments and checkout into the branch

### Create a folder called called dynamic-assignments, nside this folder, create a new file and name it env-vars.yml. We will instruct site.yml to include this playbook later.

### Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment.
### For this reason, we will now create a folder to keep each environment’s variables file. Therefore, create a new folder env-vars, then for each environment, create new YAML files which we will use to set variables.

### Your layout should now look like this.

![](./project13%20images/new%20layout.png)

### Now paste the instruction below into the env-vars.yml file.

```
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - uat.yml
            - dev.yml
            - stage.yml
            - prod.yml
            
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always
```

### Update site.yml file to make use of the dynamic assignment. site.yml should look like this

![](./project13%20images/site_yml.png)

## Step 2: Community roles

### Now it is time to create a role for MySQL database – it should install the MySQL package. You can download this role from the community instead of reinventing the wheel

### Navigate to your roles folder and install the chosen MySql role

`ansible-galaxy install geerlingguy.mysql`

### Rename it to mysql

`mv geerlingguy.mysql mysql`


### We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:

- Nginx
- Apache

`ansible-galaxy install geerlingguy.nginx`

`ansible-galaxy install geerlingguy.apache`

### Rename them to nginx and apache

![](./project13%20images/renamed%20roles.png)

### Configure the roles

### Note for the loadbalancer roles:

- Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one – this is where you can make use of variables.

- Declare a variable in defaults/main.yml file inside the Nginx and Apache roles. Name each variables enable_nginx_lb and enable_apache_lb respectively.

- Set both values to false like this enable_nginx_lb: false and enable_apache_lb: false.

- Declare another variable in both roles load_balancer_is_required and set its value to false as well

- Update both assignment and site.yml files respectively


### Create database.yml and loadbalancer.yml files in static-assignments folder where you will call your roles and update the uat inventory as well

### Your loadbalancer.yml file:

```
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
```

![](./project13%20images/lb_yml.png)

### Your site.yml, you reference your loadbalancer playbook:

```
 - name: Loadbalancers assignment
   hosts: lb
     - import_playbook: ../static-assignments/loadbalancers.yml
   when: load_balancer_is_required
```

### Now you can make use of env-vars\uat.yml file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.

### You will activate load balancer, and enable nginx by setting these in the respective environment’s env-vars file.

```
enable_nginx_lb: true
load_balancer_is_required: true
```


![](./project13%20images/enable%20nginx.png)

### uat inventory should be this:

![](./project13%20images/uat_inventory.png)

### Your site.yml file should be this:

![](./project13%20images/final%20site_yml.png)

![](./project13%20images/database%20role.png)

![](./project13%20images/nginx%20role.png)

### The same must work with apache LB, so you can switch it by setting respective environmental variable to true and other to false.

