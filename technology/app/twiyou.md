# twiyou

[twiyou](https://github.com/disksing/twiyou)（推油）是一款推特好友/推特数据监测工具。

## Docker编译

把这工具打包成了Docker镜像，上传了官方制品库，并提供了方便在K8S环节下作为CronJob运行的配置方案。

### twiyou.dockerfile
```bash
FROM alpine
RUN cd /tmp \
    && wget https://github.com/disksing/twiyou/releases/download/v0.1/twiyou_0.1_Linux_x86_64.tar.gz \
    && tar -xzvf twiyou_0.1_Linux_x86_64.tar.gz \
    && rm twiyou_0.1_Linux_x86_64.tar.gz

CMD echo Execute agaist DB ${DB_NAME} \
    && /tmp/twiyou
```

### Build image
```bash
docker build -f twiyou.dockerfile -t xiaowenz/twiyou .
```

### Execute Docker
```bash
docker run \
    -e DB_PASSWD='yourpassword' \
    -e DB_NAME=test \
    -e DB_USER='some.root' \
    -e DB_PORT=4000 \
    -e DB_HOST='some.instance.tidbcloud.com' \
    -e TWITTER_BEARER_TOKEN='YourTwitterToken' \
    -e TWITTER_USER_NAME='yourtwitterusername' \
twiyou 
```

### K8S CronJob - cj.yml

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
name: twiyou-cj
spec:
jobTemplate:
    metadata:
    name: twiyou-cj
    spec:
    template:
        spec:
        containers:
        - image: xiaowenz/twiyou
            name: twiyou-cj
            env:
            - name: DB_PASSWD
            valueFrom: 
                configMapKeyRef: 
                name: twiyou
                key: DB_PASSWD
            - name: DB_NAME
            valueFrom: 
                configMapKeyRef: 
                name: twiyou
                key: DB_NAME
            - name: DB_USER
            valueFrom: 
                configMapKeyRef: 
                name: twiyou
                key: DB_USER
            - name: DB_PORT
            valueFrom: 
                configMapKeyRef: 
                name: twiyou
                key: DB_PORT
            - name: DB_HOST
            valueFrom: 
                configMapKeyRef: 
                name: twiyou
                key: DB_HOST
            - name: TWITTER_BEARER_TOKEN
            valueFrom: 
                configMapKeyRef: 
                name: twiyou
                key: TWITTER_BEARER_TOKEN
            - name: TWITTER_USER_NAME
            valueFrom: 
                configMapKeyRef: 
                name: twiyou
                key: TWITTER_USER_NAME
        restartPolicy: OnFailure
schedule: "*/20 * * * *"
```

### K8S ConfigMap - cm.yml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
name: twiyou

data:
DB_PASSWD: 'yourpass'
DB_NAME: 'test'
DB_USER: 'some.root'
DB_PORT: '4000'
DB_HOST: 'some.instance.tidbcloud.com'
TWITTER_BEARER_TOKEN: 'YourTwitterToken'
TWITTER_USER_NAME: 'yourtwitterusername'
```

### K8S Install

```bash
kubectl apply -f cm.yml
kubectl apply -f cj.yml
```

> 注：在更正式的K8S环境中，建议使用secrets来管理关键变量而非明文的ConfigMap
