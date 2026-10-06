# Phân rã Shopee theo khả năng kinh doanh (Business Capability)

## Bối cảnh và cách tiếp cận

Giả sử cần thiết kế một nền tảng thương mại điện tử có các khả năng tương tự Shopee. Bài viết này là **phương án phân rã giả định để học kiến trúc**, không mô tả kiến trúc nội bộ thực tế của Shopee.

Phân rã *top-down* bắt đầu từ việc doanh nghiệp cần làm gì: quản lý người dùng, vận hành gian hàng, giới thiệu sản phẩm, tạo đơn, thu tiền và giao hàng. Sau đó mới xác định module nào chịu trách nhiệm cho từng khả năng. Một module có thể được triển khai như phần của ứng dụng lớn hoặc thành service riêng; ranh giới nghiệp vụ không bắt buộc tương ứng ngay với ranh giới triển khai.

## Các module chính

| # | Module / Service | Trách nhiệm chính | Dữ liệu do module sở hữu | Ngoài phạm vi |
| --- | --- | --- | --- | --- |
| 1 | **Identity & Account** | Đăng ký, đăng nhập, quản lý tài khoản và quyền truy cập của người mua, người bán. | Tài khoản, thông tin xác thực, vai trò. | Hồ sơ và hoạt động kinh doanh của gian hàng. |
| 2 | **Seller Management** | Đăng ký gian hàng, quản lý hồ sơ người bán và trạng thái hoạt động của gian hàng. | Gian hàng, hồ sơ người bán, trạng thái xét duyệt. | Nội dung sản phẩm và đơn hàng. |
| 3 | **Catalog** | Tạo, sửa, phân loại và công bố thông tin sản phẩm cùng giá niêm yết. | Sản phẩm, danh mục, biến thể, giá niêm yết. | Số lượng tồn thực tế và xếp hạng kết quả tìm kiếm. |
| 4 | **Search & Discovery** | Lập chỉ mục, tìm kiếm, lọc và sắp xếp sản phẩm để người mua khám phá hàng hóa. | Chỉ mục tìm kiếm và cấu hình xếp hạng. | Bản ghi sản phẩm gốc; dữ liệu này thuộc Catalog. |
| 5 | **Cart** | Lưu lựa chọn mua của người dùng và chuẩn bị dữ liệu đầu vào cho bước đặt hàng. | Giỏ hàng, dòng hàng đang chọn. | Đơn hàng đã xác nhận và giá cuối cùng của giao dịch. |
| 6 | **Inventory** | Theo dõi tồn kho theo sản phẩm/biến thể, giữ chỗ và giải phóng hàng khi cần. | Tồn kho, lượt giữ chỗ, lịch sử biến động tồn. | Mô tả sản phẩm và trạng thái thanh toán. |
| 7 | **Promotion** | Quản lý mã giảm giá, điều kiện áp dụng và mức giảm cho giao dịch. | Chương trình khuyến mãi, mã giảm giá, lượt sử dụng. | Giá niêm yết trong Catalog và số tiền đã thu. |
| 8 | **Order Management** | Tạo đơn, ghi nhận các dòng hàng và tổng tiền tại thời điểm đặt, quản lý vòng đời đơn hàng. | Đơn hàng, dòng hàng, trạng thái đơn, bản chụp giá. | Quyết toán thanh toán, tồn kho và vận chuyển chi tiết. |
| 9 | **Payment** | Khởi tạo yêu cầu thanh toán, tiếp nhận kết quả từ nhà cung cấp, ghi nhận thu tiền và hoàn tiền. | Giao dịch thanh toán, trạng thái thu/hoàn tiền. | Trạng thái nghiệp vụ tổng thể của đơn hàng. |
| 10 | **Shipping & Fulfillment** | Tạo yêu cầu giao hàng, kết nối đơn vị vận chuyển và theo dõi hành trình giao hàng. | Vận đơn, trạng thái giao, thông tin bàn giao. | Thu tiền và danh mục sản phẩm. |

Danh sách trên có **10 module**, nhiều hơn mức tối thiểu 6 module của đề bài. Trong triển khai thực tế, quy mô đội ngũ và lưu lượng có thể khiến một số module được gộp hoặc tách thêm.

## Làm rõ ranh giới để tránh chồng chéo

- **Catalog và Inventory:** Catalog trả lời “sản phẩm là gì, giá niêm yết bao nhiêu”; Inventory trả lời “còn bao nhiêu hàng có thể bán”. Catalog không tự cập nhật tồn kho.
- **Catalog và Search:** Catalog là nguồn dữ liệu sản phẩm gốc; Search giữ bản sao phục vụ tìm kiếm và có thể cập nhật chỉ mục khi sản phẩm thay đổi.
- **Cart và Order Management:** Cart lưu ý định mua còn có thể chỉnh sửa; Order Management tạo bản ghi giao dịch khi người dùng đặt hàng và giữ bản chụp thông tin cần thiết để đơn không thay đổi theo giỏ.
- **Order Management và Payment:** Order Management quyết định trạng thái của đơn; Payment là nguồn sự thật về giao dịch thu và hoàn tiền. Order Management nhận kết quả thanh toán qua giao diện hoặc sự kiện để cập nhật đơn.
- **Order Management và Shipping:** Đơn hàng ghi nhận tiến trình mua bán; Shipping sở hữu vận đơn và trạng thái giao thực tế. Order Management nhận thông tin giao hàng để hoàn tất vòng đời đơn.
- **Seller Management và Identity & Account:** Identity & Account quản lý danh tính và quyền truy cập; Seller Management quản lý gian hàng như một thực thể kinh doanh.

Mỗi module nên sở hữu dữ liệu nghiệp vụ của mình. Module khác truy cập thông qua API hoặc sự kiện đã thống nhất, thay vì sửa trực tiếp bảng dữ liệu của module đó. Dữ liệu sao chép để đọc, chẳng hạn chỉ mục Search, cần có cơ chế đồng bộ và chấp nhận độ trễ phù hợp.

## Ví dụ luồng đặt hàng

1. Người mua chọn sản phẩm từ **Catalog/Search** và đưa vào **Cart**.
2. Khi đặt hàng, **Order Management** lấy thông tin cần thiết, yêu cầu **Promotion** tính ưu đãi và **Inventory** giữ chỗ hàng. Đơn ghi lại giá cuối cùng tại thời điểm đặt.
3. **Payment** xử lý yêu cầu thanh toán và báo kết quả cho **Order Management**. Nếu thanh toán thất bại hoặc hết thời gian chờ, quy trình giải phóng phần hàng đã giữ chỗ.
4. Khi đơn đủ điều kiện giao, **Shipping & Fulfillment** tạo vận đơn và cập nhật quá trình giao hàng. **Order Management** nhận kết quả để cập nhật trạng thái đơn.

Luồng này minh họa sự phối hợp giữa các module mà vẫn giữ rõ nơi sở hữu từng quyết định. Khi triển khai bằng nhiều service, cần xử lý timeout, yêu cầu lặp và tình huống một bước thành công nhưng bước sau thất bại.

## Kết luận

Phân rã theo khả năng kinh doanh giúp đặt ranh giới quanh **trách nhiệm và dữ liệu nghiệp vụ**, thay vì chia theo controller, bảng dữ liệu hoặc công nghệ. Với phương án trên, mỗi module có một chức năng chính và phạm vi loại trừ rõ ràng; các module phối hợp để hoàn thành hành trình mua hàng mà không cùng sở hữu một quy tắc nghiệp vụ.
