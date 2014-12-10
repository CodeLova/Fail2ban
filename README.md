Fail2ban với iptables.

###### I. Fail2ban:
Trong khi sử dụng SSH để truy cập vào servers ta cần đảm bảo mật ở mức cao.SSH là một dịch vụ được các  người quản trị sử dụng thông qua mạng internet  việc này có thể gây nguy hiểm cho hệ thống. Bởi các hacker có thể sử dụng phương pháp tấn công vét cạn (brute force).
Bất kì dịch vụ nào  giao tiếp với internet đều có thể tạo ra mối nguy hiểm cho hệ thống.Ta có thể thấy điều này thông qua việc đọc log của ssh,thống kê các phiên truy  nhập của người dùng(hoặc những đối tượng) đang cố gắng truy cập vào hệ thống thông qua ssh bằng một phương thức nào đó.

Fail2ban là một dịch vụ có thể giảm thiểu vấn đề này bằng cách tạo ra các quy tắc mà có thể tự động thay đổi cấu hình tường lửa của bạn dựa trên một số được xác định trước cố gắng đăng nhập không thành công. Điều này sẽ cho phép các máy chủ của bạn để đáp ứng với những nỗ lực truy cập bất hợp pháp mà không có sự can thiệp từ  phía người quản trị.

###### II. Cài đặt và sử dụng fail2ban:

Trong bài này tôi cài đặt và sử dụng fail2ban trên ubuntu14.04 thao tác với dịch vụ ssh.

1. Mục tiêu:
 - Cấu hình thành công fail2ban trên ubuntu14.04.
 - Sử dụng fail2ban để giao tiếp với iptables
 - Test tính năng block ip của fail2ban.

2.Cấu hình :

- Cập nhật hệ thống trước khi cài đặt:
```
apt-get update && apt-get upgrade -y && apt-get dist-upgrade -y
```
- Cài đặt fail2ban :
```
sudo apt-get install fail2ban -y
```
-Sau khi cài đặt xong ta sẽ có các đường dẫn thư mục:
```
/etc/fail2ban/
/etc/fail2ban/action.d/ : Chứa các file cấu 
```
- Sửa file cấu hình fail2ban :
```
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
vi /etc/fail2ban/jail.local
```
- Trong file cấu hình có một số thông số cần chú ý như sau:
```
 - ignored 	: định nghĩa các ip được phép.(127.0.0.1/8)
 - bantime 	: định nghĩa thời gian  chặn khi một phiên truy cập thất bại.
 - findtime : định nghĩa thời gian truy cập hệ thông.
 - maxretry : định nghĩa số lần được phép truy cập trong khoảng thời gian được định nghĩa ở findtime.
 (Mặc định findtime = 600,maxretry = 3 có nghĩa là : từ 1 ip chỉ được phép truy cập tối đa 3 lần trong vòng 10 phút).
```
- Để định nghĩa các luật cho iptables thông qua fail2ban ta sửa trong file cấu hình với chỉ lệnh [Tên dịch vụ].Ví dụ [ssh]:
```
[ssh]
enabled = true
filter = ssh
port = ssh
logpath= /var/log/auth.log
maxretry = 2
```
- Khởi động lại dịch vụ:
```
service fail2ban restart
```
- Show tập lệnh của iptables, ta sẽ thấy như sau:
```
iptables -S 
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-N fail2ban-apache
-N fail2ban-dropbear
-N fail2ban-ssh
-N fail2ban-ssh-ddos
-A INPUT -p tcp -m multiport --dports 80,443 -j fail2ban-apache
-A INPUT -p tcp -m multiport --dports 22 -j fail2ban-ssh-ddos
-A INPUT -p tcp -m multiport --dports 22 -j fail2ban-dropbear
-A INPUT -p tcp -m multiport --dports 22 -j fail2ban-ssh
-A fail2ban-apache -j RETURN
-A fail2ban-dropbear -j RETURN
-A fail2ban-ssh -j RETURN
-A fail2ban-ssh-ddos -j RETURN
```
- Với thông số các thông số đã thiếp lập như trên.Ta sẽ test thư tính năng của nó:

Từ một máy cùng mạng với fail2ban server .Bạn thử  ssh và gõ sai mật khẩu.Kết quả sẽ như sau:
<img src="http://i.imgur.com/pDnnzMR.png">

Như vậy, ta đã test được tính năng của Fail2ban với dịch vụ ssh.


###### III. Thông tin tham khảo:
<a href="https://www.digitalocean.com/community/tutorials/how-to-install-and-use-fail2ban-on-ubuntu-14-04">Link digitalocean</a>