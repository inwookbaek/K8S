# A. ubuntu 설치
## 1. VMWare - Ubuntu Server
<img src="./images/ubuntu install/스크린샷 2025-07-08 221616.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 221659.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 221711.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 221719.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 221733.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 221739.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 221744.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 221749.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 221752.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 221757.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 221820.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 221827.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 221834.png" width="600" height="400">

## 2. ubuntu install
<img src="./images/ubuntu install/스크린샷 2025-07-08 221904.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 222133.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 222156.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 222213.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 222231.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 222251.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 222314.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 222516.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 222548.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 222614.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 222721.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 222742.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 222923.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 222946.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 223436.png" width="600" height="400">
<img src="./images/ubuntu install/스크린샷 2025-07-08 223741.png" width="600" height="400">

##### 1. VMWare 설정 - Shared Folder 
##### 2. ubuntu 설치후 스냅샷 - ubuntu initial install

## 3. ssh
```bash

# openssh 설치확인
dpkg -l | grep openssh-server
# 설치 되었다면 
# 출력결과)
# ii  openssh-server  1:9.2p1-2ubuntu2   amd64   secure shell (SSH) server

# OpenSSH를 설치했지만 서비스는 ssh.service로 작동합니다. 이미지의 Cgroup부분을 면 /system.slice/ssh.service로 
# 나와 있다.실제 설치된 패키지를 확인해 보겠습니다. apt list 명령을 활용해서 openssh와 관련해 설치된 패키지를 
# 확인하면 3개의 패키지가(openssh-client, openssh-server, open-sftp-server) 설치되어 있다.
sudo apt list openssh*

# 설치되지 않았다면 openssh 설치
sudo apt update && sudo apt install openssh-server

# openssh가 실행중인지 확인
sudo systemctl status ssh
# 출력결과)
# ● ssh.service - OpenBSD Secure Shell server
#      Loaded: loaded (/lib/systemd/system/ssh.service; enabled)
#      Active: active (running) 또는 inactive(dead)

# Active : inactive일 경우 
sudo systemctl start ssh
# 출력결과)
# ... 생략 ...
# ... port 22 확인

# ssh환경설정 - port번호 변경
sudo vi /etc/ssh/sshd_config

# config 수정 이후에는 ssh를 재시작해주어야 한다.
sudo service sshd restart
```

# B. MobaXterm 설정

<img src="./images/ubuntu install/스크린샷 2025-07-09 092015.png" width="600" height="400">

# C. ssh 환경설정

🎯 목표
* Master 서버에서 Work1, Work2, Work3 서버로 비밀번호 없이 SSH 접속
* RSA 키페어를 생성하여 공개키를 각 Worker 서버에 배포

✅ 단계별 설정 방법

#### 1. Master 서버에서 RSA 키 생성
```bash
ssh-keygen -t rsa -b 4096 -C "master@ssh"
``` 
* 경로는 기본값(~/.ssh/id_rsa)으로 Enter 입력
* 패스프레이즈는 원할 경우 설정, 아니면 그냥 Enter
* 생성되는 파일:

| 파일 경로               | 설명                      |
| ------------------- | ----------------------- |
| `~/.ssh/id_rsa`     | **개인키** (절대 유출 금지)      |
| `~/.ssh/id_rsa.pub` | **공개키** (Worker 서버에 배포) |


#### 2. 공개키를 각 Worker 서버에 복사
* 각 Work 서버에 공개키를 복사(예: 사용자 계정이 ubuntu일 경우)

```bash
ssh-copy-id ubuntu@work1_ip
ssh-copy-id ubuntu@work2_ip
ssh-copy-id ubuntu@work3_ip
# ❗ ssh-copy-id 명령어가 없다면, 수동으로:

cat ~/.ssh/id_rsa.pub | ssh ubuntu@work1_ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
# 각 Work 서버에 대해 반복합니다.
```

#### 3. Master → Worker SSH 접속 테스트
```bash
ssh ubuntu@work1_ip
ssh ubuntu@work2_ip
ssh ubuntu@work3_ip
```
✅ 비밀번호 없이 로그인되면 설정 성공입니다.

🛠️ 권한 확인 (Worker 서버에서)
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

