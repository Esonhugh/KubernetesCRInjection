## 注入产生原因

当负责分析指定 CRD 的 kubernetes controller 存在漏洞时，在攻击者可控某些云 CR 自定义资源，通过写入恶意 payload ，促使 kubernetes controller 控制器 在解析、处理、存储 CR 时或生成其他相关资源的时候触发恶意行为。

## 注入分类

注入有以下三种分类方式

### 注入存在点的注入技术分类  

1. 模版注入
2. 结束符注入 `;` `\n`  `}` `"` 等
3. 对象或对象引用注入

### 结果分类 

1. 信息泄露 - 环境变量敏感信息引入或注入可控环境 
2. 表达式注入-可执行脚本，表达式，hook, 宏, sql
3. 配置文件注入
4. \[可能的\] 文件路径，请求参数，数据库 访问控制绕过
5. 级联注入 （注入上级 CR 后生成下层各个其他 CR 组件时生成了恶意的下层组件 如Pod 的过程）

### 注入点分类 

1. annotation 注入  
2. spec 注入 不严格的 validation 配置
3. 可控制的 ref

## 案例

### jqctf2023 - 逃逸？逃译！

1. 注入点为 lb-type annotation
2. 注入类型是 换行注入 
3. 注入内容为 恶意 pod 配置
4. PoC 为 在 annotation 下注入名称后 注入整个 Pod 的配置后 注入 secret 中有的 flag 信息
5. https://paper.seebug.org/1882/ CVE-2022-21701 istio 类似

### Kubernetes - Nginx ingress controller CVE 

1. 注入点为 annotation
2. 注入类型是 结束符注入 ; 
3. 注入内容是  nginx 配置文件 或者 lua 脚本 可以任意读泄漏 serviceaccount
4. https://hackerone.com/reports/1728174 lua 表达式注入 exec 
6. https://hackerone.com/reports/1378175 nginx leak
7.  CVE-2021-25742 and CVE-2021-25746
8. CVE-2023-5044 https://raesene.github.io/blog/2023/10/29/exploiting-CVE-2023-5044/ lua 表达式 exec 

### 一个目前未被披露的漏洞 possible CVE unpublished vulnerability 

1. 背景：多租户系统中，存在 Hacker 租户
2. 注入点为 env 环境变量（租户可控）
3. 注入类型为 对象注入 不安全的ref 
4. 危害： 单租户可以直接泄漏 controller 或 controller 下的任意指定的 secrets/configmap 的指定 key 
5. 注入 env 使用 secretKeyRef 注入其他数据  https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-using-secret-data
	
## 信息

1. CRD kubernetes document: https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/
2. CRD validation kubernetes document: https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#validation
3. OpenApi v3 schema: https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.0.md#schemaObject

## 较易受影响的服务

这类漏洞产生原因是主要是由于 CRD 控制器的设计问题。因为在多用户的场景或者 SaaS 场景下，用户可能可以间接或者直接控制集群内部的某些资源（可能是 CRD 资源，也可能是其他资源），而这些资源的控制器可能直接或者间接的拼接用户可控的数据，从而导致此类注入漏洞，并且进一步危害集群内部的安全。