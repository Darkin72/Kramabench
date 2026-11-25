# Quy định chấm điểm KramaBench

Tài liệu này quy định cách chấm các câu trong `Kramabench/workload` khi đối chiếu với câu trả lời của một run QueryAI.

## Nguyên tắc chung

- Ghép câu theo ID với sáu workload chính `archeology.json`, `astronomy.json`, `biomedical.json`, `environment.json`, `legal.json` và `wildfire.json`. Dùng trường `answer` làm đáp án tham chiếu.
- Chấm `final_answer` trong `answer.json`. Có thể dùng `answer` trong `summary.json` sau khi xác nhận hai nguồn trùng nhau. Dùng log để làm rõ đơn vị hoặc cách diễn giải của câu trả lời.
- Mỗi câu có tối đa 1 điểm. Báo cáo tổng điểm trên tổng số câu và tỷ lệ phần trăm. Giữ độ chính xác của điểm từng câu khi cộng, rồi làm tròn lúc hiển thị.
- Chấm theo nghĩa của đáp án. Có thể bỏ qua khác biệt về hoa thường, dấu câu, dấu phân cách hàng nghìn, đơn vị tương đương, JSON/list/tuple và phần diễn giải nếu đáp án cốt lõi rõ ràng.
- Không suy diễn một con số từ phần giải thích nếu con số cuối cùng của run khác đáp án được hỏi.
- Khi cần quyết định cách hiểu có thể làm đổi điểm, nêu câu hỏi, hai cách chấm và mức ảnh hưởng để người yêu cầu xác nhận. Áp dụng các quyết định đã ghi theo ID trong tài liệu này.

## Cách chấm theo loại đáp án

### `numeric_exact`

- So sánh giá trị số theo độ chính xác đề yêu cầu. Nếu đề bỏ ngỏ, dùng độ chính xác thể hiện trong đáp án. Với đáp án nguyên và đề không yêu cầu làm tròn, so sánh đúng giá trị nguyên.
- `15` và `15.0` cùng giá trị. Với ký hiệu khoa học, giữ độ chính xác theo số mũ. Ví dụ, `7.95e-13` có chữ số cuối ở hàng `1e-15`.
- Cho 1 điểm khi khớp và 0 điểm khi lệch ở độ chính xác quy định.

### `numeric_approximate`

Dùng điểm RAE của KramaBench với `prediction` là số đã trích từ câu trả lời và `gold` là đáp án số.

```text
RAE = 1 / (1 + |prediction - gold| / |gold|)
```

- Điểm nằm trong khoảng 0 đến 1.
- Nếu đáp án bằng 0 thì chỉ giá trị 0 được coi là khớp.
- Quy đổi về cùng đơn vị trước khi tính. Ví dụ, `65%` tương ứng `0.65` khi đại lượng là tỷ lệ. Với đáp án `0.65` và câu trả lời `0.64`, điểm RAE là `0.984848…`.

### `string_exact`

- Chấm theo nghĩa với điểm 0 hoặc 1.
- Chấp nhận biến thể tên, viết tắt, dấu câu và cách trình bày tương đương khi không làm thay đổi thực thể được hỏi.

### `string_approximate`

- Mặc định chấm theo nghĩa với điểm 0 hoặc 1.
- Nếu câu trả lời có nhiều thành phần độc lập và đã được liệt kê trong mục đặc biệt, chấm từng thành phần rồi lấy trung bình theo trọng số đã nêu.

### `list_exact`

Dùng công thức F1 theo từng mục.

```text
F1 = 2 × số mục khớp / (số mục trong câu trả lời + số mục trong đáp án)
```

- Ghép mỗi mục bên trả lời và bên đáp án tối đa một lần. Tính cả mục thừa và mục trùng vào số mục đã trả lời.
- Với các mục có vai trò riêng, ghép theo vai trò tương ứng. Với danh sách các thực thể cùng vai trò, ghép theo thành phần.
- Chấm mục chuỗi theo nghĩa. Chấm mục số theo quy tắc `numeric_exact`, bao gồm độ chính xác của từng mục.

### `list_approximate`

- Vẫn dùng F1 theo từng mục.
- Mục số `p` khớp mục tham chiếu `g` khi `|p - g| / max(|g|, 1e-12) ≤ 0.01`, theo ngưỡng của KramaBench.
- Mục chuỗi được so sánh theo nghĩa.

## Các câu đặc biệt

### `astronomy-easy-4`

Chia đều trọng số cho ba thành phần, mỗi phần chiếm một phần ba điểm của câu.

1. Chu kỳ hoạt động Mặt Trời được chấm bằng RAE.
2. Danh sách 5 năm cực đại được chấm bằng F1.
3. Danh sách 5 năm cực tiểu được chấm bằng F1.

Các năm tham chiếu được xác định từ dữ liệu SILSO với `prominence >= 20` và `distance >= 5`. Khoảng cách 5 năm áp dụng giữa các cực trị cùng loại. Ví dụ, chu kỳ `9.2` năm so với `11` năm được `0.8594` RAE, và mỗi danh sách trả đúng 2 trong 5 năm, chỉ nêu 2 năm, được `0.5714` F1. Điểm câu là `0.6674`.