🔁 구조 요약
```less
        [ Master ]
            |
     ---------------------
     |        |         |
  [Work1]  [Work2]   [Work3]
 (공개키를 각 워커 서버에 복사)
```

✅ 추가 팁
* ssh-keygen은 한 번만 실행하면 되고, 생성한 키는 여러 서버에 복사 가능

#### 3. ✅ 공개키 자동 배포 스크립트 (ssh_key_distribute.sh)
```bash
#!/bin/bash

# 📍 설정: 복사 대상 워커 서버 목록 (IP 또는 도메인)
WORKERS=("work1_ip" "work2_ip" "work3_ip")

# 📍 SSH 접속 사용자 이름
SSH_USER="ubuntu"

# 📍 SSH 키 경로 (기본값: ~/.ssh/id_rsa.pub)
PUB_KEY="$HOME/.ssh/id_rsa.pub"

# ✅ 키가 없으면 생성
if [ ! -f "$PUB_KEY" ]; then
    echo "[INFO] RSA 키가 존재하지 않습니다. 새로 생성합니다..."
    ssh-keygen -t rsa -b 4096 -C "$USER@auto" -N "" -f "$HOME/.ssh/id_rsa"
fi

# 🔄 반복: 모든 워커에 키 복사
for HOST in "${WORKERS[@]}"; do
    echo "🔗 $HOST 서버에 공개키 배포 중..."
    ssh-copy-id -i "$PUB_KEY" "${SSH_USER}@${HOST}"
    
    if [ $? -eq 0 ]; then
        echo "✅ $HOST: 공개키 복사 성공"
    else
        echo "❌ $HOST: 공개키 복사 실패 (SSH 비밀번호 입력 실패 또는 네트워크 문제)"
    fi
done

echo "🎉 모든 서버에 키 배포 완료"
```

#### 📌 사용 방법
위 내용을 ssh_key_distribute.sh 라는 파일로 저장합니다.

```bash
nano ssh_key_distribute.sh
```
실행 권한 부여:
```bash
chmod +x ssh_key_distribute.sh
```
스크립트 실행:

```bash
./ssh_key_distribute.sh
```

#### 📌 주의사항
* 첫 실행 시 각 워커 서버의 비밀번호를 물어볼 수 있습니다.
* 이후에는 비밀번호 없이 SSH 접속 가능해집니다.
*WORKERS 배열에 실제 서버 IP 또는 도메인명을 입력하세요.

### 🧪 접속 테스트
스크립트 실행 후 Master에서:

🧪 접속 테스트
```bash
ssh ubuntu@work1_ip
# 비밀번호 없이 접속되면 성공입니다 🎉
```

# D. Network 설정

|     구분     | master | work1 | work2 | work3 |
| ------------ | ------------ |  ------------ | ------------ | ------------ |   
| `hostname`  | master.gilcns.com | work1.gilcns.com | work2.gilcns.com | work3.gilcns.com |
| `ip address`  | 192.168.164.130/24 | 192.168.164.131/24 | 192.168.164.132/24| 192.168.164.133/24

## 1. ip 변경
```bash
ip addr
cd /etc/netplan
ls -l
sudo vi 50-cloud-init.yaml
```

##### 50-cloud-init.yaml
###### **(중요)**  반드시 via: 192.168.164.2 # VMWare Gateway IP 지정
```yaml
# 50-cloud-init.yaml
network:
    version: 2
    ethernets:
      ens33:
        dhcp4: no
        addresses:
        # master -> 192.168.164.130/24,  work1~3 -> 131/~133/24
        - 192.168.164.131/24
        routes:
        - to: default
          via: 192.168.164.2 # VMWare Gateway IP 지정
        nameservers:
          addresses: [8.8.8.8, 8.8.4.4]
 ```
