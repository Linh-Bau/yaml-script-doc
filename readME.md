# 📌 YAML SCRIPT
- Đọc tài liệu chi tiết trong `./documents/yaml_script_documents.md` 
  
--- 

# 📚 USER GUILD

## YÊU CẦU MÔI TRƯỜNG VÀ CÀI ĐẶT
- Sử dụng tool `PEDownloaderV4.exe` -> tool -> getter. Tải xuống Getter. Cài môi trường `windowsdesktop-runtime-8.0.20-win-x86.exe`, `VC_redist.x86.exe` (Đối với trạm camera vision, các trạm khác không cần.)
- VSCODE extentions: `YAML`(Red Hat)
## CÁCH TRIỂN KHAI THÔNG QUA VÍ DỤ THỰC TẾ

<b>Ví dụ</b>:
```text
Cần triền khai chương trình cho product Demo. Tên trạm là TRAINING. Model_id gồm MODEL_ID_1, MODEL_ID_2. Trạm thực hiện lưu trình như sau:
1. dùng qfil FLASH image vào sản phẩm.
2. sau khi FLASH xong sẽ đợi sản phẩm lên nguồn, thông qua cổng com.
3. sau khi lên nguồn. Chuyển sản phẩm vào chế độ fastboot và hạ lệnh `fastboot oem 1`
4. Với model_id_2 sẽ cần thêm 1 bước đọc wifi mac ở fastboot và upload lên mes.
```

### CÁC BƯỚC TRIỂN KHAI
1. Tạo thư mục tương ứng tên product và tên trạm trên `/SFTP.conf.pe.02/V6`
```text
📁 Demo
  📁 TRAINING
    📁 setup
    📄 script.yaml
```
2. Phân tích yêu cầu thành logic, sau đó chuyển qua script.

Quá trình cài đặt:
- Tự độngt tải xuống môi trường tương ứng.

Quá trình test.
- Flash image tương ứng.
- Đợi sản phẩm lên nguồn hoàn tất(thường là đợi 1 ký tự nào đấy xuất hiện trên com.)
  - Mở com
  - Đợi log xuất hiện ký tự cần tìm. 
- Chuyển sản phầm vào chế độ fastboot. hạ lệnh `fastboot oem 1` 
- Đọc mac và upload mac lên MES.

3. Chuyển logic test thành script tương ứng.
Bắt đầu bằng tạo khung chương trình.
```yaml
script:
  mes_defect_code: ~
  test_environments: ~ 
  test_configurations: ~
  test_sequences: ~
  test_targets: ~ 
```
Thêm 2 model_id vào:
```yaml
script:
  mes_defect_code: ~
  test_environments: ~ 
  test_configurations: ~
  test_sequences: ~
  test_targets: 
    - model_id: MODEL_ID_1
      environment: ~
      test_config: ~
      test_sequence: ~
    - model_id: MODEL_ID_2
      environment: ~
      test_config: ~
      test_sequence: ~
```

Thêm môi trường. Xem trong `yaml_script_documents.md` để tìm môi trường QFIL, adb. Với model_id khác nhau có OS khác nhau. Gợi ý để dễ maintain thì nên chia thư mục setup cho os theo tên model id để dễ kiểm soát.
```text
📁 Demo
  📁 TRAINING
    📁 setup
      📁 MODEL_ID_A
        📄 os_for_model_id_a.zip
      📁 MODEL_ID_B
        📄 os_for_model_id_b.zip
    📄 script.yaml
```

```yaml
script:
  mes_defect_code: ~
  test_environments: 
  - &env_model_id_a
    downloads:
    - name: "QFIL TOOL" 
      type: "folder"
      from: "/SFTP.conf.pe.02/V6ENV/qfil_env" 
      to: "./setup/qfil"
      extract_to: ~

    - name: "ADB TOOL" 
      type: "folder"
      from: "/SFTP.conf.pe.02/V6ENV/adb" 
      to: "./setup/adb_tool"
      extract_to: ~

    - name: OS
      type: compressed
      from: /setup/MODEL_ID_A/os_for_model_id_a.zip
      to: ./setup
      extract_to: ~

  - &env_model_id_b
    downloads:
    - name: "QFIL TOOL" 
      type: "folder"
      from: "/SFTP.conf.pe.02/V6ENV/qfil_env" 
      to: "./setup/qfil"
      extract_to: ~

    - name: "ADB TOOL" 
      type: "folder"
      from: "/SFTP.conf.pe.02/V6ENV/adb" 
      to: "./setup/adb_tool"
      extract_to: ~

    - name: OS
      type: compressed
      from: /setup/MODEL_ID_B/os_for_model_id_b.zip
      to: ./setup
      extract_to: ~

  test_configurations: ~
  test_sequences: ~
  test_targets: 
    - model_id: MODEL_ID_1
      environment: *env_model_id_a
      test_config: ~
      test_sequence: ~
    - model_id: MODEL_ID_2
      environment: *env_model_id_b
      test_config: ~
      test_sequence: ~
```
OK, vậy là chúng ta đã setup môi trường theo từng model_id.
🧐 Nhưng mà chúng ta thấy QFIL và ADB có thể dùng chung được cho cả 2 model_id. Có thể viết để nó ngắn hơn.

