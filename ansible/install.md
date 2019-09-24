<!-- toc -->

# 用 ansible 安装软件

ansible 提供了 [yum-module][1]、[apt-module][2]、[apk-module][3]等模块，用来在不同的操作系统上安装软件。

可以用下面的方法判断目标系统类型，从而导入不同的文件：

```yaml
- name: install dependent packages
  import_tasks: centos.yml
  when: ansible_distribution == "CentOS"
```

用 setup 模块查看 ansible 支持的内置变量：

```sh
$ ansible -i inventories/production/hosts 10.19.11.7  -m setup
"ansible_distribution": "CentOS",
"ansible_distribution_file_parsed": true,
"ansible_distribution_file_path": "/etc/redhat-release",
"ansible_distribution_file_variety": "RedHat",
"ansible_distribution_major_version": "7",
"ansible_distribution_release": "Core",
"ansible_distribution_version": "7",
```

## 在 CentOS 上用 yum 安装

```yaml
- name: Install docker
  notify: Start docker
  yum:
    name: docker-ce
    state: installed
```

其中 notify 指定的安装完成后执行的 handler/main.yml 中的同名操作：

```yaml
- name: Start docker
  systemd:
    name: docker
    state: started
    daemon_reload: yes
    enabled: yes
```

## 用 pip 安装 python 包

[pip-module][4] 调用 pip 命令安装 python 包：

```yaml
- name: install docker python lib
  pip:
    name: docker
```


## 参考

[1]: https://docs.ansible.com/ansible/latest/modules/yum_module.html#yum-module "yum-module"
[2]: https://docs.ansible.com/ansible/latest/modules/apt_module.html#apt-module "apt-module"
[3]: https://docs.ansible.com/ansible/latest/modules/apk_module.html#apk-module "apk-module"
[4]: https://docs.ansible.com/ansible/latest/modules/pip_module.html#pip-module "pip-module"
