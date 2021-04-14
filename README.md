## 一键生成 FISCO-BCOS 企业级架构部署
&emsp;&emsp;本项目为区块链开源项目 [FISCO-BCOS](https://github.com/FISCO-BCOS/FISCO-BCOS) 提供了自动化生成企业级配置文件的 ansible playbook。2 群组 3 机构 6 节点的环境，可以在 30 秒内（不包括下载时间）生成配置，极大简化了部署难度，避免了手工配置容易发生的错误。

&emsp;&emsp;实验环境相关信息：
- 机器配置
  - CPU: Intel(R) Xeon(R) CPU E5-2680 v2 @ 2.80GHz * 2
  - RAM: DDR3（1333MHz）8GB * 12
  - HD: WD 4TB 7200rpm
  - OS: Manjaro Linux(Kernel 5.10.2)
- 操作系统
  - CentOS 8
  - Ubuntu 16.04, Ubuntu 18.04, Ubuntu 20.04

## 依赖软件
- Ansible: 2.10.3  

  **如果已经安装了 ansible，但版本低于 2.10 的，请运行 `pip3 install --user -U ansible` 升级到最新版。目前确认 2.9 或以下版本，不兼容部分语法。**

- Python: python3

## 注意事项
- 项目仍在重度开发中，如果你在使用时遇到问题，请先 git pull 更新再试。如果问题依然存在，请提交 issue 以便及时得到解决。非常感谢你的使用。 
- 重要提醒：已上线的联盟链，请务必备份并保管好 deploy 目录！

## 已实现的功能
1. 多群组多机构多节点的联盟链初始化 （ 已验证 3 群组 5 机构 50 节点的部署文件 )
2. 可在已初始化的机构里新增节点。 
3. 可指定 generator，ficso-bcos 和 console 的版本。
4. 可定义是否启用 console，sdk，国密。
5. 可生成部署架构图，方便部署前后的核对。 
6. 已初始化联盟链新增新群组

## 计划实现的功能
1. 支持对节点配置文件 config.ini 的定制。
2. 支持使用机构私有证书来签发链证书。
3. 支持在已初始化的联盟链中增加机构。

