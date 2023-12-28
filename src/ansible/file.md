<!-- toc -->

# 用 ansible 操作文件和 url 

## 将 url 指向的文件下载到指定目录

[get_url_module][1] 模块提供了该功能：

```yaml
- name: Download docker repo
  get_url:
    url: https://download.docker.com/linux/centos/docker-ce.repo
    dest: /etc/yum.repos.d/docker-ce.repo
```

如果 dest 指向特定用户才可以写入的目录，可能需要 [提升用户权限](./privilege.md)。

## 参考

[1]: https://docs.ansible.com/ansible/latest/modules/get_url_module.html#examples "get_url_module"
