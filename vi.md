# 12-factor
## Giới thiệu
Trong thời đại ngày nay, phần mềm thường được chuyển thành các dịch vụ, gọi là các ứng dụng web hoặc phần-mềm-như-1-dịch-vụ. Phần mềm theo 12-chuẩn là 1 phương pháp luận xây dựng phần-mềm-như-1-dịch-vụ:

- Sử dụng các định dạng được khai báo để cài đặt tự động, tối thiểu thời gian và chi phí cho các người phát triển mới tham gia vào dự án:
- Có các giao ước rõ ràng với hệ điều hành ngầm, cho phép tính di dộng tối đã giữa các môi trường thực thi.
- Trở nên phù hợp để phát triển trên các nền tảng đám mây hiện đại, giảm bớt yêu cầu từ server và các hệ thống quản trị.
- Tối thiểu sự khác nhau giữa development và production, cho phép phát triển liên tục 1 cách linh động nhất.
- Và có thể mwor rộng và không có những thay đổi đáng kể nào với công cụ, kiến trúc, hay thói quen phát triển.


Phương pháp luận 12-chuẩn có thể áp dụng cho các ứng dụng viết bằng bất kỳ ngôn ngữ lập trình nào, và sử dụng bất kỳ tập dịch vụ nền nào (database, queue, memory cache, etc).

## Background
Các tác giả của tài liệu đã áp dụng trực tiếp trong quá trình phát triển và triển khai hàng trăm ứng dụng, và  giản tiếp chứng kiến quá trình phát triển, điều hành và mở rộng của hàng trăm nghìn ứng dụng qua công việc trên nền tảng Heroku.

Tài liệu này tổng hợp tất cả kinh nghiệm và quan sát của chúng tôi trên rất nhiều các ứng dụng phần-mềm-như-1-dịch-vụ trong cuộc sống. Nó là 3 khía cạnh chính trên các ý tưởng thực hành cho phát triển ứng dụng, tập trung cụ thể vào tính linh động của phát triển liên tục 1 cách có tổ chức của ứng dụng, tính linh động trong cộng tác giữa các người phát triển làm việc trên codebase của ứng dụng, và tránh các chi phí của vận hành ứng dụng.

Động lực của chúng tôi là nâng cao nhậnận thức về vài vấn đề có hệệ thống chúng tôiôi đã gặp trong phát triển ứng dụng hiện đại, để cung cấp các từ vựng chung để bàn luận về những vấn đề này, và để cho phép 1 tập các giải pháp khái niệm rộng lớn vào các vấn đề với các thuật ngữ kèm theo. Định dạn này lấy cảm hứng bởi cuốn sách của Martin Fowler, Patterns of Enterprise Application Architecture and Refactoring.
## Ai nên đọc tài liệu này?
Bất kỳ người phát triển nào đang xây dựng các ứng dụng chạy như là 1 dịch vụ. Các kĩ sư Ops triển khai hoặc điều khiển những ứng dụng như vậy.

#### I. Codebase
###### 1 codebase được theo dõi trong việc kiểm soát các thay đổi, nhiều triển khai.

1 ứng dụng tuân theo 12-chuẩn luôn luôn được theo dõi trong 1 hệ thống kiểm soát các phiên bản, như là Git, Mercurial, hoặc Subversion. 1 bản sao chéo của cơ sở dữ liệu theo dõi thay đổi được biết như là 1 code repository, thường được viết tắt là code repo hay chỉ là repo mà thôi.

1 codebase có thể là bất cứ repo đơn lẻ nào nào (trong 1 hệ thống kiểm soát sửa đổi tập trung như Subverrsion), hoặcoặc có thể lả 1 tập các repo chia sẻ chung 1 commit gốc(trong 1 hệ thống kiểm soát sửa đổi phân cấp như Git). 

Luôn luôn có sự tương quan 1-1 giữa codebase và ứng dụng:
- Nếu có nhiều codebase, đó không phải là 1 ứng dụng - nó là 1 hệ thống phân cấp. Mỗi thành phần là 1 hệ thống được phân cấp trong 1 ứng dụng, và mỗi thành phần có thể tuân theo 12- chuẩn riêng 1 cách riêng lẻ.
- Nhiều ứng dụng chia sẻ cùng code là 1 sư vi phạm 12- chuẩn. Giải pháp ở đây là quản lý chia sẻ code vào các thư viện mà có thể được thêm vào thông qua quản lý phụ thuộc.

