# k8s etcd操作

## 证书位置

### k8s证书

| 证书                      | 路径                                                         | 说明             |
| ------------------------- | ------------------------------------------------------------ | ---------------- |
| kubernetes-ca             | /etc/kubernetes/pki/ca.crt                                   | kubernetes通用ca |
| kubernetes-front-proxy-ca | /etc/kubernetes/pki/front-proxy-ca.crt<br/>/etc/kubernetes/pki/front-proxy-client.crt | 前端代理         |
| etcd-ca                   | /etc/kubernetes/pki/etcd/ca.crt<br/>/etc/kubernetes/pki/etcd/peer.crt<br/>/etc/kubernetes/pki/etcd/server.crt | etcd相关         |



### OpenShift证书

| 证书                | 路径                                                         | 说明 |
| ------------------- | ------------------------------------------------------------ | ---- |
| master证书/node证书 | /etc/origin/master/master.server.crt<br/>/etc/origin/master/master.proxy-client.crt<br/>/etc/origin/master/master.kubelet-client.crt<br/>/etc/origin/master/server-signer.crt<br/>/etc/origin/master/master.etcd-client.crt<br/>/etc/origin/master/master.etcd-ca.crt<br/>/etc/origin/master/ca.crt<br/>/etc/origin/node/client-ca.crt<br/> |      |
| etcd证书            | /etc/etcd/ca.crt<br/>/etc/etcd/server.crt<br/>/etc/etcd/peer.crt |      |



#### 命令行获取证书

##### **ETCD用证书**

**ETCD Peer 证书**

```shell
for name in $(oc get node -o custom-columns=NAME:metadata.name | grep master)
do
echo etcd-peer-$name  
oc get secret etcd-peer-$name -n openshift-etcd -o=custom-columns=":.data.tls\.crt" | tail -1 | base64 -d | openssl x509 -noout -dates
done

# 或者

$ ssh core@master_hostname
$ sudo -i
$ for i in /etc/kubernetes/static-pod-resources/etcd-certs/secrets/etcd-all-peer/*.crt; do echo $i; openssl x509 -in $i -noout -dates; done
```

**ETCD Serving 证书**

```shell
for name in $(oc get node -o custom-columns=NAME:metadata.name | grep master)
do
echo etcd-serving-$name  
oc get secret etcd-serving-$name -n openshift-etcd -o=custom-columns=":.data.tls\.crt" | tail -1 | base64 -d | openssl x509 -noout -dates
done

# 或者

$ ssh core@master_hostname
$ sudo -i
$ for i in /etc/kubernetes/static-pod-resources/etcd-certs/secrets/etcd-all-serving/*.crt; do echo $i; openssl x509 -in $i -noout -dates; done
```

**ETCD Serving Metrics 证书**

```shell
for name in $(oc get node -o custom-columns=NAME:metadata.name | grep master)
do
echo etcd-serving-metrics-$name  
oc get secret etcd-serving-metrics-$name -n openshift-etcd -o=custom-columns=":.data.tls\.crt" | tail -1 | base64 -d | openssl x509 -noout -dates
done

# 或者

$ ssh core@master_hostname
$ sudo -i
$ for i in /etc/kubernetes/static-pod-resources/etcd-certs/secrets/etcd-all-serving-metrics/*.crt; do echo $i; openssl x509 -in $i -noout -dates; done
```

 

## 数据存储

在etcd中，资源命名是这样的

```shell
prefix/资源类型/namespace/具体资源名
例如：
/registry/deployments/default/helloworld
```

etcd的get有按照key查找和按照范围查找两种方式

```shell
kubectl get deployments  -n default
#对应etcdctl get --prefix /registry/deployments/default

kubectl get deployments/helloworld
#对应etcdctl get --prefix /registry/deployments/default/helloworld

kubectl get deployments -l hello:hello
#对应于etcdctl get --prefix /registry/deployments/default，然后将结果根据标签hello:hello过滤
```



## etcd读写

**获取etcd的endpoint**

```shell
# etcdctl endpoint status
https://127.0.0.1:2379, 17057a8cf6d6cbb3, 3.3.15, 10 MB, true, 4, 523191
```



**命令行**

etcd2命令

```shell
export ETCDCTL_API=2
etcdctl \
--key-file=/etc/kubernetes/pki/etcd/server.key \
--cert-file=/etc/kubernetes/pki/etcd/server.crt  \
--ca-file=/etc/kubernetes/pki/etcd/ca.crt \
--endpoints https://127.0.0.1:2379 \
get /registry/namespaces/default
```



