---
title: 'Spring on K8s Workshop'
disqus: hackmd
---


## Lab 1. TDD 之 Tennis Kata
### 產出
![](https://i.imgur.com/dHREUb9.png)
說明：
1. firstPlayerScoreTimes:紀錄第一個選手得分
2. secondPlayerScoreTimes:紀錄第二個選手得分
3. score()：取得選手比數，顯示方式如下表所示
4. firstPlayerScore():第一個選手得分時呼叫此方法，並於firstPlayerScoreTimes加一
5. secondPlayerScore():第二個選手得分時呼叫此方法，並於secondPlayerScoreTimes加一
### 規則
![](https://i.imgur.com/nLOIy0W.png)
1. 0比0，呼叫 score 取得 Love All
2. 1比0，呼叫 score 取得 Fifteen Love
3. 2比0，呼叫 score 取得 Forty Love
4. 0比1，呼叫 score 取得 Love Fifteen

#### 步驟
##### love_all_test
###### test code
```gherkin=
public class TennisTests {
    @Test
    public void love_all_test() {
        Tennis tennis = new Tennis();
        assertEquals("love all", tennis.score());
    }
}
```  
###### production code
```gherkin=
public class Tennis {
    public String score() {
        return "love all";
    }
}
```
###### refactor test code
```gherkin=
public class TennisTests {

    private Tennis tennis;

    @Before
    public void setUp() throws Exception {
        tennis = new Tennis();
    }

    @Test
    public void love_all_test() {
        scoreShouldBe("love all");
    }

    private void scoreShouldBe(String expected) {
        assertEquals(expected, tennis.score());
    }
}
```

##### fifteen_love_test
###### test code
```gherkin=
    @Test
    public void fifteen_love_test() {
        tennis.firstPlayerScore();
        scoreShouldBe("fifteen love");
    }
```
###### production code
```gherkin=
public class Tennis {
    private int firstPlayerScoreTimes;

    public String score() {
        if(firstPlayerScoreTimes==1)
            return "fifteen love";
        return "love all";
    }

    public void firstPlayerScore() {
        firstPlayerScoreTimes++;
    }
}
```
##### thirty_love_test
###### test code
```gherkin=
@Test
public void thirty_love_test() {
    tennis.firstPlayerScore();
    tennis.firstPlayerScore();
    scoreShouldBe("thirty love");
}
```
###### production code
```gherkin=
public String score() {
    if (firstPlayerScoreTimes == 1)
        return "fifteen love";
    if (firstPlayerScoreTimes == 2)
        return "thirty love";
    return "love all";
}
```
###### refactor production code
```gherkin=
public String score() {
    HashMap<Integer, String> scoreLookUp = new HashMap<>() {{
        put(1, "fifteen");
        put(2, "thirty");
    }};
    if (firstPlayerScoreTimes == 1||firstPlayerScoreTimes==2)
        return scoreLookUp.get(firstPlayerScoreTimes) + " love";
        return "love all";
}
```
### 參考影片
https://youtu.be/HKrsDUAeuPE


## Lab 2.建立 Kind Cluster(下列練習以 ~/workspace/kind 為基礎路徑)
![](https://hackmd.io/_uploads/SyH5N0Tjn.png)

1. 開啟一個 terminal
```gherkin=
# 跳到 ~/workspace/kind
cd ~/workspace/kind

#查看 Makefile
cat Makefile

#建置 Kubernetes 叢集
make create

#建置 Load Balancer 
make build-lb

#測試 Load Balancer
make test-lb

#刪除測試程式
make delete
```
### 修改 Kind 裡面的 Kubernete 版本
```
cd ~/workspace/kind

vim multi-nodes.yaml
```
參考文件
https://kind.sigs.k8s.io/docs/user/quick-start/#instal![](https://hackmd.io/_uploads/BJ9cEZ-n3.png)
lation

```
# three node (two workers) cluster config
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  image: kindest/node:v1.26.6@sha256:6e2d8b28a5b601defe327b98bd1c2d1930b49e5d8c512e1895099e4504007adb
- role: worker
  image: kindest/node:v1.26.6@sha256:6e2d8b28a5b601defe327b98bd1c2d1930b49e5d8c512e1895099e4504007adb
- role: worker
  image: kindest/node:v1.26.6@sha256:6e2d8b28a5b601defe327b98bd1c2d1930b49e5d8c512e1895099e4504007adb
```
版本參考
https://github.com/kubernetes-sigs/ki![](https://hackmd.io/_uploads/BkE_4bZ3h.png)
nd/releases

## Lab 3. 本機開發(下列練習以 ~/workspace/spring 為基礎路徑)

### 安裝 SDKMAN
```gherkin=
curl -s "https://get.sdkman.io" | bash
#
source "$HOME/.sdkman/bin/sdkman-init.sh"
#
sdk version
#
sdk list java
#
sdk install java 17.0.6-ms
#
sdk default java 17.0.6-ms
```

### 取得程式碼
```gherkin=
mkdir ~/workspace

cd ~/workspace

git clone git@github.com:ChunPingWang/spring-on-k8s-tutorial.git
```


1. 開啟兩個 terminal
2. 在第一個 terminal，跳到 quoters(做為 producer，用來提供資訊源)
```gherkin=
cd ~/workspace/spring/quoters

#編譯並打包程式碼
./mvnw clean package

#查看成果
ls -al target/

#應該會出現 jar 檔
java -jar target/quoters-incorporated-0.0.1-SNAPSHOT.jar
##或
./mvnw spring-boot:run
```

3. 在第二個 terminal，跳到路徑 consuming-rest
```gherkin=
#打 API，讀取 quoters 內容
curl  http://localhost:8080/api/random | jq  
#或開啟瀏覽器輸入
----------
http://localhost:8080/api/random
http://localhost:8080/api/1
http://localhost:8080/api/2
```
> 取得結果如下：
![](https://i.imgur.com/DjrIMkJ.png)

```gherkin=
cd  ~/workspace/spring/consuming-rest

# -DskipTests 跳過 JUnit 測試
./mvnw clean package -Dmaven.test.skip=true

java -jar target/consuming-rest-complete-0.0.1-SNAPSHOT.jar
##或
./mvnw spring-boot:run

```
#### 問題：consuming-rest 使用哪個 port?從哪裡改？
>增加一個 folder src/resource
>增加一個檔案 application.properties
![](https://i.imgur.com/svhX0LD.png)
>檔案內加一行 server.port=8081
```gherkin=
server.port=8081
```
>或是
>增加一個 folder src/resource
>增加一個檔案 application.yaml
>檔案內加
```gherkin=
server.port:
    8081
```
## Lab 4. 增加 Web-base 查詢頁面
>增加一個 class src/java/com/example/consumingrest/WebController.java
![](https://i.imgur.com/JKQEpz5.png)
>內容如下：
```gherkin=
package com.example.consumingrest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class WebController {
    private static final Logger log = LoggerFactory.getLogger(WebController.class);
    @Autowired
    private RestTemplate restTemplate;
    @GetMapping("/")
    public Quote getQuote() {
        Quote quote =  restTemplate.getForObject(
                "http://127.0.0.1:8080/api/random", Quote.class);
        log.info(quote.toString());
        return quote;
    }
}
```

## Lab5. 在K8s部署與執行
![](https://i.imgur.com/Xxw4cZx.png)

### Step 1，以 quoters 服務為例
![](https://i.imgur.com/Vg5QVlt.png)
```gherkin=

./mvnw clean package

#確認編譯結果
ls -al target/*.jar
```

### Step 2
![](https://i.imgur.com/cZ3paMZ.png)
```gherkin=
#打包方式如下：
# Dockerfile
https://spring.io/guides/topicals/spring-boot-docker/   
#Google JIB
https://cloud.google.com/java/getting-started/jib
#Buildpack pack
https://buildpacks.io/
#Maven
https://www.baeldung.com/spring-boot-docker-images
```

### Step 3
![](https://i.imgur.com/Jcs32Sm.png)
```gherkin=
#不測試，指定 Docker Tag；視個人 Docker Hub帳號，請修改 cpingwang 
./mvnw spring-boot:build-image -Dmaven.test.skip=true -Dspring-boot.build-image.imageName=cpingwang/restapp
# 不測試，不指定 Docker Tag
./mvnw spring-boot:build-image -Dmaven.test.skip=true
```

### Step 4
![](https://i.imgur.com/AvV1NNF.png)
```gherkin=
docker images

docker push cpingwang/restapp
```


### Step 5
![](https://i.imgur.com/CT6pJW3.png)
```gherkin=
#建立 namespace se-tap
kubectl create ns se-tap

#產生 quoters服務的deployment檔
kubectl create deploy quoters --image=cpingwang/quoters -n se-tap --dry-run=client -o yaml > deploy.yml


kubectl apply -f deploy.yml
#產生 quoters服務的ClusterIP service檔
kubectl expose deploy/quoters --type=ClusterIP --port=8080 --target-port=8080 -n se-tap --dry-run=client -o yaml > cluster_ip_svc.yml

kubectl apply -f cluster_ip_svc.yml

#或產生 quoters服務的LoadBalancer service檔，二擇一
kubectl expose deploy/quoters --type=LoadBalancer --port=8080 --target-port=8080 -n se-tap --dry-run=client -o yaml > load_balancer_svc.yml


kubectl apply -f load_balancer_svc.yml
```

### Step 6
![](https://i.imgur.com/0QYUlfS.png)
```gherkin=
#取得全部deployments
kubectl get deploy -A

#取得全部 pods
kubectl get pods -A

#部署 quoters服務
kubectl apply –f deploy.yml

#查看 deployment qouters 日誌
kubectl logs deploy/quoters -n se-tap

#查看 deployment quoters 資訊
kubectl describe deploy/quoters -n se-tap
```

### Step 7
![](https://i.imgur.com/Yq0tMOz.png)
```gherkin=
#查看 expose quoters 之前的service狀況
kubectl get svc -A

#expose quoter
kubectl –f load_balancer_svc.yml

#查看 expose quoters 之後的service狀況
kubectl get svc -A
```
### Step 8
![](https://i.imgur.com/sl48APx.png)
```gherkin=
# 如果SVC 是 CluserIP 可以用 forward service quoters 到 port 8080
kubectl port-forward svc/quoters -n se-tap 8080:8080

#開啟另外一個 terminal

curl  http://localhost:8080/api/random | jq
```

### Step 6: Consuming Rest 服務部署流程
> 先行確認 svc/quoters 的 FQDN
> 儲存下列成 ~/workspace/kind/dnsutils.yaml
```gherkin=
apiVersion: v1
kind: Pod
metadata:
  name: dnsutils
  namespace: default
spec:
  containers:
  - name: dnsutils
    image: cpingwang/alpine_dnsutils
    command:
      - sleep
      - "infinity"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always
  dnsConfig:
    searches:
      - default.svc.cluster.local
      - svc.cluster.local
      - cluster.local
    options:
      - name: ndots
        value: "2"
      - name: edns0
```

```gherkin=
#部署 DNSUtil 
kubectl apply -f ~/workspace/kind/dnsutils.yaml

#確認部署完成
kubectl get pods dnsutils
#透過 DNSUtil 確認quoters的 FQDN
kubectl exec pod/dnsutils  -it -- nslookup quoters.se-tap.svc.cluster.local 
```
![](https://hackmd.io/_uploads/BJ1gaX-h2.png)

```gherkin=
(至ConsumingRestApplication.java與 WebController.java，將 quoters ip 換成 quoters.se-tap.svc.cluster.local)
```
> Cluster IP 可用 Port-Forward 
```gherkin=
#
kubectl create deploy rest --image=cpingwang/restapp -n se-tap --dry-run=client -o yaml > deploy.yml 
#
kubectl apply –f deploy.yml
#
kubectl expose deploy/rest --type ClusterIP --port 8081 --target-port 8081 -n se-tap --dry-run=client -o yaml > svc.yml
#
kubectl apply –f svc.yml
#
kubectl logs deploy/rest -n se-tap
#
kubectl port-forward svc/rest -n se-tap 8081:8081
```
> Load Balancer 將可以自動取得對外 IP
```gherkin=
#
kubectl expose deploy/rest --type LoadBalancer --port 8081 --target-port 8081 -n se-tap --dry-run=client -o yaml > load_balancer_svc.yml
#
kubectl apply –f load_balancer_svc.yml
#
kubectl logs deploy/rest -n se-tap
#
```
![](https://hackmd.io/_uploads/Hy-cKN-33.png)
> 可用 Browser 查詢，即可查詢
![](https://hackmd.io/_uploads/rkzb5N-2h.png)


## Lab7. Carvel
Deploying Kubernetes Applications with ytt, kbld, and kapp
```gherkin=
https://carvel.dev/blog/deploying-apps-with-ytt-kbld-kapp/
```

## 學習資源
Tanzu Acdemy
https://tanzu.academy/

Kube Academy
https://kube.academy/

Spring Academy
https://spring.academy/

Coffee + Software with Josh Long
https://www.youtube.com/@coffeesoftware

VMware Tanzu Developer Center
https://tanzu.vmware.com/developer/


Developing microservices with aggregates - Chris Richardson
https://youtu.be/7kX3fs0pWwc

即將失傳的古老技藝 Vim
高見龍
https://www.youtube.com/playlist?list=PLBd8JGCAcUAH56L2CYF7SmWJYKwHQYUDI


## 推薦書籍
無瑕的程式碼－敏捷完整篇－物件導向原則、設計模式與 C# 實踐
https://www.tenlong.com.tw/products/9789864342099?list_name=rd

Clean Architecture
https://www.tenlong.com.tw/products/9789864342945?list_name=srh

單元測試的藝術, 2/e
https://www.tenlong.com.tw/products/9789864342471?list_name=srh

深入淺出設計模式, 2/e
https://www.tenlong.com.tw/products/9789865029364?list_name=srh

微服務架構設計模式
https://www.tenlong.com.tw/products/9787111624127?list_name=srh

Kent Beck 的測試驅動開發：案例導向的逐步解決之道
https://www.tenlong.com.tw/products/9789864345618?list_name=srh


###### tags: `Workshop` `Kubernetes` `Documentation`