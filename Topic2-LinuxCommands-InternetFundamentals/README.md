# Topic2-LinuxCommands-InternetFundamentals

#Linux Command Line

- Ping vietnix.vn và giải thích kết quả lệnh `ping` và `hping3`.
  ![alt text](image.png)
  - `ttl=` là gì trong ping?
    - ttl (Time To Live): Số lượng hop (bước nhảy router) tối đa mà gói tin có thể đi qua trước khi bị hủy
  - `time=` là gì trong ping?
    - tine: Là thời gian phản hồi

  ![alt text](image-1.png)
  - `Hping3`: Dùng để kiểm tra kết nối qua các port TCP/UDP cụ thể
    - Gửi 4 gói TCP SYN đến server tại port 80
    - Mục đích: Kiểm tra mạng, Kiểm tra port(TCP/UDP), Mô phỏng tấn công, Kiểm tra Firewall

- SSH Command:
  - Kết nối bằng password: `ssh user@ip_address`
    - Cách hoạt động: Client gửi username, Server yêu cầu password, nếu đúng cho login
    - Nhược điểm: Dễ bị brute-force (Phương pháp thử tất cả khả năng)
  - Kết nối bằng key: `ssh -i key.pem user@ip_address`
    - Các bước: Tạo key (ssh-keygen) tạo ra 2 file id_rsa và id_rsa.pub, Copy key lên server (ssh-copy-id username@ip), kết nối không cần nhập mật khẩu (ssh username@ip)
  - Kết nối bằng port custom: `ssh -p 2222 user@ip_address`
    - Trong file config đổi port SSH (sudo nano /etc/ssh/sshd_config)

- SCP Command:
  - Copy 1 file.
    - (scp file.txt username@ip:/path/) (Ví dụ: scp text.txt duc@192.168.1.10:/home/duc/)-> Copy từ máy lên server
    - (scp username@ip:/path/file.txt .) -> copy file tu server về máy
  - Copy 1 folder.
    - Dùng thêm -r (Recursive)
      - Local - Server : scp -r project/ duc@192.168.1.10:/home/duc/
      - Server - Local : scp -r duc@192.168.1.10:/home/duc/project
- Rsync Command:
  - Copy file.
    - (rsyn file.txt username@ip:/path/)
  - Copy folder.
    - (rsync -a folder/ username@ip:/path/) (-a: Copy đệ quy, Giữ quyền file, Giữ timestamp, Giữ cấu trúc thư mục)
  - `rsync incremental`.
    - Chỉ lấy phân thay đổi, nhanh tiết kiệm băng thông

- Cat Command:
  ![alt text](image-2.png)
  - Xem nội dung 1 file.
    - cat file.txt
  - Xem dòng thứ `<n>` trong file.
  - Ghi nhiều dòng vào 1 file bằng EOF.
    - cat <<EOF> file.txt
      hello
      word
      EOF

- Echo Command:
  - Chèn thêm 1 dòng vào cuối file.
    - cat <<EOF>> file.txt || echo "hello" >> file.txt (>>)
  - Overwrite nội dung file.
    - cat <<EOF> file.txt || echo "hello" > file.txt (>)
  - So sánh Echo và EOF
    - Echo: Ghi một dòng
    - EOF: Ghi nhiều dòng

- Tail/Head Command:

  ![alt text](image-3.png)
  - Sự khác biệt giữa `tail` và `head`.
    - tail file.txt: xem cuối file (Mặc định 10 dòng)
      - tail -n 5 file.txt (Xem 5 dòng cuối)
    - head file.txt: xem đầu file
      - head -n 5 file.txt (Xem 5 dòng đầu)
    - tail -f log.txt: realtime
  - Sự khác biệt giữa `tail` và `tailf`.

- Sed Command:
  - Find and replace string trong file.
    ![alt text](image-4.png)
    - sed 's/old/new' file.txt
      - s = substitute (Thay thế)
  - Thay thế trong từng dòng (Chỉ thay lần đầu xuất hiện trong mỗi dòng)
    - sed 's/hello/world' file.txt
  - Thay tất cả trong dòng
    - sed 's/hello/world/g' file.txt
  - Replace chỉ dòng cụ thể
    - (sed '2s/hello/world' file.txt): chỉ thay ở dòng 2
  - Replace nhiều file
    - (sed -i 's/old/new/g' \*.txt)

  Ký hiệu Nghĩa
  s substitute (replace)
  d delete
  p print
  a append (sau dòng)
  i insert (trước dòng)
  c change (thay dòng)
  -i sửa file trực tiếp

  ![alt text](image-5.png)
  - Ghi đè file, không có -i thì chỉ in ra
    - (sed -i 's/old/new/g' file)

- Traceroute/Tracert Command: Theo dõi đường đi của gói tin từ máy đến server (qua các router trung gian)
  - Thực hiện và giải thích kết quả.
    ![alt text](image-6.png)
    - Cột đầu tiên số hop, cột 2 IP router, 3 cột tiếp theo: thời gian phản hồi (3 lần đo)
    - Hop 2-4: Mạng ISP nội bộ, NAT + routing của nhà mạng
    - Hop 5-9: hạ tầng core mạng, các router lớn của ISP, chuyển tiepes traffic quốc gia
    - Hop 10: Server Vietnix

