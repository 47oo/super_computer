# Lustre Install

## lustre 安装方式
lustre提供了两种安装方式
  - 已经编译好的二进制包安装
  - 编译源码后进行安装

**[lustre下载地址](https://downloads.whamcloud.com/public/lustre/latest-release/)**
## 安装环境
1.  rehat7.3 or centos7.3
2.  lustre版本2.10.4
3.  使用root帐号
4. 禁止SELinux (Lustre服务节点) 将`/etc/selinux/config`中的`SELINUX=enforcing`变为`SELINUX=disabled`，修改完保存退出后，需要重启系统

## 带有lustre补丁的linux内核安装

1. 由于不带建议自己编写lustre内核，建议直接下载lustre内核相关文件来使用
2. **备注**：如果不使用带有lustre补丁的内核，大部分情况下编译lustre文件会报错的，内核下载地址：[带有kernel的包便是需要下载的包](https://downloads.whamcloud.com/public/lustre/latest-release/el7/server/RPMS/x86_64/)
3. 下载对应的rpm包后，进行安装，这其中，可能会出现缺少依赖，自行配置好yum源进行安装对应依赖即可
  ```shell
      # 可能会出现文件冲突，可以直接加force参数替换掉原有文件
    lustre#：rpm -ivh --force *.rpm
    lustre#：reboot
  ```
4. 获取lustre源码,生成响应初始配置
  ```shell
  git clone git://git.hpdd.intel.com/fs/lustre-release.git
  cd lustre-release
  sh autogen.sh`
  ```
  正常输出如下：
  ```shell
  configure.ac:10: installing 'config/config.guess'
  configure.ac:10: installing 'config/config.sub'
  configure.ac:12: installing 'config/install-sh'
  configure.ac:12: installing 'config/missing'
  libcfs/libcfs/autoMakefile.am: installing 'config/depcomp'
  ```
5. 或者[从这里](https://downloads.whamcloud.com/public/lustre/latest-release/el7/server/SRPMS/)获取对应源码，并使用`rpm -ivh lustre*.rpm` 进行安装，默认结果会存在/root/rpmbuild

6. 现在对lustre源码进行编译即可
  ```shell
  ./configure && make rpms
  ```
    编译成功的话会出现如下文件：
  ```shell
  ..
Wrote: /home/build/kernel/rpmbuild/SRPMS/kernel-3.10.0-514.16.1.el7_lustre.src.rpm
Wrote: /home/build/kernel/rpmbuild/RPMS/x86_64/kernel-3.10.0-514.16.1.el7_lustre.x86_64.rpm
Wrote: /home/build/kernel/rpmbuild/RPMS/x86_64/kernel-headers-3.10.0-514.16.1.el7_lustre.x86_64.rpm
Wrote: /home/build/kernel/rpmbuild/RPMS/x86_64/kernel-debuginfo-common-x86_64-3.10.0-514.16.1.el7_lustre.x86_64.rpm
Wrote: /home/build/kernel/rpmbuild/RPMS/x86_64/perf-3.10.0-514.16.1.el7_lustre.x86_64.rpm
Wrote: /home/build/kernel/rpmbuild/RPMS/x86_64/perf-debuginfo-3.10.0-514.16.1.el7_lustre.x86_64.rpm
Wrote: /home/build/kernel/rpmbuild/RPMS/x86_64/python-perf-3.10.0-514.16.1.el7_lustre.x86_64.rpm
Wrote: /home/build/kernel/rpmbuild/RPMS/x86_64/python-perf-debuginfo-3.10.0-514.16.1.el7_lustre.x86_64.rpm
Wrote: /home/build/kernel/rpmbuild/RPMS/x86_64/kernel-tools-3.10.0-514.16.1.el7_lustre.x86_64.rpm
Wrote: /home/build/kernel/rpmbuild/RPMS/x86_64/kernel-tools-libs-3.10.0-514.16.1.el7_lustre.x86_64.rpm
Wrote: /home/build/kernel/rpmbuild/RPMS/x86_64/kernel-tools-libs-devel-3.10.0-514.16.1.el7_lustre.x86_64.rpm
Wrote: /home/build/kernel/rpmbuild/RPMS/x86_64/kernel-tools-debuginfo-3.10.0-514.16.1.el7_lustre.x86_64.rpm
Wrote: /home/build/kernel/rpmbuild/RPMS/x86_64/kernel-devel-3.10.0-514.16.1.el7_lustre.x86_64.rpm
Wrote: /home/build/kernel/rpmbuild/RPMS/x86_64/kernel-debuginfo-3.10.0-514.16.1.el7_lustre.x86_64.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.F7X9cL
 + umask 022
 + cd /home//build/kernel/rpmbuild/BUILD
 + cd kernel-3.10.0-514.16.1.el7
 + rm -rf /home/build/kernel/rpmbuild/BUILDROOT/kernel-3.10.0-514.16.1.el7_lustre.x86_64
 + exit 0  
  ```
9. 安装针对Lustre的e2fsprogs
  - [下载地址](https://downloads.hpdd.intel.com/public/e2fsprogs/latest/el7/RPMS/x86_64/)
  - yum源配置，二选一即可
    ```shell
    cat << EOF >> /etc/yum.repos.d/e2fsprogs.repo
    [e2fsprogs-el7-x86_64]
    name=e2fsprogs-el7-x86_64
    baseurl=https://downloads.hpdd.intel.com/public/e2fsprogs/latest/el7/
    enabled=1
    priority=1
    EOF
    yum update e2fsprogs
    ```
10. 安装编译好的rpm包
  ```shell
  rpm -ivh *.rpm
  ```
11. 设置并加载Lustre模块，并启动Lustre服务
  ```shell
  echo 'options lnet networks=tcp0(eth0)' > /etc/modprobe.d/lustre.conf
depmod -a
modprobe lustre
systemctl restart lustre

  ```

12. 测试运行
  ```shell
  /usr/lib64/lustre/tests/llmount.sh
  ```
  服务正常的输出

```shell
  Stopping clients: onyx-21vm8.onyx.hpdd.intel.com /mnt/lustre (opts:)
Stopping clients: onyx-21vm8.onyx.hpdd.intel.com /mnt/lustre2 (opts:)
Loading modules from /usr/lib64/lustre/tests/..
detected 1 online CPUs by sysfs
libcfs will create CPU partition based on online CPUs
debug=vfstrace rpctrace dlmtrace neterror ha config                   ioctl super lfsck
subsystem_debug=all
gss/krb5 is not supported
Formatting mgs, mds, osts
Format mds1: /tmp/lustre-mdt1
Format ost1: /tmp/lustre-ost1
Format ost2: /tmp/lustre-ost2
Checking servers environments
Checking clients onyx-21vm8.onyx.hpdd.intel.com environments
Loading modules from /usr/lib64/lustre/tests/..
detected 1 online CPUs by sysfs
libcfs will create CPU partition based on online CPUs
debug=vfstrace rpctrace dlmtrace neterror ha config                   ioctl super lfsck
subsystem_debug=all
gss/krb5 is not supported
Setup mgs, mdt, osts
Starting mds1:   -o loop /tmp/lustre-mdt1 /mnt/lustre-mds1
Commit the device label on /tmp/lustre-mdt1
Started lustre-MDT0000
Starting ost1:   -o loop /tmp/lustre-ost1 /mnt/lustre-ost1
Commit the device label on /tmp/lustre-ost1
Started lustre-OST0000
Starting ost2:   -o loop /tmp/lustre-ost2 /mnt/lustre-ost2
Commit the device label on /tmp/lustre-ost2
Started lustre-OST0001
Starting client: onyx-21vm8.onyx.hpdd.intel.com:  -o user_xattr,flock onyx-21vm8.onyx.hpdd.intel.com@tcp:/lustre /mnt/lustre
UUID                   1K-blocks        Used   Available Use% Mounted on
lustre-MDT0000_UUID       125368        1736      114272   1% /mnt/lustre[MDT:0]
lustre-OST0000_UUID       350360       13492      309396   4% /mnt/lustre[OST:0]
lustre-OST0001_UUID       350360       13492      309396   4% /mnt/lustre[OST:1]

filesystem_summary:       700720       26984      618792   4% /mnt/lustre

Using TIMEOUT=20
seting jobstats to procname_uid
Setting lustre.sys.jobid_var from disable to procname_uid
Waiting 90 secs for update
Updated after 7s: wanted 'procname_uid' got 'procname_uid'
disable quota as required
```
