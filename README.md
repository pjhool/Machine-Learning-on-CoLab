# machine-learning  정리 


## SSH to Colab
```
#Generate root password
import random, string
password = ''.join(random.choice(string.ascii_letters + string.digits) for i in range(20))

#Download ngrok
! wget -q -c -nc https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
! unzip -qq -n ngrok-stable-linux-amd64.zip
#Setup sshd
! apt-get install -qq -o=Dpkg::Use-Pty=0 openssh-server pwgen > /dev/null
#Set root password
! echo root:$password | chpasswd
! mkdir -p /var/run/sshd
! echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
! echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
! echo "LD_LIBRARY_PATH=/usr/lib64-nvidia" >> /root/.bashrc
! echo "export LD_LIBRARY_PATH" >> /root/.bashrc

#Run sshd
get_ipython().system_raw('/usr/sbin/sshd -D &')

#Ask token
print("Copy authtoken from https://dashboard.ngrok.com/auth")
import getpass
authtoken = getpass.getpass()

#Create tunnel
get_ipython().system_raw('./ngrok authtoken ' + authtoken + ' && ./ngrok tcp 22 &')
#Print root password
print("Root password: {}".format(password))
#Get public address
! curl -s http://localhost:4040/api/tunnels | python3 -c \
    "import sys, json; result = json.load(sys.stdin)['tunnels'][0]['public_url'].replace('/', '').split(':');\
     print('ssh root@' + result[1] + ' -p ' + result[2]) \
    " 
```    
 
## Google Drive

```shell
!apt-get install -y -qq software-properties-common python-software-properties module-init-tools
!add-apt-repository -y ppa:alessandro-strada/ppa 2>&1 > /dev/null
!apt-get update -qq 2>&1 > /dev/null
!apt-get -y install -qq google-drive-ocamlfuse fuse

from google.colab import auth
auth.authenticate_user()
from oauth2client.client import GoogleCredentials
creds = GoogleCredentials.get_application_default()
import getpass

!google-drive-ocamlfuse -headless -id={creds.client_id} -secret={creds.client_secret} < /dev/null 2>&1 | grep URL
vcode = getpass.getpass()
!echo {vcode} | google-drive-ocamlfuse -headless -id={creds.client_id} -secret={creds.client_secret}
```

#####  


classDiagram
    class Product {
        - product_id: String
        - name: String
        - description: String
        - price: Float
        + get_details(): String
    }

    class Inventory {
        - inventory_id: String
        - product_id: String
        - quantity: Int
        + increase(quantity: Int): void
        + decrease(quantity: Int): void
    }

    class StockInService {
        - inventory_repo: InventoryRepository
        + stock_in(product_id: String, quantity: Int): void
        + publish_event(event: ProductStockedIn): void
    }

    class StockOutService {
        - inventory_repo: InventoryRepository
        + stock_out(product_id: String, quantity: Int): void
        + publish_event(event: ProductStockedOut): void
    }

    class InventoryAuditService {
        + audit_inventory(): void
    }

    class ProductStockedIn {
        - product_id: String
        - quantity: Int
    }

    class ProductStockedOut {
        - product_id: String
        - quantity: Int
    }

    Product "1" -- "*" Inventory : 관리
    Inventory "1" -- "*" StockInService : 입고
    Inventory "1" -- "*" StockOutService : 출고
    Inventory "1" -- "*" InventoryAuditService : 감사
    StockInService --> ProductStockedIn : 발행
    StockOutService --> ProductStockedOut : 발행
    
