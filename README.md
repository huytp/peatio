Hướng dẫn cài đặt chi tiết sàn giao dịch mã nguồn mở Peatio trên môi trường production 
Được cấp phép theo giấy phép MIT.

# I. Yêu cầu.
- Tất cả đều được cài đặt trên hệ điều hành ubuntu.
- Cần một máy chạy phần mềm peatio với cấu hình tối thiểu: Ram 1GB, bộ nhớ 20GB.
- Với mỗi cryptocurrency khuyến khích cài đặt trên node riêng, yêu cầu Ram tối thiểu 4GB, bộ nhớ 100GB, riêng node chạy BTC bộ nhớ cần 200GB. Chú ý trong vấn đề mở port, cài đặt iptables hợp lý để đảm bảo độ bảo mật.

# II. Cài đặt.
## 1.  Cài đặt RVM
- RVM là một phần mềm dùng để quản lý phiên bản của ruby.
- Bạn cần chạy những lệnh sau:

gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
 \curl -sSL https://get.rvm.io | bash

source /home/ubuntu/.rvm/scripts/rvm echo "source $HOME/.rvm/scripts/rvm" >> ~/.bash_profile

## 2. Cài đặt ruby 2.5.0
- Hệ thống cần sử dụng ruby phiên bản 2.5.0.
- Bạn cần chạy lệnh sau:
rvm install 2.5.0

## 3.  Cài đặt MYSQL
- Hệ thống sẽ sử dụng hệ quản lý cơ sở dữ liệu MYSQL.
- Bạn cần chạy những lệnh sau.
- Chú ý trong quá trình cài đặt sẽ yêu cầu khai báo mật khẩu cho tài khoản DB root, bận nhập vào và nhớ để sử dụng.
sudo apt-get install mysql-server mysql-client libmysqlclient-dev

## 4. Cài đặt Redis
- Redis là một hệ thống lưu trữ key-value trên RAM giúp tối ưu performance, Redis còn có cơ chế sao lưu dữ liệu trên đĩa cứng cho phép phục hồi dữ liệu khi gặp sự cố.
- Ở hệ thống Peatio, Redis giúp lưu trữ dữ liệu từ các jobs. - Bạn cần chạy những lệnh sau:
sudo apt-add-repository -y ppa:rwky/redis
sudo apt-get update
sudo apt-get install redis-server

- Để khởi chạy Redis bạn chạy lệnh sau:
redis-server &

## 5. Install RabbitMQ
- RabbitMQ cung cấp cho lập trình viên một phương tiện trung gian để giao tiếp giữa nhiều thành phần trong một hệ thống lớn, sử dụng giao thức AMQP. Ở hệ thống này RabbitMQ sẽ giao tiếp với Redis và God Deamons.
- Cần chạy những lệnh sau để cài đặt RebitMQ:
// Thêm repo của rabbitmq debian 
sudo apt-add-repository 'deb http://www.rabbitmq.com/debian/ testing main'
curl http://www.rabbitmq.com/rabbitmq-signing-key-public.asc | sudo apt-key add -
// Cài đặt rabbitmq
sudo apt-get update
sudo apt-get install rabbitmq-server

- Chạy 2 câu lệnh sau để start rabbitmq
sudo rabbitmq-plugins enable rabbitmq_management
sudo service rabbitmq-server restart

## 6. Cài đặt Bitcoind
- Bitcoind là một bitcoin full node, phần mềm này sẽ đồng bộ với tất cả các node bitcoin khác. Đây cũng là phần mềm quản lý ví (wallet). Bitcoind là một phiên bản chạy ở chế độ command (Sử dụng cho các nhà phát triển, chạy ở linux), phiên bản chạy ở giao diện là bitcoin-qt.
- Để cài đặt ta cần chạy các lệnh sau:
sudo add-apt-repository ppa:bitcoin/bitcoin
sudo apt-get update
sudo apt-get install bitcoind