etcd3命令

```shell
export ETCDCTL_API=3
etcdctl \
--key=/etc/kubernetes/pki/etcd/server.key \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--endpoints https://127.0.0.1:2379 \
--write-out=json \
get --prefix /registry/deployments/default|jq .

{
  "header": {
    "cluster_id": 16888901464889231000,
    "member_id": 3444459836023052300,
    "revision": 69522,
    "raft_term": 13
  }
}
```

- --write-out=json 返回的数据使用json格式显示，默认是使用protobuf存储



```shell
NAME:
        get - Gets the key or a range of keys

USAGE:
        etcdctl get [options] <key> [range_end]

OPTIONS:
      --consistency="l"                 Linearizable(l) or Serializable(s)
      --from-key[=false]                Get keys that are greater than or equal to the given key using byte compare
      --keys-only[=false]               Get only the keys
      --limit=0                         Maximum number of results
      --order=""                        Order of results; ASCEND or DESCEND (ASCEND by default)
      --prefix[=false]                  Get keys with matching prefix
      --print-value-only[=false]        Only write values when using the "simple" output format
      --rev=0                           Specify the kv revision
      --sort-by=""                      Sort target; CREATE, KEY, MODIFY, VALUE, or VERSION

GLOBAL OPTIONS:
      --cacert=""                               verify certificates of TLS-enabled secure servers using this CA bundle
      --cert=""                                 identify secure client using this TLS certificate file
      --command-timeout=5s                      timeout for short running command (excluding dial timeout)
      --debug[=false]                           enable client-side debug logging
      --dial-timeout=2s                         dial timeout for client connections
  -d, --discovery-srv=""                        domain name to query for SRV records describing cluster endpoints
      --endpoints=[127.0.0.1:2379]              gRPC endpoints
      --hex[=false]                             print byte strings as hex encoded strings
      --insecure-discovery[=true]               accept insecure SRV records describing cluster endpoints
      --insecure-skip-tls-verify[=false]        skip server certificate verification
      --insecure-transport[=true]               disable transport security for client connections
      --keepalive-time=2s                       keepalive time for client connections
      --keepalive-timeout=6s                    keepalive timeout for client connections
      --key=""                                  identify secure client using this TLS key file
      --user=""                                 username[:password] for authentication (prompt if password is not supplied)
  -w, --write-out="simple"                      set the output format (fields, json, protobuf, simple, table)
```





