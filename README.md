# FrozenEvent Plugin

Plugin Minecraft tổ chức sự kiện TeamWar - hệ thống thi đấu sinh tồn theo đội với cơ chế loại trừ tự động, border thu hẹp và bảo vệ arena. Hỗ trợ đầy đủ cả Paper và Folia với kiến trúc thread-safe production-ready.

## Giới thiệu

FrozenEvent là plugin tổ chức sự kiện PvP quy mô lớn (500-2000+ players) cho server Minecraft. Plugin được thiết kế với kiến trúc **3 phạm vi độc lập** (Roster Source + Arena + Loser World):

- **Roster Source (Lobby)**: Nơi chốt danh sách team tham gia - chỉ team có thành viên ở đây khi `/teamwar start` mới được đăng ký
- **Arena World**: Nơi diễn ra chiến đấu - border thu hẹp theo giai đoạn, world guard bảo vệ nghiêm ngặt, player bị loại khi chết/thoát
- **Loser World**: Nơi player bị loại được teleport đến và bị khóa không cho quay lại

**Flow hoạt động:**
1. Player tập trung ở **roster worlds** (lobby) để đăng ký
2. Admin gõ `/teamwar start 60` → hệ thống chốt danh sách team có member ở lobby
3. Đếm ngược 60 giây để player chuẩn bị (warmup cache để đảm bảo snapshot chính xác)
4. Player tự teleport vào **arena worlds** để chiến đấu
5. Border thu hẹp theo nhiều giai đoạn với auto-teleport thông minh (tier-based)
6. Player chết/thoát arena → bị loại vĩnh viễn → teleport về **loser_world** và bị khóa
7. Team cuối cùng còn người = chiến thắng, kết quả được lưu vào SQLite database với async writes

## Tính năng chính

### 🎮 Hệ thống TeamWar

- **Kiến trúc 3 phạm vi độc lập**: Roster Source (lobby), Arena (chiến trường), và Loser World hoàn toàn tách biệt
- **Đăng ký linh hoạt**: Hỗ trợ 3 chế độ roster source (LOBBY_WORLDS, SENDER_WORLD, FIXED_WORLDS)
- **Arena đa dạng**: Cấu hình FIXED_WORLDS hoặc ALL_EXCEPT cho nhiều kịch bản
- **Loại trừ nghiêm ngặt**: Player chết hoặc thoát arena = bị loại vĩnh viễn + khóa không cho quay lại
- **Đếm ngược thông minh**: Warmup cache trước khi snapshot để đảm bảo dữ liệu chính xác
- **Xử lý gián đoạn**: Tự động phát hiện và xử lý server restart giữa chừng event
- **Multi-scope state management**: Hỗ trợ nhiều war đồng thời với ScopeKey isolation

### 📊 Theo dõi và xếp hạng

- **Snapshot cache**: Hệ thống cache Folia-safe cho trạng thái player thời gian thực với warmup mechanism
- **Debounce queue**: Tối ưu recompute eliminations tránh lag spike với configurable debounce ticks
- **Database SQLite**: Lưu trữ top 10 team với thời gian sống sót và metadata đầy đủ (async writes, WAL mode)
- **PlaceholderAPI**: 10+ placeholders cho scoreboard động (is_running, totalteam, top1-10, timestamps)
- **Broadcast thông minh**: Chế độ global hoặc arena-only với Folia-safe delivery và batching support

### 🛡️ World Guard - Bảo vệ Arena

- **Initial Sweep**: Quét và đuổi tất cả player không trong roster ra khỏi arena khi start
- **Chặn người ngoài**: Player không trong roster bị eject về loser world khi cố vào arena
- **Loại trừ khi thoát**: Player thoát arena = forfeit + teleport loser world + khóa vĩnh viễn
- **Loại trừ khi chết**: Hỗ trợ 2 chế độ - chỉ trong arena hoặc anywhere (chống exploit)
- **Teleport lock**: Player bị loại bị khóa tại loser world, không thể quay lại arena
- **Bypass permission**: Admin có thể bypass tất cả quy tắc guard
- **Strict Entry Point**: Chỉ cho phép vào arena từ lobby (configurable)
- **Death Anywhere Forfeit**: Chống exploit - chết ở bất kỳ đâu cũng bị loại

