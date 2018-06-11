
[Source](https://www.codediesel.com/php/unpacking-binary-data/ "Permalink to Unpacking binary data in PHP")

# Unpacking binary data in PHP (Giải mã dữ liệu nhị phân trong PHP)

Trong PHP ta hiếm khi phải làm việc và thao tác với các file nhị phân. Tuy nhiên, khi cần thì hàm 'pack' và 'unpack' trong PHP có thể gíup ích đáng kể. Để chuẩn bị, ta sẽ bắt đầu với một vấn đề trong lập trình, điều này sẽ gíup cuộc thảo luận luôn gắn với một đến bối cảnh liên quan. Vấn đề ở đây là: Chúng ta muốn viết một hàm nhận một file ảnh làm tham số đầu vào kết quả sẽ cho chúng ta biết liệu file đó có là ảnh GIF hay không, mà không liên quan tới extension của file. Chúng ta không được phép sử dụng bất kỳ hàm của thư viện GD nào.

#### Header trong file GIF

Với yêu cầu đặt ra là chúng ta không được sử dụng bất kỳ hàm đồ họa nào, để giải quyết vấn đề chúng ta sẽ cần lấy những dữ liệu liên quan từ chính file GIF. Khác với các định dạng của các file văn bản như HTML hay XML, file GIF và hầu hết các định dạng của hình ảnh khác được lưu dưới định dạng nhị phân. Hầu hết các file nhị phân sẽ bao gồm một header nằm ở đầu của file chứa thông tin meta liên quan đến file cụ thể đó. Chúng ta có thể sử dụng thông tin này để biết được kiểu của file là gì và các thứ khác nữa, ví dụ như chiều cao và chiều rộng nếu ta đang xét đến file GIF. Một header dạng thô điển hình của file GIF được mô tả bên dưới, sử dụng trình sọan thảo kiểu hex như [WinHex][1].

![][2]

Chi tiết mô tả của header được cho như bên dưới.

        
    Offset   Độ dài   Nội dung
      0      3 bytes  "GIF"
      3      3 bytes  "87a" or "89a"
      6      2 bytes  
      8      2 bytes  
     10      1 byte   bit 0:    Global Color Table Flag (GCTF)
                      bit 1..3: Color Resolution
                      bit 4:    Sort Flag to Global Color Table
                      bit 5..7: Size of Global Color Table: 2^(1+n)
     11      1 byte   
     12      1 byte   
     13      ? bytes  
             ? bytes  
             1 bytes   (0x3b)

Vậy để kiểm tra xem liệu một file ảnh là file chuẩn GIF hay không, chúng ta cần kiểm tra 3 bytes đầu của header mà có chữ 'GIF' và 3 bytes tiếp theo, chúng sẽ cho ta biết về số của phiên bản; là '87a' hoặc '89a'. Chính những yêu cầu để thực hiện các tác vụ như trên mà hàm unpack() trở nên không thể thiếu trong PHP. Trước khi nhìn vào lời giải, chúng ta sẽ xem qua hàm unpack().

#### Sử dụng hàm unpack()

[unpack()][3] là sự bổ sung của [pack()][4] - nó chuyển hóa dữ liệu nhị phân thành mảng dựa trên định dạng cho trước. Điều này có điểm giống với _sprintf_, chuyển hóa dữ liệu chuỗi theo một vài định dạng cho trước. Hai hàm này cho phép chúng ta đọc và ghi các bộ đệm nhị phâm theo định dạng chuỗi cho trước. Điều dễ dàng cho phép một lập trình viên trao đổi dữ liệu giữa các chương trình được viết bằng các ngôn ngữ hoặc định dạng khác nhau. Hãy cùng xem ví dụ nhỏ sau đây.
      
    $data = unpack('C*', 'codediesel');
    var_dump($data);

Đoạn code trên sẽ in ra như sau, đó là các mã thập phân tương ứng với 'codediesel' :
  
    array
      1 => int 99
      2 => int 111
      3 => int 100
      4 => int 101
      5 => int 100
      6 => int 105
      7 => int 101
      8 => int 115
      9 => int 101
      10 => int 108

Trong ví dụ trên, tham số đầu tiên là định dạng chuỗi và tham số thứ hai là dữ liệu thực. Định dạng chuỗi sẽ chỉ định cách mà tham số dữ liệu được phân gỉai (parsed) như thế nào. Trong ví dụ này, phần đầu của định dạng là 'C', chỉ định rằng ta sẽ xử lý ký tự đầu của dữ liệu dưới dạng byte không dấu. Phần tiếp theo là '*', chỉ định rằng hàm sẽ áp dụng định dạng code ở phần trước lên tất cả các ký tự còn lại.

Dù điều này thoạt nhìn có vẻ khó hiểu, phần tiếp theo sẽ cung cấp ví dụ cụ thể hơn.

#### Lấy dữ liệu từ header

Dưới đây là gỉai pháp cho vấn đề GIF của chúng ta sử dụng hàm unpack(). Hàm _is_gif()_ sẽ trả về true nếu file được cho có định dạng GIF.

    function is_gif($image_file)
    {
     
        /* Mở file hình ảnh ở chế độ nhị phân */
        if(!$fp = fopen ($image_file, 'rb')) return 0;
     
        /* Đọc 20 bytes đầu của file */
        if(!$data = fread ($fp, 20)) return 0;
     
        /* Khai báo định dạng */
        $header_format = 'A6version';  # Get the first 6 bytes
    
        /* Unpack (gỉai mã) dữ liệu header */
        $header = unpack ($header_format, $data);
     
        $ver = $header['version'];
     
        return ($ver == 'GIF87a' || $ver == 'GIF89a')? true : false;
     
    }
     
    /* Run ví dụ */
    echo is_gif("aboutus.gif");

Dòng code quan trọng cần lưu ý là dòng khai báo định dạng. 'A6' chỉ định rằng hàm unpack() lấy 6 bytes đầu tiên của dữ liệu và gỉai mã chúng thành dạng chuỗi. Dữ liệu lấy được sẽ được lưu trong mảng với khóa tên là 'version'.

Một ví dụ khác được cho như bên dưới. Nó sẽ trả về thêm các thông tin header khác của file GIF, bao gồm độ rộng và độ dài của ảnh.
 
    function get_gif_header($image_file)
    {
     
        /* Mở file hình ảnh ở chế độ nhị phân */
        if(!$fp = fopen ($image_file, 'rb')) return 0;
     
        /* Đọc 20 bytes đầu của file */
        if(!$data = fread ($fp, 20)) return 0;
     
        /* Khai báo định dạng */
        $header_format = 
                'A6Version/' . # Lấy 6 bytes đầu
                'C2Width/' .   # Lấy 2 bytes tiếp theo
                'C2Height/' .  # Lấy 2 bytes tiếp theo
                'C1Flag/' .    # Lấy 1 byte tiếp theo
                '@11/' .       # Nhảy đến byte thứ 12
                'C1Aspect';    # Lấy 1 byte tiếp theo
    
        /* Unpack (giải mã) dữ liệu header */
        $header = unpack ($header_format, $data);
     
        $ver = $header['Version'];
     
        if($ver == 'GIF87a' || $ver == 'GIF89a') {
            return $header;
        } else {
            return 0;
        }
    }
     
    /* Run ví dụ */
    print_r(get_gif_header("aboutus.gif"));

Ví dụ trên sẽ in ra như sau khi chạy.
 
    Array
    (
        [Version] => GIF89a
        [Width1] => 97
        [Width2] => 0
        [Height1] => 33
        [Height2] => 0
        [Flag] => 247
        [Aspect] => 0
    )

Tiếp theo chúng ta sẽ tìm hiểu kỹ hơn cách hoạt động của định dạng khai báo. Ta sẽ tách, phân tích định dạng và đi vào chi tiết từng ký tự.

    $header_format = 'A6Version/C2Width/C2Height/C1Flag/@11/C1Aspect';    
    
    A - Đọc một byte và gỉai mã nó thành một chuỗi. 
        Số các byte để đọc sẽ được cho trong phần tiếp theo
    6 - Đọc tổng cộng 6 bytes, bắt đầu từ vị trí 0
    Version - Tên của khóa trong mảng lưu trữ mà dữ liệu được lấy về bởi 'A6' 
     
    / - Bắt đầu một định dạng code mới
    C - Gỉai mã dữ liệu tiếp theo thành byte không dấu
    2 - Đọc tổng cộng 2 bytes
    Width - Tên của khóa trong mảng
     
    / - Bắt đầu một định dạng code mới
    C - Gỉai mã dữ liệu tiếp theo thành byte không dấu
    2 - Đọc tổng cộng 2 bytes
    Height- Tên của khóa trong mảng
     
    / - Bắt đầu một định dạng code mới
    C - Gỉai mã dữ liệu tiếp theo thành byte không dấu
    1 - Đọc tổng cộng 2 bytes
    Flag - Tên của khóa trong mảng
     
    / - Bắt đầu một định dạng code mới
    @ - Dịch chuyển số byte offset theo số được chỉ định sau.
          Lưu ý rằng vị trí đầu tiên trong chuỗi nhị phân là 0. 
    11 - Dịch chuyển đến vị trí 11
     
    / - Bắt đầu một định dạng code mới
    C - Gỉai mã dữ liệu tiếp theo thành byte không dấu
    1 - Đọc tổng cộng 1 byte
    Aspect - Tên của khóa trong mảng

Tùy chọn về các định dạng khác có thể tìm thấy tại [đây][4]. Dù chúng ta mới đi qua một ví dụ nhỏ, pack/unpack có thể sử dụng cho các công việc phức tạp hơn nhiều so với những điều đã được trình bày ở đây.

Lưu ý: Kể từ phiên bản PHP 7.2.0, kiểu float và double hỗ trợ cả Big Endian và Little Endian.

[1]: http://www.x-ways.net/winhex/index-m.html
[2]: http://www.codediesel.com/wp-content/uploads/2010/09/winhex.gif "winhex"
[3]: http://php.net/manual/en/function.unpack.php
[4]: http://www.php.net/manual/en/function.pack.php

  
