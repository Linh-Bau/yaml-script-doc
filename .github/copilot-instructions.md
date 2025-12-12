
# Hướng dẫn Copilot cho repo `yaml-script-doc`

Tài liệu ngắn gọn, thực dụng giúp tác nhân AI nhanh chóng làm việc với repo này.

## Tổng quan repo
- Repo chủ yếu chứa tài liệu. Nội dung chính nằm ở `documents/yaml_script_documents.md`, mô tả định dạng `script.yaml` dùng bởi `ScriptRunner2.exe`.
- Đây là một đặc tả YAML cho tự động hoá test (khóa chính: `script` với `test_environments`, `test_configurations`, `test_sequences`, `test_targets`, `mes_defect_code`). Xem repo như spec chứ không phải code chạy được.

## Các quy ước quan trọng (dẫn từ `documents/yaml_script_documents.md`)
- Thụt lề YAML: chỉ dùng dấu cách, không dùng tab.
- Tên khóa: dùng `lower_case_with_underscores`.
- Dùng anchors & references: `&<name>` và `*<name>` để tái sử dụng environments/configs/sequences.
- Cú pháp hành động: `do` luôn ở dạng `<executer>.<method>` (ví dụ: `mes.CHECK_MBS_NO`, `mt.AUTO`).
- Biến chuỗi (string resolve): hỗ trợ `$context`, `$script_config`, `$path`, `$if`, `$str`, `$func`.

## Khi chỉnh sửa tài liệu (hướng dẫn cụ thể)
- Giữ nguyên các ví dụ YAML — nhiều ví dụ là mẫu thực thi, không sửa ý nghĩa khi refactor nội dung.
- Nếu thêm executer hoặc method mới: chèn một ví dụ ngắn vào phần `test_sequences` dùng `do: <executer>.<method>` và `with:` mô tả tham số.
- Khi sửa mô tả schema hoặc bảng, cập nhật mọi tham chiếu (anchors, `test_targets`, `test_sequences`) nếu cần.

## Quy trình phát triển liên quan repo này
- Repo không có script build/test; nhiệm vụ thường là sửa/chạy tài liệu.
- Khuyến nghị workflow cục bộ:
  - Chỉnh `documents/yaml_script_documents.md` để thay đổi spec.
  - Kiểm tra các ví dụ YAML bằng YAML linter (ví dụ `yamllint`) trước khi commit.
  - Thay đổi backward-compatible: thêm trường mới là tuỳ chọn (`~`) hoặc thêm dưới `script_configuration`.

## Điểm tích hợp & phụ thuộc bên ngoài
- `script.yaml` được thiết kế để sử dụng với `ScriptRunner2.exe`. Repo không chứa runner hay CI; coi việc chạy kịch bản là tích hợp ngoại vi.

## Tệp chính để tham khảo
- Xem `documents/yaml_script_documents.md` cho quy tắc và ví dụ chuẩn.

## Ví dụ sao chép (giữ nguyên khi thêm mẫu)
- Mẫu skeleton tối thiểu:
```
script:
  test_environments: ~
  test_configurations: ~
  test_sequences: ~
  test_targets: ~
```
- Ví dụ `do` (sao chép nguyên văn khi cần):
```
- do: mes.CHECK_MBS_NO
  with: ~
  on_success: ~
  on_fail: ~
```

## Ghi chú giọng văn
- Sử dụng tiếng việt để trả lời câu hỏi.
