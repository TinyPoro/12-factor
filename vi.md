# 12-factor
## Giới thiệu
Trong thời đại ngày nay, phần mềm thường được chuyển thành các dịch vụ, gọi là các ứng dụng web hoặc phần-mềm-như-1-dịch-vụ. Phần mềm theo 12-chuẩn là 1 phương pháp luận xây dựng phần-mềm-như-1-dịch-vụ:

- Sử dụng các định dạng được khai báo để cài đặt tự động, tối thiểu thời gian và chi phí cho các người phát triển mới tham gia vào dự án:
- Có các giao ước rõ ràng với hệ điều hành ngầm, cho phép tính di dộng tối đã giữa các môi trường thực thi.
- Thích hợp để phát triển trên các nền tảng đám mây hiện đại, giảm bớt yêu cầu từ server và các hệ thống quản trị.
- Giảm thiểu sự khác nhau giữa development và production, cho phép phát triển liên tục 1 cách nhanh nhất.
- Và có thể mở rộng và không có những thay đổi đáng kể nào với công cụ, kiến trúc, hay thói quen phát triển.


Phương pháp luận 12-chuẩn có thể áp dụng cho các ứng dụng viết bằng bất kỳ ngôn ngữ lập trình nào, và sử dụng bất kỳ tập dịch vụ sao lưu nào (database, queue, memory cache, etc).

## Background
Các tác giả của tài liệu đã áp dụng trực tiếp trong quá trình phát triển và triển khai hàng trăm ứng dụng, và  gián tiếp thấy quá trình phát triển, vận hành và mở rộng của hàng trăm nghìn ứng dụng qua công việc của chúng tôi trên nền tảng Heroku.

Tài liệu này tổng hợp tất cả kinh nghiệm và sự quan sát của chúng tôi trên rất nhiều các ứng dụng phần-mềm-như-1-dịch-vụ trong cuộc sống. Nó là 3 khía cạnh chính trên các ý tưởng thực hành cho phát triển ứng dụng, tập trung cụ thể vào tính linh động của phát triển liên tục 1 cách có tổ chức của ứng dụng, tính linh động trong cộng tác giữa các người phát triển làm việc trên codebase của ứng dụng, và tránh các chi phí của vận hành ứng dụng.

Động lực của chúng tôi là nâng cao nhận thức về vài vấn đề có hệ thống chúng tôi đã gặp trong phát triển ứng dụng hiện đại, để cung cấp các từ vựng chung để bàn luận về những vấn đề này, và để cho phép 1 tập các giải pháp khái niệm rộng lớn vào các vấn đề với các thuật ngữ kèm theo. Định dạng này lấy cảm hứng bởi cuốn sách của Martin Fowler, Patterns of Enterprise Application Architecture and Refactoring.
## Ai nên đọc tài liệu này?
Bất kỳ người phát triển nào đang xây dựng các ứng dụng chạy như là 1 dịch vụ. Các kĩ sư Ops triển khai hoặc điều khiển những ứng dụng như vậy.

#### I. Codebase
###### 1 codebase được theo dõi trong việc kiểm soát các thay đổi, nhiều triển khai.

1 ứng dụng tuân theo 12-chuẩn luôn luôn được theo dõi trong 1 hệ thống kiểm soát các phiên bản, như là Git, Mercurial, hoặc Subversion. 1 bản sao chéo của cơ sở dữ liệu theo dõi thay đổi được biết như là 1 code repository, thường được viết tắt là code repo hay chỉ là repo mà thôi.

1 codebase có thể là bất cứ repo đơn lẻ nào nào (trong 1 hệ thống kiểm soát sửa đổi tập trung như Subverrsion), hoặc có thể lả 1 tập các repo chia sẻ chung 1 commit gốc(trong 1 hệ thống kiểm soát sửa đổi phân cấp như Git). 

Luôn luôn có sự tương quan 1-1 giữa codebase và ứng dụng:
- Nếu có nhiều codebase, đó không phải là 1 ứng dụng - nó là 1 hệ thống phân cấp. Mỗi thành phần là 1 hệ thống được phân cấp trong 1 ứng dụng, và mỗi thành phần có thể tuân theo 12- chuẩn riêng 1 cách riêng lẻ.
- Nhiều ứng dụng dùng chung cùng code là 1 sư vi phạm 12- chuẩn. Giải pháp ở đây là quản lý chia sẻ code vào các thư viện mà có thể được thêm vào thông qua quản lý phụ thuộc.