```yaml
script:
  mes_defect_code: ~
  test_environments: 
  - downloads:
    - &qfil_tool
      name: "QFIL TOOL" 
      type: "folder"
      from: "/SFTP.conf.pe.02/V6ENV/qfil_env" 
      to: "./setup/qfil"
      extract_to: ~

    - &adb_tool
      name: "ADB TOOL" 
      type: "folder"
      from: "/SFTP.conf.pe.02/V6ENV/adb" 
      to: "./setup/adb_tool"
      extract_to: ~

  - &env_model_id_a
    downloads:
      - *qfil_tool
      - *adb_tool
      - name: OS
        type: compressed
        from: /setup/MODEL_ID_A/os_for_model_id_a.zip
        to: ./setup
        extract_to: ~

  - &env_model_id_b
    downloads:
      - *qfil_tool
      - *adb_tool
      - name: OS
        type: compressed
        from: /setup/MODEL_ID_B/os_for_model_id_b.zip
        to: ./setup
        extract_to: ~

  test_configurations: ~
  test_sequences: ~
  test_targets: 
    - model_id: MODEL_ID_1
      environment: *env_model_id_a
      test_config: ~
      test_sequence: ~
    - model_id: MODEL_ID_2
      environment: *env_model_id_b
      test_config: ~
      test_sequence: ~
```

Triển khai test_configuration.
Thấy rằng ở QFIL cần flash image khác nhau cho từng model. Và nếu chương trình chạy trên nhiều cổng thì mỗi fixture sẽ có một cổng com khác nhau.
```yaml
script:
  test_configurations: 
  - &configuration_model_id_a
    flags: ~ 
    script_information: 
      station_name: OS DL
      description: model_id_a
      script_version: "1.0.0.0" # phiên bản hiển thị
    script_configuration: 
      image_path: $path ./setup/os_for_model_id_a/os_for_model_id_a # thư mục được giải nén
    fixture_configuration: # danh sách object, có thể null
      - qfil_port: 10
        android_com: COM11
      - qfil_port: 12
        android_com: COM13

  - &configuration_model_id_b
    flags: ~ 
    script_information: 
      station_name: OS DL
      description: model_id_a
      script_version: "1.0.0.0" # phiên bản hiển thị
    script_configuration: 
      image_path: $path ./setup/os_for_model_id_b/os_for_model_id_b # thư mục được giải nén
    fixture_configuration: # danh sách object, có thể null
      - qfil_port: 10
        android_com: COM11
      - qfil_port: 12
        android_com: COM13
```

Ở test_sequences. Ta thấy có thể chia nó thành các item như sau.
- CHECK_MBS_NO: kiểm tra đầu vào trạm (trạm test nào cũng cần)
- FLASH_IMAGE
  - hiển thị thông báo yêu cầu công nhân ấn giữ nút để vào chế độ EDL
  - Flash image 
- BOOT_UP
  - Hiển thị thông báo yêu cầu reboot sản phẩm
  - Mở com
  - Đợi ký tự yêu cầu xuất hiện
  - Hạ lệnh vào chế độ fastboot thông qua cổng com.
- OPEN_OEM
  - Hạ lệnh oem
- GET_WIFI_MAC
  - Đọc wifi mac
  - Upload wifi mac lên hệ thống MES.

Dựa vào cách chia nhỏ như này. Có thể khai test_sequence như bên dưới.

<b>⚠️ CÁC ITEM VÀ THAM SỐ CHỈ LÀ LỆNH THAM KHẢO VỀ MẶT LOGIC. THỰC TẾ NÊN DỰA VÀO `./documents/yaml_script_documents.md`</b>

