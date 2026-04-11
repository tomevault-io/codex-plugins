# 🚀 Clarity RMS - Full Copilot Instructions

Bạn là chuyên gia Flutter/Dart cho dự án Clarity RMS. Hãy tuân thủ nghiêm ngặt cấu trúc và các nguyên tắc sau:

## 1. Kiến trúc & Cấu trúc thư mục (Clean Architecture)

- **Feature Modules**: `lib/features/<name>/`. Mỗi module gồm:
  - `domain/`: `entities/`, `repositories/` (interface), `usecases/`. **Bắt buộc**: Pure Dart, không import Flutter/Dio/Hive.
  - `data/`: `models/` (DTO), `datasources/`, `repositories/` (impl). Thực hiện mapping model -> entity tại đây.
  - `presentation/`: `pages/`, `widgets/`, `bloc/` (hoặc `cubit/`).
- **Core & Shared**:
  - `lib/core/`: Hạ tầng (DI, Network, Router, Global State).
  - `lib/shared/`: Components, styles, extensions dùng chung.

## Quy tắc bổ sung (Important)

- **Không sinh hoặc ghi mã vào file hướng dẫn**: Tuyệt đối không ghi, tạo, hoặc generate nội dung vào `.github/copilot-instructions.md` từ code hoặc các script. File này chỉ dùng làm hướng dẫn cho Copilot/agent và chỉ được chỉnh sửa thủ công bởi người phát triển khi cần.
- **`lib/core/ui` reserved**: Thư mục `lib/core/ui` chỉ dành cho "UI tokens" kỹ thuật (spacing, dimensions, radius). Không đặt widget, business logic, hoặc file cấu hình vào `lib/core/ui` nếu chúng không phải token kỹ thuật. Các component/stylesheet/định nghĩa UI khác phải nằm ở `lib/shared/styles` hoặc trong feature modules tương ứng.

### **Quy tắc UI Tokens (Bắt buộc)**

- **Mọi giá trị giao diện không được hard-coded**: Tất cả padding, margin, kích thước icon, radius, chiều cao nút, khoảng cách, và các giá trị kích thước giao diện khác phải dùng hằng số/token trong `lib/core/ui` (`AppSpacing`, `AppDimensions`, `AppRadius`, v.v.).
- **Typography & Colors**: Dùng `lib/shared/styles` (`AppTypography`, `AppColors`, `AppTheme`) hoặc `Theme.of(context)`; không dùng màu/rgb/hard-coded font size trực tiếp trong widget.
- **Kiểm soát ngoại lệ**: Trường hợp prototype/POC cần giá trị tạm thời, phải chú thích rõ `// PROTOTYPE: reason` và tạo issue/PR để chuẩn hoá trước khi merge.

### Enforcement & CI

- **Static analysis**: Bắt `flutter analyze` trong CI; thêm hoặc tùy chỉnh lint rules để phát hiện anti-patterns.
- **Custom checks**: Thêm bước CI/nghiệm thu (shell script) để grep các literal số trong `lib/**/presentation/**` hoặc kiểm tra pattern dùng `AppSpacing`/`AppDimensions`.
- **Pre-commit hooks**: Khuyến nghị pre-commit hook để phát hiện hard-coded UI numbers trước khi push.
- **Code review checklist**: Kiểm tra `lib/core/ui` token usage, DI qua `sl`, và đảm bảo Domain không import Data.

Lý do: Giữ nhất quán UI, dễ điều chỉnh toàn cục, và tránh băm vặt giá trị khiến giao diện không đồng bộ.

## 2. Nguyên tắc SOLID & DRY (Nghiêm ngặt)

### **SOLID Principles**

