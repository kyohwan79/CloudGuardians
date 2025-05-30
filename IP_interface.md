F/W:
  - with_Inbound_10.0.0.2/30
  - with_Outbound_99.99.98.1/24

main: 
  - with_FW_10.0.0.1/30
  - with_InnerNet_172.16.0.1/29

ActiveRouter:
  - with_L3SW_172.16.0.2/29
  - with_HSRP_ \
active 40 192.168.40.254/24 \
active 50 192.168.50.254/24 \
active 60 192.168.60.254/24 \
active 70 192.168.70.254/24

StandByRouter:
  - with L3SW_172.16.0.3/29
  - with_HSRP_ \
standby 40 192.168.40.254/24 \
standby 50 192.168.50.254/24 \
standby 60 192.168.60.254/24 \
standby 70 192.168.70.254/24

Branch1:
  - with_ISP_~
  - with_SW_ \
fa0/0.1: 192.168.10.254/24
fa0/0.2: 192.168.20.254/24
fa0/0.3: 192.168.30.254/24

Branch2:
  - with_ISP_~
  - with_SW_ \
fa0/0.8: 192.168.80.254/24
fa0/0.9: 192.168.90.254/24