Chỉ có duy nhất 1 codebase mỗi ứng dụng, nhưng sẽ có nhiều triển khải của ứng dụng. 1 triển khai sẽ chạy phiên bản của ứng dụng. Điều này thường là 1 phía của production, với 1 hoặc nhiều phía staging. Thêm vào đó, mỗi người phát triển có 1 bản sao của ứng dụng đang chạy trên môi trường developer nội bộ của họ, mỗi số chúng cũng được coi như 1 bản triển khai

Codebase là chung giữa tất cả các bản triển khai, mặc dù các phiên bản khác nhau có thể hoạt động trong mỗi bản triển khai. Ví dụ, 1 người phát triển có vài commit chưa được triển khai đến staging, staging có vài commit chưa được triển khai tới production. Nhưng tất cả chúng đều chia sẻ cùng codebase, vì thế làm chún có thể được định dạng như là các triển khai khác nhau của cùng 1 ứng dụng
#### II. các phụ thuộc
###### Khai báo rõ ràng và tách biệt các phụ thuộc

Hầu hết các ngôn ngữ lập trình cho phép 1 hệ thống packaging phân phối các thư viện hỗ trợ, như là CPAN cho Perl hoặc Rubygems cho Ruby. Các thư viện được cài đặt thông qua hệ thống packaging có thể được cài đặt mức-hệ-thống ( được hiểu như là “các site package”) hoặc chỉ trọng phạm vi thư mục chứa ứng dụng ( được biết như là “vendoring” hoặc “bundling”).

1 ứng dụng theo 12-chuẩn không bao giờ phụ thuộc vào các tồn tại ngầm của các gói mức-hệ-thống. Nó khai báo tất cả các phụ thuộc, hoàn thiện và chính xác, thống qua 1 biểu thị khai báo phụ thuộc. Hơn nữa, nó sử dụng 1 công cụ tách biệt phụ thuộc trong quá trình thực thi để đảm bảo rằng không có phụ thuộc ngầm nào “lọt vào”  từ các hệ thống xung quanh. Đặc tả phụ thuộc đầy đủ và rõ ràng được áp dụng thống nhất cho cả production và developer.

Ví dụ, Bundler cho Ruby cho phép định dạng biểu thị `Gemfile` cho việc khai báo phụ thuộc và `bundle exec` cho việc tách biệt phụ thuộc. Trong Python có 2 công cụ riêng biệt cho các bước này - Pip được sử dụng để khai báo và Virtualenv để tách biệt. Thâm chí C có Autoconf cho việc khai báo phụ thuộc và link tĩnh có thể cung cấp tách biệt các phụ thuộc. Bất kể dùng tập công cụ nào, khai báo và tách biệt phụ thuộc luôn phải được sử dụng cùng nhau - chỉ một là không đủ để thỏa mãn 12-chuẩn.

1 lợi ích của việc khai báo phụ thuộc rõ ràng là nó đơn giản hóa việc cài đặt cho những người phát triển mới của ứng dụng. Người phát triển mới có thể lấy codebase của ứng dụng về máy phát triển của họ, với điều kiện tiên quyết chỉ yêu cầu ngôn ngữữ chạy và quản lý phụ thuộc được cài đặt. Họ sẽ cóó thể cài đặt bất cứ thứ gì cần thiết để chạy code của ứng dụng với 1 lệnh xây dựng xác định. Ví dụ, lấy xây dựng cho Ruby/Bundler là `bundle install`, còn với Clojureojure/Leiningen là `lein dép`.

Các ứng dụng theo 12-chuẩn cũng không phụ thuộc vào các tồn tại ngầm của bất kỳ công cụ hệ thống nào. Ví dụ bao gồm cả với ImageMagick hay curl. Trong khi nhưng công cụ này có thể tồn tại trên nhiều hay thậm chí là hầu hết các hệ thống, không có gì bảo đảm rằng chúng sẽ tồn tại trên tất cả các hệ thống mà ứng dụng có thể chạy trong tương lại, hay 1 phiên bản được tìm thấy trong hệ thống tương lai có thể tương thích với ứng dụng. Nếu ứng dụng cần 1 công cụ hệ thống, công cụ đó cần được gán vào ứng dụng.

#### III. Cấu hình
###### Lưu cấu hình trong môi trường
Cấu hình của 1 ứng dụng là bất cứ thứ gì có khả năng thay đổi giữa các bản triển khai (môi trường staging, production, developer, vân vân). Nó bao gồm:

