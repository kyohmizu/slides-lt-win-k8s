class: center, middle, blue
## Kubernetes に Windows ノードを<br/>0から追加するのは大変だった話

---
### whoami

.left-small[
    ![image](https://pbs.twimg.com/profile_images/994762110792953856/EheEvqBY_400x400.jpg)
]

.right-large[
- Kyohei Mizumoto(@kyohmizu)

- C# Software Engineer

- Interests
    - Docker
    - Kubernetes
    - Golang
]
---
### アドベントカレンダーに投稿しました

.zoom0[
<u><https://qiita.com/kyohmizu/items/dffdd49123b1e47c3ac4></u>
]

<center><img src="qiita.png" width=100%></center>

---
### 今日話すこと

- Windows on Kubernetes

- Windowsノード追加の手順

- ハマったポイント

---
class: center, middle, blue
## Windows on Kubernetes

---
### Windows on Kubernetes

.zoom2[
- Kubernetes V1.14 でGA

- コンテナネットワークには CNIプラグインを使用

Windowsコンテナについては以前話をしました
.zoom2[<u><https://speakerdeck.com/kyohmizu/windowskontenaru-men></u>]
]

---
### 導入方法

.zoom2[
.tmp[
- マネージドサービスを利用する
  - AKS の Windowsノードプール
]

.tmp[
- [kubernetes.io](https://kubernetes.io/docs/setup/production-environment/windows/user-guide-windows-nodes/) のガイドに従う
  - kubeadm + flannel を使用したノードの追加
]

.tmp[
- [Microsoft SDN](https://github.com/microsoft/SDN/tree/master/Kubernetes) の Kubernetes 導入用スクリプトを使用する
  - flannel (ドキュメントあり)
  - wincni (ドキュメントなし)
]
]

---
### 導入方法

.zoom2[
.tmp[
- マネージドサービスを利用する
  - AKS の Windowsノードプール
]

.tmp[
- [kubernetes.io](https://kubernetes.io/docs/setup/production-environment/windows/user-guide-windows-nodes/) のガイドに従う
  - kubeadm + flannel を使用したノードの追加
]

.tmp[
- [Microsoft SDN](https://github.com/microsoft/SDN/tree/master/Kubernetes) の Kubernetes 導入用スクリプトを使用する
  - flannel (ドキュメントあり)
  - <span style="color:red">wincni (ドキュメントなし) ←こちらを採用</span>
]
]

---
class: center, middle, blue
## Windowsノード追加の手順

---
### クラスタを準備

.zoom1[
<u><https://github.com/ivanfioravanti/kubernetes-the-hard-way-on-azure></u>

```bash
$ az vm list -d -g kubernetes -o table
Name          ResourceGroup    PowerState    PublicIps       Location
------------  ---------------  ------------  --------------  ----------
controller-0  kubernetes       VM running    XX.XXX.XXX.XXX  japaneast
controller-1  kubernetes       VM running    XX.XXX.XXX.XXX  japaneast
controller-2  kubernetes       VM running    XX.XXX.XXX.XXX  japaneast
worker-0      kubernetes       VM running    XX.XXX.XXX.XXX  japaneast
worker-1      kubernetes       VM running    XX.XXX.XXX.XXX  japaneast
worker-2      kubernetes       VM running    XX.XXX.XXX.XXX  japaneast

$ kubectl get no
NAME       STATUS   ROLES    AGE     VERSION
worker-0   Ready    <none>   12m     v1.15.0
worker-1   Ready    <none>   7m26s   v1.15.0
worker-2   Ready    <none>   5m59s   v1.15.0
```
]

---
class: header-margin
### ノード用 Windows Server VM を準備

<center><img src="win-server.png" width=100%></center>

.zoom1[
Hyper-V 分離コンテナを使用する場合は VMサイズに注意しましょう
.zoom1[https://docs.microsoft.com/en-us/azure/virtual-machines/windows/nested-virtualization]
]

---
### Windows Server の操作

.zoom1[
.zoom01[
<u><https://docs.microsoft.com/en-us/virtualization/windowscontainers/kubernetes/joining-windows-workers></u>
]

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
docker tag mcr.microsoft.com/windows/nanoserver:1809 `
mcr.microsoft.com/windows/nanoserver:latest
```
]

---
### Windows Server の操作

.zoom1[
```powershell
ls config,*exe

  Directory: C:\k

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       12/22/2019  11:49 AM           6254 config
-a----       12/11/2019   1:14 PM       40086016 kube-proxy.exe
-a----       12/11/2019   1:20 PM       47195136 kubectl.exe
-a----       12/11/2019   1:20 PM      119127552 kubelet.exe
```
]

---
### スクリプトをダウンロード

.zoom01[
<u><https://github.com/microsoft/SDN/blob/master/Kubernetes/windows/start.ps1></u>

```powershell
Start-BitsTransfer `
https://github.com/microsoft/SDN/raw/master/Kubernetes/windows/start.ps1
```
]

.zoom1[
ドキュメントのない wincni 用のスクリプトを採用した理由：

- flannel は事前にクラスタにインストールが必要
- hard way のクラスタは CNIプラグインを使用している
  <u><https://github.com/containernetworking/plugins></u>
  - flannel を使う必要がない
- ちなみに flannel をインストールしても動作します  
  (最初はこちらでやっていて、後で気づきましたorz)
]

---
class: center, middle, blue
## ハマったポイント

---
### ①ノードの podCIDR が未設定

.zoom1[
```bash
# 何も表示されない
$ kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}'
```
]

---
### ②自分の podCIDR を取得できない？


---
### ③必要な param の不足

---
### 完了！

.zoom2[
```bash
$ kubectl get no
NAME         STATUS   ROLES    AGE    VERSION
win-server   Ready    <none>   173m   v1.16.4
worker-0     Ready    <none>   28h    v1.15.0
worker-1     Ready    <none>   28h    v1.15.0
worker-2     Ready    <none>   28h    v1.15.0
```
]


---
### 参考

.zoom2[

]

---
class: center, middle, blue
## Thank you!
