# AWS Vanilla K8S Environment

## Cloudformation.yaml

VPC, EC2.. for Vanilla K8S

* v1.23.6
* Flannel CNI
* CRI(Containerd)
* 2 * public subnet, 2 * private subnet
* 1 * master node, 3 * worker node
* ubuntu 22.04, t3.medium





## After Install, access master node

```bash

# 마스터 노드 SSH 접속 후 설치 로그 확인 : 반드시 아래 설치 스크립트 완료를 확인!
sudo tail -f /var/log/cloud-init-output.log
...(생략)...
>>>> K8S Controlplane Config End <<<<
Cloud-init v. 22.1-14-g2e17a0d6-0ubuntu1~22.04.5 finished at Fri, 20 May 2022 06:09:56 +0000. Datasource DataSourceEc2Local.  Up 154.07 seconds

# 마스터 노드 SSH 종료 후 다시 SSH 접속
exit
ssh ~

# CNI/StorageClass 등 설치 스크립트 실행
exa -bghHliSR .
cat efs.txt
./final.sh

# 쿠버네티스 정상 상태 확인
kubectl cluster-info
systemctl status kubelet
kubectl get node -v7
kubectl get node -owide

# 파드 확인
kubectl get pod -A

# storageclasses 확인
kubectl get storageclasses
NAME                   PROVISIONER                                                     RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)   rancher.io/local-path                                           Delete          WaitForFirstConsumer   false                  39s
nfs-client             cluster.local/nfs-provisioner-nfs-subdir-external-provisioner   Delete          Immediate              true                   36s

# 볼륨 확인
lsblk
df -h -T -t ext4
duf
duf -all
exportfs -v

# AWS EFS 볼륨 마운트 확인 : EFS 파일시스템 ID는 실습환경에 따라 다름
duf -hide local,special
╭───────────────────────────────────────────────────────────────────────────────────────╮
│ 2 network devices                                                                     │
├─────────────────────────┬──────┬──────┬───────┬──────┬──────┬─────────────────────────┤
│ MOUNTED ON              │ SIZE │ USED │ AVAIL │ USE% │ TYPE │ FILESYSTEM              │
├─────────────────────────┼──────┼──────┼───────┼──────┼──────┼─────────────────────────┤
│ /nfs4-share             │ 8.0E │   0B │  8.0E │      │ nfs4 │ fs-0086c2e0fbde36ef0.ef │
│                         │      │      │       │      │      │ s.ap-northeast-2.amazon │
│                         │      │      │       │      │      │ aws.com:/               │
│ /var/lib/kubelet/pods/8 │ 8.0E │   0B │  8.0E │      │ nfs4 │ fs-0086c2e0fbde36ef0.ef │
│ e0a5a4d-1cf8-4081-988e- │      │      │       │      │      │ s.ap-northeast-2.amazon │
│ 19d1aa8950f3/volumes/ku │      │      │       │      │      │ aws.com:/               │
│ bernetes.io~nfs/nfs-sub │      │      │       │      │      │                         │
│ dir-external-provisione │      │      │       │      │      │                         │
│ r-root                  │      │      │       │      │      │                         │
╰─────────────────────────┴──────┴──────┴───────┴──────┴──────┴─────────────────────────╯

df -h -T -t nfs4
Filesystem                                              Type  Size  Used Avail Use% Mounted on
fs-0086c2e0fbde36ef0.efs.ap-northeast-2.amazonaws.com:/ nfs4  8.0E     0  8.0E   0% /nfs4-share


# (옵션) EC2 기본 정보 확인
hostnamectl
cat /etc/hostname
cat /etc/hosts
ip -br -c -4 addr
cat /etc/resolv.conf
resolvectl status
ip -c route
iptables -t filter -S
iptables -t nat -S
systemctl list-unit-files | grep 'enabled         enabled'
lsmod

# (옵션) 모니터링
# https://github.com/aksakalli/gtop
docker run --rm -it --name gtop --net="host" --pid="host" aksakalli/gtop
```

## After all tests, delete cloudformation

```bash
aws cloudformation delete-stack --stack-name myk8s
```
