# 📌 YAML SCRIPT
- Đọc tài liệu chi tiết trong `./documents/yaml_script_documents.md` 
  
--- 

# 📚 USER GUILD

## ⚙️ YÊU CẦU MÔI TRƯỜNG VÀ CÀI ĐẶT
- Sử dụng tool `PEDownloaderV4.exe` -> tool -> getter. Tải xuống Getter. Cài môi trường `windowsdesktop-runtime-8.0.20-win-x86.exe`, `VC_redist.x86.exe` (Đối với trạm camera vision, các trạm khác không cần.)
- VSCODE extentions: `YAML`(Red Hat)

## 📏 RULES
**Để cho format thống nhất. Khi viết script tuân theo những nguyên tắc sau.**
1. tên `key` trong `yaml` luôn viết dạng `lower_case`.
```yaml
key: value # key là phần giá trị bên trái, đại diện tên khóa. bên phải là giá trị của khóa.
```
2. name của `test_item` luôn để là `UPPER_CASE` và không chứa ký tự đặc biệt.
```yaml
- name: CHECK_MBS_NO # OK
  lower_limit: ~ 
  upper_limit: ~ 

- name: CHECK MBS NO # KHÔNG TỐT, CHỨA DẤU CÁCH
  lower_limit: ~ 
  upper_limit: ~ 

- name: check mbs no # KHÔNG TỐT, CHỮ THƯỜNG
  lower_limit: ~ 
  upper_limit: ~ 

- name: check/mbs\no # KHÔNG TỐT, chứa ký tự đặc biệt
  lower_limit: ~ 
  upper_limit: ~ 
```
3. return `error_code` luôn để là `UPPER_CASE` và không chứa ký tự đặc biệt.
  
```yaml
- do: return.FAIL
  with:
    error_code: DOWNLOAD_K81_FIRMWARE_FAIL # OK


  error_code: DOWNLOAD K81 FIRMWARE_FAIL # Không tốt

  error_code: DOWNLOAD K81-FIRMWARE_FAIL # Không tốt
```

4. Đặt tên. môi trường sẽ đặt là `env_<tên_riêng>`. `cf_<tên_riêng>`. `seq_<tên_riêng>`. chú ý tên viết thường, không có dấu cách.
```yaml
script:
  mes_defect_code: ~
  test_environments: ~
  test_configurations:
    - &cf_dev # ...
  test_sequences:
    - &seq_dev #...
  test_targets:
    - model_id: Demo
      environment: ~
      test_config: *cf_dev
      test_sequence: *seq_dev
```