📖 각 항목 설명
| 항목                     | 의미                                                         |
| ---------------------- | ---------------------------------------------------------- |
| `network:`             | 네트워크 설정의 루트 블록 시작                                          |
| `version: 2`           | Netplan 구성 파일의 형식 버전. 대부분의 Ubuntu 버전에서는 `2` 사용             |
| `ethernets:`           | 유선 네트워크 인터페이스들을 설정하는 섹션                                    |
| `ens33:`               | 설정할 실제 **이더넷 장치 이름**. `ip link`로 이름 확인 가능.                 |
| `dhcp4: no`            | IPv4 주소를 DHCP로 자동 받지 않도록 설정 (수동 IP 사용)                     |
| `addresses:`           | 수동으로 설정할 IP 주소 목록                                          |
| `- 192.168.164.131/24` | 이 머신의 IP 주소. CIDR 표기법으로, `/24`는 서브넷 마스크 `255.255.255.0` 의미 |
| `routes:`              | 이 머신에서 나가는 트래픽의 경로(라우팅) 설정                                 |
| `- to: default`        | 기본 경로 (인터넷 등 모든 외부 네트워크를 향한 경로) 설정                         |
| `via: 192.168.164.2`   | 위 "default" 경로로 보낼 때 사용할 **게이트웨이(라우터)** 주소                 |
| `nameservers:`         | DNS 서버 설정 (도메인 이름을 IP로 변환하는 서버)                            |
| `addresses:`           | 사용할 DNS 서버 목록. 여기선 Google DNS 사용 (`8.8.8.8`, `8.8.4.4`)    |


## 2. IP 적용

```bash
sudo netplan apply
```

## 3. hostname 변경

```bash
hostname
hostnamectl
sudo hostnamectl set-hostname master.test.com # work1~3.gilcns.com
```

## 4. ping 테스트 
### 🧰 ping 명령어의 주요 옵션 (Ubuntu 기준)
| 옵션         | 설명                                            |
| ---------- | --------------------------------------------- |
| `-c <횟수>`  | 핑을 보낼 횟수를 지정합니다 (예: `-c 4` → 4번 핑 전송)         |
| `-i <간격>`  | 핑을 보내는 간격(초 단위, 기본 1초)                        |
| `-t <ttl>` | 패킷의 TTL(Time To Live) 값을 설정합니다                |
| `-s <크기>`  | 보낼 패킷의 크기(바이트) 설정 (기본 56바이트 → 실제 ICMP는 64바이트) |
| `-q`       | 조용한 출력 (결과 요약만 표시)                            |
| `-W <초>`   | 각 응답을 기다리는 시간(초 단위)                           |
| `-f`       | 플러딩 핑 (매우 빠르게 핑을 보냄, 루트 권한 필요)                |
| `-n`       | 도메인 이름 해석 없이 숫자로만 출력                          |

```bash
ping -c 3 192.168.164.130
ping -c 3 192.168.164.131
ping -c 3 192.168.164.132
ping -c 3 192.168.164.133
````

```bash
sudo vi /etc/hosts

# 맨뒤에 추가
# 192.168.164.130 master.gilcns.com
# 192.168.164.131 work1.gilcns.com
# 192.168.164.132 work2.gilcns.com
# 192.168.164.133 work2.gilcns.com

ping -c 3 work1.gilcns.com
ping -c 3 work2.gilcns.com
ping -c 3 work3.gilcns.com
```
## 5. ssh 접속

### 1. 비밀번호로 접속
```bash
ssh gilbaek@192.168.164.131
exit
ssh gilbaek@work2.gilcns.com
exit
ssh gilbaek@work3.gilcns.com
exit
```

### 2. 공개키접속

#### 1️⃣ Worker 서버에 OpenSSH 설치
* Work1, Work2, Work3에서 아래 명령어 실행:

```bash
sudo apt update
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

#### 2️⃣ Master 서버에서 SSH 키 생성
* Master에서 실행:

```bash
ssh-keygen -t rsa -b 4096 -C "gilbaek@master.gilcns.com"
```

#### 🔐 ssh-keygen 명령어 옵션 설명
| 옵션           | 설명                | 예시 값                          | 비고                            |
| ------------ | ----------------- | ----------------------------- | ----------------------------- |
| `ssh-keygen` | SSH 키 생성 명령어      | -                             | RSA, ECDSA, ED25519 등 키 생성 가능 |
| `-t`         | 키의 타입(Type)을 지정   | `rsa`                         | RSA 알고리즘 사용 (가장 호환성이 높음)      |
| `-b`         | 키 길이(bit)를 지정     | `4096`                        | 보안 강화를 위해 4096bit 사용 권장       |
| `-C`         | 키에 주석(Comment) 추가 | `"gilbaek@master.gilcns.com"` | 키 식별 용도 (주로 이메일/호스트 이름)       |

