START: ansible-playbook chạy
    ↓
┌───────────────────────────────────────────────────────┐
│ BƯỚC 1: GATHERING FACTS (Thu thập thông tin hệ thống) │
└───────────────────────────────────────────────────────┘
    ↓
    ├─ Kết nối SSH đến host
    ├─ Chạy module "setup"
    ├─ Thu thập system facts
    │
    └─→ Tạo biến tự động:
        ├─ ansible_os_family: "Debian"
        ├─ ansible_distribution: "Ubuntu"
        ├─ ansible_hostname: "ubuntuthuong"
        └─ ... hàng trăm facts khác
    ↓
┌───────────────────────────────────────────────────────┐
│ BƯỚC 2: LOAD ROLE "lab-apache"                        │
└───────────────────────────────────────────────────────┘
    ↓
┌───────────────────────────────────────────────────────┐
│ BƯỚC 3: TỰ ĐỘNG LOAD vars/main.yml                    │
│ (Ansible convention - không cần khai báo)             │
└───────────────────────────────────────────────────────┘
    ↓
    └─→ Load biến từ vars/main.yml:
        ├─ apache_document_root: "/var/www/html"
        ├─ apache_dir_mode: "0755"
        ├─ apache_file_mode: "0644"
        ├─ apache_owner: "root"
        └─ apache_group: "root"
    ↓
    [MEMORY STATE]
    ┌─────────────────────────────────────┐
    │ ansible_os_family = "Debian"        │
    │ apache_document_root = "/var/www/html"│
    │ apache_dir_mode = "0755"            │
    │ apache_file_mode = "0644"           │
    │ apache_owner = "root"               │
    │ apache_group = "root"               │
    └─────────────────────────────────────┘
    ↓
┌───────────────────────────────────────────────────────┐
│ BƯỚC 4: CHẠY tasks/main.yml                           │
└───────────────────────────────────────────────────────┘
    ↓
┌───────────────────────────────────────────────────────┐
│ TASK 1: include_vars "{{ ansible_os_family }}.yml"    │
└───────────────────────────────────────────────────────┘
    ↓
    ├─ Thay thế biến: ansible_os_family = "Debian"
    ├─ Kết quả: include_vars "Debian.yml"
    ├─ Tìm file: vars/Debian.yml ✓
    │
    └─→ Load biến từ vars/Debian.yml:
        ├─ apache_pkg: "apache2"
        ├─ apache_service: "apache2"
        └─ apache_config_dir: "/etc/apache2"
    │
    └─→ vars/RedHat.yml: KHÔNG được load ✗
    ↓
    [MEMORY STATE - CẬP NHẬT]
    ┌─────────────────────────────────────┐
    │ ansible_os_family = "Debian"        │
    │ apache_document_root = "/var/www/html"│
    │ apache_dir_mode = "0755"            │
    │ apache_file_mode = "0644"           │
    │ apache_owner = "root"               │
    │ apache_group = "root"               │
    │ apache_pkg = "apache2"         ← MỚI│
    │ apache_service = "apache2"     ← MỚI│
    │ apache_config_dir = "/etc/apache2" ← MỚI│
    └─────────────────────────────────────┘
    ↓
┌───────────────────────────────────────────────────────┐
│ TASK 2: Install Apache package                        │
└───────────────────────────────────────────────────────┘
    ↓
    package:
      name: "{{ apache_pkg }}"  ← Lấy từ memory = "apache2"
      state: present
    ↓
┌───────────────────────────────────────────────────────┐
│ TASK 3: Ensure document root exists                   │
└───────────────────────────────────────────────────────┘
    ↓
    file:
      path: "{{ apache_document_root }}"  ← "/var/www/html"
      owner: "{{ apache_owner }}"         ← "root"
      group: "{{ apache_group }}"         ← "root"
      mode: "{{ apache_dir_mode }}"       ← "0755"
    ↓
┌───────────────────────────────────────────────────────┐
│ TASK 4: Deploy index template                         │
└───────────────────────────────────────────────────────┘
    ↓
    template:
      src: index.html.j2
      dest: "{{ apache_document_root }}/index.html"
      mode: "{{ apache_file_mode }}"      ← "0644"
    ↓
┌───────────────────────────────────────────────────────┐
│ TASK 5: Ensure Apache service is running              │
└───────────────────────────────────────────────────────┘
    ↓
    service:
      name: "{{ apache_service }}"  ← "apache2"
      state: started
    ↓
END: Playbook hoàn thành