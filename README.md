# Triển khai Cụm Alertmanager với Ansible
Đây là playbook Ansible dùng để triển khai một cụm Alertmanager với 4 node. Playbook này bao gồm việc cài đặt Alertmanager, cấu hình cluster, mở các cổng tường lửa cần thiết, và kiểm tra trạng thái của cụm.
### Mục lục
[1. Yêu cầu](#yeu-cau)

[2. Cấu trúc thư mục](#cau-truc-thu-muc)

[3. Các biến cấu hình](#cac-bien-cau-hinh)

[4. Hướng dẫn triển khai](#huong-dan-trien-khai)

[5. Chi tiết các tác vụ](#chi-tiet-cac-tac-vu)

[6. Kiểm tra trạng thái cluster](#kiem-tra-trang-thai-cluster)

<a name="yeu-cau"></a>

## 1. Yêu cầu
- Ansible 2.9 trở lên được cài đặt trên máy điều khiển.
- Các máy đích (node) chạy hệ điều hành Linux (Ubuntu/Debian hoặc CentOS/RHEL).
- Người dùng có quyền sudo trên các máy đích.
- Kết nối SSH giữa máy điều khiển và các máy đích (khuyến nghị sử dụng SSH Key).

<a name="cau-truc-thu-muc"></a>

## 2. Cấu trúc thư mục
```
alertmanager-cluster/
├── main.yml
├── inventory
├── group_vars/
│   └── alertmanager_nodes/
│       └── vault.yml  # Chứa biến được mã hóa bằng Ansible Vault
└── roles/
    └── alertmanager/
        ├── tasks/
        │   └── main.yml
        ├── templates/
        │   ├── alertmanager.yml.j2
        │   └── alertmanager.service.j2
        └── vars/
            └── main.yml
```
<a name="cac-bien-cau-hinh"></a>

## 3. Các biến cấu hình
- ``version``: Phiên bản Alertmanager cần cài đặt (mặc định: ``0.27.0``).
- ``alertmanager_user``: Tên người dùng chạy dịch vụ Alertmanager (mặc định: ``alertmanager``).
- ``alertmanager_group``: Tên nhóm của người dùng Alertmanager (mặc định: ``alertmanager``).
- ``alertmanager_home``: Thư mục cấu hình của Alertmanager (mặc định: ``/etc/alertmanager``).
- ``alertmanager_data``: Thư mục lưu trữ dữ liệu của Alertmanager (mặc định: ``/var/lib/alertmanager``).
- ``alertmanager_port``: Cổng HTTP của Alertmanager (mặc định: ``9093``).
- ``alertmanager_cluster_port``: Cổng cluster của Alertmanager (mặc định: ``9094``).

<a name="huong-dan-trien-khai"></a>

## 4. Hướng dẫn triển khai
- **Clone hoặc tải về playbook**
```
git clone https://github.com/hieuny613/alertmanager-cluster.git
cd alertmanager-cluster
```
- **Cập nhật tệp ``inventory``**
Chỉnh sửa tệp ``inventory`` và thêm thông tin các node trong cluster:
```
[alertmanager_nodes]
node1 ansible_host=192.168.1.1 ansible_user=your_username
node2 ansible_host=192.168.1.2 ansible_user=your_username
node3 ansible_host=192.168.1.3 ansible_user=your_username
node4 ansible_host=192.168.1.4 ansible_user=your_username
```
**Lưu ý**: Khuyến nghị sử dụng SSH Key để xác thực.
- **(Tùy chọn) Sử dụng Ansible Vault cho biến nhạy cảm**
Nếu cần lưu trữ mật khẩu hoặc thông tin nhạy cảm, sử dụng Ansible Vault:
```
ansible-vault create group_vars/alertmanager_nodes/vault.yml
```

Thêm các biến vào ``vault.yml``:
```
ansible_password: your_password
ansible_become_password: your_sudo_password
```
- **Chạy playbook**
Thực thi playbook với lệnh:
```
ansible-playbook -i inventory playbook.yml --ask-become-pass
```
Hoặc nếu sử dụng Ansible Vault:
```
ansible-playbook -i inventory main.yml --ask-become-pass --ask-vault-pass
```
<a name="chi-tiet-cac-tac-vu"></a>

## 5. Chi tiết các tác vụ
- **Tạo người dùng và nhóm** ``alertmanager``: Không có thư mục home và không có quyền đăng nhập.
- **Tải và cài đặt Alertmanager**: Sử dụng biến ``version`` để tải đúng phiên bản.
- **Cấu hình Alertmanager**: Sử dụng tệp mẫu ``alertmanager.yml.j2``.
- **Cấu hình dịch vụ systemd**: Sử dụng tệp mẫu ``alertmanager.service.j2``.
- **Kiểm tra trạng thái Alertmanager**: Gửi yêu cầu HTTP để kiểm tra xem dịch vụ đã sẵn sàng chưa.

<a name="kiem-tra-trang-thai-cluster"></a>

## 6. Kiểm tra trạng thái cluster
Playbook sẽ kiểm tra và hiển thị danh sách các peer trong cluster:
```
- name: Display Alertmanager cluster peers
  debug:
    msg: "Cluster Peers: {{ alertmanager_status.json.cluster.peers | map(attribute='address') | join(', ') }}"
```
Đầu ra mẫu:
```
ok: [node1] => {
    "msg": "Cluster Peers: 192.168.1.1:9094, 192.168.1.2:9094, 192.168.1.3:9094, 192.168.1.4:9094"
}
```