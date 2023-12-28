<!-- toc -->

# 使用 ansible 操作时，用户权限相关的设置

使用 ansible 操作时，有时候需要切换到特权账号，比如 root。

## 切换到特权用户（root）

使用登录用户 `ops` 操作时，因为目标权限设置，ops 用户没有 /etc 目录的写入权限：

```sh
$ ansible -i inventories/production/hosts -u ops  all   -m command  -a "touch /etc/a"
10.19.11.7 | FAILED | rc=1 >>
touch: cannot touch ‘/etc/a’: Permission deniednon-zero return code

10.19.117.30 | FAILED | rc=1 >>
touch: cannot touch ‘/etc/a’: Permission deniednon-zero return code
```

依然使用 ops 用户，加上参数 `-b --become-user=root` 提升到 root 用户，提升方法用 `--become-method` 指定，默认是 sudo：

```sh
$ ansible -i inventories/production/hosts -u ops -b --become-user=root all   -m command  -a "touch /etc/a"
10.19.11.7 | CHANGED | rc=0 >>
10.19.117.30 | CHANGED | rc=0 >>
```

使用 playbook 时，也可以在 playbook 文件中设置：

```yaml
- hosts: all
  gather_facts: no
  become: true
  become_user: root
```