- Nguồn xử lý cơ sở dữ liệu, Memcaches, và các dịch vụ hỗ trợ khác.
- Thông tin đăng nhập cho các dịch vụ bên ngoài như Amazon S3 hoặc Twitter
- Các giá trị của mỗi-triển-khai như là tên máy chủ của triển khai

Các ứng dụng đôi khi lưu trữ cấu hình như các hằngằng số trong codeode. Điều này xung đột với 12-chuẩn, khi nó yêu cầu `tính tách biệt 1 cách nghiêm ngặt trong cấu hình ở code`. Cấu hình thay đổi  giữa các triển khai, còn code thì không.

1 kiểm thử litmus kiểm tra ứng dụng có tất cả cấuấu hình chính xác với code là khi codebae có thể tạo mã nguồn mở bất cứ khi nào, mà không ảnh hưởng bất cứ thông tin xác thực nào.

Lưu ý rằng định nghĩa `cấu hình` không bao gồm các cấu hình nội bộ ứng dụng, ví dụ như là `config/routes.rb` trong Rails, hoặc cách các module được kết nối trong Spring. Loại cấu hình này không thay đổi giữa các bản triển khai, và do đó được thực hiện tốt nhất trong code.

1 cách tiếp cận khác với cấu hình là sử dụng các file cấu hình không được kiểm tra trong trình quản lý thay đổi, như là `config/database.yml` trong Rails. Đây là 1 cải thiện lớn so với việc sử dụng các hằng số được kiểm soát trongong code repo, nhưng vẫn còn những nhược điểm: nó rất dễ kiểm soát lỗi 1 file cấu hình trong repo, có 1 xu hướng là các file cấu hình sẽ nằm rải rác ở những chỗ khác nhau và trong những định dạng khác nhau, khiên nó khó để thấy và kiểm soát tất cả các cấu hình trong 1 chỗ. Hơn nữa những định dạng này có xu hướng riêng biệt với từng ngôn ngữ hay framework.

Ứng dụng theo 12-chuẩn lưu trữ cấu trình trong các biến môi trường ( thường đọc tắt là env vars hoặc env). ENV vars thường dễ thay đổi giữa các bản triển khai mà không phải thay đổi code, không giống như các file cấu hình, chỉ 1 chút thay đổi của chúng được kiểm soát phụ thuocj bởi code repo; và không giống các file cấu hình thông thường,  các cơ chế cấu hình khác như Java System Properties, chúng là các chuẩn ngôn ngữ và OS-agnostic.

1 khía cạnh khác của quản lý cấu hình là gom nhóm. Đôi khi các ứng dụng gom cấu hình thành các tên nhóm ( thường gọi là “các môi trường”) được đặt tên sau các triển khai cụ thể, như là môi trường `development`, `test`, và `production` trong Rails. Phương pháp này không được gọn cho lắm: vì sẽ có nhiều triển khai của ứng dụng được tạo ra, các tên môi trường mới sẽ cần thiết, như là `staging` hoặc `qa`. Vì dự án sẽ càng phát triển hơn, các người phát triển có thể sẽ thêm các môi trường đặc biệt riêng của họ như `joes-staging`, kết quả của việc kết hợp đống cấu hình này sẽ khiến việc quản lý triển khải của ứng dụng trở nên mong manh.

Trong các ứng dụng 12-chuẩn, env vars là các điều khiển chi tiết, mỗi chúng trực giao đầy đủ với các env vars khác. Chúng không vào giờ được nhóm lại với nhau như làà "các môi trường", nhưng thay vào đó chúng quản lý độc lập cho từng triển khai. Đây là 1 mô hình nâng cao sự mượt mà, ứng dụng sự mở rộng 1 cách tự nhiên đến nhiều triển khai hơn trong suốt vòng đời của nó.

#### IV. Các dịch vụ sao lưu
###### Coi các dịch vụ sao lưu như các nguồn đi kèm
1 dịch vụ sao lưu là bất cứ dịch vụ nào mà ứng dụng sử dụng trong suốt kết nối như là 1 phần của quá trình điều hành thông thường của nó. Các ví dụ bao gồm các kho dữ liệu ( như MySQL hoặc CouchDB), các hệ thống nhắn tin/ xếp hàng ( như RabbitMQ hoặc Beanstalkd), các dịch vụ SMTP cho gửi  email(như Postfix) và các hệ thống caching ( như Memcaches).