#### 📁 생성되는 파일
| 파일 경로               | 설명                 | 공유 여부       |
| ------------------- | ------------------ | ----------- |
| `~/.ssh/id_rsa`     | 개인 키 (Private Key) | ❌ 절대 공유 금지  |
| `~/.ssh/id_rsa.pub` | 공개 키 (Public Key)  | ✅ 서버에 복사 가능 |


### 3️⃣ SSH 공개키를 Worker 서버로 복사
* Master에서 아래 명령어를 각각 실행:
* work서버 저장위치 : /home/gilbaek/.ssh/authorized_keys
```bash
ssh-copy-id gilbaek@192.168.164.131
ssh-copy-id gilbaek@work2.gilcns.com
ssh-copy-id gilbaek@work3.gilcns.com
```

### 4️⃣ SSH 접속 테스트
* Master에서 접속 확인:
```bash
ssh gilbaek@192.168.164.131
ssh gilbaek@work2.gilcns.com
ssh gilbaek@work3.gilcns.com
```
### 5️⃣ (선택) SSH 별칭 설정
* Master 서버의 ~/.ssh/config 파일에 아래 내용 추가:

```ssh
Host work1
    HostName 192.168.164.131
    User gilbaek

Host work2
    HostName 192.168.164.132
    User gilbaek

Host work3
    HostName 192.168.164.133
    User gilbaek
```

* 별칭으로 접속
```bash
ssh work1
ssh work2
ssh work3
```

### 🔒 보안 팁 (선택)
* 방화벽에서 SSH 포트 허용:

```bash
sudo ufw allow ssh

# 패스워드 인증 비활성화 (키 인증만 허용):
sudo vi /etc/ssh/sshd_config
# 다음 항목 수정 또는 추가: PasswordAuthentication no

# 설정 적용:
sudo systemctl restart ssh
```

### 📜 자동 설정 스크립트

#### setup_ssh_workers.sh
```bash
#!/bin/bash

# Worker 서버 목록 (IP 또는 호스트 이름)
WORKERS=("192.168.1.101" "192.168.1.102" "192.168.1.103")
USER="ubuntu"

# SSH 키가 없으면 생성
if [ ! -f "$HOME/.ssh/id_rsa.pub" ]; then
    echo "[+] SSH 키 생성 중..."
    ssh-keygen -t rsa -b 4096 -C "$USER@master" -N "" -f "$HOME/.ssh/id_rsa"
else
    echo "[*] SSH 키가 이미 존재합니다. 건너뜁니다."
fi

# Worker 서버에 SSH 키 배포
for i in "${!WORKERS[@]}"; do
    HOST="${WORKERS[$i]}"
    echo "[+] $HOST에 SSH 키 복사 중..."
    ssh-copy-id "$USER@$HOST"
done

# ~/.ssh/config에 호스트 별칭 추가
CONFIG="$HOME/.ssh/config"

echo "[+] SSH 설정파일에 호스트 별칭 추가 중..."

for i in "${!WORKERS[@]}"; do
    HOST="${WORKERS[$i]}"
    INDEX=$((i+1))
    ALIAS="work$INDEX"

    # 이미 항목이 존재하는지 확인
    if grep -q "Host $ALIAS" "$CONFIG" 2>/dev/null; then
        echo "[*] $ALIAS 별칭이 이미 존재합니다. 건너뜁니다."
    else
        echo -e "\nHost $ALIAS\n    HostName $HOST\n    User $USER" >> "$CONFIG"
        echo "[+] $ALIAS 설정 추가됨."
    fi
done

echo "[✅] 설정 완료! 이제 다음과 같이 접속할 수 있습니다:"
echo "    ssh work1"
echo "    ssh work2"
echo "    ssh work3"
```

* 실행
```bash
chmod +x setup_ssh_workers.sh
./setup_ssh_workers.sh
```