### 🌍 Border - Thu hẹp thông minh

- **Multi-phase shrinking**: Cấu hình nhiều giai đoạn với tốc độ và damage khác nhau
- **Sequential schedule**: Thời gian chờ giữa các phase để player điều chỉnh vị trí
- **Auto-teleport tiers**: Hệ thống teleport phân tầng theo khoảng cách (0-5, 5-20, 20-100 blocks)
- **Accumulate seconds**: Tích lũy thời gian qua các tier, chuyển đổi thông minh
- **Reset on end**: Tự động reset border về initial radius với countdown announcement
- **Folia-compatible**: Border service hoàn toàn thread-safe cho Folia với per-world state tracking
- **Configurable center**: Hỗ trợ SPAWN hoặc FIXED coordinate mode

### ⚙️ Đăng ký nghiêm ngặt (Strict Registration)

- **3 cấp độ kiểm tra**: All players in lobby, all online members in lobby, all members online
- **Eligibility rules**: Áp dụng gamemode filter (SPECTATOR/CREATIVE) cho registration
- **Team blacklist**: Hỗ trợ blacklist theo team ID (UUID) hoặc team name
- **Error messages**: Hiển thị danh sách player/team vi phạm với giới hạn tùy chỉnh
- **World validation**: Kiểm tra tất cả worlds (roster, arena, loser) đã load trước khi start
- **BetterTeams integration**: Reflection-based hook hỗ trợ nhiều phiên bản BetterTeams

### 🚪 Lobby Close - Đóng lobby tự động

- **Tự động đóng lobby**: Sau thời gian cấu hình, lobby đóng và ép player vào arena
- **Teleport command**: Hỗ trợ CONSOLE hoặc PLAYER mode cho lệnh teleport tùy chỉnh
- **Grace period**: Thời gian chờ sau teleport thất bại trước khi xử thua
- **Retry mechanism**: Thử lại teleport nhiều lần với interval cấu hình
- **Warning system**: Cảnh báo player trước khi đóng lobby ở các mốc thời gian
- **Auto-forfeit**: Player không vào arena sau grace period bị tự động loại

## Yêu cầu server

- **Phiên bản Minecraft**: 1.20.x - 1.21.11
- **Loại server**: Paper hoặc Folia
- **Plugin bắt buộc**: BetterTeams (để quản lý đội)
- **Plugin khuyến nghị**: PlaceholderAPI (để hiển thị thống kê trên scoreboard)

## Hướng dẫn cài đặt

1. Tải file plugin từ releases
2. Đặt file `.jar` vào thư mục `plugins` của server
3. Cài đặt plugin BetterTeams (bắt buộc)
4. Cài đặt PlaceholderAPI (khuyến nghị)
5. Khởi động lại server
6. Cấu hình plugin trong `plugins/FrozenEvent/config.yml`
7. Chạy lệnh `/frozenevent debug` để kiểm tra

## Kiểm tra sau khi cài đặt

1. **Kiểm tra plugin đã tải**: Chạy `/plugins` - FrozenEvent phải màu xanh
2. **Kiểm tra lệnh**: Chạy `/frozenevent` để xem menu trợ giúp
3. **Chạy chẩn đoán**: Chạy `/frozenevent debug` để kiểm tra:
   - Plugin BetterTeams đã được phát hiện
   - PlaceholderAPI đã được phát hiện (nếu cài)
   - Không có lỗi trong cấu hình
4. **Test sự kiện**: Thử bắt đầu sự kiện test với `/frozenevent teamwar start 10`

**Kết quả mong đợi từ lệnh debug:**
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
  - Active Wars: 0