```json
$ etcdctl --key=/etc/kubernetes/pki/etcd/server.key --cert=/etc/kubernetes/pki/etcd/server.crt  --cacert=/etc/kubernetes/pki/etcd/ca.crt --endpoints https://127.0.0.1:2379 --write-out=json  get --prefix /registry/namespaces | jq .

{
  "header": {
    "cluster_id": 16888901464889231000,
    "member_id": 3444459836023052300,
    "revision": 70251,
    "raft_term": 13
  },
  "kvs": [
    {
      // base64解码后为字符串：/registry/namespaces/default
      "key": "L3JlZ2lzdHJ5L25hbWVzcGFjZXMvZGVmYXVsdA==",
      "create_revision": 197,
      "mod_revision": 197,
      "version": 1,
      // value的base64解码后为protobuf数据
      "value": "azhzAAoPCgJ2MRIJTmFtZXNwYWNlEogCCu0BCgdkZWZhdWx0EgAaACIAKiQzMTJmYmE1Mi1kYjBjLTRkZTAtOTcwMS01Y2E5NzY2YTUyM2YyADgAQggIwIDvngYQAFomChtrdWJlcm5ldGVzLmlvL21ldGFkYXRhLm5hbWUSB2RlZmF1bHR6AIoBfQoOa3ViZS1hcGlzZXJ2ZXISBlVwZGF0ZRoCdjEiCAjAgO+eBhAAMghGaWVsZHNWMTpJCkd7ImY6bWV0YWRhdGEiOnsiZjpsYWJlbHMiOnsiLiI6e30sImY6a3ViZXJuZXRlcy5pby9tZXRhZGF0YS5uYW1lIjp7fX19fUIAEgwKCmt1YmVybmV0ZXMaCAoGQWN0aXZlGgAiAA=="
    },
    {
      // base64解码：/registry/namespaces/kube-flannel
      "key": "L3JlZ2lzdHJ5L25hbWVzcGFjZXMva3ViZS1mbGFubmVs",
      "create_revision": 1151,
      "mod_revision": 1151,
      "version": 1,
      "value": "azhzAAoPCgJ2MRIJTmFtZXNwYWNlEp0FCoIFCgxrdWJlLWZsYW5uZWwSABoAIgAqJDVjZWIyODQxLTcxY2YtNDhmMi04ZmQyLTY2NWU5Y2I4MDVkMjIAOABCCAjPhe+eBhAAWisKG2t1YmVybmV0ZXMuaW8vbWV0YWRhdGEubmFtZRIMa3ViZS1mbGFubmVsWjAKInBvZC1zZWN1cml0eS5rdWJlcm5ldGVzLmlvL2VuZm9yY2USCnByaXZpbGVnZWRizQEKMGt1YmVjdGwua3ViZXJuZXRlcy5pby9sYXN0LWFwcGxpZWQtY29uZmlndXJhdGlvbhKYAXsiYXBpVmVyc2lvbiI6InYxIiwia2luZCI6Ik5hbWVzcGFjZSIsIm1ldGFkYXRhIjp7ImFubm90YXRpb25zIjp7fSwibGFiZWxzIjp7InBvZC1zZWN1cml0eS5rdWJlcm5ldGVzLmlvL2VuZm9yY2UiOiJwcml2aWxlZ2VkIn0sIm5hbWUiOiJrdWJlLWZsYW5uZWwifX0KegCKAYUCChlrdWJlY3RsLWNsaWVudC1zaWRlLWFwcGx5EgZVcGRhdGUaAnYxIggIz4XvngYQADIIRmllbGRzVjE6xQEKwgF7ImY6bWV0YWRhdGEiOnsiZjphbm5vdGF0aW9ucyI6eyIuIjp7fSwiZjprdWJlY3RsLmt1YmVybmV0ZXMuaW8vbGFzdC1hcHBsaWVkLWNvbmZpZ3VyYXRpb24iOnt9fSwiZjpsYWJlbHMiOnsiLiI6e30sImY6a3ViZXJuZXRlcy5pby9tZXRhZGF0YS5uYW1lIjp7fSwiZjpwb2Qtc2VjdXJpdHkua3ViZXJuZXRlcy5pby9lbmZvcmNlIjp7fX19fUIAEgwKCmt1YmVybmV0ZXMaCAoGQWN0aXZlGgAiAA=="
    },
    {
      "key": "L3JlZ2lzdHJ5L25hbWVzcGFjZXMva3ViZS1ub2RlLWxlYXNl",
      "create_revision": 48,
      "mod_revision": 48,
      "version": 1,
      "value": "azhzAAoPCgJ2MRIJTmFtZXNwYWNlEpgCCv0BCg9rdWJlLW5vZGUtbGVhc2USABoAIgAqJDBkZWI4ODM5LWRhYjktNDM3NC04YjQ4LTc1YjAwMzc1MjRlODIAOABCCAi/gO+eBhAAWi4KG2t1YmVybmV0ZXMuaW8vbWV0YWRhdGEubmFtZRIPa3ViZS1ub2RlLWxlYXNlegCKAX0KDmt1YmUtYXBpc2VydmVyEgZVcGRhdGUaAnYxIggIv4DvngYQADIIRmllbGRzVjE6SQpHeyJmOm1ldGFkYXRhIjp7ImY6bGFiZWxzIjp7Ii4iOnt9LCJmOmt1YmVybmV0ZXMuaW8vbWV0YWRhdGEubmFtZSI6e319fX1CABIMCgprdWJlcm5ldGVzGggKBkFjdGl2ZRoAIgA="
    },
    {
      "key": "L3JlZ2lzdHJ5L25hbWVzcGFjZXMva3ViZS1wdWJsaWM=",
      "create_revision": 46,
      "mod_revision": 46,
      "version": 1,
      "value": "azhzAAoPCgJ2MRIJTmFtZXNwYWNlEpACCvUBCgtrdWJlLXB1YmxpYxIAGgAiACokMDFiNmVhMWUtY2Y5Zi00MTIzLTgxZjItZDIzZTFlNDc2YTRhMgA4AEIICL+A754GEABaKgoba3ViZXJuZXRlcy5pby9tZXRhZGF0YS5uYW1lEgtrdWJlLXB1YmxpY3oAigF9Cg5rdWJlLWFwaXNlcnZlchIGVXBkYXRlGgJ2MSIICL+A754GEAAyCEZpZWxkc1YxOkkKR3siZjptZXRhZGF0YSI6eyJmOmxhYmVscyI6eyIuIjp7fSwiZjprdWJlcm5ldGVzLmlvL21ldGFkYXRhLm5hbWUiOnt9fX19QgASDAoKa3ViZXJuZXRlcxoICgZBY3RpdmUaACIA"
    },
    {
      "key": "L3JlZ2lzdHJ5L25hbWVzcGFjZXMva3ViZS1zeXN0ZW0=",
      "create_revision": 12,
      "mod_revision": 12,
      "version": 1,
      "value": "azhzAAoPCgJ2MRIJTmFtZXNwYWNlEpACCvUBCgtrdWJlLXN5c3RlbRIAGgAiACokMTA4YjY4NDktNzllYi00YmM3LTkxMjktN2VjNjVjM2FlMTBhMgA4AEIICL6A754GEABaKgoba3ViZXJuZXRlcy5pby9tZXRhZGF0YS5uYW1lEgtrdWJlLXN5c3RlbXoAigF9Cg5rdWJlLWFwaXNlcnZlchIGVXBkYXRlGgJ2MSIICL6A754GEAAyCEZpZWxkc1YxOkkKR3siZjptZXRhZGF0YSI6eyJmOmxhYmVscyI6eyIuIjp7fSwiZjprdWJlcm5ldGVzLmlvL21ldGFkYXRhLm5hbWUiOnt9fX19QgASDAoKa3ViZXJuZXRlcxoICgZBY3RpdmUaACIA"
    }
  ],
  "count": 5
}
```





