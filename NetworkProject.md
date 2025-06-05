# Network Project Simulating a Small enterprise
---
## **Requisites**
- Usage of the network address 10.10.0.0/23 for all the addressing needs
- It needs to have 3 branches
- In each branch there is 3 local networks
  - Sales: 32 PCs
  - Human Resources: 20 PCs
  - Administration: 10 PCs
- No layer 2 and 3 connectivity between local networks
- There are 2 servers made accessible for any PC in the entreprise, one for file sharing and another for a web page
- All Addressing is made through DHCP implemented in one router
- Conection between the entreprise and the ISP is made with PPPoE
- There must be implemented NAT to a unique address in the router connected to the internet
- Usage of ACLs
- Usage of OSPF for all addressing needs

--- 

## Topology
![image](https://user-images.githubusercontent.com/100608872/253207767-6e00223f-b22b-442d-ae27-e657e3e5bf51.png)
![image](https://user-images.githubusercontent.com/100608872/253208334-4f004c0a-2e9e-4518-b590-36f67f1fb828.png)

---

## Addressing

- Sede Leiria:
  - Rede Vendas: 10.10.0.0/26 (64 addresses)
  - Rede Recursos Humanos: 10.10.0.64/27 (32 addresses)
  - Rede Administra¸c˜ao: 10.10.0.96/28 (16 addresses)
- Filial Porto:
  - Rede Vendas: 10.10.0.112/26 (64 addresses)
  - Rede Recursos Humanos: 10.10.0.192/27 (32 addresses)
  - Rede Administra¸c˜ao: 10.10.0.224/28 (16 addresses)
- Filial Lisboa:
  - Rede Vendas: 10.10.1.0/26 (64 addresses)
  - Rede Recursos Humanos: 10.10.1.64/27 (32 addresses)
  - Rede Administra¸c˜ao: 10.10.1.112/27 (16 addresses)
- Servidores
  - Web: 10.10.50.0/30
  - FTP: 10.10.50.4/30
- FrameRelays
  - Router-Porto: 10.10.100.2
  - Router-Lisboa: 10.10.200.2
  - Leiria-Sede Router-Lisboa: 10.10.200.1
  - Leiria-Sede Router-Porto: 10.10.100.1

---

## Configurations

### ACL 

```Config
ip access-list standard 1
remark AclVendas
permit host 10.10.30.2
permit 10.10.50.0 0.0.0.3
permit 10.10.50.4 0.0.0.3
permit 10.10.0.0 0.0.0.63
permit 10.10.0.128 0.0.0.63
permit 10.10.1.0 0.0.0.63
deny any
ex

ip access-list standard 2
remark AclRec
permit host 10.10.30.2
permit 10.10.50.0 0.0.0.3
permit 10.10.50.4 0.0.0.3
permit 10.10.0.64 0.0.0.31
permit 10.10.0.192 0.0.0.31
permit 10.10.1.64 0.0.0.31
deny any
ex

ip access-list standard 3
remark AclAdmin
permit host 10.10.30.2
permit 10.10.50.0 0.0.0.3
permit 10.10.50.4 0.0.0.3
permit 10.10.0.96 0.0.0.15
permit 10.10.0.224 0.0.0.15
permit 10.10.1.96 0.0.0.15
deny any
ex

access-list 10 remark AclNat
access-list 10 permit 10.10.0.0 0.0.1.255
access-list 10 permit 10.10.50.0 0.0.0.7
access-list 10 deny any
ex
```

### DHCP

```config
ip dhcp excluded-address 10.10.1.62
ip dhcp excluded-address 10.10.1.94
ip dhcp excluded-address 10.10.1.110

ip dhcp excluded-address 10.10.0.190
ip dhcp excluded-address 10.10.0.222
ip dhcp excluded-address 10.10.0.238

ip dhcp excluded-address 10.10.30.1
ip dhcp excluded-address 10.10.0.62
ip dhcp excluded-address 10.10.0.94
ip dhcp excluded-address 10.10.0.110

ip dhcp excluded-address 10.10.50.2
ip dhcp excluded-address 10.10.50.6

ip dhcp pool Lan-Leiria-Vendas
default-router 10.10.0.62
network 10.10.0.0 255.255.255.192
exit

ip dhcp pool Lan-Leiria-RecursosHumanos
default-router 10.10.0.94
network 10.10.0.64 255.255.255.224
exit

ip dhcp pool Lan-Leiria-Administracao
default-router 10.10.0.110
network 10.10.0.96 255.255.255.240
exit

ip dhcp pool Lan-Lisboa-Vendas
default-router 10.10.1.62
network 10.10.1.0 255.255.255.192
exit

ip dhcp pool Lan-Lisboa-RecursosHum
default-router 10.10.1.94
network 10.10.1.64 255.255.255.224
exit

ip dhcp pool Lan-Lisboa-Administracao
default-router 10.10.1.110
network 10.10.1.96 255.255.255.240
exit

ip dhcp pool Lan-Porto-Vendas
default-router 10.10.0.190
network 10.10.0.128 255.255.255.192
exit

ip dhcp pool Lan-Porto-RecursosHumanos
default-router 10.10.0.222
network 10.10.0.192 255.255.255.224
exit

ip dhcp pool Lan-Porto-Administracao
default-router 10.10.0.238
network 10.10.0.224 255.255.255.240
exit

ip dhcp pool FTP-Server
default-router 10.10.50.2
network 10.10.50.0 255.255.255.252
exit

ip dhcp pool WEB-Server
default-router 10.10.50.6
network 10.10.50.4 255.255.255.252
exit
```

### NAT

```config
interface f0/0
ip nat inside
ex

interface f1/0
ip nat inside
ex

interface f1/1
ip nat inside
ex

interface f1/1.10
ip nat inside
ex

interface f1/1.20
ip nat inside
ex

interface f1/1.30
ip nat inside
ex

interface s0/0/0
ip nat inside
ex

interface s0/0/0.1
ip nat inside
ex

interface s0/0/0.2
ip nat inside
ex

interfacef0/1
ip nat outside
ex

ip nat inside source list 1 interface f0/1 overload
```

### PPPoE - Incomplete

- Client

```config
interface f0/1
pppoe enable
pppoe-client dial-pool-number 1
no sh
exit

interface Dialer 1
encapsulation ppp
mtu 1492
ip address negotiated
dialer pool 1
ppp authentication chap callin
ppp chap hostname cisco
ppp chap password class
exit

ip route 0.0.0.0 0.0.0.0 dialer 1
```
- Server

```config
bba-group pppoe global
virtual-template 1
exit

int fa 0/0
no shutdown
ip mtu 1492
pppoe enable group global
ip address 10.10.30.1 255.255.255.252
exit

interface virtual-Template 1
ppp authentication chap callin
ip unnumbered fastEthernet 0/0
peer default ip address pool PPPoEPOOL
exit

ip local pool PPPoEPOOL 10.10.30.2 10.10.30.10
username cisco password class
```
---

Project made in conjunction with my colleague Tiago Pereira.