- Tạo thư mục .bitcoin và tạo file config:
mkdir -p ~/.bitcoin
touch ~/.bitcoin/bitcoin.conf
vim ~/.bitcoin/bitcoin.conf
- Nội dung file bitcoin.conf như sau:
daemon=1
testnet=1 #khi chạy mainnet chỉnh lại thành server=1
rpcuser=USERNAME
rpcpassword=PASSWORD
rpcport=18332 #Khi chạy mainnet chỉnh lại thành rpcport=8332

// Gửi thông báo tới hệ thống khi nhận được coins
walletnotify=/usr/local/sbin/rabbitmqadmin publish routing_key=peatio.deposit.coin payload='{"txid":"%s", "currency":"btc"}'

- Khởi chạy bitcoind:
bitcoind

- Để dừng chạy lệnh:
bitcoin-cli stop
- Tham khảo thêm các lệnh cung cấp cho bitcoin RPC tại: https://en.bitcoin.it/wiki/Original_Bitcoin_client/API_calls_list

## 7. Cài đặt PhantomJS
- Chạy những lệnh sau để cài đặt:
sudo apt-get update
sudo apt-get install build-essential chrpath git-core libssl-dev libfontconfig1-dev

cd /usr/local/share

PHANTOMJS_VERISON=1.9.8
sudo wget https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-$PHANTOMJS_VERISON-linux-x86_64.tar.bz2

sudo tar xjf phantomjs-$PHANTOMJS_VERISON-linux-x86_64.tar.bz2

sudo ln -s /usr/local/share/phantomjs-$PHANTOMJS_VERISON-linux-x86_64/bin/phantomjs /usr/local/share/phantomjs
sudo ln -s /usr/local/share/phantomjs-$PHANTOMJS_VERISON-linux-x86_64/bin/phantomjs /usr/local/bin/phantomjs
sudo ln -s /usr/local/share/phantomjs-$PHANTOMJS_VERISON-linux-x86_64/bin/phantomjs /usr/bin/phantomjs

## 8. Cài đặt JavaScript Runtime
- Chạy những lệnh sau để cài đặt:
curl -sL https://deb.nodesource.com/setup_8.x | sudo bash -
sudo apt-get install nodejs

## 9. Install ImageMagick
- Đây là phần mềm hỗ trợ cho việc tải ảnh lên hệ thống.
sudo apt-get install imagemagick

## 10. Cài đặt và cấu hình Peatio
a).  Clone dự án về từ Github của Peatio.
mkdir project
cd project
git clone https://github.com/rubykube/peatio.git
cd peatio
git checkout 1-7-stable 

- Cài đặt gem:
gem install bundle
bundle install

- Generate những config mặc định được viết sẵn.
bin/init_config

- Cài đặt yarn và cài đặt những thư viện sử dụng cho giao diện.
sudo npm install -g yarn
bundle exec rake tmp:create yarn:install assets:precompile

- Cấu hình lại trong file production.rb
vi config/environments/production.rb
tìm và chỉnh vào sau thành: config.assets.compile = true

b). Cài đặt database.
vi config/database.yml

Cần sửa lại những thông số sau trong database.yml:
host: localhost
username: root
password: [Mật khẩu mà bạn khai báo khi cài đặt mysql]

- Tạo DB, tạo bảng, khởi tạo một số dữ liệu khai báo trong seed:
RAILS_ENV=production bundle exec rake db:setup

c). Chạy daemons.
- Yêu cầu phải chạy Rabbitmq và redis trước, ở trên mình đã hướng dẫn chạy 2 ứng dụng này rồi.
- Hãy chắc chắn bạn đang đứng ở thư mục /peatio
- Chỉnh lại tham số sau để chạy ở chế độ production:
vi lib/daemons/daemons.god 
tại đây chỉnh tham số RAILS_ENV thành:
RAILS_ENV  = ENV.fetch('RAILS_ENV', 'production')
- OK, tiến hành start deamon bằng lệnh sau:
god -c lib/daemons/daemons.god
d) Khởi tạo liability proof
RAILS_ENV=production bundle exec rake solvency:liability_proof

e) Cấu hình trong config/application.yml
vi config/application.yml
e1. Cấu hình the Google Authentication
OAUTH2_SIGN_IN_PROVIDER: google