Chỉ có duy nhất 1 codebase mỗi ứng dụng, nhưng sẽ có nhiều triển khải của ứng dụng. 1 triển khai sẽ chạy phiên bản của ứng dụng. Điều này thường là 1 phía của production, với 1 hoặc nhiều phía staging. Thêm vào đó, mỗi người phát triển có 1 bản sao của ứng dụng đang chạy trên môi trường developer nội bộ của họ, mỗi số chúng cũng được coi như 1 triển khai

Codebase là chung giữa tất cả các triển khai, mặc dù các phiên bản khác nhau có thể hoạt động trong mỗi triển khai. Ví dụ, 1 người phát triển có vài commit chưa đượcđược triển khai đến staging, staging có vài commit chưa được triển khai tới production. Nhưng tất cả chúng đều chia sẻ cùng codebase, vì thế làm chún có thể được định dạng như là các triển khai khác nhau của cùng 1 ứng dụng
#### II. các phụ thuộc
###### Khai báo rõ ràng và các phụ thuộc cô lập

Hầu hết các ngôn ngữ lập trình cho phép 1 hệ thống packaging phân phối các thư viện hỗ trợ, như là CPAN cho Perl hoặc Rubygems cho Ruby. Các thư viện được cài đặt thông qua hệ thống packaging có thể được cài đặt mức-hệ-thống ( được hiểu như là “các site package”) hoặc chỉ trọng phạm vi thư mục chứa ứng dụng ( được biết như là “vendoring” hoặc “bundling”).

1 ứng dụng theo 12-chuẩn không bao giờ phụ thuộc vào các tồn tại ngầm của các gọi mức-hệ-thống. Nó khai báo tất cả các phụ thuộc, hoàn thiện và chính xác, thống qua 1 biểu thị khai báo phụ thuộc. Hơn nữa, nó sử dụng 1 công cụ cô lập phụ thuộc trong quá trình thực thi để đảm bảo rằng không có phụ thuộc ngầm nào “lọt vào”  từ các hệ thống xung quanh. Đặc tả phụ thuộc đầy đủ và rõ ràng được áp dụng thống nhất cho cả productiont và developer.

Ví dụ, Bundler cho Ruby cho phép định dạng biểu thị `Gemfile` cho việc khai báo phụ thuộc và `bundle exec` cho việc cô lập phụ thuộc. Trong Python có 2 công cụ riêng biệt cho các bước này - Pip được sử dụng để khai báo và Virtualenv để cô lập. Thâm chí C có Autoconf cho việc khai báo phụ thuộc và link tĩnh có thể cung cấp cô lập các phụ thuộc. Bất kể dùng tập công cụ nào, khai báo và cô lập phụ thuộc luôn phải được sử dụng cùng nhau - chỉ một là không đủ để thỏa mãn 12-chuẩn.

1 lợi ích của việc khai báo phụ thuộc rõ ràng là nó đơn giản hóa việc cài đặt cho những người phát triển mới của ứng dụng. Người phát triển mới có thể lấy codebase của ứng dụng về máy phát triển của họ, với điều kiện tiên quyết chỉ yêu cầu ngôn ngữữ chạy và quản lý phụ thuộc được cài đặt. Họ sẽ cóó thể cài đặt bất cứ thứ gì cần thiết để chạy code của ứng dụng với 1 lệnh xây dựng xác định. Ví dụ, lấy xây dựng cho Ruby/Bundler là `bundle install`, còn với Clojureojure/Leiningen là `lein dép`.

Các ứng dụng theo 12-chuẩn cũng không phụ thuộc vào các tồn tại ngầm của bất kỳ công cụ hệ thống nào. Ví dụ bao gồm cả với ImageMagick hay curl. Trong khi nhưng công cụ này có thể tồn tại trên nhiều hay thậm chí là hầu hết các hệ thống, không có gì bảo đảm rằng chúng sẽ tồn tại trên tất cả các hệ thống mà ứng dụng có thể chạy trong tương lại, hay 1 phiên bản được tìm thấy trong hệ thống tương lai có thể tương thích với ứng dụng. Nếu ứng dụng cần 1 công cụ hệ thống, công cụ đó cần được gán vào ứng dụng.