[FrozenEvent] === End Debug Status ===
```

Nếu gặp vấn đề:
- **BetterTeams: Not Found** → Cài đặt plugin BetterTeams
- **BetterTeamsHook: Not Available** → Kiểm tra phiên bản BetterTeams tương thích
- **TeamWar Module: Failed** → Xem console logs để tìm lỗi chi tiết

## Hướng dẫn sử dụng

### Lệnh quản lý sự kiện

**Lệnh cơ bản:**
- `/frozenevent` hoặc `/fe` - Hiển thị menu trợ giúp
- `/frozenevent debug` - Kiểm tra trạng thái plugin và phát hiện lỗi
- `/frozenevent reload` - Tải lại cấu hình (cần quyền admin)

**Lệnh TeamWar:**
- `/frozenevent teamwar start <giây>` - Bắt đầu sự kiện với thời gian đếm ngược
- `/frozenevent teamwar end` - Kết thúc sự kiện đang diễn ra (force end, không lưu kết quả)
- `/frozenevent teamwar alive` - Xem danh sách team còn lại trong sự kiện
- `/frozenevent teamwar top` - Xem top 3 team từ sự kiện vừa kết thúc
- `/frozenevent teamwar status` - Xem trạng thái chi tiết sự kiện
- `/frozenevent teamwar teams` - Xem danh sách tất cả các team với ID

### Ví dụ tổ chức sự kiện

**Bắt đầu sự kiện với đếm ngược 60 giây:**
```
/frozenevent teamwar start 60
```
Hệ thống sẽ:
1. Chốt danh sách team có member ở lobby (roster worlds)
2. Đếm ngược 60 giây để player chuẩn bị
3. Thông báo cho tất cả player
4. Player tự teleport vào arena để chiến đấu
5. Border bắt đầu thu hẹp theo cấu hình
6. Tự động loại team khi tất cả member bị loại (chết/thoát arena)
7. Công bố team chiến thắng khi chỉ còn 1 team

**Lưu ý quan trọng:**
- **Roster worlds** (nơi đăng ký): Cấu hình trong `teamwar.roster-source.worlds`
- **Arena worlds** (nơi chiến đấu): Cấu hình trong `teamwar.arena.worlds`
- **Loser world** (nơi bị loại): Cấu hình trong `teamwar.world-guard.loser-destination.world`

**Bắt đầu ngay không đếm ngược:**
```
/frozenevent teamwar start 0
```

**Xem team đang còn lại:**
```
/frozenevent teamwar alive
```

**Xem top 3 team từ sự kiện vừa kết thúc:**
```
/frozenevent teamwar top
```
Hiển thị 3 team có thời gian sống sót lâu nhất.

**Xem trạng thái sự kiện:**
```
/frozenevent teamwar status
```

**Xem danh sách tất cả các team:**
```
/frozenevent teamwar teams
```

**Kết thúc sự kiện sớm:**
```
/frozenevent teamwar end
```
Lưu ý: Force end sẽ KHÔNG lưu kết quả vào database.

### Quyền (Permissions)

- `frozenevent.admin` - Quyền quản trị plugin (reload, debug) - Mặc định: OP
- `frozenevent.teamwar.admin` - Bắt đầu/kết thúc sự kiện - Mặc định: OP
- `frozenevent.teamwar.view` - Xem thông tin sự kiện - Mặc định: Tất cả người chơi
- `frozenevent.teamwar.guard.bypass` - Bỏ qua world guard (admin) - Mặc định: OP

## Cấu hình plugin

### Quick Setup - Cài đặt nhanh

File cấu hình: `plugins/FrozenEvent/config.yml`

```yaml
teamwar:
  enabled: true
  
  # 1. Lobby đăng ký (nơi chốt danh sách team)
  roster-source:
    mode: LOBBY_WORLDS
    worlds:
      - "lobby"
  
  # 2. Arena chiến đấu (nơi diễn ra war)
  arena:
    mode: FIXED_WORLDS
    worlds:
      - "world_nether"
  
  # 3. Loser world (nơi player bị loại)
  world-guard:
    enabled: true
    loser-destination:
      world: "loser_world"
      x: 0.5
      y: 80.0
      z: 0.5
  
  # 4. Broadcast toàn server
  broadcast:
    global: true
```

**Flow với config trên:**
- Player đứng ở **lobby** → Admin gõ `/teamwar start 60`
- Chỉ team có người ở lobby mới được tham gia
- Player tự teleport vào **world_nether** để chiến đấu
- Player chết/thoát → teleport về **loser_world**

### Cấu hình Roster Source (Nơi đăng ký)

**Chế độ LOBBY_WORLDS (khuyến nghị):**
```yaml
roster-source:
  mode: LOBBY_WORLDS
  worlds:
    - "lobby"
