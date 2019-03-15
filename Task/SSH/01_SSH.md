# SSH
SSH-Secure Shell là một giao thức điều khiển từ xa cho phép người dùng truy cập và thao tác trên server ở xa thông qua Internet. Giao thức này giống với giao thức telnet và một số giao thức khác nhưng nó thiết lập cơ chế bảo mật tốt hơn. Nó thực hiện mã hóa tất cả dữ liệu và câu lệnh được trao đổi giữa 2 máy. 
## Cách hoạt động
SSH hoạt động bằng mô hình client-server cho phép chứng thực an toàn giữa 2 máy thừ xa và mã hóa dữ liệu được truyền giữa chúng.
SSH vận hành trên TCP port 22 mặc định (có thể thay đổi nếu cần). Server sẽ nghe port 22 (hoặc bất kỳ port nào SSH được gán vào) cho những kết nối tới. Nó sẽ thiết lập kết nối an toàn khi chứng thực giữa client và môi trường shell đang mở thành công.
Client phải bắt đầu kết nối SSh bằng cách tạo ra TCP handshake với server, bảo đảm có thể thiết lập kết nối đối xứng, xác thực thông tin của server có khớp dữ liệu cũ không, và so sánh thông tin đăng nhập của user kết nối để xác thực đúng kết nối.
Có 2 giai đoạn để thiết lập kết nối: trước tiên cả 2 bên đống ý chuẩn mã hóa để bảo vệ giao tiếp. Thứ 2 user phải được xác thực. Nếu thông tin đăng nhập khớp user có quyền truy cập.</br>
Cú pháp thiết lập kết nối SSH:
```sh
ssh user@địa_chỉ
```
Trong đó: 
 - `user` là tên của user ta muốn kết nối vào(có thể là root, user1, user2...)
 - `Địa_chỉ` là địa chỉ của máy ta muốn kết nối vào có thể là địa chỉ IP hoặc domain name.

Có 2 cách để xác thực kết nối SSH. 
- Sử dụng password của user mà ta muốn kết nối vào để xác thực.
- Sử dụng một cặp key(private key và public key) để xác thực.

## Xác thực bằng password
Cách xác thực này khá đơn giản khi ta thiết lập kết nối nó sẽ hỏi ta password của user ta muốn truy nhập vào. Nếu ta nhập đúng thì ta có thể kết nối vào được. Nó có ưu điểm rất lớn là dễ và không cần phải cấu hình trước bất kỳ điều gì. Nhưng với cách xác thực này cũng có nhược điểm của nó. Mỗi lần kết nối nó lại hỏi lại mật khẩu gây phiền phức và còn một nhược điểm khác liên quan tới vấn đề bảo mật đó là ta phải nhập mật khẩu mỗi lần kết nối nếu không may bị người khác biết được mật khẩu đó thì rủi ra là rất lớn.
## Xác thực bằng key
Với cách xác thực này chúng ta có 1 cặp key (private key và public key). Chúng ta có thể coi private key như một chiếc chìa khóa còn public key như chiếc ổ khóa. Public key ta sẽ đặt trên những nơi mà ta muốn kết nối vào (trên user root, user thường của máy ở xa). Còn private key là chìa khóa nên ta phải cất giữ trên máy của mình. 
Quá trình chứng thực diễn ra như sau:
 - Client gửi thông tin user đực sử dụng để đăng nhập vào server. Server sẽ kiểm tra xem có user này trong hệ thống không. Nếu có thì tiếp tục quá trình kiểm tra cặp key.
 - Client gửi thông tin ID của cặp key được sử dụng để chứng thực đến server.
 - Server kiểm tra file `authorized_keys` để kiểm tra tài khoản mà user login dựa vào key ID.
 - Nếu có 1 public key trùng khớp với ID được tìm thấy trong file, thì server sẽ khởi tạo một chuỗi string+number (challenge) và sử dụng public key để mã hóa chuỗi đó thành một thông điệp (message).
 - Server gửi thông điệp đó đến cho client
 - Nếu Client có private key tương ứng với public key đó thì Client sẽ có khả năng giải mã mesage nhận được từ server để khôi phục chuỗi ký tự ban đầu.
 - Sau khi giải mã thành công thì chuỗi ký tự đó sẽ được kết hợp với shared session key (được sử dụng để mã hóa kênh truyền) và được tính toán giá trị MD5 hash ra một chuỗi mới.
 - Client sẽ gửi chuỗi MD5 ngược về phía server như trả lời thông điệp mã hóa từ phía server.
 - Server sẽ sử dụng shared session key và chuỗi ký tự ban đầu chưa mã hóa, tính toán giá trị MD5 hash sau cùng sẽ so với MD5 hash mà Client gửi tơi server. Nếu trùng khớp thì client sẽ được phép truy cập server.
 