Các dịch vụ sao lưu  như cơ sở dữ liệu thông thường được quản lý bởi cùng các quản trị hệ thống như là các triển khai trong quá trình chạy của ứng dụng. Ngoài những dịch vụ được quản lý cục bộ này, ứng dụng còn có thể có các dịch vụ được cung cấp và quản lý bởi các bên thứ ba. Ví dụ bao gồm các dịch vụ SMTP (như Postmark), các dịch vụ thu thập số liệu ( như là New Relic hay Loggly), các dịch vụ sở hữu nhị phân( nhưu là Amazon S3), và thậm chí các dịch vụ sử dụng APO-truy-cập ( như là Twitter, Google Maps, hay Last.fm).

Code cho ứng dụng theo 12-chuẩn không phân biệt các dịch vụ cục bộ hay là bên thứ ba. Đối với ứng dụng, cả 2 đều là các nguồn đi kèm, được truy cập thông qua 1 URL hay các định vị/ thông tin xác thực khác được lưu trữ trong cấu hình. 1 triển khai của ứng dụng theo 12-chuẩn nên có thể hoán đổi giữa cơ sở dữ liệu MySQL cục bộ với 1 cái được quản lý bởi 1 bên thứ ba (như là Amazon RDS) mà không cần bất kỳ thay đổi nào trong code của ứng dụng. Tương tự, 1 máy chủ SMTP cục bộ có thể được tráo đổi với 1 dịch vụ SMTP bên thứ ba (như Postmark) mà không phải thay đổi code. Trong cả 2 trường hợp, chỉ có các tài nguyên được xử lý trong cấu hình cần được thay đổi.

Mỗi dịch vụ sao lưu riêng biệt là 1 tài nguyên. Ví dụ, 1 cơ sở dữ liệu MySQL là 1 tài nguyên, 2 cơ sở dữ liệu MySQL ( sử dụng  song song tại tầng ứng dụng) được coi là 2 tài nguyên riêng biệt. Ứng dụng theo 12-chuẩn coi các cơ sở dữ liệu này như là các tài nguyên đi kèm, chỉ định kết nối lỏng lẻo của chúng với triển khai mà chúng đi kèm.

Các tài nguyên có thể được gắn kèm cũng như được tách rời khỏi các triển khai 1 cách tùy ý. Ví dụ, nếu cơ sở dữ liệu của ứng dụng  hoạt động lỗi do các vấn đề phần cứng, người quản trị ứng dụng có thể chuyển sang 1 máy chủ cơ sở dữ liệu mới được khôi phục từ lần backup gần nhất. Cơ sở dữ liệu production hiện tại có thể được gỡ bỏ, và cơ sở dữ liệu mới sẽ được thêm vào - tất cả đều không cần bất kỳ thay đổi code nào.

#### V. Xây dựng, xuất bản và chạy
###### Tách rời rõ ràng giai đoạn xây dựng và chạy
1 codebase được chuyển thành 1 triển khai(không-phải-phát-triển) qua ba giai đoạn:

- Giai đoạn xây dựng là 1 quá trình chuyển đổi mà chuyển 1 code repo thành 1 gói thực thi, được biết như là 1 kiến trúc. Sử dụng 1 phiên bản code tại 1 commit cụ thể qua quá trình triển khai, bước xây dựng sẽ lấy các phụ thuộc vendors và biên dịch các file nhị phân và asset.
- Giai đoạn xuất bản lấy kiến trúc đượct ạo từ giai đoạn xây dựng và ghép chúng với cấu hình triển khai hiện tại. Kết quả xuất bảo sẽ bảo gồm cả kiến trúc và cấu hình và sẵn sáng để thực thi ngay lập tức trong môi trường thực thi.
- Giai đoạn chạy(cũng được biết là "thời gian chạy") chạy ứng dụng trong môi trường thực thi, bằng cách xuất 1 tập các tiến trình của ứng dụng với 1 xuất bản được chọn.

Ứng dụng theo 12 chuẩn sử dụng tách biệt rõ ràng giữa các giai đoạn xây dựng, xuất bản và chạy. Ví dụ, không thể tạo các thay đổi đến code trong thời gian chạy, vì không có cách nào để truyền những thay đổi này về giai đoạn xây dựng cả.