GOOGLE_CLIENT_ID: 321905830771-um33gdv48uo38bu3i7hs8gv6er52mpoh.apps.googleusercontent.com

GOOGLE_CLIENT_SECRET: isdrFxVDuGCcOHOw6pUWXZvB

- Vào  HYPERLINK "https://console.developers.google.com/"https://console.developers.google.com/ để tạo và lấy những key trên.

e2. Cấu hình các thứ khác:

ADMIN:  HYPERLINK "mailto:example@example.com"example@example.com #tên email sẽ là admin.
URL_HOST: peatio.example #tên miền website

f) Tạo secret key:

rake secret
copy đoạn mã sau khi tạo ra.
Mở file config/secret.yml

chỗ production/secret_key_base, thay thế đoạn mã trên vào <%= ENV["SECRET_KEY_BASE"] %>

g). Khởi động rails server:

bundle exec puma -e production -p 3000 --pidfile tmp/pids/puma.pid -d

## 11. Run Peatio Trading UI
cd ~/project
git clone https://github.com/rubykube/peatio-trading-ui.git
cd peatio-trading-ui
git checkout 1-7-stable 

- Chỉnh lại code lỗi:
vi app/views/markets/_market_list.html.erb
tìm tới dòng 45:
<%= link_to market.fetch(:name), '/trading/' + market.fetch(:id) %>
chỉnh lại thành
<%= link_to market.fetch(:name), '/trading/' + market.fetch(:id).to_s %>

- Cài đặt gem:
bundle install
- Khởi tạo những config đã cấu hình sẵn.
bin/init_config

- Vào file config/application.yml tìm và cấu hình lại:
PLATFORM_ROOT_URL: irie.fashion #đường dẫn website chính của bạn

- Giống ở trên, tạo secret key bằng lệnh sau:
rake secret

Vào file config/secret.yml, tìm dòng production/secret_key_base
thay thế đoạn mã đã copy ở trên thay thế vào: <%= ENV["SECRET_KEY_BASE"] %>

- Chạy lệnh sau để precompile assets:
bundle exec rake assets:precompile

- Cấu hình trong production.rb
vi config/environments/production.rb
tìm và chỉnh vào sau thành: config.assets.compile = true

- Khởi chạy peatio GUI ở port 4000
bundle exec puma -e production -p 4000 --pidfile tmp/pids/puma.pid -d

## 12. Cài đặt và cấu hình nginx
- Cài đặt nginx
sudo apt-get update
sudo apt-get install nginx
sudo ufw allow 'Nginx HTTP'
systemctl status nginx

- Cấu hình nginx
 sudo vi /etc/nginx/sites-available/default
xóa hết tất cả mọi thứ và thay thế thành những đoạn code bên dưới:
server {
  server_name http://irie.fashion;
  listen      80 default_server;

  location ~ ^/(?:trading|trading-ui-assets)\/ {
    proxy_pass http://127.0.0.1:4000;
  }

  location / {
    proxy_pass http://127.0.0.1:3000;
  }
}

* ở location thứ nhất, có port 4000 là đang sử dụng từ server peatio GUI.
Ở location thứ hai, có port 3000 là đang sử dụng từ server peatio.
server_name chính là tên miền của website.

- Khởi động lại nginx
sudo systemctl restart nginx

# III. Cấu hình ở trang admin.

Vào bảng quản trị, vào phần currencies, view BTC, 
- Chỗ JSON RPC endpoint, thay thế thành endpoint của mình (đoạn cấu hình bitcoind)
 HYPERLINK "http://user/"http://username: HYPERLINK "mailto:password@127.0.0.1"password@127.0.0.1 HYPERLINK "":18332

- Wallet URL template (use #{address} as placeholder): ở testnet thì để thành:
 HYPERLINK "https://testnet.blockchain.info/address/"https://testnet.blockchain.info/address/ HYPERLINK ""#{address}
- Transaction URL template (use #{txid} as placeholder): ở testnet để thành: https://testnet.blockchain.info/tx/#{txid}
