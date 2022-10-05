/**
 * @author [daoNguyen]
 * @email [sydaonguyen1999@gmail.com ]
 * @create date 2022-10-05 14:48:55
 * @modify date 2022-10-05 14:48:55
 * @desc [Tài liệu tham khảo các bước Setup để deploy NEXTJS trên server Ubuntu 20.x ]
 */

# NEXTJS-DOCCUMENT
# Setup deploy NEXTJS trên server Ubuntu 20.x
## Các bước cấu hình đầu tiên cho server
 - Nginx hoặc Apache (nên cài Nginx vì tính ứng dụng rộng rãi )
 - Nodejs (nên cài NVM để quản lý version node cho nhiều dự án )
 - pm2 để start dự án NEXTJS (niếu dự án build static thì không cần cài )

## Cài đặt Nginx trên Ubuntu 20.04
 ### Cài đặt nginx
 -  Cài đặt nginx bằng lệnh:
       `$ sudo apt update`
       `$ sudo apt install nginx`
 -  Điều chỉnh tường lửa: 
       `$ sudo ufw app list`
       `$ sudo ufw allow 'Nginx HTTP'`
       `$ sudo ufw status`
 -  Kiểm tra trạng thái nginx trên server:
       `$ systemctl status nginx`
    -   Sau khi thực hiện lệnh chúng ta sẽ có kết quả là thông tin nginx service
    -   Kiểm tra kết quả trả về và chú ý dòng : `Active: active (running) since Fri 2020-04-20 16:08:19 UTC; 3 days ago`. Kết quả có dòng này tức là chúng ta đã active thành công nginx cho server Ubuntu.
    - Có thể kiểm tra thông qua trình duyệt bằng cách gõ địa chỉ ip vào thanh tìm kiếm như 1 địa chỉ website thông thường.
 -  Kiểm tra version nginx(Có thể không cần thiết):
       `$ nginx -v`

## Setup thư mục dự án
 ### Tạo đường dẫn lưu trữ
  - Thông thường thư mục dự án sẽ được lưu trữ tại : `/var/www/you_project`
  - Tạo mới thư mục chứa dự án:
       `$ sudo mkdir -p /var/www/you_project`
  - Sau khi đã tạo thành công thì tiến hành phân quyền dự án(Có thể cần thiết hoặc không)
  - Vào thư mục chứa dự án của chúng ta và tiến hành clone từ git hoặc có thể tạo mới dự án.
        `$ cd /var/www/you_project ` -> `$ git clone ...`
 ### Cấu hình ánh xạ port cho tên nginx
  - Các config của nginx cho website điều nằm trong `/etc/nginx/...`
  - Có thể dùng các file cấu hình mặc định của nginx hoặc tạo mới.
  - Vào file cấu hình để trỏ thư mục dự án:
    -   Dùng file mặc định của nginx, niếu muốn tạo mới thì tiến hành tạo file mới nằm trong thư mục này và đổi default thành tên file mới đã tạo:
         `$ sudo nano /etc/nginx/sites-available/default`

    -   Dán đoạn code config sau và tiến hành lưu

         server {
                listen 80;
                server_name `you_domain.com`; 
                index index.html;
                error_log  /var/log/nginx/hssg-error.log;
                access_log /var/log/nginx/hssg-access.log;

                proxy_read_timeout 1200;
                proxy_connect_timeout 1200;
                proxy_send_timeout 1200;

                location / {
                    proxy_set_header Host $host;
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_pass http://localhost:3000;
                }
            }
        -   server_name `you_domain.com`;  domain cần chạy website
        -   proxy_pass http://localhost:3000;  port dự án lúc start 
    -   Sau khi config xong tiến hành lưu file và kiểm tra lại
          `$ sudo nginx -t`
    - Xem kết quả về niếu thành công thì restart lại nginx và tiến hành build source.
          `$ sudo systemctl restart nginx`
    - Mỗi lần chỉnh sửa file cấu hình nginx thì phải restart nginx để nhận cấu hình mới.

## Build và start dự án next
 ### Cài pm2 để start dự án:
  - Các bước cài đặt pm2:
         `$ npm install pm2 -g `
  - Kiểm tra version
          `$ pm2 -v `
 ### Cài đặt Nodejs (NVM)
  - Cài đặt NVM
         `$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash `
  - Chỉnh sửa tệp lệnh `~/.bashrc`
         `$ nano ~/.bashrc`
  - Thêm các dòng sau ở cuối file và lưu

         export NVM_DIR="$HOME/.nvm"
            [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm

  - Lưu file và reload để nhận chỉnh sửa mới:
         `$ source ~/.bashrc`
  - Kiểm tra NVM 
          `$ nvm -v`
  - Cài đặt Node.js version
          `$ nvm install node ` -> Cài đặt mặc định version node mới nhất
          `$ nvm install 6.14.4 ` -> Cài đặt mặc định version node cụ thể (ví dụ 6.14.4)
  - Kiểm tra version node:
          `$ node -v`
 ### Build và start NEXTJS
  - Build source
      - Vào thư mục dự án chạy lệnh build:
           `$ npm run build`
  - Sau khi build thành công vào thư mục dự án và start
           `$ pm2 start npm --name "name_project" -- start`
  - Sau khi đã hoàn thành tất cả các bước trên tiến hành kiểm tra lại bằng địa chỉ IP server hoặc domain(Niếu đã trỏ ) bằng trình duyệt web.

Tham khảo: 
- https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04
- https://pm2.io/docs/runtime/guide/installation/
- https://www.vultr.com/docs/install-nvm-and-node-js-on-ubuntu-20-04/