## 设计思路
1. 参考[官方文档](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/enterprise_tools/tutorial_detail_operation.html)，示例架构 2 群组 3 机构 6 节点。
1. 所有文件在本地目录生成。方便分发给不同的真实机构，由机构的运维人员自行部署启动。
1. 更详细的内容请看[《Ansible for FISCO BCOS，企业级部署的速度与激情》](https://mp.weixin.qq.com/s/YKwcfCGx_1TAu_hoXnqrMw)

## 操作指导

### 1.  联盟链初始化

#####  1.1 安装依赖
```
pip3 install --user -U -r requirements.txt
```

##### 1.2 复制项目
&emsp;&emsp;复制一份 inventory 配置。假设新环境是 'my_inventory'。

```shell
cp -R inventories/sample inventories/my_inventory
```

##### 1.3 修改 init.yml 配置文件

```shell
nano inventories/my_inventory/group_vars/all/init.yml

# 文件配置样例如下

# 变量注释
# name:           （必填）机构名称
# create_genesis: 是否生成创世区块，同组(根据 main_group_id 判断)必须有且只有 1 个创世区块机构。（不设置则默认为 false）
# nodes:          （必填）机构各节点连接信息。格式 <ip>:<节点数>。如果只填写 IP，则默认为 1 个节点。同 IP 多节点会自动递增对应端口。
# main_group_id:  （必填）群组编号
# extra_group_id: （可选）额外要加入的目标群组编号

agencies:
 - name: A
   create_genesis: true
   nodes:
     - 172.17.8.101:5
     - 172.17.8.102:5
   main_group_id:
     - 1
   extra_group_id:
     - 2
 - name: B
   nodes:
     - 172.17.8.103:5
     - 172.17.8.104:5
   main_group_id:
     - 1
 - name: C
   create_genesis: true
   nodes:
     - 172.17.8.105:5
     - 172.17.8.106:5
   main_group_id:
     - 2

# true 表示使用国密模式
fisco_gm_enabled: true
```

请根据文件头部的变量注释，编排好目标配置的信息。更多变量设置请查看 `roles/fisco_bcos/defaults/main.yml` 文件。

##### 1.4 初始化联盟链  
###### 1.4.1 验证配置
&emsp;&emsp;执行以下命令，生成架构图，来确认配置是否有问题。

```
ansible-playbook -i inventories/my_inventory/hosts.ini fisco_bcos.yml -t archimate
```

###### 温馨提示
- 报错处理
如果执行时无法显示中文，或 python 报错 `UnicodeEncodeError: 'ascii' codec can't encode characters`，请执行 `export LC_ALL=C.UTF-8` 后再试。
- 操作系统兼容
由于精力有限，无法覆盖多个 Linux 发行版。如果生成的图片不能正确显示中文，请根据你的 Linux 发行版，查找 Java 显示中文的解决方法。欢迎提交你的解决方案到我们的 issue 列表里。**

###### 1.4.2 验证输出
输出结果类似于：

```
TASK [fisco_bcos : 生成图片文件] ***********************************************************************************************
Monday 07 December 2020  22:17:04 +0800 (0:00:00.855)       0:00:05.108 *******
ok: [localhost] => (item=/home/haibin/Workspaces/ansible-for-fisco-bcos/inventories/v2.7.0/deploy/archimate/agency_C_topo.png)
ok: [localhost] => (item=/home/haibin/Workspaces/ansible-for-fisco-bcos/inventories/v2.7.0/deploy/archimate/agency_B_topo.png)
ok: [localhost] => (item=/home/haibin/Workspaces/ansible-for-fisco-bcos/inventories/v2.7.0/deploy/archimate/agency_A_topo.png)
ok: [localhost] => (item=/home/haibin/Workspaces/ansible-for-fisco-bcos/inventories/v2.7.0/deploy/archimate/chain_topo.png)
```

###### 1.4.3 执行部署
&emsp;&emsp;查看图片，确认没问题后，可以开始生成部署文件。


```
ansible-playbook -i inventories/my_inventory/hosts.ini fisco_bcos.yml
```

###### 1.4.4 检查执行结果
- 执行过程中出现的 [WARNING] 信息可以忽略。

- 如无意外，你会在 `inventories/my_inventory/deploy` 目录下找到相关的配置内容。例如：

```
find inventories/my_inventory/deploy/agency_*/fisco* -type d -name "node_*" | sort -n

inventories/my_inventory/deploy/agency_A/fisco_deploy_agency_A/node_172.17.8.101_30300
inventories/my_inventory/deploy/agency_A/fisco_deploy_agency_A/node_172.17.8.102_30300
inventories/my_inventory/deploy/agency_B/fisco_deploy_agency_B/node_172.17.8.103_30300
inventories/my_inventory/deploy/agency_B/fisco_deploy_agency_B/node_172.17.8.104_30300
inventories/my_inventory/deploy/agency_C/fisco_deploy_agency_C/node_172.17.8.105_30300
inventories/my_inventory/deploy/agency_C/fisco_deploy_agency_C/node_172.17.8.106_30300
```

- 查看 `inventories/my_inventory/deploy/node_list.yml` 文件，内容类似于：

```
# 机构名称:群组编号:节点IP:P2P端口:Channel端口:RPC端口:额外目标群组编号（没有则为 0）

node_list:
  - A:1:172.17.8.101:30300:20200:8545:[2]
  - A:1:172.17.8.102:30300:20200:8545:[2]
  - B:1:172.17.8.103:30300:20200:8545:[0]
  - B:1:172.17.8.104:30300:20200:8545:[0]
  - C:2:172.17.8.105:30300:20200:8545:[0]
  - C:2:172.17.8.106:30300:20200:8545:[0]
  - C:2:172.17.8.107:30300:20200:8545:[0]
```

###### 1.4.5 节点部署
- 确保相关的 IP 和端口能访问即可
- 把相关文件夹上传到对应的服务器上，执行启动命令即可。

### 2. 已有机构扩展节点

&emsp;&emsp;**目前仅支持在 init.yml 配置中，给已有的机构增加节点，不支持 agencies 里增加机构。**

###### 2.1 修改配置
- 假设第一次初始化的配置是

```
	agencies:
	 - name: A
	   create_genesis: true
	   nodes:
	     - 172.17.8.101
	     - 172.17.8.102
	   main_group_id: 
	     - 1
	   extra_group_id:
	     - 2
	 - name: B
	   nodes:
	     - 172.17.8.103
	     - 172.17.8.104
	   main_group_id: 
	     - 1
	 - name: C
	   create_genesis: true
	   nodes:
	     - 172.17.8.105
	     - 172.17.8.106
	   main_group_id: 
	     - 2
```

- 假设要给机构 A 的 172.17.8.101 和 172.17.8.102 服务器，分别增加至 5 个节点，就把 `172.17.8.101` 改成 `172.17.8.101:5`，`172.17.8.102` 改成 `172.17.8.102:5`。也可以增加 IP 和节点，例如 `172.17.8.110:5`：

```
	agencies:
	 - name: A
	   create_genesis: true
	   nodes:
	     - 172.17.8.101:5
	     - 172.17.8.102:5
	     - 172.17.8.110:5
	   main_group_id: 1
	   extra_group_id:
	     - 2
```

###### 2.2 生成新机构

```
ansible-playbook -i inventories/my_inventory/hosts.ini fisco_bcos.yml
```

&emsp;&emsp;执行完成后，在对应的机构目录下，你可以看到类似 `fisco_deploy_agency_A_expand_1917645e6744e1360fba72fa4cf8cc47` 这样的目录，就是新增的节点配置了。

&emsp;&emsp;把相关节点文件夹传送到对应 IP 的服务器上，通过 [控制台](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/console_of_java_sdk.html)、[SDK](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/sdk/index.html) 或 [WeBase 管理平台](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/webase/webase.html) 进行管理。

### 3. 已初始化联盟链新增群组
###### 3.1 修改配置
- 假设第一次初始化的配置是

```
	agencies:
	 - name: A
	   create_genesis: true
	   nodes:
	     - 172.17.8.101
	     - 172.17.8.102
	   main_group_id: 
	     - 1
	   extra_group_id:
	     - 2
	 - name: B
	   nodes:
	     - 172.17.8.103
	     - 172.17.8.104
	   main_group_id: 
	     - 1
	 - name: C
	   create_genesis: true
	   nodes:
	     - 172.17.8.105
	     - 172.17.8.106
	   main_group_id: 
	     - 2
```

- 假设要给机构 A，B 新增群组 4

```
	agencies:
	 - name: A
	   create_genesis: true
	   nodes:
	     - 172.17.8.101
	     - 172.17.8.102
	   main_group_id: 
	     - 1
	     - 4
	   extra_group_id:
	     - 2
	- name: B
	   nodes:
	     - 172.17.8.103
	     - 172.17.8.104
	   main_group_id: 
	     - 1
	     - 4
```
###### 3.2 生成新群组
&emsp;&emsp;在 ansible-for-fisco-bcos 目录中执行如下命令
```
ansible-playbook -i inventories/my_inventory/hosts.ini fisco_bcos.yml
```
###### 3.3 查看执行结果
&emsp;&emsp;到机构 A 和 B 的节点目录中查看是否生成群组 4 的群组文件 ( group.4.genesis, group.4.ini)

## 如何参与？
我们欢迎大家在遵循 Apache-2.0 的前提下，一起来完善代码。请 fork 本项目，提交你的 pull request。
同时，我们需要定义一些代码规范。

1. 请复制一份 inventory/sample，重命名在同级目录中。避免 sample 目录文件冲突。
1. 每个 task 必须有 name，用中文注释。
1. 尽可能的使用 ansible 模块，不鼓励直接使用 shell 命令。


## 贡献者

本项目由 [FISCO BCOS 自动化工具兴趣小组](https://github.com/FISCO-BCOS/Wiki/blob/master/FISCO%20BCOS%E8%87%AA%E5%8A%A8%E5%8C%96%E5%B7%A5%E5%85%B7%E7%A0%94%E5%8F%91%E5%85%B4%E8%B6%A3%E5%B0%8F%E7%BB%84README.md) 成员共同编写。