## Một số dòng hay sử dụng trong File cấu hình SSH
```sh
vi /etc/ssh/sshd_config

17 #Port 22
38 #PermitRootLogin yes
47 #AuthorizedKeysFile      .ssh/id_rsa.pub
64 #PermitEmptyPasswords no
65 #PasswordAuthentication no
69 #ChallengeResponseAuthentication no
```
- Lưu ý bỏ dấu `#` để command có hiệu lực
- Restart lại service khi có những thay đổi `systemctl restart ssd`
#### Các bước cấu hình:
- Ta có thể tạo key-pair trên máy
Trước tiên ta tạo key trên máy client. Sử dụng lệnh
```sh
[user1@server2 ~]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/user1/.ssh/id_rsa):
Created directory '/home/user1/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/user1/.ssh/id_rsa.
Your public key has been saved in /home/user1/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:2O3xBl61Jn2tw+qjVKBD8T7nlK+0giViNh0uVLaqiI4 user1@server2
The key's randomart image is:
+---[RSA 2048]----+
|        .        |
|        oo       |
|       o..o   .  |
|      .+o+ . + ..|
|     ..+S.* B + o|
|      B ++.@ = o |
| . . + + ++ = =  |
|o . .   ...o.+ . |
|E.        .+=.   |
+----[SHA256]-----+
```
Ta thấy khi tạo key nó hỏi ta chỗ lưu file. Bạn có thể chọn chỗ lưu nhưng thường sẽ để mặc định bằng cách nhấn `enter`
Và sẽ được hỏi `passphrase` bạn có thể nhập hoặc không để bảo vệ private key của bạn. Nếu bạn nhập thì hãy nhớ mật khẩu này.
##### Một số option của câu lệnh `ssh-keygen`
 * `-b` sau đó là số bit để định số bit cho key.
 * `-t` để định thuật toán tạo key. Có các loại sau: rsa, dsa, ecdsa, ed25519. Nếu không có option này thì mặc định sẽ là `rsa`
 * `-f` để chọn chọn vị trí lưu file key.
 * `-p` để thay đổi passphrase. Cú pháp `ssh-keygen -p -P pass_cũ -N pass_mới -f keyfile`
 * `-y` để tạo một public key từ 1 private key.
Đây là một số option thông dụng để biết thêm các option khác ta sử dụng lệnh `ssh-keygen --help`
Ta kiểm tra bằng cách cd vào thư mục `.ssh` và kiểm tra trong xem đã có file chưa.
```sh
[user1@server2 ~]$ ll .ssh/
total 8
-rw-------. 1 user1 user1 1675 Jan 22 11:15 id_rsa
-rw-r--r--. 1 user1 user1  395 Jan 22 11:15 id_rsa.pub
```
Trong thư mục này chúng ta quan tâm đến 2 file đó là `id_rsa` là file chưa private key và file `id_rsa.pub` là file chứa public key.
Tiếp theo ta cần đưa public key lên server ta muốn kết nối ssh. Có 2 cách để ta đưa public key lên trên server. 
Cách thủ công: Ta đăng nhập vòa server mà cụ thể là đăng nhập vào user ta muốn truy nhập ssh vào. 
 * Tạo thư mục `.ssh` bằng lệnh `mkdir .ssh`
 * `chmod 700 .ssh` để phân quyền cho thư mục
 * cd vào thư mục `.ssh` và tạo file `authorized_keys` vào dùng lệnh `chmod 600 authorized_keys` để phân quyền cho file đó.
 * Ta copy nội dung file `id_rsa.pub` trên client vào file `authorized_keys` vừa tạo.
Cách khác là ta sử dụng lệnh `ssh-copy-id -i .ssh/id_rsa.pub user@địa_chỉ` 
 * Cách này ta phải đảm bảo rằng ta có thể ssh vào user trên máy mà ta muốn ssh vào bằng cách sử dụng password.