### `environment-hard-7`

Đề để mở đơn vị đầu ra. Chấp nhận chênh lệch ở cả hai thang, mỗi thang làm tròn đến hai chữ số thập phân.

- `0.11` theo thang tỷ lệ.
- `10.87` theo thang điểm phần trăm.

Hai cách biểu diễn xuất phát từ chênh lệch chưa làm tròn khoảng `0.1087474449` theo thang tỷ lệ. Đây là ngoại lệ làm tròn dành riêng cho câu này.

### `astronomy-hard-9`

Lấy độ trễ tốt nhất làm đáp án. Ví dụ, với độ trễ 42 giờ và 24 cặp dữ liệu hợp lệ, giá trị để chấm là 42 giờ.

### `astronomy-hard-12`

Chấm theo giá trị trong workload dù công thức trường geopotential có thể được diễn giải khác nhau từ mô tả đề.

### `biomedical-hard-4`

Chỉ chấm tên histological grade. Không phân biệt hoa thường giữa `FIGO Grade 2` và `FIGO grade 2`.

### `biomedical-hard-8`

Chỉ chấm danh sách protein sites. Sample IDs và tên thuốc đi kèm không tạo thêm mục phải khớp.

### `environment-hard-9`

Chấm F1 theo tên ba bãi biển và ghép theo thành phần.

### `environment-hard-20`

Với câu trả lời `[Bucks Creek, Bucks Creek, Schoolhouse Pond]`, một mục `Bucks Creek` khớp. Câu trả lời và đáp án đều có ba mục, nên F1 là `1/3`.

### `legal-hard-14`

Chỉ chấm tên 5 vùng đô thị. Số lượng báo cáo đi kèm là thông tin giải thích.

### `legal-hard-17`

Workload dùng tỷ lệ công bố `3.04%` và cho đáp án `32542`. Phép tính từ số đếm gốc cho `32587`, được 0 điểm theo chuẩn workload.

### `legal-hard-24`

So khớp vùng đô thị Los Angeles–Long Beach–Anaheim, CA qua các biến thể dấu phẩy trước mã bang và hậu tố `Metropolitan Statistical Area`.

### `legal-hard-2` và `wildfire-easy-13`

Chấp nhận tên rút gọn `Miami-Fort Lauderdale-West Palm Beach, FL` cho vùng đô thị đầy đủ và `Great Basin Coordination Center` cho `Great Basin Area` trong ngữ cảnh câu hỏi điều phối trực thăng.

### `wildfire-easy-1`

Khi workload chỉ có đáp án số, chấm tổng acres. Khoảng thời gian là ngữ cảnh kiểm tra và không tạo thêm điểm riêng.

### `wildfire-easy-3`

Chấm danh sách tên bang đầy đủ. Dấu phẩy, xuống dòng và thứ tự không có ý nghĩa.

### `wildfire-hard-4`

Chấm chi phí trên mỗi acre. Năm được nêu cùng không thay thế cho đại lượng chi phí.

### `wildfire-hard-16`

Chấm theo vị trí của cặp `[chi-square statistic, p-value]`. Làm tròn từng số theo độ chính xác của đáp án, lần lượt là `6.326` và `0.787`.

### `wildfire-hard-18`

Đáp án cần đủ hai kết luận về việc tăng mức độ dập cháy. Việc này không giúp đám cháy tắt sớm hơn và giúp giảm số công trình bị ảnh hưởng.

### `wildfire-hard-21`

Chấm F1 theo thành phần ba bang. Nếu câu trả lời và đáp án đều có ba mục và chỉ khớp California, điểm là `1/3` dù California nằm ở vị trí khác.

## Ghi nhận bất đồng của workload

- Nếu câu hỏi, subtasks và `answer` không hoàn toàn nhất quán, dùng `answer` làm chuẩn điểm chính.
- Ghi rõ bất đồng trong báo cáo chấm, không tự động cộng điểm vì run có thể hợp lý theo một cách diễn giải khác.
- Các câu trả lời dạng “không đủ dữ liệu” chỉ được chấp nhận khi workload cũng quy định đó là đáp án.

| ID | Điểm cần ghi chú khi có bất đồng |
|---|---|
| `archeology-hard-9` | Đề yêu cầu chọn bản ghi cuối khi có nhiều thành phố cổ khớp. Bước tính tham chiếu chọn bản ghi có rank số lớn nhất. |
| `biomedical-easy-2` | Tham chiếu chỉ dùng các ca `Case_excluded == No` khi tính tuổi trung bình. |
| `environment-easy-3` | Tham chiếu nhóm theo tên bãi, mã thành phố và mã cộng đồng trước khi ghép hai năm. |
| `wildfire-hard-12` | Đề để mở phương pháp kiểm tra dịch chuyển ngày bắt đầu. Tham chiếu xét xu hướng trung bình theo năm và cho đáp án `No`. |
| `wildfire-hard-14` | Đề để mở ngưỡng AQI cho “generally unsafe”. Tính điểm RAE theo đáp án `0.65`. |
