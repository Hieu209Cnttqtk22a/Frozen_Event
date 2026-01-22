# FrozenEvent Plugin

Plugin Minecraft cho máy chủ Paper/Folia cung cấp tính năng TeamWar - hệ thống thi đấu loại trừ đội với theo dõi và thông báo thời gian thực.

## Tính năng

- **Hỗ trợ nhiều phạm vi**: Chạy chiến tranh trong thế giới cụ thể hoặc toàn bộ máy chủ
- **Theo dõi loại trừ thời gian thực**: Tự động phát hiện đội bị loại dựa trên trạng thái online
- **Hệ thống đếm ngược**: Đếm ngược có thể tùy chỉnh trước khi chiến tranh bắt đầu
- **Bảng xếp hạng**: Theo dõi và hiển thị top đội trong chiến tranh
- **Tích hợp PlaceholderAPI**: Hiển thị thống kê chiến tranh trong scoreboard, chat, v.v.
- **Tương thích Folia**: Hỗ trợ đầy đủ cả Paper và Folia
- **Tích hợp BetterTeams**: Tích hợp liền mạch với plugin BetterTeams

## Yêu cầu

- **Máy chủ**: Paper 1.20.x - 1.21.11 hoặc Folia 1.20.1 - 1.21.11
- **Java**: 17 trở lên
- **Plugin phụ thuộc**:
  - BetterTeams (bắt buộc)
  - PlaceholderAPI (tùy chọn, cho placeholders)

## Cài đặt

1. Tải file `FrozenEvent-X.X.X.jar` mới nhất từ releases
2. Đặt file jar vào thư mục `plugins` của máy chủ
3. Cài đặt plugin BetterTeams nếu chưa có
4. (Tùy chọn) Cài đặt PlaceholderAPI để hỗ trợ placeholders
5. Khởi động lại máy chủ
6. Cấu hình plugin trong `plugins/FrozenEvent/config.yml`

## Kiểm tra sau khi cài đặt

Sau khi cài đặt, kiểm tra plugin hoạt động đúng:

1. **Kiểm tra plugin đã tải**: Chạy `/plugins` và xác nhận FrozenEvent màu xanh
2. **Kiểm tra lệnh chính**: Chạy `/frozenevent` để xem thông báo trợ giúp
3. **Chạy lệnh debug**: Chạy `/frozenevent debug` để kiểm tra:
   - Phát hiện runtime (Folia hay Paper)
   - Trạng thái cấu hình TeamWar
   - Plugin BetterTeams và hook có sẵn không
   - PlaceholderAPI có sẵn không (tùy chọn)
   - Chiến tranh đang hoạt động (nếu có)
4. **Kiểm tra TeamWar**: Chạy `/frozenevent teamwar` để xác nhận TeamWar khả dụng
5. **Kiểm tra console**: Xem console logs để tìm lỗi hoặc cảnh báo

**Ví dụ kết quả lệnh debug:**
```
[FrozenEvent] === FrozenEvent Debug Status ===

Runtime: Paper/Spigot

TeamWar Config:
  - Enabled: Yes

Plugin Dependencies:
  - BetterTeams: Present & Enabled
  - BetterTeamsHook: Available
  - PlaceholderAPI: Present & Enabled

TeamWar Module:
  - Status: Initialized
  - Active Wars: 1
    • all (RUNNING)

[FrozenEvent] === End Debug Status ===
```

Nếu thấy vấn đề trong kết quả debug:
- **BetterTeams: Not Found** → Cài đặt plugin BetterTeams
- **BetterTeamsHook: Not Available** → Kiểm tra tương thích phiên bản BetterTeams
- **TeamWar Module: Failed to Initialize** → Kiểm tra console logs để tìm lỗi
- **TeamWar Module: Not Loaded** → Kiểm tra `teamwar.enabled: true` trong config.yml

## Lệnh

### Lệnh chính

| Lệnh | Mô tả | Quyền |
|------|-------|-------|
| `/frozenevent` hoặc `/fe` | Hiển thị trợ giúp plugin | - |
| `/frozenevent reload` | Tải lại cấu hình | `frozenevent.admin` |
| `/frozenevent debug` | Hiển thị trạng thái và chẩn đoán plugin | `frozenevent.admin` |
| `/frozenevent teamwar start <giây> <all|world>` | Bắt đầu TeamWar với đếm ngược | `frozenevent.teamwar.admin` |
| `/frozenevent teamwar end <all|world>` | Kết thúc TeamWar đang hoạt động | `frozenevent.teamwar.admin` |
| `/frozenevent teamwar alive` | Hiển thị đội còn sống trong chiến tranh hiện tại | `frozenevent.teamwar.view` |
| `/frozenevent teamwar top <1-10>` | Hiển thị top N đội | `frozenevent.teamwar.view` |
| `/frozenevent teamwar status [scope]` | Hiển thị trạng thái chiến tranh (debug) | `frozenevent.teamwar.view` |

### Ví dụ lệnh

```bash
# Kiểm tra trạng thái plugin và phụ thuộc
/frozenevent debug

# Bắt đầu chiến tranh ở tất cả thế giới với đếm ngược 30 giây
/frozenevent teamwar start 30 all

# Bắt đầu chiến tranh ở thế giới cụ thể ngay lập tức
/frozenevent teamwar start 0 world_nether

# Kết thúc chiến tranh ở tất cả thế giới
/frozenevent teamwar end all

# Hiển thị đội còn sống
/frozenevent teamwar alive

# Hiển thị top 5 đội
/frozenevent teamwar top 5

# Hiển thị trạng thái tất cả chiến tranh đang hoạt động
/frozenevent teamwar status

# Hiển thị trạng thái phạm vi cụ thể
/frozenevent teamwar status all

# Tải lại cấu hình
/frozenevent reload
```

