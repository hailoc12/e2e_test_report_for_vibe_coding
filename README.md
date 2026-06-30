# E2E Test Report for Vibe Coding

> Một bộ skill giúp bạn **kiểm thử trọn vẹn** bất kỳ sản phẩm nào được AI tạo ra — và tự động viết ra một bản báo cáo QA đẹp, trung thực, có bằng chứng rõ ràng.

Khi bạn dựng xong một sản phẩm bằng vibe coding, câu hỏi đầu tiên luôn là: *"Nó có thực sự chạy đúng không?"* Skill này giúp bạn trả lời bằng **kết quả kiểm thử cụ thể** thay vì cảm giác *"có vẻ ổn"*. Kết quả đầu ra là một báo cáo (DOCX / PDF) có ma trận kiểm thử 5 tầng, bảng điểm theo 7 tiêu chí, danh sách tính năng, và ảnh chụp minh chứng cho từng kết luận.

Nếu sản phẩm có trợ lý AI hay chatbot RAG, skill còn chấm riêng **độ chính xác của phần AI** — đối chiếu từng câu trả lời với nguồn xác thực để bắt lỗi bịa (hallucination).

> *"Không tin lời nói — chỉ tin kết quả kiểm thử."*

---

## Báo cáo mẫu trông như thế nào?

Dưới đây là báo cáo do chính skill tạo ra (dùng cho một sản phẩm hư cấu tên `DemoKB`, chỉ nhằm minh họa — không phải ảnh chụp giả):

<p align="center">
  <img src="assets/sample-cover.png" alt="Trang bìa với huy hiệu điểm" width="42%">
  &nbsp;
  <img src="assets/sample-rubric.png" alt="Bảng điểm rubric tô màu" width="42%">
</p>

<p align="center">
  <em>Bên trái: trang bìa có ô điểm số lớn, tô màu theo mức đánh giá. Bên phải: bảng điểm 7 tiêu chí, ô mức độ được tô màu, dòng tổng được làm nổi bật.</em>
</p>

<p align="center">
  <img src="assets/sample-detail.png" alt="Thẻ chi tiết từng tiêu chí" width="46%">
</p>

<p align="center">
  <em>Mỗi tiêu chí đi kèm một thẻ chi tiết: tình huống kiểm thử / kết quả mong đợi / kết quả thực tế / kết luận, kèm ảnh minh chứng. Nhờ vậy người đọc thấy được từng kiểm thử thay vì chỉ một con điểm chung.</em>
</p>

Muốn xem bản đầy đủ, mở file mẫu: [`QA-Report-Sample.docx`](e2e-test-report/samples/QA-Report-Sample.docx) · [`QA-Report-Sample.pdf`](e2e-test-report/samples/QA-Report-Sample.pdf)

---

## Vì sao cần một skill riêng?

Sản phẩm do AI tạo ra thường đặc biệt ở ba điểm sau, và skill này được thiết kế để xử lý từng điểm:

| Tình huống hay gặp | Cách skill xử lý |
|--------------------|------------------|
| Báo cáo tự khen là "tốt" nhưng không có kiểm thử gì | Mỗi kết luận đều phải có minh chứng: một tình huống kiểm thử cụ thể + ảnh chụp + một câu lý do dựa trên kết quả thật |
| Chỉ thử trang chủ và phần đăng nhập rồi cho rằng "sản phẩm chạy được" | Bắt buộc duyệt toàn bộ tính năng: rà theo từng vai trò người dùng, lập danh sách tính năng, rồi thử từng cái |
| Tưởng trợ lý AI là "thông minh" trong khi thực ra chỉ trả lời sáo rỗng giống nhau mọi câu | Thử bằng ít nhất hai câu hỏi khác nhau, đối chiếu với nguồn xác thực để phân biệt AI thật sự tạo câu trả lời với dạng kịch bản cố định |

### Một ý cần nhớ: "không kiểm thử được" khác với "bị hỏng"

Đây là ranh giới quan trọng nhất. Nếu công cụ kiểm thử không vào được trang, không có tài khoản thử, hoặc trang chưa kịp tải — kết luận đúng là **chưa xác định được (UNVERIFIED)**, chứ không phải lỗi. Skill kèm sẵn bộ kỹ thuật để chứng minh một lỗi thực sự là của sản phẩm, không phải do hạn chế của công cụ kiểm thử.

---

## Cài đặt

### Yêu cầu trước

- Một client AI coding hỗ trợ định dạng skill SKILL.md (xem phần dưới)
- Python 3.10 trở lên, kèm các thư viện:
  ```bash
  pip3 install --break-system-packages python-docx playwright
  python3 -m playwright install chromium
  ```