```
Chỉ team có member ở lobby khi start mới tham gia.

**Chế độ SENDER_WORLD:**
```yaml
roster-source:
  mode: SENDER_WORLD
```
Admin đứng ở đâu thì roster là world đó. Tiện khi có nhiều lobby.

**Chế độ FIXED_WORLDS:**
```yaml
roster-source:
  mode: FIXED_WORLDS
  worlds:
    - "registration_area"
    - "waiting_room"
```
Giống LOBBY_WORLDS nhưng có thể nhiều world.

### Cấu hình Đăng ký nghiêm ngặt

```yaml
roster-source:
  registration:
    # RULE 1: Tất cả player online phải ở lobby
    require-all-online-players-in-lobby: false
    
    # RULE 2: Tất cả member ONLINE của team phải ở lobby
    require-all-online-team-members-in-lobby: false
    
    # RULE 3: Tất cả member của team phải online
    require-all-team-members-online: false
    
    # Áp dụng quy tắc gamemode (CREATIVE/SPECTATOR không tính)
    apply-eligibility-rules-to-registration: true
```

**Ví dụ thực tế:**
Team "Dragons" có 3 người: A (lobby), B (nether), C (offline)

- **Mặc định**: Team Dragons ĐƯỢC tham gia (A ở lobby)
- **require-all-online-team-members-in-lobby: true**: KHÔNG (B ở nether)
- **require-all-team-members-online: true**: KHÔNG (C offline)

### Cấu hình Arena (Nơi chiến đấu)

**Chế độ FIXED_WORLDS (khuyến nghị):**
```yaml
arena:
  mode: FIXED_WORLDS
  worlds:
    - "world_nether"
    - "world_the_end"
```
War chỉ diễn ra trong các world này. Border + world guard chỉ hoạt động ở đây.

**Chế độ ALL_EXCEPT:**
```yaml
arena:
  mode: ALL_EXCEPT
  all-except:
    blacklist-worlds:
      - "lobby"
      - "loser_world"
```
War diễn ra ở MỌI world TRỪ blacklist.

### Cấu hình World Guard

```yaml
world-guard:
  enabled: true
  
  # Điểm đến cho player bị loại
  loser-destination:
    world: "loser_world"
    x: 0.5
    y: 80.0
    z: 0.5
    yaw: 0.0
    pitch: 0.0
  
  rules:
    # Quét ban đầu: đuổi player không trong roster ra khỏi arena khi start
    initial-sweep-enabled: true
    
    # Người ngoài vào arena → đuổi về loser world
    outsider-enter-event-world: "EJECT_TO_LOSER"
    
    # Player thoát arena → loại + teleport loser world
    participant-leave-event-world: "FORFEIT_AND_TELEPORT_LOSER"
    
    # Player chết trong arena → loại + respawn loser world
    participant-death-in-event-world: "FORFEIT_AND_RESPAWN_LOSER"
    
    # Khóa player bị loại không cho quay lại arena
    eliminated-teleport-lock: true
```

**Quy tắc hoạt động:**
1. **Initial Sweep**: Khi start, đuổi tất cả player không trong roster ra khỏi arena
2. **Outsider**: Player không trong roster cố vào arena → đuổi ra
3. **Leave**: Player trong roster thoát arena → bị loại vĩnh viễn
4. **Death**: Player chết trong arena → bị loại vĩnh viễn
5. **Lock**: Player đã bị loại không thể quay lại arena

### Cấu hình Border

```yaml
border:
  enabled: true
  start-on-war-start: true
  initial-radius: 5000.0
  
  # Tâm border
  center:
    mode: SPAWN  # SPAWN hoặc FIXED
    x: 0.0
    z: 0.0
  
  # Các giai đoạn thu hẹp
  phases:
    - id: "phase-1"
      start-after: "30s"
      target-radius: 2500.0
      duration: "3m"
      damage: { buffer: 1.0, amount: 0.3 }
    
    - id: "phase-2"
      start-after: "30s"
      target-radius: 1000.0
      duration: "2m"
      damage: { buffer: 1.0, amount: 0.4 }
    
    - id: "final"
      start-after: "15s"
      target-radius: 25.0
      duration: "60s"
      damage: { buffer: 1.5, amount: 1.0 }