Các công cụ triển khai thường sẽ chấp nhận các công cụ quản lý xuất bản, đáng chủ ý nhất là có thể rool back về xuất bản trước đó. í dụ công cụ triển khai Capistranoapistrano lưu các xuất bản trong 1 thư mục con gọi là `releases`, nơi mà xuất bản hiện tại là 1 liên kết tượng trưng đến thư mục xuất bản hiện tại. Lệnh `rollback` của nó khiến nó rất dễ dàng để rollback lại xuất bản trước 1 cách nhanh chóng.

Mỗi xuất bản luôn nên có 1 ID xuất bản riêng, như là mốc thời gian xuất bản ( như là `2011-04-06-20:32:17`) hay 1 số tự tăng (như là `v100`). Các xuất bản kiểu như chỉ có thể thêm vào và 1 xuất bản không thể chỉnh sửa 1 khi nó được tạo ra. Bất kỳ thay đổi nào đều phải tạo 1 xuất bản mới. 

Các kiến trúc sẽ được khởi tạo bởi người phát triển ứng dụng khi mà code mới được triển khai. Mặt khác, thực thi thời gian chạy có thể tự động xảy ra trong  số trường hợp như là server khởi động lại, hoặc 1 tiến trình crash được khởi động lại bởi quản lý tiến trình. Vì thế, giai đoạn chạy nên được giữ càng ít phần chuyển tiếp càng tốt, vì các vấn đề ngăn ứng dụng chạy có thể khiến nó hỏng trong giữa đêm khi mà không có người lập trình nào làm việc cả. Quá trình xây dựng có thể phức tạp hơn, vì các lỗi luôn luôn tồn tại xung quanh 1 nhà phát triển đang thực hiện triển khai.

#### VI. Các tiến trình
###### Thực thi ứng dụng như là 1 hay nhiều  các tiến trình không trạng thái
Ứng dụng được thực thi trong môi trường thực thi như là 1 hay nhiều hơn các tiến trính/

Trong trường hợp đơn giản nhất, code là 1 kịch bản độc lập, môi trường thực thi là laptop cục bộ của người phát triển với ngôn ngữ thời gian chạy đã được cài đặt, và tiến trình được thực hiện thông qua dòng lệnh ( ví dụ, `python my_script.py`). Mặt khác, 1 triển khai production của 1 ứng dụng tinh vi có thể sử nhiều nhiều loại tiến trình, khởi tạo từ 0 hoặc nhiều hơn các tiến trình chạy.

Các tiến trình theo 12-chuẩn là không trạng thái và không chia sẻ bất cứ gìì cả. Bất kỳ dữ liệu nào cần dùng phải được lưu trữữ trong 1 dịch vụ nền có trạng thái, thường là 1 cơ sở dữ liệu.

Bộ nhớ hoặc filesystem của tiến trình có thể được sử dụng như là 1 tóm tắt,  1 bộ nhớ đệm đơn-chiều. Ví dụ, tải 1 file lớn, nghiên cứu nó, và lưu các kết quả của sự nghiên cứu trong cơ sởở dữ liệu. Ứng dụng theo 12-chuẩn không bao giờ  giả định bất cứ thứ gì được cache trong bộ nhớ hay trên đĩa sẽ khả dụng trong 1 yêu cầu hay công việc trong tương lai - với nhiều tiến trình mỗi loại đang chạy, cơ hội cho 1 yêu cầu trong tương lai được phục vụ bởi 1 tiến trình khác là cao. Thậm chí ngay cả khi đang chạy duy nhất 1 tiến trình, 1 khởi động lạiại ( bắt nguồn từ triển khai code, thay đổi cấu hình hay môi trường thực thi  chuyển tiến trình sang 1 ví trí vật lí khác) thường xóa sạch tất cả trạng thái cục bộ ( bộ nhớ và filesystem).

Các đóng gói asset như django-asetpackager sử dụng filesystem như là bộ nhớ đệm cho các asset được biên dịch. 1 ứng dụng theo 12-chuẩn thích thực hiện việc biên dịch này trong suốt giai đoạn xây dựng. Các đóng gói asset như Jammit và asset pipeline Rails có thể được cấu hình thành các asset package trong giai đoạn xây dựng.