- Netstat Command: Kiểm tra các kết nối mạng đang hoạt động, Kiểm tra các port đang lắng nghe,Thống kê lưu lượng và giao thức, Hiển thị bảng định tuyến
  - Hiển thị các socket đang listen.
    ![alt text](image-7.png)
    - 0.0.0.0:\* : Cho phép mọi IP kết nối
    - localhost:domain : DNS server local (Port 53)
    - mDNS (Multicast DNS): dùng để tìm thiết bị trong LAN (AirPlay,printer,...)
    - UNIX sockkets: Giao tiếp nội bộ trong Linux

  - Không resolve hostname: Bình thường nếu không có -n, máy tính sẻ thực hiện một truy vấn DNS để xem địa chỉ IP đó thuộc về tên miền nào
    - netstat -n

  - Không resolve portname: Hệ điều hành có file thường là /etc/services để tra cứu tên của các công này, nếu không có Resolve nó sẻ giữ nguyên :80 :443

  - Display process name/PID.
    ![alt text](image-8.png)
    - Có thêm cột PID/Program name: cho biết ai lầ chủ nhân của dòng đso

  - Chỉ hiển thị socket TCP (HTTP, SSH, HTTPS)
    ![alt text](image-9.png)
    - Local Address: các con số sau dấu: (như 51478) là các Ephemeral Ports (cổng tạm thời). Khi mở một trang web, máy tính tự động chọn một số cổng ngẫu nhiên trong dải số cao để gửi yêu cầu đi

  - Chỉ hiển thị socket UDP (DNS, DHCP, HTTPS)
    ![alt text](image-10.png)
    - Tại sao lại có 443 (TCP). Các dòng IP như 104.18.32.47:443 (google) đang dùng giao thức QUIC chạy trên nền UDP để giúp lướt web nhanh hơn, giảm độ trẽ so với TCP truyền thống.
    - (0 ryo:bootpc \_gateway:bootps ): Đây là giao thức DHCP. Máy tính đang nói chuyện với Router(\_gateway:bootps) để xin cấp phát hoặc duy trì địa chỉ IP nội bộ .

- Sort Command:
  - Theo thứ tự tăng dần.
    ![alt text](image-11.png)
    - sort file.txt: Sắp xếp tăng dần (ascending)

  - Theo thứ tự giảm dần.
    - sort -r file.txt (-r = reserve)

  - Theo column.
    - sort -k2 file.txt
    - sort -k2 -r file.txt
    - sort -k1,1 -k2,2 file.txt (sort theo cột 1 trước nếu giống thì sort theo cột 2)

- Uniq Command:
  - Lọc các dòng lặp lại (hoạt động tốt khi dữ liệu đã được sort trước)
    ![alt text](image-12.png)
    - Vì chỉ xóa các dòng lặp lại liền kề nên cần short trước
  - Lọc và đếm số lượng dòng lặp lại.
    ![alt text](image-13.png)
  - (-d): Chỉ hiện dòng trùng

    ![alt text](image-14.png)

  - (-u): Chỉ hiện dòng không trùng

    ![alt text](image-15.png)

- Wc Command:

  ![alt text](image-16.png)
  - Đếm số dòng: wc -l file.txt
  - Đếm số ký tự: wc -m file.txt (bao gồm khoảng trắng + xuống dòng)
  - Đếm số từ: wc -w file.txt

- Chmod, Chown, Chattr Command:
  - Phân quyền bằng số và chữ.
  - Đổi owner user/group.
  - Set Immutable Attribute.

- Find Command:
  - Tìm file đuôi `.log`.
  - Tìm folder tên `abc`.
  - Tìm file tên `abc`.
  - Tìm file `abc` và đặt quyền read only.

- Cp Command:
  - Copy file.
  - Copy folder.

- Mv Command:
  - Di chuyển/đổi tên file/folder.

- Cut Command:
  - Lấy ký tự thứ `<n>`.
  - Lấy từ ký tự `<n>` trở về sau.
  - Lấy đến ký tự thứ `<n>`.

- Dig Command:
  - Kiểm tra record A, MX, NS.
  - Kiểm tra record A, MX, NS với custom DNS.

- Tar/Zip/Unzip Command:
  - Nén/giải nén `tar.gz`.
  - Nén/giải nén `.zip`.

- Mount/Umount Command:
  - Thêm ổ cứng `sdb` ~ 5gb.
  - Kiểm tra số lượng ổ cứng.
  - Mount vào `/mnt/test`.
  - Umount `/mnt/test`.

- Symbolic Links, Hard Links Command:
  - Định nghĩa Sym Link.
  - Định nghĩa Hard Link.
  - Ví dụ về Sym Link và Hard Link.

- Ls Command:
  - Liệt kê file/thư mục.
  - Liệt kê file/thư mục và thuộc tính.
  - Show file ẩn.

- Ps Command:
  - Show tiến trình.
  - Kill tiến trình.

- Top Command:
  - Kiểm tra tài nguyên CPU.
  - Giải thích các thông số.

- Free Command:
  - Giải thích các thông số về RAM.

- Df Command:
  - Xem dung lượng disk.
  - Phân vùng `/` là gì.
