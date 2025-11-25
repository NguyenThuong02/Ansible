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