```

### Cấu hình Auto-Teleport

```yaml
border:
  auto-teleport:
    enabled: true
    only-when-waiting-between-phases: true
    near-edge-buffer: 5.0
    push-inside: 1.0
    check-interval-ticks: 20
    
    tiers:
      - upto: 20.0
        delay-seconds: 3
      - upto: 100.0
        delay-seconds: 10
    
    else-delay-seconds: -1
```

**Cách hoạt động:**
- Player ở 0-5 blocks ngoài border → teleport ngay
- Player ở 5-20 blocks ngoài → teleport sau 3 giây
- Player ở 20-100 blocks ngoài → teleport sau 10 giây
- Player ở >100 blocks ngoài → không teleport (để tự chết)

### Cấu hình Broadcast

```yaml
broadcast:
  global: true
```

- **true**: Tất cả player online thấy thông báo (khuyến nghị)
- **false**: Chỉ player trong arena thấy thông báo

### Cấu hình khác

```yaml
teamwar:
  # Hành vi khi chỉ có 1 team
  single-team-behavior: AUTO_WIN  # AUTO_WIN hoặc BLOCK
  
  # Bỏ qua player ở chế độ này
  spectator:
    ignore-gamemodes:
      - SPECTATOR
      - CREATIVE
  
  # Cảnh báo khi còn ít team
  ruleA:
    enabled: true
    threshold: 2
  
  # Blacklist team
  blacklist:
    team-ids: []
    team-names: []
```

### Tùy chỉnh thông báo

File thông báo: `plugins/FrozenEvent/messages.yml`

Bạn có thể tùy chỉnh tất cả thông báo của plugin, bao gồm:
- Thông báo đếm ngược
- Thông báo bắt đầu/kết thúc sự kiện
- Thông báo team bị loại
- Thông báo chiến thắng
- Cảnh báo border
- Thông báo world guard
- Và nhiều thông báo khác

Tất cả thông báo hỗ trợ màu sắc Minecraft và PlaceholderAPI.

## Tích hợp Scoreboard (PlaceholderAPI)

Plugin hỗ trợ hiển thị thông tin sự kiện trực tiếp trên scoreboard thông qua PlaceholderAPI.

### Placeholders có sẵn

**Kiểm tra trạng thái sự kiện:**
- `%frozenevent_is_running_all%` - Kiểm tra war có đang chạy không (trả về "true" hoặc "false")

**Sự kiện đang diễn ra (RUNNING):**
- `%frozenevent_totalteam_all%` - Số team còn lại trong sự kiện đang diễn ra

**Kết quả sự kiện đã kết thúc (từ database):**
- `%frozenevent_all_top1%` đến `%frozenevent_all_top10%` - Top 10 team từ sự kiện vừa kết thúc
- `%frozenevent_all_last_saved%` - Thời gian lưu kết quả
- `%frozenevent_all_last_ended%` - Thời gian sự kiện kết thúc

**Lưu ý quan trọng:** 
- Placeholder `is_running` trả về "true" nếu war đang ở phase RUNNING, "false" nếu không
- Placeholder `totalteam` hiển thị thông tin **sự kiện đang RUNNING**
- Placeholder `top` hiển thị kết quả **sự kiện ĐÃ KẾT THÚC** (lưu trong database)
- Trong lúc RUNNING, không có placeholder nào hiển thị danh sách team đang thi đấu
- Chỉ có thể xem danh sách team bằng lệnh `/teamwar alive`

### Ví dụ Scoreboard

**Scoreboard động (tự động chuyển đổi):**
```yaml
# Sử dụng plugin scoreboard hỗ trợ điều kiện như TAB, Oraxen, etc.
# Khi war đang chạy:
- condition: "%frozenevent_is_running_all%=true"
  lines:
    - "&6&l⚔ TEAMWAR ⚔"
    - ""
    - "&7Team còn lại: &e%frozenevent_totalteam_all%"
    - ""
    - "&7Đang thi đấu..."
    - ""
    - "&7server.com"

# Khi không có war:
- condition: "%frozenevent_is_running_all%=false"
  lines:
    - "&6&l⚔ TOP TEAMWAR ⚔"
    - ""
    - "&eTop trận trước:"
    - "&61. &f%frozenevent_all_top1%"
    - "&e2. &f%frozenevent_all_top2%"
    - "&73. &f%frozenevent_all_top3%"
    - ""
    - "&7server.com"