## Quyền (Permissions)

| Quyền | Mô tả | Mặc định |
|-------|-------|----------|
| `frozenevent.admin` | Quyền admin plugin | op |
| `frozenevent.teamwar.admin` | Bắt đầu/kết thúc chiến tranh | op |
| `frozenevent.teamwar.view` | Xem thông tin chiến tranh | true |

## Cấu hình

### Các khóa cấu hình chính (`config.yml`)

```yaml
teamwar:
  enabled: true
  
  # Hành vi phạm vi lệnh
  command-scope:
    mode: auto  # "auto" hoặc "all-only"
    
  # Cài đặt placeholder
  placeholders:
    none-value: "-"  # Giá trị hiển thị khi không có dữ liệu
    
  # Cài đặt spectator
  spectator:
    ignore-gamemodes:
      - SPECTATOR
      - CREATIVE
      
  # Sắp xếp lệnh top
  top-sort: SNAPSHOT  # "SNAPSHOT" hoặc "ALPHABET"
  
  # Hành vi đội đơn khi chỉ có 1 đội đủ điều kiện khi bắt đầu chiến tranh
  single-team-behavior: AUTO_WIN  # "BLOCK" hoặc "AUTO_WIN"
  
  # Quy tắc A: Ngưỡng đội tối thiểu
  ruleA:
    enabled: true
    threshold: 2  # Thông báo khi còn số đội này
    
  # Cài đặt giai đoạn bắt đầu
  start:
    countdown:
      show-last-seconds: 10  # Chỉ hiển thị đếm ngược trong N giây cuối
      tick-message:
        enabled: true
      started-message:
        enabled: true
        
  # Cài đặt giai đoạn kết thúc
  end:
    broadcast:
      enabled: true  # Thông báo cho tất cả người chơi
    private:
      enabled: true  # Tin nhắn riêng cho người tham gia
    skip-disbanded: true  # Bỏ qua đội đã giải tán trong kết quả
    
  # Cài đặt định dạng thời gian
  time-format:
    suffix:
      days: "d"
      hours: "h"
      minutes: "m"
      seconds: "s"
    show-hours: false
```

### Tin nhắn (`messages.yml`)

Tất cả tin nhắn có thể tùy chỉnh trong `messages.yml`. Các đường dẫn tin nhắn chính:

- `teamwar.prefix` - Tiền tố cho tất cả tin nhắn TeamWar
- `teamwar.countdown.tick` - Tin nhắn đếm ngược
- `teamwar.started` - Tin nhắn chiến tranh bắt đầu
- `teamwar.ruleA.triggered` - Thông báo Quy tắc A
- `teamwar.alive-command.*` - Tin nhắn lệnh alive
- `teamwar.top-command.*` - Tin nhắn lệnh top
- `teamwar.end.broadcast.*` - Tin nhắn thông báo kết thúc
- `teamwar.end.private.*` - Tin nhắn kết thúc riêng tư
- `teamwar.errors.*` - Tin nhắn lỗi

## PlaceholderAPI Placeholders

### Placeholder tổng đội

| Placeholder | Mô tả |
|-------------|-------|
| `%frozenevent_totalteam_all%` | Tổng đội còn sống trong phạm vi "all" |
| `%frozenevent_totalteam_world_<tênThếGiới>%` | Tổng đội còn sống trong thế giới cụ thể |

### Placeholder top đội

| Placeholder | Mô tả |
|-------------|-------|
| `%frozenevent_all_top1%` đến `%frozenevent_all_top10%` | Tên đội top 1-10 trong phạm vi "all" |
| `%frozenevent_world_<tênThếGiới>_top1%` đến `%frozenevent_world_<tênThếGiới>_top10%` | Tên đội top 1-10 trong phạm vi thế giới |

### Ví dụ Placeholder

```yaml
# Ví dụ scoreboard
- "&6TeamWar"
- "&7Còn sống: &e%frozenevent_totalteam_all%"
- "&7Top 1: &a%frozenevent_all_top1%"
- "&7Top 2: &a%frozenevent_all_top2%"
- "&7Top 3: &a%frozenevent_all_top3%"

# Cụ thể theo thế giới
- "&7Đội thế giới: &e%frozenevent_totalteam_world_world_nether%"
- "&7Dẫn đầu: &a%frozenevent_world_world_nether_top1%"
```

**Lưu ý**: Placeholders trả về giá trị `none-value` đã cấu hình (mặc định: "-") khi:
- Không có chiến tranh đang hoạt động
- Chiến tranh đang ở giai đoạn COUNTDOWN (chưa có snapshot)
- Chỉ số top yêu cầu vượt quá số đội có sẵn

## Hỗ trợ

Để báo cáo lỗi, đặt câu hỏi hoặc yêu cầu tính năng, vui lòng liên hệ tác giả plugin hoặc tạo issue trong repository.

## Giấy phép

[Giấy phép của bạn ở đây]

## Credits

- Được xây dựng cho máy chủ Paper/Folia
- Tích hợp với plugin BetterTeams
- Hỗ trợ PlaceholderAPI
