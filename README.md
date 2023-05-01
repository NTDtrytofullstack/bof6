# bof6
## Ý tưởng.
- Hôm nay ta đến với 1 dạng khác của bof5 khó hơn và phức tạp hơn.
![image](https://user-images.githubusercontent.com/130078745/235438748-d4fcd377-4863-450f-859f-db5d80044eed.png)
![image](https://user-images.githubusercontent.com/130078745/235438762-a0862949-f832-474d-8640-df763899dd1c.png)
![image](https://user-images.githubusercontent.com/130078745/235438776-aecc9aa0-c649-4cbb-a04c-d525bf47384d.png)
- Ở bof5 ta sử dụng gadget để có thể giải quyết nó 1 cách dễ dàng nhưng bof6 thì lại khác sau khi check gadget thì ko có 1 gadget nào thực thi việc trỏ đến địa chỉ của nó cả mà chỉ toàn là lệnh thực thi.
- Sau khi vào terminal để kiểm tra , ta tạo 1 breakponit ở hàm **get_name** thử nhập dữ liệu ở **buf** thì ta thấy dữ liệu đc lưu ở thanh ghi **rsp** và **rsi** , khi check thì ta ko thể xài theo như cách **bof5** ta từng làm đc mà lần này ta phải lưu chuỗi **/bin/sh** ở stack và kiếm cách để có thể leak ra đc địa chỉ của stack bởi vì nó là địa chỉ động , ta ko thể dự đoán 1 giá trị cụ thể nào cả mà chỉ có thể leak nó ra theo từng đợt.
![image](https://user-images.githubusercontent.com/130078745/235439317-aeef6eb5-1383-45eb-8153-8ae38bc51300.png)
![image](https://user-images.githubusercontent.com/130078745/235439355-b06b3ef7-e8b1-4802-a065-85b05daa7ac8.png)
![image](https://user-images.githubusercontent.com/130078745/235439460-cfe1d2f1-2e8c-4ac8-ae7b-30d6f1c62fb4.png)
- Sau khi check gadget.
![image](https://user-images.githubusercontent.com/130078745/235439515-6857500b-102c-4c06-a6f1-a701288f649a.png)
![image](https://user-images.githubusercontent.com/130078745/235439625-3ab3b8f1-3824-4be8-b788-869bfc25ab5d.png)
- Và trong hàm **get_wish** ta thấy đc 1 lỗi buffer overflow ta sẽ khai thác điểm này , tạo 1 shell code trên thanh ghi stack , sau đấy overwrite save rip của hàm **get_wish** để nó trỏ đến địa chỉ chứa chuỗi **/bin/sh** để ta có thể thu đc shell.
- Với điều kiện bit NX tắt thì thanh ghi stack mới xài đc.
![image](https://user-images.githubusercontent.com/130078745/235440100-65e7a22f-51a0-4db0-b8c0-39a4a3a7a919.png)
## Tự Hành.
- bước đầu để có thể leak đc địa chỉ stack ta cần phải biết là lệnh read() , khi nhập dữ liệu vào nó sẽ ko tự động thêm byte **\0*8 vào cuối và khi nhập nối với 1 chuỗi khác thì nó sẽ nối vào với nhau thì chính cách này ta có thể leak ra đc địa chỉ stack như sau.
![image](https://user-images.githubusercontent.com/130078745/235441469-541b74a0-8ba1-4e78-a6b9-a6d2919a0d39.png)
- từ chỗ nhập khi ta nhập đủ 8byte thì khi in ra nó đã bị nối liền với chuỗi địa chỉ phía dưới.
- Ta sẽ tiến hành viết tool leak ra địa chỉ sau khi nhập 80byte, và thưởng thì ta sẽ leak địa chỉ stack tại địa chỉ rbp vừa đủ offset 80byte.
![image](https://user-images.githubusercontent.com/130078745/235442108-28c6f49b-d033-451c-a027-8aadefa1dad8.png)
- Bước đầu ta nhập 1 để vào hàm **get_name** nhập vào đấy 80byte chạy thử tools và xem DEBUG.
![image](https://user-images.githubusercontent.com/130078745/235442478-37a582b1-0b32-41d4-ab8c-baa61f15841c.png)
![image](https://user-images.githubusercontent.com/130078745/235442572-7f1bf944-51fa-401a-a5f0-63baafed6272.png)
- Ta chỉ cần nhận đủ 80byte 'A' và 6 byte địa chỉ là đc.
![image](https://user-images.githubusercontent.com/130078745/235443006-208f4da2-1c47-4881-9474-d9c8699d761f.png)
- Với lệnh u64 ta cần nhập đủ 8byte nên ta cần phải thêm 2 byte null và in ra địa chỉ leak đc để xài cho việc viết tools dễ hơn.
- Chạy thử tools thì ta xem đc địa chỉ stack đc leak là.
![image](https://user-images.githubusercontent.com/130078745/235443344-d270011f-aeb5-4076-a4a2-6cc87fc2e699.png)
- Bước tiếp là ta tự tao 1 chuỗi **/bin/sh** bằng mã assembly.
- Nhập chuỗi vào lệnh read() trong hàm get_wish
- Bởi vì yêu cầu nhập tới 544 byte nhưng chuỗi **/bin/sh** là ko đủ vì vậy ta sử dụng **ljust()** với công dụng bù vào chỗ còn thiếu , trừ đi 8 byte save rip vì vậy ta sẽ nhập chỉ có 536 byte.
- VD như ở đây ta chỉ nhập có 4 ký tự nhưng dk là 10 thì nó sẽ tự thêm vào 6 byte null: ![image](https://user-images.githubusercontent.com/130078745/235444034-d9fcc1da-0199-48c2-9cca-4a9d1b7168c4.png)
![image](https://user-images.githubusercontent.com/130078745/235444245-8e003d85-2701-4d08-879f-b954ac489f06.png)
- Khi chayj thử và so sánh địa chỉ , địa chỉ mà ta leak ra đc lớn hơn địa chỉ nhập vào thực tế vì vậy để mà có thể trỏ chính xác tới điah chỉ chứa **/bin/sh** ta cần trừ đi bớt để có thể trỏ tới đúng nơi.
![image](https://user-images.githubusercontent.com/130078745/235445343-37e6334f-dba8-4f85-8408-1e89959fa3e8.png)
![image](https://user-images.githubusercontent.com/130078745/235445279-f8583dcc-805a-4cb2-b94d-ae3906da3df6.png)
- Và ở đây ta cũng bị dư đi 16byte làm chỉ save rip trỏ trật địa chỉ stack vì vậy ta cũng trừ đi bớt 16byte nhập vào , ta viết tiếp câu lệnh để có thể overwrite save rip.
![image](https://user-images.githubusercontent.com/130078745/235445822-701d4766-7a4e-4aaf-8569-fdfda76e708a.png)
![image](https://user-images.githubusercontent.com/130078745/235445733-1c4c156d-adea-4adb-84e2-dcac750918a1.png)
- Sau khi đã hoàn thành tools ta cùng chạy thử xem như nào.
![image](https://user-images.githubusercontent.com/130078745/235446011-59e930eb-4861-4dec-8495-14c29f760b29.png)
- Chạy thử thì chương trình ta viết đã đúng và đã lấy đc shell.

