1. Создаю вирутальные машины `vagrant up`. Проверяю виртуальные машины `vagrant status`
```
[tesla@ol85 homework05]$ vagrant status
Current machine states:

nfss                      running (virtualbox)
nfsc                      running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```
2. Подключаюсь к NFS серверу, и заходу под su
```
[tesla@ol85 homework05]$ vagrant ssh nfss
[vagrant@nfss ~]$ sudo -i
```
3. Устанавливаю `nfs-utils`
```
yum install nfs-utils
.....
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : 1:nfs-utils-1.3.0-0.68.el7.2.x86_64                          1/2
  Cleanup    : 1:nfs-utils-1.3.0-0.66.el7.x86_64                            2/2
  Verifying  : 1:nfs-utils-1.3.0-0.68.el7.2.x86_64                          1/2
  Verifying  : 1:nfs-utils-1.3.0-0.66.el7.x86_64                            2/2

Updated:
  nfs-utils.x86_64 1:1.3.0-0.68.el7.2

Complete!
```
nfs-utils уже были, обновились
4. Включаю firewall `systemctl enable firewalld --now`
```
[root@nfss ~]# systemctl enable firewalld --now
Created symlink from /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service to /usr/lib/systemd/system/firewalld.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/firewalld.service to /usr/lib/systemd/system/firewalld.service.
```
5. Проверяю его статус
```
[root@nfss ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2022-02-20 16:41:28 UTC; 52s ago
     Docs: man:firewalld(1)
 Main PID: 2911 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─2911 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid

Feb 20 16:41:27 nfss systemd[1]: Starting firewalld - dynamic firewall daemon...
Feb 20 16:41:28 nfss systemd[1]: Started firewalld - dynamic firewall daemon.
Feb 20 16:41:28 nfss firewalld[2911]: WARNING: AllowZoneDrifting is enabled...w.
Hint: Some lines were ellipsized, use -l to show in full.
```
6. Разрешаю доступ к сервисам NFS и перезагружаю правила firewall
```
[root@nfss ~]# firewall-cmd --add-service="nfs3" \
> --add-service="rpc-bind" \
> --add-service="mountd" \
> --permanent
success
[root@nfss ~]# firewall-cmd --reload
success
```
7. Включаю NFS `systemctl enable nfs --now`
```
[root@nfss ~]# systemctl enable nfs --now
Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.
```
8. Проверяю через ss, что нужные порты открыты
```
[root@nfss ~]# ss -tnplu | grep 2049
udp    UNCONN     0      0         *:2049                  *:*
udp    UNCONN     0      0      [::]:2049               [::]:*
tcp    LISTEN     0      64        *:2049                  *:*
tcp    LISTEN     0      64     [::]:2049               [::]:*
[root@nfss ~]# ss -tnplu | grep 20048
udp    UNCONN     0      0         *:20048                 *:*                   users:(("rpc.mountd",pid=3074,fd=7))
udp    UNCONN     0      0      [::]:20048              [::]:*                   users:(("rpc.mountd",pid=3074,fd=9))
tcp    LISTEN     0      128       *:20048                 *:*                   users:(("rpc.mountd",pid=3074,fd=8))
tcp    LISTEN     0      128    [::]:20048              [::]:*                   users:(("rpc.mountd",pid=3074,fd=10))
[root@nfss ~]# ss -tnplu | grep 111
udp    UNCONN     0      0         *:111                   *:*                   users:(("rpcbind",pid=347,fd=6))
udp    UNCONN     0      0      [::]:111                [::]:*                   users:(("rpcbind",pid=347,fd=9))
tcp    LISTEN     0      128       *:111                   *:*                   users:(("rpcbind",pid=347,fd=8))
tcp    LISTEN     0      128    [::]:111                [::]:*                   users:(("rpcbind",pid=347,fd=11))
```
9. Создаю директорию и настраиваю права доступа. Для дальнейшего доступа через nfs
```
[root@nfss ~]# mkdir -p /srv/share/upload
[root@nfss ~]# chown -R nfsnobody:nfsnobody /srv/share
[root@nfss ~]# chmod 0777 /srv/share/upload
```
10. Настриваю директорию в /etc/exports
```
[root@nfss ~]# cat << EOF > /etc/exports
> /srv/share 192.168.50.11/32(rw,sync,root_squash)
> EOF
[root@nfss ~]# cat /etc/exports
/srv/share 192.168.50.11/32(rw,sync,root_squash)
```
11. Выгружаю exports и проверяю
```
[root@nfss ~]# exportfs -r
[root@nfss ~]# exportfs -s
/srv/share  192.168.50.11/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```
12. Настройки сервера nfs выполнены, подключаюсь к клиенту nfs, и захожу под root
```
[tesla@ol85 homework05]$ vagrant ssh nfsc
[vagrant@nfsc ~]$ sudo -i
```
13. Устанавливаю/обновляю nfs-utils
```
[root@nfsc ~]# yum install nfs-utils
...
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : 1:nfs-utils-1.3.0-0.68.el7.2.x86_64                                                                      1/2
  Cleanup    : 1:nfs-utils-1.3.0-0.66.el7.x86_64                                                                        2/2
  Verifying  : 1:nfs-utils-1.3.0-0.68.el7.2.x86_64                                                                      1/2
  Verifying  : 1:nfs-utils-1.3.0-0.66.el7.x86_64                                                                        2/2

Updated:
  nfs-utils.x86_64 1:1.3.0-0.68.el7.2

Complete!
```
14.