1 vài hệ thống web phụ thuộc vào các “sticky sessions” - nó là , các dữ liệu phiên của người dùng được lưu đệm trong bộ nhớ của các tiến trình của ứng dụng và đợi những yêu cầu trong tương lai từ cùng 1 người sẽ được điều hướng tới cùng 1 tiến trình. Các tiến trình  dính là những vi phạm của 12-chuẩn và không nên bao giờ được sử dụng hay phụ thuộc vào nó. Các dữ liệu trang thái phiên là 1 thay thế tốt cho các kho dữ liệu yêu cầu thời gian hết hạn, như là Memcaches hay Redis.

#### VII. Ràng buộc cổng
###### Xuất các dịch vụ bằng ràng buộc cổng
Các ứng dụng web đôi khi được thực thi bên trong 1 nơi chứa webserver. Ví dụ, các ứng dụng php có thể chạy như là 1 module bên trong Apache HTTPD, hay các ứng dụng Java có thể chạy bên trong Tomcat.

Ứng theo theo 12-chuẩn hoàn toàn độc lập và không phụ thuộc vào các đơn ảnh thời gian chạy của 1 webserver vào môi trường thực thi để tạo 1 dịch vụ web-facing. Ứng dụng web xuất HTTP như 1 dịch vụ bằng cách ràng buộc với 1 cổng, và nghe các yêu cầu tới từ cổng đó.

Trong môi trường phát triển cục bộ, cácác nhà phát triển vào 1 URL dịch vụ như `http://localhost:5000/` để truy cập vào dịch vụ xuất bởi ứng dụng của họ.Trong triển khai, tầng điều hướng xử lý các yêu cầu điều hướng từ 1 tên miền public-facing đến các tiến trình web port-bound.

Điều này thường được triển khai bằng các sử dụng các khai báo phụ thuộc để thêm 1 thư viện webserver vào ứng dụng, như là Tornado cho Python, Thin cho Ruby, hay Jetty cho Java và các ngôn ngữ chuẩn JVM khác. Điều này xảy ra hoàn toàn trong không gian người dùng, đó là, bên trong code của ứng dụng. Các giao ước với môi trương thực thi được ràng buộc tới 1 cổng để phục vụ các yêu cầu.

HTTP không phải dịch vụ duy nhất có thể được xuất bằng ràng buộc cổng. Gần đây thì bất kỳ loại phần mềm máy chủ nào đều có thể chạy thông qua 1 tiến trình ràng buộc với 1 cổng và đợi các yêu cầu đến. Các ví dụ bao gồm ejabberd ( gọi XMPP), và Redis (gọi giao thức Redis).

Cũng lưu ý rằng các tiếp cận ràng-buộc-cổng có nghĩa là 1 ứng dụng có thể trở thành 1 dịch vụ nền cho 1 ứng dụng khác, bằng cách cung cấp URL đến ứng dụng nền như 1 xử lý tài nguyên trong cấu hình cho ứng dụng sử dụng.

#### VIII. Xử lý đồng thời
###### Tăng khả năng thông qua mô hình tiến trình
Bất kỳ chương trình máy tính nào, 1 khi chạy, được đại diện bởi 1 hoặc nhiều hơn các tiến trình. Các ứng dụng web có rất nhiều hình thức thực thi tiến tình. Ví dụ, các tiến trình PHP chạy như các tiến trình con của Apache, bắt đầu  theo yêu cầu như nhu cầuầu của lượng request. Các tiến trình Java lại cóó 1 cách tiếp cận khác, với  JVM cung cấp 1 uberprocess lớn lưu trữ khối tài nguyên hệ thống (CPU và bộ nhớ) lúc khởi động, với xử lý đồng thời được quản lý nội bộ thông qua các luồng. Trong cả 2 trường hợp, các tiến trình chạy chỉ hiển thị tối thiểu đến các người lập trình của ứng dụng.

Trong ứng dụng 12-chuẩn, các tiến trình là tập các lớp đầu tiên. Các tiến trình trong ứng dụng theo 12 chuẩn tuân theo mô hình tiến trình unix để chạy các daemon(chương trình chạy nền) dịch vụ. Sử dụng môô hình này, người phát triển có thể thiết kế ứng dụng của họ để xử lý các khối lượng công việc khác nhau bằng cách giao mỗi loại công việc cho 1 loại tiến trình. Ví dụ các yêu cầu HTTP có thể được xử lý bằng 1 tiến trình web, và các nhiệm vụ nền chạy dài được xử lý bởi các tiến trình worker.

