https://developern.tistory.com/entry/VMware-ubuntu-vm%EC%9D%98-%EA%B3%B5%EC%9C%A0%ED%8F%B4%EB%8D%94-Shared-Folders-%EC%84%A4%EC%A0%95-%EB%B0%A9%EB%B2%95
![image](https://github.com/user-attachments/assets/8b33f451-0e03-43b1-9efb-12e9513b4d45)
사진에 add까지 하고 머신을 가동하면 /mnt/hgfs/<폴더이름> 으로 만드렁진다.
//위처럼 설정을 했는데도 안될 시 조치하는 방법!
sudo vmware-hgfsclient
sudo nano /etc/fstab
/*
vmhgfs-fuse	/mnt/hgfs fuse defaults,allow_other 0 0
*/
sudo reboot
