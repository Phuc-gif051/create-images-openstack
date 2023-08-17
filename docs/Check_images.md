# Các kiểm tra Images cơ bản sau khi đóng Template

Chúng ta sẽ tạo VM từ Images và kiểm tra

[Kiểm tra có thể tạo dung lượng min_size bao nhiêu (Dung lượng này = dung lượng file `qcow2` chúng ta tạo lúc đóng Images)](#b1-kiểm-tra-có-thể-tạo-dung-lượng-min_size-bao-nhiêu-dung-lượng-này--dung-lượng-file-qcow2-chúng-ta-tạo-lúc-đóng-images)

[Kiểm tra khả năng tự động Extend của Volume](#b2-kiểm-tra-khả-năng-tự-động-extend-của-volume)

[Kiểm tra truyền password qua cloud-init](#b3-kiểm-tra-truyền-password-qua-cloud-init)

[Add thêm IP xem có nhận không](#b4-add-thêm-ip-xem-có-nhận-không)

[Thử tính năng reset password qua nova](#b5-thử-tính-năng-reset-password-qua-nova)

[Kiểm tra xem tab log của VM](#b6-kiểm-tra-xem-tab-log-của-vm)

___

## B1: Kiểm tra có thể tạo dung lượng min_size bao nhiêu (Dung lượng này = dung lượng file `qcow2` chúng ta tạo lúc đóng Images)

Tạo VM với dung lượng Volume = min_size (Ở đây linux là 10GB, windows trắng là 25GB,...) xem có tạo được không 

![](../images/check_images/vol1.png)

## B2: Kiểm tra khả năng tự động Extend của Volume 

Tạo VM với dung lượng lớn hơn min_size, sau khi tạo VM thì tiến hành login vào VM kiểm tra xem VM có nhận đủ dung lượng root disk không 

![](../images/check_images/vol2.png)

## B3: Kiểm tra truyền password qua cloud-init 

Hiện tại mới thực hiện được với image của linux

Truyền cloud-init khi create VM, Login thử bằng password truyền vào xem có login được không 

![](../images/check_images/cloud-init.png)

## B4: Add thêm IP xem có nhận không 

![](../images/check_images/addip.png)

## B5: Thử tính năng reset password qua nova

Để VM running và login vào Controller node sử dụng `nova set-password <VM_ID>` để set password cho VM xem có nhận password mới không 
Lấy ID của VM

![](../images/check_images/id.png)

Set paswd mới cho VM

![](../images/check_images/setpasswd.png)

Trong trường hợp muốn reset password tài khoản root hay Administrator thông qua QEMU-guest-agent, thì trước đó image đã phải được gắn metadata os_type để OpenStack biết và gắn user cho câu lệnh.
Trường os_type được set khi đứng sau cờ --property, có thể gắn cho image đã có sẵn với lệnh:

```sh
openstack image set --property os_type='linux-or-windows' <image-ID>
```

Gắn vào lúc tạo image:

```sh
openstack image create <name-image> --disk-format qcow2 --container-format bare --file <source-file> --share --min-disk 15 --property hw_qemu_guest_agent=yes --property os_distro=<windows or linux> --property os_type=<windows or linux>
```

Set-password cho VM:

sử dụng nova:

```sh
nova set-password <vm-id or name>
```

Sử dụng openstack:

```sh
openstack server set --root-password <vm-id or name>
```

Nhập pass theo chỉ dẫn.


### Trong khi set-password thì có 2 trường lỗi có thể xảy ra như sau


1. ERROR (Conflict): Failed to set admin password on xxx because error setting admin password (HTTP 409) (Request-ID: req-xxx)

Điều này cho thấy rằng không có tên người dùng tương ứng (mặc định) trong hệ thống OpenStack và máy ảo hiện tại. Cần phải xác nhận rằng thuộc tỉnh os_admin_user của máy chủ OpenStack và bên trong VM or image là giống nhau. Với OpenStack có 2 kiểu user mặc định là root cho linux và Administrator cho Windows, vì thế khi đặt lại password sẽ mặc định đặt lại cho 2 tài khoản này trên 2 dòng OS.

2. ERROR (Conflict): QEMU guest agent is not enabled (HTTP 409) (Request-ID: req-xxx)

Tình huống này cho thấy lỗi: hiện tại trên máy ảo qemu-guest-agent hoặc thuộc tính hw_gemu_guest_agent hư hỏng, kiểm tra và sửa đổi

#### qemu-guest-agent kiểm tra

1. Xác nhận rằng thuộc tính của VM là chính xác

- Xác nhận trên VM được tạo ra

```sh
openstack server show <vm-name>
```

- Xác nhận trên image dùng để tạo VM

```sh
openstack image show <image-name> | grep qemu
```

Cả hai tồn tại hw_gemu_guest_agent='yes'

2. Truy cập vào VM và xác nhận QEMU-guest-agent vẫn đang hoạt động bình thường

- trên linux:

```sh
systemctl status qemu-guest-agent
```

- Trên windows: truy cập vào Service tìm trong list được liệt kê.

- Thay đổi thuộc tính cho image với cờ --property

openstack image set

--property os_admin_user=Administrator <image-name or id>

--property os_distro=windows --property os_type=windows

Tham khảo thêm tại:

<https://blog.csdn.net/q965844841qq/article/details/111467457>

## B6: Kiểm tra xem tab log của VM

Trong quá trình boot VM tiến hành truy cập tab `log` xem có log MV hiển thị không 

![](../images/check_images/log.png)

## B7: Kiểm tra app của VM

Bước này chúng ta kiểm tra hoạt động của các app trên VM sau khi running như DA, Plesk, WHM

# Edit images sau khi đóng Template

Cài đặt libguest tools
```sh 
yum install -y libguestfs || apt-get install -y libguestfs-tools
```

Chỉnh sửa Images

```sh 
root@nhcephbka02:/var/lib/libvirt/images# guestfish --rw -a  u16-qemuagent.img 

Welcome to guestfish, the guest filesystem shell for
editing virtual machine filesystems and disk images.

Type: 'help' for help on commands
      'man' to read the manual
      'quit' to quit the shell

><fs> run

 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒⟧ 00:00
><fs> list-filesystems
/dev/sda1: ext4
><fs> mount /dev/sda1 /

```

## Đổi thông tin DA sau khi tạo VM từ Template

- Login vào VM

```sh
ssh root@<VM_IP>
```

- Check IP Public server

```sh
ip a
```

- Chạy script change IP
```sh 
cd /usr/local/directadmin/scripts
./ipswap.sh 192.168.122.36 <ip-public-server>
```

- Chạy script get License
```sh 
cd /usr/local/directadmin/scripts
./getLicense.sh
service directadmin restart || systemctl restart directadmin
```

- Reboot
```sh 
init 6 
```

- Kiểm tra thông tin và đăng nhập 
```sh 
cat /usr/local/directadmin/scripts/*.txt
```

- Truy cập Dashboard DA kiểm tra 
http://<ip-public-server>:2222

# Đổi thông tin IP WHM sau khi tạo VM từ Template

CentOS
``` sh
# Update license Cpanel
/usr/local/cpanel/cpkeyclt

IP=$(ip a | grep 255 | awk '{print $2}' | cut -d '/' -f1)

# Replace IP
Example: 
replace 123.30.145.16 103.28.36.104 -- /var/cpanel/mainip 
replace 123.30.145.16 103.28.36.104 -- /etc/hosts
replace 123.30.145.16 103.28.36.104 -- /etc/wwwacct.conf
replace 123.30.145.16 103.28.36.104 -- /usr/local/apache/conf/httpd.conf

```

Thông tin đăng nhập MySQL 
```sh 
cat /root/.my.cnf
```

Truy cập 
- WHM: https://<ip-public-server>:2083
- Cpanel: https://<ip-public-server>:2087
- Mail: https://<ip-public-server>:2095

# Đổi thông tin IP Plesk sau khi tạo VM từ Template

Đổi thông tin password đăng nhập Plesk

```
https://support.plesk.com/hc/en-us/articles/115001761193-How-to-change-the-IP-address-of-Plesk-server-

https://docs.plesk.com/en-US/12.5/advanced-administration-guide-win/system-maintenance/changing-ip-addresses.49727/
```

Đăng nhập MySQL 
```sh 
plesk db
```

Truy cập 
- Plesk: https://<ip-public-server>:8443