Điều này không ngăn các tiến trình độc lập xử lý các kênh ghép nội bộ của chúng, thông qua các luồng bên trong VM thời gian chạy, hoặc mô hình bất đồng bộ/sự kiện được tìm thấy trong các công cụ như là EventMachine, Twisted, hay Node.js. Nhưng 1 VM độc lập chỉ có thể mở rộng rất lớn ( tăng theo chiều dọc), do vậy ứng đụng cũng phải có thể có đa tiến trình chạy trên nhiều máy vật lý.

Mô hình tiến trình thực sự hiệu quả khi đến lúc mở rộng(dãn theo chiều ngang). Bản chất không chia sẻ, có thể phân chia theo chiều dọc của các tiến tình ứng dụng theo 12 chuẩn có nghĩa là thêm nhiều xử lý đồng thời là 1 hoạt động đơn giản và chắc chắn. Mảng các loại tiến trình và số lượng tiến trình của mỗi loại được biết như là hệ thống tiến trình.

Các tiến trình của ứng dụng theo 12-chuẩn không bao giờ nên daemonize hay viết các file PID. Thay vào đó, dựa trên quản lý tiến trình của hệ điều hành( như là systemd, 1 quản lý tiến trình phân tán trên nền tảng đám mây, hay 1 công cụ như Foreman trong phát triển) để quản lý các luồng ra, phản hồi các tiến trình hỏng, và xử lý các khởi động lại và tắt do người dùng khởi tạo.

#### IX. Tính sẵn dùng
###### Tối đa hóa sự vững chắc với khởi động nhanh và tắt  1 cách thanh nhã
Các tiến trình của ứng dụng theo 12 chuẩn phải sẵn dùng, tức là nó có thể bắt đầu hay kết thúc ngay lập tức. Điều này tạo điều kiển cho việc mở rộng mềm dẻo và nhanh chóng, triển khai nhanh chóng code hay các thay đổi cấu hình, và sự chắc chắn của các triển khai production. 

Các tiến trình nên cố gắng để tối thiểu thời gian khởi động. Lý tưởng là 1 tiến trình  mất 1 vài giây từ lúc lệnh gọi được thực thi cho tới khi tiến trình bắt đầu và sẵn sàng để nhận các yêu cầu từ công việc. Thời gian khởi động ngắn cho phép các tiển trình triển khai và mở rộng nhanh hơn, và nó chắc chắn hơn, vì quản lý tiến trình có thể chuyển các tiến trình đến các máy vật lý mới khi được đảm bảo dễ dàng hơn.

Các tiến trình tắt 1 cách thanh nhã khi chúng nhận được 1 tín hiệu SIGTERM từ trình quản lý tiến trình. Với 1 tiến trình web, tắt 1 cách thanh nhã đạt được bằng cách ngừng lắng nghe trên cổng dịch vụ (vì thế từ chối bất cứ yêu cầu mới nào), cho phép bất cứ yêu cầu hiện tại hoàn thành, và sau đó thoạt. Ngụ ý trong mô hình này là các yêu cầu HTTP sẽ ngắn( không nhiều hơn vài giây), hoặc trong trường hợp lâu, máy khách nên ngay lập tức thử kết nối lại khi kết nối bị mất.

Với 1 tiến trình worker, tắt 1 cách thanh ngã đạt được bằng cách đưa công việc hiện tại về hàng chờ việc. Ví dụ, trên RabbitMQ worker có thể gửi 1 `NACK`; trên Beanstalkd, công việc được đưa về hàng chờ tự động mỗi khi worker ngắt kết nối. Các hệ thống lock-based như Delayed Job cần phải chắc triển khai khóa của chúng trên các bản ghi công việc. Ngụ ý trong mô hình ngày là tất cả các công việc là lặp lại, thứ thường có thể đạt được bằng cách gói các kết quả trong 1 giao dịch, hoặc khiến hoạt động không thay đổi giá trị.

Các tiến trình cũng nên mạnh mẽ trước những biến cố bất thình lình, trong trường hợp bị hỏng ở phần cứng phía dưới. Trong khi đây là điều thường ít xảy ra hơn  so với tắt 1 cách thanh nhã với `SIGTERM`, nó vẫn có thể xảy ra. 1 lời khuyên là sử dụng  nền hàng chờ chắc chắn, như là Beanstalkd, nó sẽ đưa các công việc về hàng chờ khi các máy khách ngắt kết nối hoặc quá hạn. Cung như vậy, 1 ứng dụng theo 12 chuẩn được thiết kế để xử lý những kết thúc không ngờ, không thanh nhã. Các thiết kế chỉ-hỏng đưa khái niệm này đến kết luận logic của nó.


