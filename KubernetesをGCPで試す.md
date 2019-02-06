
# 0, 用語の理解

Kubernetesは概念が命。
用語を理解しないと始まらない。 (そして用語が沢山あるのだ…)

[Kubernetesの用語の理解](https://qiita.com/sheepland/private/cdff472ba2d37784a125)


# 1, 準備

### GCPのコンソールにアクセスできることを確認。

https://console.cloud.google.com/home/dashboard

### プロジェクトの作成

この[リソースの管理](https://console.cloud.google.com/cloud-resource-manager)画面からプロジェクトを作成してください。
プロジェクト名は`k8s-test`にします。

### よく使う画面をピン留めする

以下の4つの画面はこのハンズオンでよく見るのでピン留めしておきます。(左上のメニューからサービス名にマウスオーバーするとピンのアイコンがでるのでクリックするとピン留めできます。ピン留めするとメニューの上のほうに固定されます)

![スクリーンショット 2018-10-15 22.33.23.png](https://qiita-image-store.s3.amazonaws.com/0/14124/bfb473eb-1ee7-d6eb-475b-3b0315f13301.png)



###  Google Cloud Platform サービスの有効化  

さらに、GCPではサービスを利用する前に有効化する必要があります。
検索窓に「api」とうち、「APIとサービス」を選びます
![スクリーンショット 2018-05-09 1.51.46.png](https://qiita-image-store.s3.amazonaws.com/0/14124/5c6bbf31-f1ba-1412-d8da-a99c8be00b7b.png)
そして次に「APIとサービスの有効化」ボタンを押します。
![スクリーンショット 2018-05-09 1.56.35.png](https://qiita-image-store.s3.amazonaws.com/0/14124/1956a8ad-0085-f4c4-8485-d724b672b1a1.png)

表示された検索窓に以下の文字列をうって検索し、それぞれ有効化してください。
もしかしたら既に有効化済みのものもあるかも。
また有効化には時間がかかるので、別タブ開いて次のAPIを有効化すると時間の節約になります。

* Compute Engine API
* Kubernetes Engine API
* Cloud Build API
* Container Registry API

### gcloudツールのインストール

https://cloud.google.com/sdk/docs/quickstart-mac-os-x
の"始める前に"の1〜4を行う。
"始める前に"のセクションだけやればいいです。"SDK の初期化"のセクションはやらなくて大丈夫です。


### パスを通す
以下の設定で勝手にパスが通ります。

bashの人

```sh:.bashrc
# bashの人は以下でよい。ただし"$HOME/path/to"の部分は適切に修正すること。
source $HOME/path/to/google-cloud-sdk/completion.bash.inc
source $HOME/path/to/google-cloud-sdk/path.bash.inc

#エイリアスはお好みで(kubectlコマンドは後でインストールするが先にaliasだけ設定しておく)
alias kc=kubectl 
```

zshの人

```sh:.zshrc
# zshの人は以下でよい。ただし"$HOME/path/to"の部分は適切に修正すること。
source $HOME/path/to/google-cloud-sdk/completion.zsh.inc
source $HOME/path/to/google-cloud-sdk/path.zsh.inc

#エイリアスはお好みで(kubectlコマンドは後でインストールするが先にaliasだけ設定しておく)
alias kc=kubectl
```

```
# shellを再起動する
$ exec $SHELL -l
```

```
# gcloudコマンドが通ることを確認
$ gcloud version

# 最新版にアップデート(ちょっと時間がかかるかも)
$ gcloud components update
```

### プロジェクトIDの設定をする

```
$ PROJECT_ID=hogehoge #ここにプロジェクトIDを設定する(プロジェクトIDの取得の仕方は下の画像を参照)
```

[リソースの管理](https://console.cloud.google.com/cloud-resource-manager)画面に行き、以下をコピーする。
![スクリーンショット 2018-10-10 23.43.37.png](https://qiita-image-store.s3.amazonaws.com/0/14124/ac943a9f-ded9-8b2e-8ee1-a1620d3aea9f.png)


```
$ echo $PROJECT_ID # k8s-test以外の文字が出力されることを確認
$ gcloud config set project $PROJECT_ID
$ gcloud config list | grep project # 設定されていることを確認
```

### リージョンの設定をする

```
$ gcloud config set compute/zone asia-northeast1-a
```

### 認証をする

```
$ gcloud auth login
```

上記コマンドを入力するとブラウザが立ち上がりOAuth認証が始まりますので、プロジェクト作成に利用したアカウントで認証を行ってください。認証が成功すると、gcloudコマンドからGCPを操作できるようになります。

### kubectlのインストール

kubectlはKubernetesを操作するためのCLIツールです

```
$ gcloud components install kubectl
```

# 2, ハンズオンの準備

```bash
$ cd 適当な作業ディレクトリ
$ git clone https://github.com/akira-kuriyama/kubernetes-handson.git
$ cd kubernetes-handson/source
$ echo $PROJECT_ID # k8s-test以外の文字が出力されることを確認
$ sed -i '' "s/PROJECT_ID/$PROJECT_ID/g" *.yaml
```

以降は `kubernetes-handson/source`以下のyamlファイルを使ってハンズオンを進めます。

# 3, Kubernetes Clusterの作成

Kubernetes Clusterの作成します。(デフォルトで3台のNodeが作成されます)

```
$ gcloud container clusters create my-k8s
```

https://console.cloud.google.com/kubernetes で確認できます。
もしくは`$ gcloud container clusters list`で確認できる。

完了まで数分(5,6分)かかる。

Kubernetes Clusterが作成されると以下のようになる。
![スクリーンショット 2018-05-09 3.10.24.png](https://qiita-image-store.s3.amazonaws.com/0/14124/63e6bb1d-1e3a-61df-4732-e88c64798549.png)

また、 https://console.cloud.google.com/compute/instances をみるとGCEインスタンスが作成されているのが分かる。これがnodeとなる。

クラスタが作成できたら、kubectlが正しくクラスタに接続して操作を行えるように、kubernetesクラスタの認証情報をセットします。

```
$ gcloud container clusters get-credentials my-k8s
``` 


確認

```
$ kubectl cluster-info
```

以下のような表示がされたらOK

```bash
$ kubectl cluster-info
Kubernetes master is running at https://35.200.6.30
GLBCDefaultBackend is running at https://35.200.6.30/api/v1/namespaces/kube-system/services/default-http-backend/proxy
Heapster is running at https://35.200.6.30/api/v1/namespaces/kube-system/services/heapster/proxy
KubeDNS is running at https://35.200.6.30/api/v1/namespaces/kube-system/services/kube-dns/proxy
kubernetes-dashboard is running at https://35.200.6.30/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```



# 4, podを配置

これから"◯◯ファイルを作成"というのが出てきますが、すでにファイルはwgetして手に入れているのでスキップして下さい。

### 最初にimageを作成

hello.goを作成。8080ポートでまちうけ、Hello, World!を返すプログラム。


```go:hello.go
package main

import (
        "io"
        "net/http"
        "os"
)

func main() {
        http.HandleFunc("/", hello)
        http.ListenAndServe(":8080", nil)
}

func hello(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "text/plain")
        // 出力メッセージは環境変数があればそれを使い、なければ"Hello, World!"を使う
        message := os.Getenv("CUSTOM_MESSAGE")
        if message == "" {
                message = "Hello, World!"
        }
        io.WriteString(w, message)
}
```

次にDockerfileを作成

```txt:Dockerfile
FROM alpine:3.6
EXPOSE 8080
ADD hello-world /hello-world 
CMD ["/hello-world"]
```

次にcloudbuild.yamlを作成

```yaml:cloudbuild.yaml
steps:
- name: 'gcr.io/cloud-builders/go:alpine'
  env: ['PROJECT_ROOT=my-project']
  args: ['build', '-o', 'hello-world', 'hello.go']
- name: 'gcr.io/cloud-builders/docker'
  env: ['PROJECT_ROOT=my-project']
  args: ['build', '--tag=asia.gcr.io/PROJECT_ID/my-project/hello-world:latest', '.']
images: ['asia.gcr.io/PROJECT_ID/my-project/hello-world:latest']
```

次にGoogle Container Register(以下、GCR) にイメージを登録します。

```
$ gcloud builds submit --config=cloudbuild.yaml .
```

https://console.cloud.google.com/gcr にアクセスして登録されたことを確認。
`$ gcloud builds list`でも確認できる。

### Podの作成

次にpodファイルを作成

```yaml:pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:
  containers:
    - image: asia.gcr.io/PROJECT_ID/my-project/hello-world
      imagePullPolicy: Always
      name: hello-world
```

そしていよいよpodをKubernetes上に作成

```
kubectl create -f pod.yaml
```

podが作成されたか確認する

```bash
$ kubectl get pods
NAME          READY     STATUS    RESTARTS   AGE
hello-world   1/1       Running   0          5s
```

またGCPのKubernetes画面のワークロードの画面でも確認できる。
https://console.cloud.google.com/kubernetes/workload

pod内のコンテナにコマンドを実行する。

```
kubectl exec -it pod名(kubectl get podの左側のNAME) date
```

pod内のコンテナにログインする

```
kubectl exec -it pod名(kubectl get podの左側のNAME) /bin/sh
```

pod内に複数のコンテナが動いている場合は、以下のようにコンテナ名を指定する

```
kubectl exec $pod_name --container $container_name $command
```

※ ちなみに`kubectl get pod`とか `kubectl get po`でもいける。

また、
`$ kubectl get pod -o wide`で追加情報が表示される。get pod以外にも

```
kubectl get deployments
kubectl get replicasets
kubectl get services
```

があり、それぞれ `-o wide`が使える。

あとpod(やserviceやreplicasetやdeployment)の定義を知りたい場合は、
`kubectl get pod -o yaml`のように`-o yaml`でみれる。便利。
podの詳細情報を見たい場合は、
`kubectl describe pod`や`kubectl describe pod pod名`でみれる

# 5, podへのアクセス

pod内のコンテナへ簡単にアクセスするための方法として、port forwardingがあります。

$POD_NAMEは `kubectl get pods` したときに一番左側に表示されているやつ。

```
$ kubectl port-forward $POD_NAME 8080
```

で、別タブで、

```
$ curl http://localhost:8080/
```


確認できたら、いったん削除しましょう
`kubectl delete pods $POD_NAME`
もしくは
`kubectl delete -f pod.yaml`

# 6, 設定値を設定ファイルから与える

まず以下のような設定ファイルを作成

```yaml:configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-world-config
data:
  hello-world.message: "Hello World, ConfigMap!"
```

以下を実行する。

```
$ kubectl create -f configmap.yaml
# 以下で作成されていることを確認
$ kubectl get configmaps
```

次にpod定義ファイルを作成。envセクションがポイント。configMapKeyRefとか。

```yaml:pod-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world-configmap
spec:
  containers:
    - image: asia.gcr.io/PROJECT_ID/my-project/hello-world
      imagePullPolicy: Always
      name: hello-world
      env:
        - name: CUSTOM_MESSAGE
          valueFrom:
            configMapKeyRef:
              name: hello-world-config
              key: hello-world.message
```

で、
`$ kubectl create -f pod-configmap.yaml` でpodを作成し、
`$ kubectl port-forward $POD_NAME 8080` をして、
別タブで、
`$ curl http://localhost:8080/`
すると、"Hello World, ConfigMap!"とでるはず。

確認できたら、いったん削除しましょう
`kubectl delete pods $POD_NAME`
もしくは
`kubectl delete -f pod-configmap.yaml`

# 7, ReplicaSetを使ったデプロイ

```yaml:replicaset.yaml
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  name: hello-world-replicaset
spec:
  replicas: 3
  template:
    metadata:
      labels:
        name: hello-world
    spec:
      containers:
        - image: asia.gcr.io/PROJECT_ID/my-project/hello-world
          imagePullPolicy: Always
          name: hello-world
```

以下でreplicasetを生成する

```
$ kubectl create -f replicaset.yaml
```


`$ kubectl get pods`
3つ作成されているのが分かる。
次に試しにpodの一つを手動で消してみる。
`$ kubectl delete pods $POD_NAME`
すると、podが3つに戻ることが確認できます。
`$ kubectl get pods`

つぎに、replicaset.yaml内の`replicas: 3`を`replicas: 1`にして、
`$ kubectl replace -f replicaset.yaml` とやって設定を変更し、
`$ kubectl get pods`とすると、pod数が1になっていることが確認できます。

ちなみに
`$ kubectl get replicasets`でReplicaSetの状態を確認できます。

確認が終わったら消しておきましょう。
`$ kubectl delete replicaset hello-world-replicaset`
もしくは
`$ kubectl delete -f replicaset.yaml`

# 8, Deploymentを使ったデプロイ

```yaml:deployment.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: hello-world-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        name: hello-world-pod
    spec:
      containers:
        - image: asia.gcr.io/PROJECT_ID/my-project/hello-world
          imagePullPolicy: Always
          name: hello-world-pod
```

次にデプロイをする
`$ kubectl create -f deployment.yaml`

**これでdeploymentとreplicasetとpodが作成される。**

`$ kubectl get pod,replicasets,deployments`で確認

もしくは `$ kubectl get all`で確認

あとは、GCPのKubernetesのワークロードの画面で、deploymentを確認することができる。
https://console.cloud.google.com/kubernetes/workload


※ このdeploymentは消さなくてよいです。

# 9, Serviceの作成
 

LoadBalancerも一緒に作成してくれるServiceを作成します(`type: LoadBalancer`)。
このtypeはIngressがなくても静的IPが割り当てられます。

yaml内の`selector`に`name: hello-world-pod`と指定されているが、これは`name: hello-world-pod`のlabelを持ったpodと紐付けるという意味。
`port: 80`はこのserviceがうけつけるport。`targetPort: 8080`はpodが開放しているport。

```yaml:lb-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-world-lb-svc
spec:
  type: LoadBalancer
  selector:
    name: hello-world-pod
  ports:
  - port: 80
    targetPort: 8080
```

serviceのデプロイ
`$ kubectl create -f lb-service.yaml`

`$ kubectl get services hello-world-lb-svc -w`
-wを指定すると、変更や再登録があった際に更新された値が表示されて便利。(wはwatchのw)

EXTERNAL-IP が `<pending>` から有効な IP アドレス に変わったら(数分かかります)、そのアドレスにアクセスしてみましょう。Hello Worldが出力されるはずです。
`$ curl http://$EXTERNAL_IP/`

`type: LoadBalancer`を指定したので、load balancerも作成されたという次第です。

serviceには[他にも種類がある](https://qiita.com/sheepland/private/cdff472ba2d37784a125#service%E3%81%AE%E7%A8%AE%E9%A1%9E)。

ちなみに、以下のようにリソース名称は省略できる。

- `kubectl get pods`は`kubectl get po`
- `kubectl get deployments`は`kubectl get deploy`
- `kubectl get services`は`kubectl get svc`

省略名称は、`kubectl get`と打つと`aka`で表示される。
あと`kubectl get`でリソースの全種類が見れる。


# 10, Ingressの作成

次はServiceとIngressを作って以下の構成にします。

![スクリーンショット 2018-10-14 22.13.15.png](https://qiita-image-store.s3.amazonaws.com/0/14124/77a66678-f351-d3e5-5db6-53888728d5b4.png)


Serviceはトランスポート層(L4)のプロキシを行うためのしくみです。
アプリケーション層(L7)のプロキシを行うためには、Ingressを使用する必要があります。
Ingressは以下が可能。

* load balancing
* SSLの終端
* URLのパスごとにどのserviceにリクエストを割り振るかの設定
* name-based virtual hosting (Apacheのバーチャルホスト相当)


```yaml:service.yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-world-svc
spec:
  type: NodePort
  selector:
    name: hello-world-pod
  ports:
    - port: 8080
```

`serviceName: hello-world-svc`で紐付けるserviceを指定しています。

```yaml:ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world-ing
spec:
  rules:
  - http:
     paths:
      - path: /*
        backend:
          serviceName: hello-world-svc
          servicePort: 8080
```
Serviceの作成
`$ kubectl create -f service.yaml`

Ingressの作成
`$ kubectl create -f ingress.yaml`

確認

```
$ kubectl get ingress hello-world-ing -w
```


しばらくするとADDRESS欄に外部IPが付与される。
しかし外部IPが付与されたあとでもロードバランサがトラフィックを処理する準備が整うまで、HTTP 404 や HTTP 500 などのエラーが発生することがあります。正常にレスポンスが返るようになるまで結構かかる(10分以上?)ので気長に待ちましょう。。
`kubectl describe ingress`のレスポンス内のAnnotationsのbackendsのUnknownがHEALTHYになったら正常にレスポンスを返すようになる。
もしくはGCPのKubernetesの検出と負荷分散の画面でingressのステータスが正常になるまで待ちましょう
https://console.cloud.google.com/kubernetes/discovery
また、ロードバランサーは以下の画面から見れます
https://console.cloud.google.com/net-services/loadbalancing/loadBalancers/list

# 11, dashboard

dashboard機能もあります

以下を実行して表示される文字列をコピーします

```
$ kubectl config view -o json | jq '.users[] | select(.name=="'$(kubectl config current-context)'") | .user."auth-provider".config."access-token"'
```

proxyを立ち上げます

```
$ kubectl proxy
```

そして、 http://127.0.0.1:8001/ui へアクセス。

認証が必要になるので、以下の画面が表示されたら、「Token」を選び、さっきコピった文字列を「Enter token」テキストボックスにペーストします。で、「SIGN IN」ボタンクリック。
<img width="1049" alt="スクリーンショット 2018-05-09 19.48.11.png" src="https://qiita-image-store.s3.amazonaws.com/0/14124/a747f002-1910-43f8-2498-e32a7e85d2ea.png">


proxyコマンドを使う以外にも方法はあります。参考: https://qiita.com/sheepland/items/0ee17b80fcfb10227a41

# 12, 最後に

終わったらまとめて消しましょう。

```sh
$ kubectl delete service hello-world-lb-svc
$ kubectl delete service hello-world-svc
$ kubectl delete ingress hello-world-ing
$ kubectl delete deployment hello-world-deployment

# クラスタごと削除したい方はこちら。クラスタは残しておきたい人は後述するnodeサイズを0にする方法のほうがよい。
$ gcloud container clusters delete my-k8s
```

Kubernetesのリソースの全削除は以下でも可能
`kubectl delete all --all`

GCPのKubernetesはnodeの数だけGCEインスタンスを立ち上げています。なのでその分料金が発生しています。
しかしGCEの画面からインスタンスを削除してもKubernetesが自動的にインスタンスを復旧(立ち上げ直す)しちゃいます。
なので、以下のコマンドでnode数を0にするとよいです。こうするとclusterに登録したリソース(deploymentやserviceやingress等)の情報が失われません。sizeを再度例えば3にすると復旧します。
`$ gcloud container clusters resize my-k8s --size=0`


# その他のkubectlコマンド

https://kubernetes.io/docs/reference/kubectl/overview/#operations を参照

例えば以下がある

```
kubectl describe po
kubectl describe po POD名
kubectl logs pod名
kubectl explain pod
kubectl explain pod.spec
```

# Kubernetesをもっと知りたい、触りたい

* Kubernetes完全ガイド http://amzn.asia/d/9QFJw5X
    * これを買えば間違いがない
* 公式のWebでできるチュートリアル。Kubernetesクラスタを手元で立てずにチュートリアルができる。
    * https://kubernetes.io/docs/tutorials/kubernetes-basics/cluster-interactive/
* ローカルでKubernetesクラスタを簡単にたてるためのアプリケーション
    * https://github.com/kubernetes/minikube