```

**Scoreboard trong lúc sự kiện đang diễn ra:**
```yaml
- "&6&l⚔ TEAMWAR ⚔"
- ""
- "&7Team còn lại: &e%frozenevent_totalteam_all%"
- ""
- "&7Đang thi đấu..."
- ""
- "&7server.com"
```

**Scoreboard hiển thị kết quả sự kiện trước:**
```yaml
- "&6&l⚔ TOP TEAMWAR ⚔"
- "&7━━━━━━━━━━━━━━"
- ""
- "&eKết quả trận trước:"
- "&61. &f%frozenevent_all_top1%"
- "&e2. &f%frozenevent_all_top2%"
- "&73. &f%frozenevent_all_top3%"
- ""
- "&7Kết thúc: &e%frozenevent_all_last_ended%"
- ""
- "&7━━━━━━━━━━━━━━"
```

**Scoreboard kết hợp (đang chạy + kết quả trước):**
```yaml
- "&6&l⚔ TEAMWAR ⚔"
- "&7━━━━━━━━━━━━━━"
- ""
- "&eSự kiện hiện tại:"
- "&7Còn lại: &a%frozenevent_totalteam_all% &7team"
- ""
- "&eTop trận trước:"
- "&61. &f%frozenevent_all_top1%"
- "&e2. &f%frozenevent_all_top2%"
- "&73. &f%frozenevent_all_top3%"
- ""
- "&7━━━━━━━━━━━━━━"
```

**Lưu ý:** 
- Placeholders hiển thị giá trị `-` (hoặc giá trị `none-value` đã cấu hình) khi:
  - `totalteam`: Không có sự kiện đang diễn ra hoặc đang trong giai đoạn đếm ngược
  - `top`: Chưa có sự kiện nào kết thúc hoặc không đủ team cho vị trí được yêu cầu

## Câu hỏi thường gặp (FAQ)

**Q: Làm sao để chỉ định world nào tham gia sự kiện?**
A: Cấu hình 2 phạm vi:
- **Roster worlds** (nơi đăng ký): `teamwar.roster-source.worlds: ["lobby"]`
- **Arena worlds** (nơi chiến đấu): `teamwar.arena.worlds: ["world_nether"]`

**Q: Team nào được tham gia sự kiện?**
A: Chỉ team có ít nhất 1 member ở roster worlds (lobby) khi admin gõ `/teamwar start` mới được tham gia.

**Q: Team bị loại khi nào?**
A: Team bị loại khi TẤT CẢ member bị loại bởi world guard (chết hoặc thoát arena). Không bị loại khi rời lobby.

**Q: Player bị loại có thể quay lại không?**
A: Không. Player bị loại sẽ bị khóa tại loser world và không thể quay lại arena (nếu `eliminated-teleport-lock: true`).

**Q: Border có tự động reset sau mỗi sự kiện không?**
A: Có, border sẽ tự động reset về cấu hình ban đầu khi sự kiện kết thúc.

**Q: Làm sao để thay đổi thông báo của plugin?**
A: Chỉnh sửa file `plugins/FrozenEvent/messages.yml` và reload plugin.

**Q: Plugin có hoạt động với server Folia không?**
A: Có, plugin hỗ trợ đầy đủ cả Paper và Folia với thread-safe design.

**Q: Force end có lưu kết quả không?**
A: Không. Lệnh `/teamwar end` sẽ kết thúc sự kiện ngay lập tức mà KHÔNG lưu kết quả vào database. Chỉ có auto-end (khi còn 1 team) mới lưu kết quả.

## Cấu hình nâng cao

### Performance Tuning (500-2000+ players)

File: `plugins/FrozenEvent/performance.yml`

```yaml
performance:
  enabled: true  # Bật OptimizedTeamWarModule
  
  # Tối ưu broadcast
  broadcast:
    batch-size: 50  # Gửi message theo batch
    delay-ticks: 1  # Delay giữa các batch
  
  # Tối ưu cache
  cache:
    warmup-delay: 20  # Delay warmup cache (ticks)
    cleanup-interval: 6000  # Cleanup interval (ticks)