```yaml
script:
  test_sequences: 
  -  test_items:
    - &check_mbs_no
      name: CHECK_MBS_NO 
      lower_limit: ~ 
      upper_limit: ~ 
      steps: 
        - do: mes.CHECK_MBS_NO 
          with: ~
          on_success: ~
           on_fail: ~

    - &flash_image
      name: FLASH_IMAGE
      steps: 
        # show dialog
        - do: dialog.show
          with: 
            title: THÔNG BÁO
            message: Xin chuyển sản phẩm vào chế độ EDL!
          on_success: ~
          on_fail: ~

        - do: qfil.AUTO
          with: 
            working_path: $path ./setup/qfil # xem ở phần download. 
            comport: $script_config qfil_port
            image_path: $script_config image_path # dựa theo config
            ttimeout: 300
          on_success: ~
          on_fail: ~

    - &boot_up
      name: BOOT_UP
      steps: 
        # show dialog
        - do: dialog.show
          with: 
            title: THÔNG BÁO
            message: Xin reboot sản phẩm
          on_success: ~
          on_fail: ~

        # mở com
        - do: com.open
          with: 
            name: android_com
            port: $script_config android_com
            baudrate: 115200
          on_success: ~
          on_fail: ~

        # đợi bootup
        - do: com.wait_string
          with: 
            name: android_com
            expect: "m440: $"
            timeout: 150
          on_success: ~
          on_fail: 
            - do: return.FAIL
              with:
                error_code: BOOT_UP_ERROR

        - do: com.send_string
          with: 
            name: android_com
            command: reboot fastboot
            sleep: 10
          on_success: ~
          on_fail: ~

    - name: OPEN_OEM
      steps: 
        # lệnh cmd
        - do: cmd.execute
          with: 
            name: _
            command: fastboot oem console 1
            working_path: $path ./setup/adb_tool
            timeout: 15
          on_success: 
            - do: return.PASS
          on_fail: 
            - do: return.FAIL
              with:
                error_code: OPEN_OEM_FAIL

    - &open_oem
      name: OPEN_OEM
      steps: 
        # lệnh cmd
        - do: cmd.execute
          with: 
            name: _
            command: fastboot oem console 1
            working_path: $path ./setup/adb_tool
            timeout: 15
          on_success: 
            - do: return.PASS
          on_fail: 
            - do: return.FAIL
              with:
                error_code: OPEN_OEM_FAIL

    - &get_mac
      name: GET_MAC
      steps: 
        # lệnh cmd
        - do: cmd.execute
          with: 
            name: get_mac
            command: fastboot getvar wifi_mac
            working_path: $path ./setup/adb_tool
            timeout: 15
          on_success: ~
          on_fail: 
            - do: return.FAIL
              with:
                error_code: GET_MAC_FAIL
        # đọc mac từ chuỗi response => function chưa có. Giả định đã có function tên là func.read_value
        - do: func.read_value
          with: 
            source: CMD[get_mac][log]
            expect: re_(\d{2}:){5}\d{2}
            index: 0
            replace: ':'
            replace_with: ''
            convert: to_upper
            var_name: PROD[wifi_mac]
          on_success: ~
          on_fail: 
            - do: return.FAIL
              with:
                error_code: READ_MAC_FAIL
        # upload mac lên mes
        - do: mes.UPLOAD_WIFI_MAC
          with: 
            mac: $context  PROD[wifi_mac]
          on_success: ~
          on_fail: ~
          
    - &seq_for_model_id_a
      *check_mbs_no
      *boot_up
      *open_oem

    - &seq_for_model_id_n
      *check_mbs_no
      *boot_up
      *get_mac
```

Sau đó thực hiện update  `test_targets`:
```yaml
  test_targets: 
    - model_id: MODEL_ID_1
      environment: *env_model_id_a
      test_config: *configuration_model_id_a
      test_sequence: *seq_for_model_id_a
    - model_id: MODEL_ID_2
      environment: *env_model_id_b
      test_config: *configuration_model_id_a
      test_sequence: *seq_for_model_id_a
```

Trên là toàn bộ quá trình hoàn tất cho 1 trạm. Các trạm khác có thể tham khảo logic triển khai tương tự.
Tài liệu dùng `./documents/yaml_script_documents.md` tham khảo.

---

# 📌 MỘT SỐ TIÊU CHUẨN CHUNG
- Tham khảo `./documents/yaml_script_documents.md - ĐỊNH DẠNG CHUNG YAML`
- ❌ Đừng dùng các ký tự đặc biệt. Tránh dấu cách. Hãy dùng gạch dưới thay thế.
Ví dụ:
```yaml
# không nên
name: CHECK H/W id # không nên. Dễ lỗi
steps: 
  # lệnh cmd
  ...
  on_fail:
    - do: return.FAIL
      with:
        error_code: check h/w id # không nên. chứa ký tự đặc biệt, lúc ghi log file sẽ lỗi

  # nên
  on_fail:
    - do: return.FAIL
      with:
        error_code: CHECK_HW_ID # dùng chữ hoa, gạch dưới để biểu thị lỗi
```
- ❌ Đặt tên biến rõ ràng. Khi nhìn sẽ dễ check.
Ví dụ:
```yaml
# không nên
  ...
  - do: com.send_string
    with:
      name: raptor
      command: logcat -c # nhưng lệnh này là adb mà, nên thực hiện ở com android chứ?
    

  # nên
  - do: com.send_string
    with:
      name: android_com
      command: logcat -c # rõ ràng.
```
- ❌ setup môi trường chỉ nên chứa đủ cái nó cần. Không nên bỏ tất cả vào rồi để tải xuống. Các file môi trường cần được tinh gọn và kiểm tra trước khi up lên. Ví dụ trạm K81 FLASH BOOTLOADER thì chỉ nên chứa file bootloader, k nên kèm những thứ ở trạm khác.
ví dụ:
```text
📁 Demo
  📁 TRAINING
    📁 setup
      📁 MODEL_ID_A
        📄 os_for_model_id_a.zip
        📄 other_files
      📁 MODEL_ID_B
        📄 os_for_model_id_b.zip
        📄 other_files
    📄 script.yaml
```