```sh
[user1@server1 ~]$ ssh-copy-id -i .ssh/id_rsa.pub user1@172.16.80.199
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ".ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
user1@172.16.80.199's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'user1@172.16.80.199'"
and check to make sure that only the key(s) you wanted were added.
```
```sh
[user1@server1 ~]$ scp user1@172.16.80.199:/home/user1/.ssh/id_rsa.pub ~/.ssh/
The authenticity of host '172.16.80.199 (172.16.80.199)' can't be established.
ECDSA key fingerprint is SHA256:SL7vZolGq493UzMOT0oGJMSIih1xjfyWh3p5B0xO8yE.
ECDSA key fingerprint is MD5:15:ad:62:ac:e7:55:b7:6f:6b:dd:45:c1:27:4f:0c:f5.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.16.80.199' (ECDSA) to the list of known hosts.
user1@172.16.80.199's password:
id_rsa.pub                                            100%  395   221.2KB/s   00:00
```
 * Với cách này thì hệ thống sẽ tự tao thư mục `.ssh` và file `authorized_keys` trên máy server. Nó sẽ đặt thư mục này trong thư mục của user đó. 

*Lưu ý* SSH key sẽ không thể hoạt động nếu ta đang bật `selinux` ta cần phải tắt nó đi bằng cách vào file `/etc/selinux/config` tìm dòng `SELINUX=enforcing` để sửa lại thành `SELINUX=disabled` sau đó reboot lại server.

Để bật xác thực kết nối SSH bằng key ta cần cấu hình một vàì thông số trong file `/etc/ssh/sshd_config`
*Lưu ý*: mỗi lần làm việc với file `/etc/ssh/sshd_config` xong ta cần thực hiện lệnh `service sshd restart` thì thay đổi của ta mới được update.
Ta cần sửa một số thông số sau:
 * `PubkeyAuthentication yes`
 * `AuthorizedKeysFile .ssh/authorized_keys`

Sau đó restart lại SSH
Để đảm bảo tính bảo mật hơn bằng việc kết nối SSH ta có thể thiết lập để chỉ cho phép SSH bằng key chứ ko cho SSH bằng pass. Để làm điều này ta cũng vào file `/etc/ssh/sshd_config` để tìm và sửa thông số sau:
 * `PasswordAuthentication no`

Sau đó restart lại SSH
Như đã biết thì tài khoản `root` trong linux có đặc quyền cao nhất. Nếu để mất tài khoản này vào tay người khác thì gần như chấm hết. Vì vậy cách tốt nhất là không cho SSH truy cập vào tài khoản `root` và nếu cần thì switch sang `root` bằng lệnh `su` khi cần thiết.
Để làm điều này ta cũng vào file `/etc/ssh/sshd_config` sau đó tìm đến dòng:
 * `#PermitRootLogin no`
Sau đó sửa lại thành:
 * `PermitRootLogin no`

Sau đó restart lại SSH
Bây giờ ta không thể SSH vào máy bằng tài khoản `root` được nữa.
```sh
[user2@server2 ~]$ ssh root@172.16.80.198
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
[user2@server2 ~]$ ssh user1@172.16.80.198
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```
Ta cũng có thể giới hạn những user có thể login SSH vào hệ thống bằng cách vào file `/etc/ssh/sshd_config` tìm dòng `AllowUsers` và thêm những user bạn cho phép dùng SSH để login vào.
Ví dụ ở đâu tôi chỉ cho user `user1` và `user2` dùng ssh để login vào hệ thống thì tôi thêm như sau: 
```sh
AllowUsers user1 user2
```
*Chú ý*: Ta cũng có thể tạo ra public key từ một private key có sẵn bằng cách sử dụng lệnh `ssh-keygen -y -f file_private_key > file_public_key`
Trong đó:
 * `file_private_key` là file chứa key private mà ta muốn tạo một public key từ key đó.
 * `file_public_key ` là file mà ta sẽ lưu public key mới tạo ra.

Như ví dụ trên tôi đã tạo ra một public key nữa và tôi đặt nó trong file `id_rsa1.pub`

Chúng ta có thể thấy rằng 1 private key có thể sinh ra rất nhiều các public key. Và các public key này là khác nhau nhưng vẫn sẽ được giải mã bằng private key mà chúng được sinh ra.

## Thiết lập phiên SSH giữa Client-Server
Muốn thực hiện SSH sử dụng key-pair ta làm như sau:
### Trên máy Linux
Muốn thực hiện được SSH thì `user` phải có `private key`
### Trên máy Window
Ta sử dụng các Tool dùng để SSH. Ở đây tôi dùng MobaXterm. Các Tool khác cũng sẽ tương tự.
- Trước tiên ta phải có `key private`. Có thể lấy key này ở thư mục `cat /user1/home/.ssh/id_rsa/
- Copy file này ra Text sau đó chọn đường dẫn đến file này như hình dưới:
<img src=https://i.imgur.com/UYEs8BN.png>
