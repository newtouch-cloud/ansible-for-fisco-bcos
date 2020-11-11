# 一键生成 FISCO-BCOS 企业级架构部署
本项目由[上海新致软件技术有限公司](http://www.newtouch.com) NS1 部门研发，为区块链开源项目 [FISCO-BCOS](http://www.fisco-bcos.org/) 提供了自动化生成企业级配置文件的 ansible-playbook。2 群组 3 机构 6 节点的环境，可以在 30 秒内（除下载时间）生成配置，极大简化了部署难度，避免了手工配置容易发生的错误。

# 已实现的功能
1. 多群组多机构多节点的联盟链初始化配置。目前测试生成 3 群组 5 机构 50 节点的部署文件是没问题的。
1. 已初始化的机构，新增节点。
1. 支持国密版。

# 计划实现的功能
1. 支持对节点配置文件 config.ini 的定制。
1. 支持使用机构私有证书来签发链证书。
1. 支持在已初始化的联盟链中增加机构。

## 设计思路
1. 参考[官方文档](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/enterprise_tools/tutorial_detail_operation.html)，示例架构 2 群组 3 机构 6 节点。
1. 所有文件在本地目录生成。方便分发给不同的真实机构，由机构的运维人员自行部署启动。

## 相关版本信息
* [FISCO](https://github.com/FISCO-BCOS/FISCO-BCOS/blob/master/docs/README_CN.md): v2.6.0
* [Ansible](https://www.ansible.com/): 2.10.1

## 部署环境要求
* Linux。目前在 Ubuntu 16.04 上启动通过。
* 系统建议安装有时间同步服务，例如 chrony。如果是离线环境，请参照官方文档做好内网时间同步。
* 启动文件应部署在外挂数据盘，建议采用分布式存储。
* 各机构服务器之间可以访问相关的 IP 和监听端口（具体可以查看生成的 deploy 目录内容，后面有详述）。

## 联盟链初始化
1. 复制一份 inventory 配置。假设新环境是 'my_inventory'。

   ```
   > cp -R inventories/sample inventories/my_inventory
   ```

1. 编辑 init.yml 配置文件

   ```
   > nano inventories/my_inventory/group_vars/init.yml
   ```

   例如
	
	```
	# true 表示使用国密模式
	fisco_gm_enabled: true
	
	# 变量注释
	# name:           （必填）机构名称
	# create_genesis: 是否生成创世区块，同组必须有且只有 1 个创世区块机构。（不设置则默认为 false）
	# nodes:          （必填）机构各节点连接信息。格式 <ip>:<节点数>。如果只填写 IP，则默认为 1 个节点。同 IP 多节点会自动递增对应端口。參考 https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/enterprise_tools/tutorial_detail_operation.html#id5。生产环境中请确保各机构不共用服务器，避免私钥证书的泄漏。由于节点多群组共享网络带宽、CPU和内存资源，因此为了保证服务的稳定性，一台机器上不推荐配置过多节点。
	# main_group_id:  （必填）群组编号
	# extra_group_id: （可选）额外要加入的目标群组编号
	
	agencies:
	 - name: A
	   create_genesis: true
	   nodes:
	     - 172.17.8.101:5
	     - 172.17.8.102:5
	   main_group_id: 1
	   extra_group_id:
	     - 2
	 - name: B
	   nodes:
	     - 172.17.8.103:5
	     - 172.17.8.104:5
	   main_group_id: 1
	 - name: C
	   create_genesis: true
	   nodes:
	     - 172.17.8.105:5
	     - 172.17.8.106:5
	   main_group_id: 2
	```

   请根据文件头部的变量注释，编排好目标配置的信息。

1. 开始生成配置文件。

   ```
   > ansible-playbook -i inventories/my_inventory/hosts.ini fisco_bcos.yml
   ```

1. 如无意外，你会在 `inventories/my_inventory/deploy` 目录下找到相关的配置内容。例如：

   ```
    > find inventories/my_inventory/deploy/agency_*/fisco* -type d -name "node_*" | sort -n

    inventories/my_inventory/deploy/agency_A/fisco_deploy_agency_A/node_172.17.8.101_30300
    inventories/my_inventory/deploy/agency_A/fisco_deploy_agency_A/node_172.17.8.102_30300
    inventories/my_inventory/deploy/agency_B/fisco_deploy_agency_B/node_172.17.8.103_30300
    inventories/my_inventory/deploy/agency_B/fisco_deploy_agency_B/node_172.17.8.104_30300
    inventories/my_inventory/deploy/agency_C/fisco_deploy_agency_C/node_172.17.8.105_30300
    inventories/my_inventory/deploy/agency_C/fisco_deploy_agency_C/node_172.17.8.106_30300
   ```

1. 查看 `inventories/my_inventory/deploy/node_list.yml` 文件，内容类似于：

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
	确保相关的 IP 和端口能访问即可

1. 把相关文件夹上传到对应的服务器上，执行启动命令即可。

# 已有机构扩展节点

目前仅支持在 ini.yml 配置中，给已有的机构增加节点，不支持 agencies 里增加机构。

假设第一次初始化的配置是 


	```
	agencies:
	 - name: A
	   create_genesis: true
	   nodes:
	     - 172.17.8.101
	     - 172.17.8.102
	   main_group_id: 1
	   extra_group_id:
	     - 2
	 - name: B
	   nodes:
	     - 172.17.8.103
	     - 172.17.8.104
	   main_group_id: 1
	 - name: C
	   create_genesis: true
	   nodes:
	     - 172.17.8.105
	     - 172.17.8.106
	   main_group_id: 2
	```
   
要给机构 B 的 172.17.8.103 服务器增加 3 个节点，就把 `172.17.8.103` 改成 `172.17.8.103:4`(因为默认是 1 个节点，所以增加 3 个就是 4)。也可以增加 IP 和节点，例如：
   
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
   
然后再次执行

	```
	> ansible-playbook -i inventories/my_inventory/hosts.ini fisco_bcos.yml
	```

执行完成后，在对应的机构目录下，你可以看到类似 `fisco_deploy_agency_A_expand_1917645e6744e1360fba72fa4cf8cc47` 这样的目录，就是新增的节点配置了。