#### X. Dev/prod parity (bình đẳng dev/prod)
###### Giữ cho giai đoạn development, staging, và production giống nhau nhất có thể

Trước đây, có 1 khoảng trống đáng kể giữa development ( 1 người phát triển tạo các chỉnh sửa trực tiếp tới các triển khai cục bộ của ứng dụng) vàà production(1 triển khai đang chạy của ứng dụng tiếp cận bởi các người dùng cuối). Những khoảng trống này biểu lộ trong 3 điểm:

- Khoảng trống thời gian: 1 người phát triển có thể làm việc trên code mất vài ngàyày, vài tuần, hay thâm chí vài tháng trước khi đẩy lên production.
- Khoảng cách về nhân sự: Các người phát triển viết code, các kĩ sư óp triển khai nó.
- Khoảng cách về công cụ: các người phát triển có thể sử dụng 1 stack như Ngĩn, SQLite, và OS X, trong khi triển khai production sử dụng Apache, MySQL, và Linuxx.

Ứng dụng theo 12 chuẩn được thiết kế để triển khai liên tục bằng cách giữ các khoảng cách giữa development và production nhỏ. Nhìn vào 3 cái khoảng trống được mô tả bên trên:

- Làm khoảng cách thời gian nhỏ: 1 người phát triển có thểể viết code và triển khai nó chỉ vài giờ hay thậm chí vài phút sau đó.
- Làm khoảng cách nhân sự nhỏ lại: những người phát triển viết code liên quan chặt chẽ với việc triển khai nó và theo dõi các hoạt động của nó trong production.
- Khiến khoảng cách các công cụ nhỏ lại: làm development và production giống nhau nhất có thể.

Tổng kết trong bảng dưới đây:

|  |  Ứng dụng cổ điển |  Ứng dụng theo 12 chuẩn |
| ------------- |:-------------:| -----:|
| Thời gian giữa các triển khai |  Hàng tuần |  Hàng giờ |  
| Tác giả code và người triển khai code |  Những người khác nhau |  Cùng người |  
| Các môi trường Dev và production |  Khác ngau |  Giống nhau hết cỡ có thể | 

[Các dịch vụ sao lưu][3], như là cơ sở dữ liệu của ứng dụng, hệ thống hàng đợi, hoặc bộ nhớ đệm, là thứ mà bình đằng dev/prod rất quan trọng.
such as the app's database, queueing system, or cache, is one area where dev/prod parity is important. Many languages offer libraries which simplify access to the backing service, including _adapters_ to different types of services. Some examples are in the table below.


| Loại |  Ngôn ngữ |  Thư viện |  Adapters |  
| ------------- |:-------------:| -----:|-----:|
| Database |  Ruby/Rails |  ActiveRecord |  MySQL, PostgreSQL, SQLite |  
| Queue |  Python/Django |  Celery |  RabbitMQ, Beanstalkd, Redis |  
| Cache |  Ruby/Rails |  ActiveSupport::Cache |  Memory, filesystem, Memcached | 

Developers sometimes find great appeal in using a lightweight backing service in their local environments, while a more serious and robust backing service will be used in production. For example, using SQLite locally and PostgreSQL in production; or local process memory for caching in development and Memcached in production.

**The twelve-factor developer resists the urge to use different backing services between development and production**, even when adapters theoretically abstract away any differences in backing services. Differences between backing services mean that tiny incompatibilities crop up, causing code that worked and passed tests in development or staging to fail in production. These types of errors create friction that disincentivizes continuous deployment. The cost of this friction and the subsequent dampening of continuous deployment is extremely high when considered in aggregate over the lifetime of an application.

Lightweight local services are less compelling than they once were. Modern backing services such as Memcached, PostgreSQL, and RabbitMQ are not difficult to install and run thanks to modern packaging systems, such as [Homebrew][4] and [apt-get][5]. Alternatively, declarative provisioning tools such as [Chef][6] and [Puppet][7] combined with light-weight virtual environments such as [Docker][8] and [Vagrant][9] allow developers to run local environments which closely approximate production environments. The cost of installing and using these systems is low compared to the benefit of dev/prod parity and continuous deployment.

Adapters to different backing services are still useful, because they make porting to new backing services relatively painless. But all deploys of the app (developer environments, staging, production) should be using the same type and version of each of the backing services.
