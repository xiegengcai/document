### 一. Zookepper（10.85~10.87）合并成集群，dubbo-admin入口统一合并为http://192.168.10.84:8080，使用版本dubbo版本号区分各个环境，详见如下解释
### 二. 统一配置环境划分

1. local 环境：开发人员本地开发过程中使用，dubbo服务版本号加“-dev”或“-local”后缀。如你正常开发订单服务，把订单的版本好加“-local”后缀，其它保留“-dev”后缀依赖10.88服务；如果两个开发人员进行联调，两个模块都改成“-local”后缀进行联调
1. dev 环境：192.168.10.88业务集成组内部开发环境。每天发布版本使用，dubbo服务版本号加“-dev”后缀。该环境版本号不允许修改
1. dev-test 环境：192.168.10.86研发内部联调（与VSIM-CORE联调）版本。版本发布频率低于dev环境（原则三天更新一次）。dubbo服务版本号加“-dev-test”后缀。该环境版本号不允许修改
1. qa 环境：192.168.10.87准QA环境，提前提供给QA介入测试用。每周更新一次版本。dubbo服务版本号不带后缀（只保留数字）。该环境版本号不允许修改