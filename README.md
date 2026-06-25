# Ecommerce Data Pipeline — Tổng quan Project

## 1. Mục tiêu


Theo dõi và phân tích dữ liệu sản phẩm từ các shop bán hàng trên **Shopee** và **TikTok Shop**, nhằm:
- Theo dõi biến động giá / lượt bán / rating theo thời gian
- Phân tích tổng quan ngành hàng: sản phẩm nào bán tốt, shop nào dẫn đầu
- Cung cấp dashboard trực quan (Power BI) để ra quyết định nhanh

## 2. Phạm vi dữ liệu

| Hạng mục | Chi tiết |
|---|---|
| Sàn TMĐT | Shopee, TikTok Shop |
| Input | Danh sách link **shop** (không phải link sản phẩm lẻ) |
| Trường dữ liệu lấy | Tên sản phẩm, giá, lượt bán (sold), số lượt review, số sao (rating), thông tin shop |
| Tần suất cào | Hàng ngày (daily) |
| Quy mô | Nhỏ — vài shop, tối đa ~200 sản phẩm/shop |

> _(Insert thêm: số lượng shop cụ thể đang theo dõi, ngành hàng tập trung, ngày bắt đầu thu thập dữ liệu...)_

## 3. Kiến trúc Workflow

```
config/shops.xlsx (danh sách link shop)
        │
        ▼
┌─────────────────────┐     ┌─────────────────────┐
│   Scrape Shopee      │     │  Scrape TikTok Shop  │
│   (Playwright)       │     │   (Playwright)       │
└──────────┬───────────┘     └──────────┬───────────┘
           │                            │
           ▼                            ▼
   raw_shopee.csv                raw_tiktok.csv
           │                            │
           └─────────────┬──────────────┘
                          ▼
              ┌───────────────────────┐
              │  Python xử lý (pandas) │
              │  Clean, gộp 2 sàn,     │
              │  chuẩn hoá schema      │
              └───────────┬───────────┘
                          ▼
              cleaned_products.xlsx / .csv
                          │
                          ▼
      ┌──────────────────────────────────────┐
      │         SQL Server (SSMS 21)          │
      │                                        │
      │   Bronze layer                         │
      │   → dữ liệu thô, append theo ngày      │
      │              ▼                         │
      │   Silver layer                         │
      │   → làm sạch, đúng kiểu dữ liệu, dedupe│
      │              ▼                         │
      │   Gold layer                           │
      │   → bảng tổng hợp: xu hướng giá,       │
      │     ranking sản phẩm                   │
      └──────────────────┬─────────────────────┘
                          ▼
                Power BI Dashboard
              (xu hướng giá, top sản phẩm)
```

> _(Có thể chèn ảnh sơ đồ trực quan vào đây sau, ví dụ: `![workflow](./docs/workflow.png`)_

## 4. Chi tiết từng giai đoạn

### 4.1. Thu thập dữ liệu (Scraping)
- Công cụ: Playwright (Python)
- Input: `config/shops.xlsx`
- Output: `data/raw/raw_shopee_YYYYMMDD.csv`, `data/raw/raw_tiktok_YYYYMMDD.csv`

> _(Insert thêm: các trường hợp edge-case đã gặp, cách xử lý chống bot, giới hạn đã biết...)_

### 4.2. Xử lý dữ liệu (Transform)
- Công cụ: Python (pandas)
- Việc cần làm: ép kiểu số (giá, lượt bán...), chuẩn hoá tên cột giữa 2 sàn, loại bỏ trùng lặp
- Output: `data/processed/cleaned_products_YYYYMMDD.xlsx/csv`

> _(Insert thêm: các rule làm sạch cụ thể đã áp dụng, ví dụ cách convert "1,2 Tr" → 1200000)_

### 4.3. Mô hình dữ liệu (Bronze / Silver / Gold)

| Layer | Vai trò | Bảng chính |
|---|---|---|
| Bronze | Lưu nguyên trạng dữ liệu thô theo từng ngày, không sửa/xoá | `bronze.shop_products_raw` |
| Silver | Dữ liệu đã làm sạch, đúng kiểu, sẵn sàng truy vấn | `silver.shop_products` |
| Gold | Bảng tổng hợp phục vụ trực tiếp cho BI | `gold.price_trend`, `gold.shop_ranking` |

> _(Insert thêm: schema chi tiết từng bảng, các business rule tính toán ở Gold layer)_

### 4.4. Trực quan hoá (Power BI)
- Nguồn dữ liệu: kết nối trực tiếp tới các bảng Gold trên SQL Server
- Các dashboard dự kiến:

> _(Insert thêm: danh sách cụ thể các trang/dashboard, ví dụ: "Trang 1 — Tổng quan giá theo ngày", "Trang 2 — So sánh shop"...)_

## 5. Tự động hoá

| Thành phần | Công cụ |
|---|---|
| Lập lịch chạy | Windows Task Scheduler |
| Script điều phối | `run_pipeline.bat` |
| Versioning code | Git + GitHub (chỉ push code, không push data) |

> _(Insert thêm: giờ chạy cụ thể, cách xử lý khi 1 bước lỗi giữa pipeline, cách nhận thông báo khi pipeline fail...)_

## 6. Vấn đề đã biết / Rủi ro

> _(Khung để ghi lại các vấn đề gặp phải theo thời gian, ví dụ:)_

- Shopee/TikTok Shop có cơ chế chống bot — có thể gặp captcha hoặc bị chặn tạm thời nếu cào quá nhanh/nhiều
- Cấu trúc HTML/JSON nội bộ của các sàn có thể thay đổi theo thời gian, cần kiểm tra lại selector/regex định kỳ
- _(thêm các vấn đề khác khi gặp phải)_

## 7. Lịch sử thay đổi

> _(Khung changelog, cập nhật dần theo thời gian)_

| Ngày | Thay đổi |
|---|---|
| _(YYYY-MM-DD)_ | Khởi tạo project, thiết kế kiến trúc tổng thể |