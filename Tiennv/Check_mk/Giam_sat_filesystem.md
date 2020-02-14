## Giám sát file system

Mặc định thì Check_mk sẽ gộp chung các vùng nhớ của chúng ta lại với nhau để giám sát dưới thông số `DISK IO SUMMARY`. Tuy nhiên như thế này chúng ta không thể theo dõi được các vùng nhớ hay ô nhớ khác trên hệ thống. Để tách riêng từng vùng nhớ chúng ta vào mục `Host & Service Parameters` để thực hiện việc này.

Các bước thực hiện:

- Trong phần `WATO - CONFIGURATION` chọn `Host & Service Parameters` -> `Parameters for Discovered Services`:

<img src="img/128.png">

- Tại mục `STORAGE, FILESYSTEMS AND FILES` chọn `Discovery mode for Disk IO check`:

<img src="img/129.png">

- Chọn `Create rule in folder`

<img src="img/130.png">

- Điền các thông tin và ấn `Save`:

<img src="img/131.png">

- Sau đó, trong phần `WATO - CONFIGURATION` chọn `Hosts` -> `Discovery`:

<img src="img/132.png">

- Chọn `Start`:

<img src="img/133.png">

- Cuối cùng là Active Change:

<img src="img/134.png">

<img src="img/135.png">

- Kiểm tra kết quả:

<img src="img/136.png">