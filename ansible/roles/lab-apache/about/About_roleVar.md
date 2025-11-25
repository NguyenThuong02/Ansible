Vì include_vars chỉ load ĐÚNG file được chỉ định!
- name: Include OS-specific variables
  include_vars: "{{ ansible_os_family }}.yml"


### Quá trình thực tế:

Khi ansible_os_family = "Debian":
├─ Ansible thay thế biến: "{{ ansible_os_family }}.yml"
├─ Kết quả: "Debian.yml"
├─ Ansible CHỈ load file: vars/Debian.yml
└─ KHÔNG load vars/RedHat.yml

Khi ansible_os_family = "RedHat":
├─ Ansible thay thế biến: "{{ ansible_os_family }}.yml"
├─ Kết quả: "RedHat.yml"
├─ Ansible CHỈ load file: vars/RedHat.yml
└─ KHÔNG load vars/Debian.yml

Giải thích đơn giản:
include_vars giống như bạn nói với Ansible: "Hãy mở DUY NHẤT file có tên này và load biến từ đó"

KHÔNG phải là: "Load tất cả file .yml trong thư mục vars/"

Ví dụ minh họa:

Trường hợp 1: Chạy trên Ubuntu (Debian family)
# ansible_os_family = "Debian"
- name: Include OS-specific variables
  include_vars: "{{ ansible_os_family }}.yml"
  # ↓ Thay thế biến
  include_vars: "Debian.yml"
  # ↓ Chỉ mở file này
  # vars/Debian.yml ✓ LOAD
  # vars/RedHat.yml ✗ KHÔNG load
  # vars/main.yml   ✗ KHÔNG load (đã load tự động trước đó)
Kết quả:
yamlapache_pkg: "apache2"         # Từ Debian.yml
apache_service: "apache2"     # Từ Debian.yml
apache_config_dir: "/etc/apache2"  # Từ Debian.yml

Trường hợp 2: Chạy trên CentOS (RedHat family)
# ansible_os_family = "RedHat"
- name: Include OS-specific variables
  include_vars: "{{ ansible_os_family }}.yml"
  # ↓ Thay thế biến
  include_vars: "RedHat.yml"
  # ↓ Chỉ mở file này
  # vars/RedHat.yml ✓ LOAD
  # vars/Debian.yml ✗ KHÔNG load
  # vars/main.yml   ✗ KHÔNG load (đã load tự động trước đó)
Kết quả:
yamlapache_pkg: "httpd"           # Từ RedHat.yml
apache_service: "httpd"       # Từ RedHat.yml
apache_config_dir: "/etc/httpd"    # Từ RedHat.yml


### Timeline thực tế:

T0: Gathering Facts
    └─ ansible_os_family = "Debian"

T1: Load vars/main.yml (tự động)
    └─ apache_document_root = "/var/www/html"
    └─ apache_dir_mode = "0755"

T2: include_vars "Debian.yml"
    └─ String interpolation: "{{ ansible_os_family }}.yml"
    └─ Kết quả: "Debian.yml"
    └─ Ansible mở ĐÚNG file: vars/Debian.yml
    └─ Load vào memory:
        ├─ apache_pkg = "apache2"
        ├─ apache_service = "apache2"
        └─ apache_config_dir = "/etc/apache2"
    
    └─ File vars/RedHat.yml: KHÔNG được đụng đến ✗

T3: Sử dụng biến
    └─ apache_pkg có giá trị: "apache2"

### Tóm lại:
include_vars: "{{ ansible_os_family }}.yml" = "Chỉ load file có tên là giá trị của biến ansible_os_family"

* Nếu ansible_os_family = "Debian" → Chỉ load Debian.yml
* Nếu ansible_os_family = "RedHat" → Chỉ load RedHat.yml
* KHÔNG BAO GIỜ load cả hai cùng lúc!

Đây là cách Ansible biết load đúng file tương ứng với hệ điều hành! 😊