- LibreOffice (`soffice`) — chỉ cần khi bạn muốn xuất báo cáo ra PDF

### 1. Claude Code *(khuyên dùng)*

```bash
# Lấy mã nguồn về
git clone https://github.com/hailoc12/e2e_test_report_for_vibe_coding.git
cd e2e_test_report_for_vibe_coding

# Dùng cho mọi project (cài cho toàn bộ tài khoản)
cp -R e2e-test-report ~/.claude/skills/

# Hoặc chỉ dùng cho project hiện tại
cp -R e2e-test-report .claude/skills/
```

Khởi động lại Claude Code, sau đó gọi:
```
/e2e-test-report https://your-app.example.com
```

> Nếu không muốn dùng git: trên trang repo, bấm **Code → Download ZIP**, giải nén rồi chép thư mục `e2e-test-report/` vào thư mục skill của client.

### 2. Codex CLI

Codex đọc skill thông qua file `~/.codex/AGENTS.md` (hoặc `AGENTS.md` trong workspace) và cấu hình `~/.codex/config.toml`. Cách đơn giản nhất là chép nội dung [`e2e-test-report/SKILL.md`](e2e-test-report/SKILL.md) vào `~/.codex/AGENTS.md`, sau đó yêu cầu:
```
Dùng skill e2e-test-report để kiểm thử <đối tượng cần test>
```

### 3. Antigravity

Chép thư mục `e2e-test-report/` vào vị trí Antigravity quét skill (thường là `~/.antigravity/skills/` hoặc `.skills/` trong workspace), khởi động lại IDE rồi gọi. Vì Antigravity là ứng dụng Electron, skill dùng Quartz API để chụp đúng cửa sổ của ứng dụng (không chụp toàn màn hình, tránh lẫn thông tin nhạy cảm).

> Vị trí thư mục skill khác nhau tùy client và hệ điều hành. Nguyên tắc chung: đặt thư mục `e2e-test-report/` (thư mục chứa `SKILL.md`) vào nơi client của bạn quét. Chi tiết thêm trong [`docs/INSTALL.md`](e2e-test-report/docs/INSTALL.md).

---

## Bắt đầu nhanh

```bash
# 1. Khai báo sản phẩm cần kiểm thử
$EDITOR e2e-test-report/harness/qa_teams.py   # thay demo.example.com bằng URL và tài khoản thật

# 2. Chạy bộ công cụ (Playwright headless) — thu thập bằng chứng và ảnh chụp
cd e2e-test-report/harness
python3 qa_harness.py demo          # kiểm tra đăng nhập và nhận diện trợ lý AI
python3 feature_explore.py demo     # rà soát tính năng theo từng vai trò
python3 ai_probe.py demo            # (nếu có chatbot) thử độ chính xác và chống bịa

# 3. Chấm điểm và tạo báo cáo DOCX
cd ../report-template
$EDITOR sample_data.py              # điền mức điểm và ghi chú từ kết quả kiểm thử
python3 build_report_docx.py        # -> test_report_output/QA-Report-<Tên>.docx

# 4. (tùy chọn) xuất ra PDF
soffice --headless --convert-to pdf --outdir test_report_output test_report_output/QA-Report-*.docx
```

Cách nhàn nhất: mở client AI, gọi `/e2e-test-report <URL>` rồi để skill dẫn bạn từng bước.

---

## Cấu trúc thư mục

```
e2e_test_report_for_vibe_coding/
├── README.md                         ← bạn đang đọc
├── LICENSE                           ← giấy phép MIT
├── docs/
│   └── e2e-test-report-guide.pdf     ← sách hướng dẫn chi tiết (xem cuối trang)
├── assets/                           ← ảnh minh họa cho README
└── e2e-test-report/                  ← skill (chép vào client của bạn)
    ├── SKILL.md                      ← toàn bộ phương pháp kiểm thử
    ├── docs/
    │   ├── INSTALL.md
    │   └── GETTING-STARTED.md
    ├── harness/                      ← bộ tự động hóa Playwright để thử web thật
    │   ├── qa_teams.py               ← KHAI BÁO: điền sản phẩm của bạn vào đây
    │   ├── qa_harness.py             # kiểm tra đăng nhập, nhận diện AI, chụp ảnh
    │   ├── feature_explore.py        # rà soát tính năng theo vai trò
    │   ├── ai_probe.py               # thử độ chính xác và chống bịa của AI
    │   └── README.md
    ├── report-template/              ← trình tạo báo cáo DOCX
    │   ├── build_report_docx.py      # tạo báo cáo (python-docx)
    │   ├── sample_data.py            # dữ liệu mẫu (DemoKB) — thay bằng dữ liệu thật
    │   ├── make_sample_shots.py      # tạo ảnh minh họa trung tính cho mẫu
    │   └── samples/                  # ảnh minh họa cho mẫu
    └── samples/                      ← báo cáo mẫu
        ├── QA-Report-Sample.docx
        ├── QA-Report-Sample.pdf
        └── SUMMARY.md                # cách gộp nhiều báo cáo thành một bảng xếp hạng
```

