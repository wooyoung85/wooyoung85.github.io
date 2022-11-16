> 윈도우 WSL Ubuntu 환경에서 진행하였습니다.  
> 자세한 내용은 [🚀 WSL 사용하기](https://wooyoung85.tistory.com/70) 를 참고해 주세요^^

<br/>

# Prerequisite

- 환경변수

  ```bash
  RESOURCE_GROUP="polling-rg"
  DB_NAME="polling-$(date +%Y%m%d)-db"
  DB_ADMIN_USER="pollingdbadmin"
  DB_ADMIN_PASSWORD="qwer12#$"
  DB_SKU="GP_Gen5_2"
  LOCATION="koreacentral"
  ACR_NAME="pollingacr"
  APP_SERVICE_PLAN="polling-appservice-plan"
  POLLING_APP_SERVER="polling-app-server"
  POLLING_APP_CLIENT="polling-app-client"
  MY_IP_ADDRESS=$(curl -s http://ipinfo.io/ip)
  ```

- 리소스 그룹 생성
  ```bash
  az group create --name $RESOURCE_GROUP --location koreacentral
  ```
- 소스코드 다운로드

  ```bash
  git clone https://github.com/wooyoung85/polling-app
  ```

  > 소스코드 및 작업폴더 경로 : `~/polling-app` (`/home/<username>/polling-app`)

# Azure Database for MySQL

```bash
# DB 생성
az mysql server create --resource-group $RESOURCE_GROUP --name $DB_NAME \
--admin-user $DB_ADMIN_USER --admin-password $DB_ADMIN_PASSWORD \
--location $LOCATION --sku-name $DB_SKU

# Azure 서비스 접근 가능
az mysql server firewall-rule create --resource-group $RESOURCE_GROUP --server-name $DB_NAME \
--name AllowAllAzureIps --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0

# 내 IP 접근 가능
az mysql server firewall-rule create --resource-group $RESOURCE_GROUP --server-name $DB_NAME \
--name ClientIPAddress_wooyoung --start-ip-address $MY_IP_ADDRESS --end-ip-address $MY_IP_ADDRESS

# mysql client 설치 (필요시에만)
sudo apt-get install mysql-client

# MY SQL SSL 접속을 위한 인증서 다운로드
##301 redirect 를 고려하여 -L 옵션 적용
curl -L -o ~/polling-app/$DB_NAME.crt.pem https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem

# 테이블 생성
echo "create database polling_app;
      create table polling_app.roles (id bigint auto_increment primary key, name varchar(60) null, unique (name));
      create table polling_app.choices (id bigint not null auto_increment, text varchar(40), poll_id bigint not null, primary key (id)) engine=InnoDB;
      create table polling_app.polls (id bigint not null auto_increment, created_at datetime not null, updated_at datetime not null, created_by bigint, updated_by bigint, expiration_date_time datetime not null, question varchar(140), primary key (id)) engine=InnoDB;
      create table polling_app.user_roles (user_id bigint not null, role_id bigint not null, primary key (user_id, role_id)) engine=InnoDB;
      create table polling_app.users (id bigint not null auto_increment, created_at datetime not null, updated_at datetime not null, email varchar(40), name varchar(40), password varchar(100), username varchar(15), primary key (id)) engine=InnoDB;
      create table polling_app.votes (id bigint not null auto_increment, created_at datetime not null, updated_at datetime not null, choice_id bigint not null, poll_id bigint not null, user_id bigint not null, primary key (id)) engine=InnoDB;
      alter table polling_app.roles add constraint UK_nb4h0p6txrmfc0xbrd1kglp9t unique (name);
      alter table polling_app.users add constraint UKr43af9ap4edm43mmtq01oddj6 unique (username);
      alter table polling_app.users add constraint UK6dotkott2kjsp8vw4d0m25fb7 unique (email);
      alter table polling_app.votes add constraint UK8um9h2wxsdjrgx3rjjwvny676 unique (poll_id, user_id);
      alter table polling_app.choices add constraint FK1i68hpih40n447wqx4lpef6ot foreign key (poll_id) references polls (id);
      alter table polling_app.user_roles add constraint FKh8ciramu9cc9q3qcqiv4ue8a6 foreign key (role_id) references roles (id);
      alter table polling_app.user_roles add constraint FKhfh9dx7w3ubf1co1vdev94g3f foreign key (user_id) references users (id);
      alter table polling_app.votes add constraint FKomskymhxde3qq9mcukyp1puio foreign key (choice_id) references choices (id);
      alter table polling_app.votes add constraint FK7trt3uyihr4g13hva9d31puxg foreign key (poll_id) references polls (id);
      alter table polling_app.votes add constraint FKli4uj3ic2vypf5pialchj925e foreign key (user_id) references users (id);
      insert into polling_app.roles(name) values('ROLE_USER');
      insert into polling_app.roles(name) values('ROLE_ADMIN');" \
      | mysql -u$DB_ADMIN_USER@$DB_NAME -p$DB_ADMIN_PASSWORD -h$DB_NAME.mysql.database.azure.com --ssl --ssl-ca=/home/wooyoung/pollingapp/$DB_NAME.crt.pem -v

# DB 접속
mysql -u$DB_ADMIN_USER@$DB_NAME -p$DB_ADMIN_PASSWORD -h$DB_NAME.mysql.database.azure.com -Dpolling_app --ssl --ssl-ca=/home/wooyoung/polling-app/$DB_NAME.crt.pem

MySQL [polling_app]> show tables;
+-----------------------+
| Tables_in_polling_app |
+-----------------------+
| choices               |
| polls                 |
| roles                 |
| user_roles            |
| users                 |
| votes                 |
+-----------------------+
6 rows in set (0.004 sec)

MySQL [polling_app]> exit
```

> ⭐ `Azure Database for MySQL` 서버와 SSL 통신하는 데 필요한 인증서는 https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem 에서 다운로드 받을 수 있음

> 😅 mysql client 접속 시 SSL certificate 파일 경로는 꼭 절대 경로로 설정해 주어야 함
>
> ```bash
> # 상대 경로를 사용하면 에러남
> mysql -u $DB_ADMIN_USER@$DB_NAME -p$DB_ADMIN_PASSWORD -h $DB_NAME.mysql.database.azure.com --ssl --ssl-ca=~/pollingapp/$DB_NAME.>crt.pem
> ERROR 2026 (HY000): SSL connection error: Error while reading file.
> ```

# Azure Container Registry

```bash
az acr create --resource-group $RESOURCE_GROUP --name $ACR_NAME --sku Standard --admin-enabled true
az acr login --name $ACR_NAME
```

# Polling Application 도커 이미지 Build And Push

```bash
cd ~/polling-app/$POLLING_APP_SERVER/
az acr build --registry $ACR_NAME --image "$ACR_NAME.azurecr.io/$POLLING_APP_SERVER" --file Dockerfile_MultiStage .

cd ~/polling-app/$POLLING_APP_CLIENT/
az acr build --registry $ACR_NAME --image $ACR_NAME.azurecr.io/$POLLING_APP_CLIENT --build-arg REACT_APP_API_BASE_URL="https://$POLLING_APP_SERVER.azurewebsites.net/api" --file Dockerfile_MultiStage .
```

# Azure Web App for Containers

```bash
#Azure Web App for Containers 생성
az appservice plan create -g $RESOURCE_GROUP -n $APP_SERVICE_PLAN --is-linux --sku B1

az webapp create -g $RESOURCE_GROUP -p $APP_SERVICE_PLAN -n $POLLING_APP_SERVER --deployment-container-image-name $ACR_NAME.azurecr.io/$POLLING_APP_SERVER

az webapp config appsettings set -g $RESOURCE_GROUP -n $POLLING_APP_SERVER --settings WEBSITES_PORT=8080 SPRING_DATASOURCE_URL="jdbc:mysql://$DB_NAME.mysql.database.azure.com:3306/polling_app?useSSL=true&serverTimezone=UTC&useLegacyDatetimeCode=false" SPRING_DATASOURCE_USERNAME="$DB_ADMIN_USER@$DB_NAME" SPRING_DATASOURCE_PASSWORD="$DB_ADMIN_PASSWORD"
az webapp restart -g $RESOURCE_GROUP -n $POLLING_APP_SERVER

az webapp create -g $RESOURCE_GROUP -p $APP_SERVICE_PLAN -n $POLLING_APP_CLIENT -i $ACR_NAME.azurecr.io/$POLLING_APP_CLIENT
az webapp restart -g $RESOURCE_GROUP -n $POLLING_APP_CLIENT
```

# Test

- 브라우저에서 polling-app 사이트 접속 👉 [https://polling-app-client.azurewebsites.net](https://polling-app-client.azurewebsites.net)

- 회원가입 및 로그인 후 투표 기능 테스트

# 리소스 그룹 삭제

```bash
az group delete --name $RESOURCE_GROUP
```

## 참고자료

[Configure SSL connectivity in your application to securely connect to Azure Database for MySQL](https://learn.microsoft.com/en-us/azure/mysql/single-server/how-to-configure-ssl#step-1-obtain-ssl-certificate)  
[MySQL and SSL connection failing ERROR 2026 (HY000)](https://stackoverflow.com/questions/20459056/mysql-and-ssl-connection-failing-error-2026-hy000)  
[Spring Boot Docker](https://spring.io/guides/topicals/spring-boot-docker/)
