## 1 số câu lệnh dùng để quản lý máy ảo

Để sử dụng cũng như quản lý các máy ảo, ta thường sử dụng các câu lệnh `virsh command`. Đây là bộ công cụ dòng lệnh để tương tác với libvirtd có hỗ trợ quản lý KVM. Cấu trúc cơ bản của 1 câu lệnh sẽ như sau

`virsh [Option]... <command> <domain> [ARG]...`

Các lệnh thường dùng:

- Liệt kê các máy ảo đang chạy:

`virsh list`

- Liệt kê tất cả các máy ảo:

`virsh list --all`

- Bật máy ảo:

`virsh start <VM_name>`

- Tự động bật máy ảo khi khởi động hệ thống:

`virsh autostart <VM_name>`

- Reboot máy ảo:

`virsh reboot <VM_name>`

- Lưu trạng thái đang hoạt động của máy ảo vào một file và sau này restore lại:

`virsh save <VM_name> <VM_name_time>.state`

Tuỳ chọn time để đánh dấu thời điểm cho dễ nhớ.

- Restore lại máy ảo vừa lưu:

`virsh restore <VM_name_time>.state`

- Tắt máy ảo:

`virsh shutdown <VM_name>`

- Để console tới máy ảo:

`virsh console <VM_name>`

- 1 số option khác của lệnh `virsh`:

`dominfo`: Hiển thị thông tin máy ảo

`domstate`: Hiển thị trạng thái máy ảo

`setmem`: Set RAM cho máy ảo

`setmaxmem`: Sets maximum memory limit for the hypervisor

`dommemstat`: Lấy số liệu thống kê bộ nhớ

`setvcpus`: Thay đổi số vCPU cho máy ảo. Lưu ý không support trên RHEL 5

`vcpucount`: Số lượng vCPU máy ảo

`vcpuinfo`: Hiển thị vCPU của máy ảo

`vcpupin`: Controls the virtual CPU affinity of a guest.

`domblklist`: Liệt kê tất cả các block

`domblkinfo`: Hiển thị thông tin block size

`domblkstat`: Hiển thị thông tin block device của máy ảo

`domiflist`: Liệt kê các virtual interfaces máy ảo

`domifaddr`: Hiển thị địa chỉ network interface cho 1 máy ảo đang chạy

`domifstat`: Hiển thị thông tin network của máy ảo

`attach-device`: Gắn thiết bị vào máy ảo

`attach-disk`: Gắn ổ đĩa mới vào máy ảo

`attach-interface`: Gắn network interface mới vào máy ảo

`detach-device`: Gỡ thiết bị khỏi máy ảo

`detach-disk`: Gỡ ổ đĩa khỏi máy ảo

`detach-interface`:	Gỡ network interface máy ảo