```

**Khuyến nghị cho server lớn:**
- Border phases: Giảm số phases, tăng duration
- Border announce: Chỉ dùng TITLE mode, giảm số announcements
- Auto-teleport: Tăng check-interval-ticks lên 20-40
- Broadcast: Bật batch mode trong performance.yml

### Module Architecture

Plugin hỗ trợ 2 module implementations:

**TeamWarModule** (Standard):
- Dùng cho server < 500 players
- Full features không tối ưu
- Config: `performance.enabled: false`

**OptimizedTeamWarModule** (Performance):
- Dùng cho server 500-2000+ players
- Batched broadcasts, optimized cache
- Config: `performance.enabled: true`

Module được chọn tự động khi plugin load dựa trên config.

### Folia Compatibility

Plugin tự động detect Folia runtime:
- `FoliaSchedulerAdapter`: Region-based scheduling
- `PlayerSnapshotCache`: Thread-safe player access
- `Broadcaster`: Region-aware message delivery
- Tất cả state mutations qua `SchedulerAdapter.runNow()`

Không cần cấu hình thêm, plugin tự động adapt.

## Hỗ trợ và liên hệ

Nếu gặp vấn đề hoặc cần hỗ trợ:
1. Chạy `/frozenevent debug` và gửi kết quả
2. Kiểm tra console logs để tìm lỗi
3. Xem file `server/latest.log` để debug chi tiết
4. Liên hệ tác giả plugin hoặc tạo issue

### Debug Checklist

**Plugin không load:**
- Kiểm tra Java version >= 17
- Kiểm tra Paper/Folia version tương thích
- Xem console logs khi server start

**BetterTeams not found:**
- Cài đặt BetterTeams plugin
- Kiểm tra version tương thích
- Chạy `/frozenevent debug` để verify

**TeamWar không start:**
- Kiểm tra worlds đã load (`/frozenevent debug`)
- Verify roster-source và arena config
- Xem console logs để tìm error message
- Kiểm tra strict registration rules

**Border không hoạt động:**
- Verify `teamwar.border.enabled: true`
- Kiểm tra arena worlds config
- Xem BorderService logs trong console

**PlaceholderAPI không hoạt động:**
- Cài đặt PlaceholderAPI plugin
- Chạy `/papi reload` sau khi install
- Verify expansion registered: `/papi list`

---

**Phát triển bởi:** FrozenEvent Team  
**Phiên bản:** 1.0.0  
**Tương thích:** Minecraft 1.20.x - 1.21.11 (Paper/Folia)  
**Yêu cầu:** Java 17+, BetterTeams (bắt buộc), PlaceholderAPI (khuyến nghị)  
**License:** Proprietary

## Tính năng nổi bật

- ✅ **Thread-safe architecture**: Hỗ trợ đầy đủ Paper và Folia với SchedulerAdapter pattern và single-writer design
- ✅ **Kiến trúc 3 phạm vi**: Roster Source, Arena, và Loser World hoàn toàn độc lập, linh hoạt cấu hình
- ✅ **PlayerSnapshotCache**: Folia-safe player state access với warmup mechanism và world index optimization
- ✅ **DebounceQueue**: Tối ưu recompute eliminations tránh lag spike với configurable debounce ticks
- ✅ **World Guard nghiêm ngặt**: Initial sweep, teleport lock, death anywhere forfeit, strict entry point
- ✅ **Border multi-phase**: Sequential schedule với auto-teleport tiers thông minh và accumulate seconds
- ✅ **Lobby close system**: Tự động đóng lobby với retry mechanism, grace period, và auto-forfeit
- ✅ **Strict registration**: 3 cấp độ kiểm tra với eligibility rules, blacklist, và world validation
- ✅ **Database persistence**: SQLite với async writes, WAL mode, read cache, và connection pooling
- ✅ **PlaceholderAPI**: 10+ placeholders cho scoreboard động với real-time updates
- ✅ **Interrupted state handling**: Tự động xử lý server restart giữa chừng event với state recovery
- ✅ **Runtime command registration**: Hỗ trợ cả legacy và Paper Plugin API với dynamic registration
- ✅ **Broadcaster system**: Folia-safe message delivery với global/arena-only modes và batching
- ✅ **Performance optimized**: OptimizedTeamWarModule cho server 500-2000+ players với batched operations
- ✅ **Tùy chỉnh hoàn toàn**: Messages, time format, timezone, broadcast modes, và performance tuning
- ✅ **BetterTeams integration**: Reflection-based hook hỗ trợ BetterTeams 4.x, 5.x, 6.x

## Kiến trúc kỹ thuật

### Core Components

**TeamWarService** (1798 lines): Service chính quản lý state machine của TeamWar
- Multi-scope state management (ScopeKey: all hoặc specific world)
- Phase transitions: IDLE → COUNTDOWN → RUNNING → IDLE
- Snapshot creation với warmup cache mechanism
- Recompute eliminations với debounce queue
- Auto-end logic và force-end handling
- Integration với BorderService, WorldGuardService, LobbyCloseService

**SchedulerAdapter**: Abstraction layer cho Paper/Folia scheduling
- `PaperSchedulerAdapter`: Standard Bukkit scheduler
- `FoliaSchedulerAdapter`: Region-based scheduling cho Folia
- Single-writer pattern đảm bảo thread-safety
- `runNow()`, `runLater()`, `runRepeating()` APIs
- Entity-specific task scheduling cho Folia regions

**PlayerSnapshotCache**: Folia-safe player state cache
- Cache world name, gamemode, online status của players
- Warmup mechanism trước khi snapshot (configurable delay)
- Tự động update qua event listeners
- Thread-safe access cho Folia regions
- World index optimization cho fast lookups

**BorderService** (578 lines): Quản lý world border shrinking
- Multi-phase sequential schedule với configurable delays
- Per-world border state tracking
- Auto-teleport với tier system (distance-based delays)
- Accumulate seconds mechanism cho smooth transitions
- Reset mechanism với countdown announcements
- Folia-compatible với region-aware scheduling

**TeamWarWorldGuardService**: Bảo vệ arena và xử lý forfeit
- Initial sweep: eject non-roster players khi war start
- Outsider enter: block và eject về loser world
- Participant leave: forfeit + teleport + lock
- Death handling: forfeit + respawn/teleport loser world
- Teleport lock: prevent eliminated players từ quay lại
- Strict entry point: chỉ cho phép vào từ lobby
- Death anywhere forfeit: anti-exploit mechanism

**LobbyCloseService**: Tự động đóng lobby sau khi war start
- Countdown với warning announcements ở các mốc thời gian
- Teleport command execution (CONSOLE/PLAYER mode)
- Retry mechanism với configurable interval và max retries
- Grace period trước khi forfeit
- Integration với WorldGuardService cho auto-elimination

**BetterTeamsHook**: Integration với BetterTeams plugin
- `ReflectionBetterTeamsHook`: Reflection-based implementation hỗ trợ nhiều versions
- Team lookup by UUID và player
- Member listing với online status
- Compatibility với BetterTeams 4.x, 5.x, 6.x

**TeamWarModule vs OptimizedTeamWarModule**:
- `TeamWarModule`: Standard implementation cho server < 500 players
- `OptimizedTeamWarModule`: Performance-optimized cho 500-2000+ players
  - Batched broadcasts với configurable batch size
  - Optimized cache với TTL và max size
  - Async database operations với connection pooling
  - Partial recompute với threshold-based triggering

### Design Patterns

- **State Machine**: Phase transitions với guard conditions
- **Observer Pattern**: Event listeners notify service về state changes
- **Strategy Pattern**: RosterWorldResolver, ArenaWorldResolver
- **Adapter Pattern**: SchedulerAdapter cho Paper/Folia
- **Debouncing**: DebounceQueue tối ưu recompute operations
- **Cache Pattern**: PlayerSnapshotCache với warmup
- **Repository Pattern**: TeamWarLastResultStore cho persistence
- **Single-Writer Pattern**: Tất cả state mutations qua SchedulerAdapter.runNow()

### Thread Safety & Folia Compatibility

Plugin tự động detect Folia runtime và adapt:
- `FoliaSchedulerAdapter`: Region-based scheduling
- `PlayerSnapshotCache`: Thread-safe player access với concurrent collections
- `Broadcaster`: Region-aware message delivery
- Tất cả state mutations qua `SchedulerAdapter.runNow()` đảm bảo single-writer
- Entity-specific tasks cho player operations trong Folia regions