- **S (Single Responsibility)**: Mỗi Class/Widget/Function chỉ làm một việc duy nhất. Tách biệt hoàn toàn Logic (Bloc/UseCase) khỏi UI.
- **O (Open/Closed)**: Đối tượng mở để mở rộng nhưng đóng để sửa đổi. Sử dụng `abstract class` và `interfaces` để thay đổi hành vi mà không can thiệp code lõi.
  - Ví dụ: Đăng ký nhiều `PaymentMethod` bằng cách thêm class mới triển khai `PaymentMethodInterface` thay vì sửa `PaymentService`.
- **L (Liskov Substitution)**: Lớp con phải có khả năng thay thế lớp cha mà không làm hỏng tính đúng đắn của chương trình. Tránh ném `UnimplementedError` khi override.
  - Ví dụ: `MockAuthRepository` phải trả về cùng kiểu dữ liệu và cùng ràng buộc tiền/ hậu điều kiện như `RealAuthRepository`.
- **I (Interface Segregation)**: Chia nhỏ các Interface lớn thành nhiều Interface cụ thể. Không ép module phụ thuộc vào các phương thức nó không dùng.
  - Ví dụ: Thay vì `IUserRepository` có 12 method, tạo `UserReadRepository` và `UserWriteRepository` nếu client chỉ cần đọc dữ liệu.
- **D (Dependency Inversion)**: Module cấp cao không phụ thuộc vào module cấp thấp. Cả hai phụ thuộc vào Abstractions. UseCase luôn gọi Repository Interface thay vì Implementation.

### **DRY (Don't Repeat Yourself)**

- **Centralized Components**: Kiểm tra `lib/shared/widgets/` trước khi tạo mới.
- **Logic Reuse**: Đưa logic tính toán/định dạng lặp lại vào `extensions` hoặc `utils`.
- **UI Reuse**: Widget nội bộ trang dùng private method `_buildXxx`, widget dùng chung cho feature đưa vào `presentation/widgets/`.

## 3. Dependency Injection (GetIt)

- **Centralized Locator**: Sử dụng `sl` từ `lib/core/di/locator.dart`.
- **Nguyên tắc**: Ưu tiên **Constructor Injection**.
- Chỉ dùng `sl<T>()` tại `injection_container.dart` hoặc khi khởi tạo Bloc/Cubit. Tránh gọi `sl()` trong logic nghiệp vụ.

## 4. Networking & State Management

- **Microservice support**: Sử dụng `ApiClient` để tách `baseUrl` và interceptor theo từng microservice; đăng ký mỗi service client riêng trong DI module. Kết hợp `lib/core/infrastructure/network/api_client.dart` + `lib/core/infrastructure/network/api_client_factory.dart`.
- **HTTP Client**: Sử dụng `Dio`. Xử lý qua `NetworkHandler` và `ApiResponseHandler` tại `lib/core/infrastructure/network/`.
- **Auth**: Token flow nằm tại `AuthInterceptor` (sử dụng `tokenDio` riêng).
- **State**: Sử dụng `flutter_bloc`. Global cubit nằm ở `lib/core/global_state/`.
- **Codegen**: Sử dụng `Freezed`, `json_serializable`, `envied`. Nhắc chạy: `flutter pub run build_runner build --delete-conflicting-outputs`.

## 5. Quy ước Code & UI

- **Đặt tên**: File `snake_case`, Class `PascalCase`. UseCase dùng phương thức `call()`.
- **UI**:
  - Widget tái sử dụng -> `presentation/widgets/`.
  - Widget nội bộ trang -> private method `_buildXxx`.
- **Navigation**: `GoRouter`. Cấu hình tại `lib/core/router/app_router.dart`.
- **Env**: Không commit secrets. Dùng `envied` với file generated `lib/config/env.g.dart`.

## 6. Quy trình & Phản hồi

- **Thứ tự viết code**: Domain -> Data -> Presentation.
- **Trước khi hoàn tất**: Nhắc người dùng chạy `flutter analyze` và đăng ký DI/Router.
- **Ngôn ngữ**: Giải thích và phản hồi bằng **Tiếng Việt**.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bathanhdev)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/bathanhdev)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
