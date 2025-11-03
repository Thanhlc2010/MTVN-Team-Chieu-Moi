# zMaps

> **Zimperium MAPS (Mobile Application Protection Suite)** là bộ giải pháp bảo mật ứng dụng di động giúp **bảo vệ mã nguồn, ngăn tấn công khi runtime, phát hiện mối đe dọa và đảm bảo vận hành an toàn, dễ dàng.**
> 

# **Tổng quan sản phẩm**

## 4 năng lực chính của sản phẩm và bài toán nó xử lý:

1. Code / Intellectual Property Protection (Offline Attack)
- Bảo vệ mã nguồn:
    - **Reverse Engineering:** Hacker giải mã file APK/IPA để đọc mã nguồn hoặc logic kinh doanh.
    - **Tampering:** Sửa đổi ứng dụng (chèn malware, bỏ qua cơ chế xác thực, gỡ quảng cáo, v.v.).
    - **Repackaging:** Đóng gói lại app giả mạo và phát tán.
- MAPS dùng các kỹ thuật:
    - **Code obfuscation** (làm rối mã, biến đổi logic để khó đọc).
    - **Integrity checks** (kiểm tra tính toàn vẹn).
    - **Anti-debugging / Anti-hooking** (chặn debug hoặc hook bằng Frida, Magisk, v.v.).
1. Run-Time Protection (Online Attack)
- **Bảo vệ ứng dụng khi đang chạy** (runtime) khỏi các mối đe dọa trên **thiết bị, mạng và malware** như:
    - Ứng dụng chạy trên **thiết bị đã root/jailbreak**.
    - **Kết nối mạng không an toàn**, tấn công MITM, proxy, DNS spoofing.
    - **Malware can thiệp vào tiến trình ứng dụng**, keylogger hoặc overlay attack.
- MAPS cung cấp SDK (zShield/zDefend) mà bạn **tích hợp vào app**, cho phép nó **tự phát hiện và phản ứng** ngay trong runtime.
1. Threat Visibility
- Có zConsole nhìn được tường tận các mối đe dọa đang xảy ra:
    - Thiết bị root, mạng nào không an toàn, app nào đang bị tấn cống, …
    - Hiển thị log, trending và phân tích ở zconsole
    - Hỗ trợ tích hợp SIEM
1. Ease of Operations
- **Triển khai và vận hành dễ dàng**, vì:
    - Tích hợp SDK vào app là **no code / low code** (chỉ cần vài dòng).
    - Có thể tích hợp vào **CI/CD pipeline** để tự động bảo vệ app khi build.
    - Quản lý qua **cloud-based console**, không cần hạ tầng phức tạp.

# Chi tiết sản phẩm

![image.png](image.png)

Cơ bản thì nó có 4 module chính là zScan, zKeybox, zShield và zDefend

Ở giai đoạn phát triển ta có zScan để quét trước code tìm ra các lỗ hổng bảo mật giống quét của thg Dam

Tiếp theo ở giai đoạn Run-time sẽ có 3 thằng còn lại để bảo vệ

- zKeybox để giữ các mã khóa và thông tin nhạy cảm trong ứng dụng khỏi bị trích xuất hoặc giải mã.
- zShield sẽ sửa cái source code của app để ngăn app không bị tấn công bằng các obfuscation (làm rối mã code) hoặc biện pháp anti-tampering
- zDefend để chạy runtime threat như là app bị cài vào máy bị jailbreak, app bị tampering, app bị cài ở nguồn cài không uy tín, app vào máy có mạng ko tốt

Cuối cùng thì thằng zConsole sẽ là nơi quản trị

![image.png](image%201.png)

Hình này sẽ mô tả chính xác hơn về cách hiểu của cái con sản phẩm này

![image.png](image%202.png)

Hình này sẽ nói về việc mỗi module được áp dụng trong từng giai đoạn của dự án

- Đầu tiên ở plan và develop zDefend sẽ cung cấp threat có thể có để có build app, zKeybox sẽ đảm nhiệm vai trò lưu trữ các mã khóa qtrong
- Tiếp đến sẽ có 1 vòng lặp ở build và test sẽ liên tục tạo các bài test để test và sửa lại cho đến khi hoàn thành
- Giữa giai đoạn hoàn thành và deploy zShield sẽ đi vào và sửa source code để ngăn tampering, … sau đó zScan sẽ quét lại lần nữa
- Và cuối cùng zDefend sẽ đảm nhận bảo vệ app theo thời gian thực

## zScan

![image.png](image%203.png)

zScan sẽ phân tích được cả mã tính và mã lúc chạy, thực thi

### Mã tĩnh

Ở đây cũng phân tích tìm ra lỗ hổng như dam:

- Dữ liệu nhạy cảm ở đâu
- Cơ chế nào dùng để mã hóa
- Code này này permision ở đâu
- Data thì được lưu như nào
- Nó “nói chuyện với SSL” ra sao

### Mã động

Cho thử vào môi trường ảo hóa để tìm tiếp các lỗ hổng như

- Thành phần dễ tổn thương
- Dữ liệu nào nó lấy từ ng dùng
- App send data qua đâu
- Có khả năng bị leak ko

zScan cũng có các tiêu chuẩn như **NIAP**, **GDPR**, **MASVS**

![image.png](image%204.png)

Với việc kết hợp zShield (làm rối mã code) và zDefend bảo vệ thời gian thực tức là cả ofline và online, ta có thể bảo vệ nhiều lớp, nôm na là toàn diện.

