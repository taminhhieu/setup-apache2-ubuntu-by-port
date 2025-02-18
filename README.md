Để setup một dự án mới chạy trên demo.pro.vn:3030 (một cổng khác trên server Ubuntu của bạn), bạn cần thực hiện một số bước cấu hình Apache 2 và hệ thống của bạn để có thể chạy cùng lúc nhiều ứng dụng hoặc dự án khác nhau. Sau đây là các bước hướng dẫn để đạt được điều này:
1. Mở Cổng 3030 Trong Tường Lửa (Firewall)
   - Trước khi cấu hình Apache, bạn cần đảm bảo rằng cổng 3030 đã được mở trên tường lửa của Ubuntu (nếu bạn đang sử dụng ufw hoặc một firewall khác). ( có thể bỏ qua )
     + Kiểm tra trạng thái firewall:
       ```bash
       sudo ufw status
       ```
     + Mở cổng 3030:
       ```bash
       sudo ufw allow 3030
       ```
     + Kiểm tra lại:
       ```bash
       sudo ufw status
       ```
2. Cài Đặt Apache và Cấu Hình Virtual Host Cho Cổng 3030:
   ### Giả sử bạn đã cài đặt Apache và có một dự án đang chạy trên demo.pro.vn. Bạn muốn cài đặt một dự án khác chạy trên cổng 3030, và các bước thực hiện như sau:
   #### a. Tạo Virtual Host cho cổng 3030
   Trong Apache, chúng ta sẽ tạo một VirtualHost cho demo.pro.vn:3030 và trỏ nó đến thư mục chứa source code của dự án mới.
   - Tạo file cấu hình VirtualHost cho cổng 3030:
     Mở hoặc tạo một file cấu hình Apache mới cho dự án của bạn:
     ```bash
     sudo nano /etc/apache2/sites-available/demo.pro.vn-3030.conf
     ```
     Thêm nội dung sau vào file này:
     ```apache
     <VirtualHost *:3030>
        ServerAdmin webmaster@demo.pro.vn
        ServerName demo.pro.vn
        DocumentRoot /var/www/demo-pro-vn-3030  # Thư mục chứa source code của dự án mới
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    
        # Nếu cần SSL, có thể cấu hình thêm SSL ở đây (optional)
        # SSLEngine on
        # SSLCertificateFile /path/to/certificate.crt
        # SSLCertificateKeyFile /path/to/private.key
      </VirtualHost>
     ```
   - Lưu lại và đóng file.
   #### b. Kích Hoạt VirtualHost
   Kích hoạt file cấu hình vừa tạo và restart lại Apache để thay đổi có hiệu lực:
   ```bash
   sudo a2ensite demo.pro.vn-3030.conf
   sudo systemctl restart apache2
   ```
   #### c. Đảm bảo Apache đang lắng nghe cổng 3030
   Mặc định, Apache sẽ lắng nghe các cổng 80 và 443. Bạn cần đảm bảo rằng Apache cũng sẽ lắng nghe cổng 3030.
   Mở file cấu hình ports.conf:
   ```bash
   sudo nano /etc/apache2/ports.conf
   ```
   Thêm dòng sau vào:
   ```apache
   Listen 3030
   ```
   Sau đó, restart lại Apache:
   ```bash
   sudo systemctl restart apache2
   ```
3. Chỉnh Sửa hosts Nếu Cần
   Nếu bạn đang phát triển trên máy cá nhân hoặc máy chủ khác, bạn có thể cần chỉnh sửa file hosts để trỏ demo.pro.vn đến địa chỉ IP của server.
   - Mở file hosts:
     ```bash
     sudo nano /etc/hosts
     ```
   - Thêm dòng sau vào file (trong trường hợp bạn đang thử nghiệm trên máy cá nhân hoặc một máy khác):
     ```php
     <Your_Server_IP> demo.pro.vn
     ```
     Ví dụ:
     ```php
     192.168.1.100 demo.pro.vn
     ```
4. Cấu Hình Dự Án Mới (nếu có)
   Đảm bảo rằng dự án mới của bạn đã được đặt đúng thư mục /var/www/demo-pro-vn-3030 và có quyền truy cập hợp lệ.
   - Đảm bảo quyền sở hữu thư mục cho Apache (user mặc định là www-data):
     ```bash
     sudo chown -R www-data:www-data /var/www/demo-pro-vn-3030
     sudo chmod -R 755 /var/www/demo-pro-vn-3030
     ```
5. Kiểm Tra Lại
   Bây giờ, bạn có thể kiểm tra lại:
   - Truy cập trang web bằng cổng 3030:
     Mở trình duyệt và truy cập:
     ```arduino
     http://demo.pro.vn:3030
     ```
     Nếu mọi thứ được cấu hình đúng, bạn sẽ thấy ứng dụng của bạn chạy trên cổng 3030.
6. (Optional) Cấu Hình Reverse Proxy Nếu Cần
   Nếu ứng dụng của bạn chạy trên một cổng khác như 3030 và là ứng dụng Node.js hoặc một server khác (chẳng hạn như Express hoặc Nginx), bạn có thể cần cấu hình Apache như một reverse proxy để chuyển tiếp yêu cầu đến server đằng sau.
   Ví dụ: Cấu hình reverse proxy trong file demo.pro.vn-3030.conf:
   ```apache
   <VirtualHost *:3030>
      ServerAdmin webmaster@demo.pro.vn
      ServerName demo.pro.vn
      DocumentRoot /var/www/demo-pro-vn-3030
  
      ProxyPreserveHost On
      ProxyPass / http://127.0.0.1:3030/
      ProxyPassReverse / http://127.0.0.1:3030/
  
      ErrorLog ${APACHE_LOG_DIR}/error.log
      CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
   ```
   Lưu ý bạn sẽ cần phải cài đặt module proxy và proxy_http trong Apache nếu chưa cài:
   ```bash
   sudo a2enmod proxy
   sudo a2enmod proxy_http
   sudo systemctl restart apache2
   ```

### Kết Luận
Với các bước trên, bạn đã cấu hình thành công Apache để phục vụ hai dự án:
  - Dự án cũ sẽ chạy trên demo.pro.vn (cổng mặc định 80).
  - Dự án mới sẽ chạy trên demo.pro.vn:3030.
