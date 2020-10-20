# FISCO-BCOS 企业级自动化运维部署

## 设计思路
1. Github https://github.com/FISCO-BCOS/generator.git
1. 参考[官方文档](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/enterprise_tools/tutorial_detail_operation.html)，示例架构 2 群组 3 机构 6 节点。最小支持单群组、单机构、多节点。
1. 所有文件在本地目录生成。方便分发给不同的真实机构，由机构的运维人员自行部署启动。

## 相关版本信息
* FISCO： v2.6.0
* Ansible: 2.10.1

## 系统要求
* Linux。目前在 Ubuntu 16.04 上启动通过。
* 系统必须安装有时间同步服务，例如 chrony。如果是离线环境，请参照官方文档做好内网时间同步。