## Offline protection

![image.png](image%205.png)

Nó sẽ có 2 cái chính mà hacker dùng để tấn công vào app đó là reversing và tampering

### Reversing

> Cái này là hacker cầm đoạn apk sau đó đảo ngược từ đó ra mã nguồn
> 

Bảo vệ bằng cách:

- Làm rối mã nguồn
- Chống debug là chống việc người dùng mở f12 lên xem luồng chạy hoặc  1 công cụ nào đó
- Nén mã và làm đa dạng mã (nén mã là việc khiến cho 1 đoạn mã code bị nén lại và sẽ được khởi chạy/ thực thi qua 1 dạng loader nhỏ, đa dạng mã là việc khiến cho để làm được điều A thì có 3 4 cách, sẽ tạo ra nhiều cách như vậy)
- Và qtrong nhất là control flow flattening

![image.png](image%206.png)

### Tampering

> Đây là hacker trực tiếp vào sửa đổi luôn đoạn mã code
> 

Và nó bảo vệ bằng cách:

- Tự thêm các hàm checksum để kiểm tra có bị sửa file hoặc bộ nhớ ko
- **Anti-Debug & Anti-Hooking:** Phát hiện khi có debugger, Frida, Magisk can thiệp.
- Cơ chế kiểm tra chồng lấp (*Overlapping Checkers*) giúp phát hiện sửa đổi tinh vi. ( cơ chế này là việc làm cho code nó thành nhiều lớp và chèn các mã nhỏ vào từng lớp để kiểm tra, việc này giúp ta kiểm tra được nhiều trường hợp)

## Online protection

![image.png](image%207.png)

Cơ bản thì thằng zDefend hay nói rộng là zimperium sẽ bảo vệ app qua cơ chế là nhìn từ ngoài vào, sẽ là kiểu giám sát app của mình hoạt động ra sao chứ không như cách truyền thống sẽ đi từ trong ra là khi app nổ, app chết thì đi tìm các cách giải quyết dựa vào signature 

cái qtrong nhất là thg zim này không bắt người dùng phải cập nhật lại app khi có tấn công, áp dụng cả ml vào trong này để học hành vi tìm ra được các threat mà chưa có trong signature, 

Các cách tấn công sẽ được  cập nhật liên tục từ các threat mới

**Machine learning**

![image.png](image%208.png)

## zKeybox

### Mục đích chính của sản phẩm

- Thực hiện các thao tác mật mã (encryption, signing,…) trong 1 môi trường an toàn
- Ngăn kẻ tấn công **trích xuất khóa bí mật** từ ứng dụng, dù thiết bị bị root/jailbreak hay bị debug.
- Dựa trên nguyên lý **white-box cryptography** → mọi dữ liệu, biến, và thao tác bên trong đều ở trạng thái mã hóa (không bao giờ xuất hiện dạng plaintext).

### Cách sử dụng zKeyBox Constructor

Đầu tiên là phải tạo 1 cái lược đồ mật mã riêng cho app của mk sau đó sẽ tạo ra zKeyBox library được thiết kế riêng

Tạo lược đồ bằng phần mềm tên là zKeyBox Constructor
Lược đồ này là **một đồ thị (graph)** mô tả **các thao tác mật mã sẽ được thực thi, thứ tự thực hiện, và loại khóa được sử dụng** trong từng bước.

Sau đó sẽ có 2 loại bản dùng thử và trả phí của zKeybox

### zKeybox sẽ cung cấp khá nhiều loại cơ chế và áp dụng được trên nhiều platform

### Các chức năng chính

- **Diversification**: Mỗi thư viện zKeybox thì được tạo nên từ 1 seed ngẫu nhiên dẫn đến việc zKeybox ở mỗi lần tạo sẽ khác nhau
- **zShield protection**: cơ bản thì khi request cái gói zKeybox thì sẽ có được chọn option là “tamper-resistant edition” sẽ được bảo vệ bằng zShield
- **Key Exporting and Importing**

![image.png](image%209.png)

Mỗi khi export hoặc import thì sẽ cần 1 cái key riêng để wrap lại cái key của người dùng. 

Export và import key này sẽ được gen trên cái Export Import Key Tool qua 1 dòng lệnh, mỗi 1 dynamic key thì có thể có nhiều key mỗi loại nma luôn phải có ít nhất 1 trong 2

- Device ID cái này ta có thể cho thêm vào lúc export và import để tăng tính xác thực
- 3 yếu tố cần để import thành công
    - khóa nhập phải giống khóa xuất
    - 2 khóa này cần được tạo ra từ Export Import Key Tool từ cùng 1 gói zKeybox
    - Nếu có device ID thì phải giống
- **Key Serialization**

Đây như kiểu 1 tính năng đơn giản hóa của export và import nhưng có các sự khác biệt sau

- Dành riêng cho zKeybox chứ không đi theo các phiên bản app/ thiết bị được
- không cần export và import key
- **Internal Random Number Generator**

Với việc sinh số ngẫu nhiên thì thg này có riêng 1 cái gọi là internal Deterministic Random Bit Generator (DRBG) nhưng mà có thể dùng các cái entropy khác được 

- **Externalization of Platform Dependencies**

Khi zKeybox sử dụng có thể sử dụng 1 số hàm ăn theo hệ điều hành, việc này dẫn tới 1 số tuân thủ có thể ko đạt

Chức năng này cho phép chúng ta cầm cái hàm mà zKeybox dùng về, sửa đổi và nhét lại vào app để tuân thủ