**存储格式**

除了`/registry/apiregistration.k8s.io`是直接存储JSON格式的，其他资源默认都不是使用JSON格式直接存储，而是通过protobuf格式存储，

> openshift项目已经开发了一个强大的辅助工具[etcdhelper](https://github.com/openshift/origin/tree/master/tools/etcdhelper)可以读取etcd内容并解码proto。
>
> [etcdhelper](etcdhelper.go)

```shell
etcdhelper -cacert /etc/kubernetes/pki/etcd/ca.crt \
           -key /etc/kubernetes/pki/etcd/server.key \
           -cert /etc/kubernetes/pki/etcd/server.crt \
           get /registry/namespaces/default && echo
```

支持以下参数:

- `-key` - points to `master.etcd-client.key`
- `-cert` - points to `master.etcd-client.crt`
- `-cacert` - points to `ca.crt`

支持以下命令:

- `ls` - list all keys starting with prefix
- `get` - get the specific value of a key
- `dump` - dump the entire contents of the etcd





etcd的数据类型

```shell
/registry/apiregistration.k8s.io
/registry/clusterrolebindings
/registry/clusterroles
/registry/configmaps
/registry/controllerrevisions
/registry/daemonsets
/registry/deployments
/registry/events
/registry/leases
/registry/masterleases
/registry/minions
/registry/namespaces
/registry/persistentvolumeclaims
/registry/persistentvolumes
/registry/pods
/registry/podsecuritypolicy
/registry/priorityclasses
/registry/ranges
/registry/replicasets
/registry/rolebindings
/registry/roles
/registry/secrets
/registry/serviceaccounts
/registry/services
/registry/statefulsets
/registry/storageclasses
```



**kubectl get xxx**

/registry/{resource_name}/{namespace}/{resource_instance}



`minions`其实就是node信息，Kubernetes之前节点叫`minion`，因此还是使用的`/registry/minions`







## 参考

- [Openshift各组件Master/Node/Etcd/Router/Registry证书维护](https://www.jianshu.com/p/ffc4d6369d4e)
- [OpenShift 4 - 查看关键证书到期日期](https://blog.csdn.net/weixin_43902588/article/details/117089091)
- [etcd在k8s中](https://www.jianshu.com/p/677c11695e18)
- [kubernetes的PKI证书](https://blog.csdn.net/rendongxingzhe/article/details/124682560)
- [如何读取Kubernetes存储在etcd上的数据](https://zhuanlan.zhihu.com/p/94685947)
- [k8s学习笔记之etcd](https://www.jianshu.com/p/478ba38a938e)