---

## Phương pháp (tóm tắt)

### Ma trận kiểm thử 5 tầng

`Unit` (từng thành phần) → `Integration` (liên kết) → `Functional` (theo yêu cầu) → `UAT` (theo kịch bản người dùng) → `Documentation Compliance` (theo HDSD). Nếu một tầng thấp có quá nhiều lỗi (>50%), skill dừng lại và báo cho bạn, không chạy các tầng cao hơn cho đỡ tốn công.

### Báo cáo hai trục cho sản phẩm phần mềm

| Trục | Nội dung đo | Kết quả |
|------|-------------|---------|
| **A — QA hệ thống** | Kiểm thử chức năng, liên kết, theo kịch bản người dùng, theo HDSD qua mọi vai trò | Điểm /100 kèm danh sách lỗi |
| **B — Độ chính xác AI** *(khi có chatbot)* | Hội thoại nhiều lượt, đối chiếu nguồn xác thực | Điểm /10 kèm các phát hiện |

### Bảng điểm 7 tiêu chí (/100, có trọng số)

| # | Tiêu chí | Trọng số |
|---|----------|----------|
| C1 | Mức độ phát hiện và bao phủ tính năng | 15% |
| C2 | Đăng nhập và phân quyền | 15% |
| C3 | Chức năng cốt lõi | 25% |
| C4 | Chất lượng AI *(nếu có chatbot)* | 15% |
| C5 | Độ tin cậy và sẵn sàng thực tế | 15% |
| C6 | Trải nghiệm và độ hoàn thiện | 10% |
| C7 | Đã triển khai và có thể kiểm thử | 5% |

Mỗi tiêu chí được chấm từ 1 đến 5 (mức 0 là chưa xác định được, sẽ bị loại ra khỏi tổng điểm để không kéo điểm xuống oan). Tổng điểm được **tính lại bằng mã nguồn**, không dựa vào con số do mô hình tự nhẩm. Chi tiết đầy đủ nằm trong [`e2e-test-report/SKILL.md`](e2e-test-report/SKILL.md).

---

## Chống kết luận sai — những kỹ thuật cốt lõi

Skill bao hàm một bộ kỹ thuật đúc kết từ các lỗi thật gặp phải khi kiểm thử:

- **Mỗi vai trò dùng một phiên trình duyệt riêng** — tránh tình trạng phiên đăng nhập của vai trò trước làm vai trò sau bị "lỗi giả".
- **Xác minh đăng nhập bằng ba cách** — kiểm tra token, so sự thay đổi của menu theo vai trò, và chụp ảnh; không kết luận dựa trên việc URL có đổi hay cookie có trống.
- **Tìm tài khoản ngay trên trang đăng nhập** — khi cấu hình không có tài khoản, skill rà bốn nguồn (gợi ý trên giao diện, nút Demo/Showcase, form điền sẵn, đường dẫn `/demo`) trước khi ghi "thiếu tài khoản".
- **Nhận diện đúng ô chat AI** — tìm đúng vị trí trò chuyện, bỏ qua ô tìm kiếm; chụp đúng dòng trả lời của bot chứ không chụp nhầm phần cuối trang.
- **Hỏi ít nhất hai câu khác nhau** — để phân biệt AI tạo câu trả lời thật với dạng kịch bản trả lời sáo rỗng.
- **Kiểm tra backend độc lập** — dùng `curl` thẳng vào các endpoint sức khỏe, không suy đoán "backend chết" chỉ vì giao diện tải được.

Toàn bộ chi tiết nằm trong phần **HARNESS PLAYBOOK** của `SKILL.md`.

---

## Sách hướng dẫn chi tiết

Nếu bạn muốn đọc phương pháp dưới dạng một cuốn sách nhỏ, mở [`docs/e2e-test-report-guide.pdf`](docs/e2e-test-report-guide.pdf) — sách đi kèm, viết bằng tiếng Việt.

---

## Giấy phép

[MIT](LICENSE) — tự do sử dụng, phân phối lại và chỉnh sửa. Sản phẩm mẫu `DemoKB` chỉ là hư cấu, không liên quan đến sản phẩm thật nào.

## Đóng góp

Skill này được cập nhật liên tục sau mỗi lần kiểm thử. Mọi đóng góp qua pull request hoặc issue đều được hoan nghênh.