#### III. Cấu hình
###### Lưu cấu hình trong môi trường
Cấu hình của 1 ứngứng dụng là bất cứ thứ gì giống như là sự thay đổi giữa các triển khai (môi trường staging, production, developer, vân vân). Nó bao gồm:

- Nguồn xử lý cơ sở dữ liệu, Memcaches, và các dịch vụ hỗ trợ khác.
- Thông tin đăng nhập cho các dịch vụ bên ngoài như Amazon S3 hoặc Twitter
- Các giá trị của mỗi-triển-khai như là tên máy chủ của triển khai

Các ứng dụng đôi khi lưu trữ cấu hình như các hằngằng số trong codeode. Điều này xung đột với 12-chuẩn, khi nó yêu cầu `tính tách biệt 1 cách nghiêm ngặt trong cấu hình ở coe`. Cấu hình thay đổi  giữa các triển khai, còn code thì không.

1 kiểm thử litmus kiểm tra ứng dụng có tất cả cấuấu hình chính xác với code là khi codebae có thể tạo mã nguồn mở bất cứ khi nào, mà không ảnh hưởng bất cứ thông tin xác thực nào.

Lưu ý rằng định nghĩa `cấu hình` không bao gồm các cấu hình nội bộ ứng dụng, ví dụ như là `config/routes.rb` trong Rails, hoặc cách các module được kết nối trong Spring. Loại cấu hình này không thay đổi giữa các triển khai, và ví thể hoàn thiện nhất trong code.

1 cách tiếp cận khác với cấu hình là sử dụng các file cấu hình không được kiểm soát trong kiểm soát thay đổi, như là `config/database.yml` trong Rails. Đây là 1 cải thiện lớn so với việc sử dụng các hằng số được kiểm soát trongong code repo, nhưng vẫn còn những nhược điểm: nó rất dễ kiểm soát lỗi 1 file cấu hình trong repo, có 1 xu hướng là các file cấu hình sẽ nằm rải rác ở những chỗ khác nhau và trong những định dạng khác nhau, khiên nó khó để thấy và kiểm soát tất cả các cấu hình trong 1 chỗ. Hơn nữa những định dạng này có xu hướng riêng biệt với từng ngôn ngữ hay framework.

Ứng dụng theo 12-chuẩn lưu trữ cấu trình trong các biến môi trường ( thường đọc tắt là env vars hoặc env). ENV vars thường dễ thay đổi giữa các triển khai mà không phải thay đổi code, không giống như các file cấu hình, chỉ 1 chút thay đổi của chúng được kiểm soát phụ thuocj bởi code repo; và không giống các file cấu hình thông thường,  các cơ chế cấu hình khác như Java System Properties, chúng là các chuẩn ngôn ngữ và OS-agnostic.

1 khía cạnh khác của quản lý cấu hình là gom nhóm. Đôi khi các ứng dụng gom cấu hình thành các tên nhóm ( thường gọi là “các môi trường”) được đặt tên sau các triển khai cụ thể, như là môi trường `development`, `test`, và `production` trong Rails. Phương pháp này không được gọn cho lắm: vì sẽ có nhiều triển khai của ứng dụng được tạo ra, các tên môi trường mới sẽ cần thiết, như là `staging` hoặc `qa`. Vì dự án sẽ càng phát triển hơn, các người phát triển có thể sẽ thêm các môi trường đặc biệt riêng của họ như `joes-staging`, kết quả của việc kết hợp đống cấu hình này sẽ khiến việc quản lý triển khải của ứng dụng trở nên mong manh.

Trong các ứng dụng 12-chuẩn, env vars là các điều khiển chi tiết, mỗi chúng trực giao đầy đủ với các env vars khác. Chúng không vào giờ được nhóm lại với nhau như làà "các môi trường", nhưng thay vào đó chúng quản lý độc lập cho từng triển khai. Đây là 1 mô hình nâng cao sự mượt mà, ứng dụng sự mở rộng 1 cách tự nhiên đến nhiều triển khai hơn trong suốt vòng đời của nó.


