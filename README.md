# FISCO-BCOS 企业级自动化运维部署

## 设计思路
1. 参考[官方文档](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/enterprise_tools/tutorial_detail_operation.html)，示例架构 2 群组 3 机构 6 节点。最小支持单群组、单机构、多节点。
1. 所有文件在本地目录生成。方便分发给不同的真实机构，由机构的运维人员自行部署启动。

## 相关版本信息
* FISCO: v2.6.0
* Ansible: 2.10.1

## 部署环境要求
* Linux。目前在 Ubuntu 16.04 上启动通过。
* 系统必须安装有时间同步服务，例如 chrony。如果是离线环境，请参照官方文档做好内网时间同步。
* 启动文件应部署在外挂数据盘，建议采用分布数存储。
* 各机构服务器之间可以访问相关的 IP 和监听端口（具体可以查看生成的 deploy 目录内容，后面有详述）。

## 初始化
1. 复制一份 inventory 配置。假设新环境是 'my_inventory'。

    ```
    > cp -R inventories/sample inventories/my_inventory

    ```
1. 编辑 init.yml 配置文件

    ```
    > nano inventories/my_inventory/group_vars/init.yml
    ```

    根据文件头部的变量注释，编排好目标配置的信息。

1. 开始生产配置文件。

    ```
    > ansible-playbook -i inventories/my_inventory/hosts.ini init.yml

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

1. 查看 inventories/my_inventory/deploy/node_list.yml 文件，内容类似于：
	
	```
	# 机构名称:群组编号:节点IP:P2P端口:Channel端口:RPC端口:额外目标群组编号

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

1. 每个机构目录下还会有 console 控制台目录，可用于基本管理。建议使用 WeBase 作为管理平台。
