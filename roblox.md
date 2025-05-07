**Kinh Nghiệm Hack Roblox: Tổng Quan Về Các Cấp Độ Bảo Mật**

Trong Roblox, **Các Cấp Độ Bảo Mật (Security Context Levels)** quy định mức độ tin tưởng và quyền truy cập mà các script và hành động khác nhau có thể đạt được trong môi trường game. Hiểu rõ về các cấp độ này rất quan trọng, đặc biệt trong bối cảnh hack, exploit và thực thi script.

### Các Cấp Độ Bảo Mật

Các cấp độ trong Roblox giao động từ **0 đến 8**, theo [Wiki Roblox](https://roblox.fandom.com/wiki/Security_context):

| Cấp Độ | Mô tả                                                           |
| :----: | :-------------------------------------------------------------- |
|   0    | Luồng ẩn danh (Anonymous threads)                               |
|   1    | Hành động do người dùng khởi tạo trong Roblox Studio            |
|   2    | Đối tượng BaseScript trong bất kỳ DataModel nào                 |
|   3    | BaseScript trong DataModel, ở nơi được tạo bởi Roblox           |
|   4    | BaseScript trong DataModel, nếu tác giả là Roblox               |
|   5    | Studio command bar, "Execute Script", thông qua tham số -script |
|   6    | Plugin Studio, COM API                                          |
|   7    | Web service API                                                 |
|   8    | Nhận dữ liệu qua replication                                    |

### Lưu ý quan trọng:

-   **LocalScript** và **Script** do người dùng tạo ra sẽ chạy ở **cấp độ 2**.
-   **Executor** (dùng trong các tool hack) có thể tuý chỉnh cấp độ, nhưng **mặc định là cấp 7**.

### Executor so với LocalScript/Script

Executor hoạt động ở **cấp 7** nên có **quyền lực mạnh hơn** so với LocalScript hoặc Script thông thường. Nó cho phép:

-   Thực hiện các thao tác API dịch vụ web (Web Service API).
-   Vượt qua nhiều hạn chế mà script cấp 2 gặp phải.

Tuy nhiên, **executor không hoàn toàn không thể bị phát hiện**. Khi inject vào Roblox, một phần nhỏ của bộ nhớ sẽ bị thay đổi. Khu vực nhớ này có thể bị **hook** hoặc giám sát. Roblox developers có thể phát hiện executor thông qua:

-   **Memory leak** (rò rỉ bộ nhớ)
-   **Stack overflow** (tràn ngăn xếp)
-   **Hành vi nghi vấn**

Tuy nhiên, việc dò tìm này **vô cùng khó khăn** vì developer các executor có thể **vá lỗ hổng chỉ sau vài giờ** sau khi bị lộ.

---

### Một Cách Phòng Chống Khác: Phát hiện hành vi nhân vật/người chơi

Thay vì cố gắng phát hiện executor, chúng ta có thể **phát hiện các hành động bất thường của nhân vật hoặc người chơi** như:

-   **FireServer** bất thường
-   **InvokeServer** sai lệch
-   **WalkSpeed** vượt giới hạn
-   **MoveDirection** bất thường
-   **Position** dịch chuyển không hợp lý

Ví dụ một đoạn kiểm tra tốc độ đơn giản bằng LocalScript:

```lua
while true do
    if char.Humanoid.WalkSpeed >= limitSpeed then
        Player:Kick('Cheater')
    end
    task.wait(1)
end
```

Hoặc kiểm tra dịch chuyển bất thường:

```lua
local character = game.Players.LocalPlayer.Character
local oldPosition = character.HumanoidRootPart.Position
while true do
    local newPosition = character.HumanoidRootPart.Position
    local magnitude = (newPosition - oldPosition).Magnitude

    if magnitude >= 100 then
        player:Kick('Teleport ?')
    end
    oldPosition = newPosition
    task.wait(5)
end
```

**Giải thích:**

-   Nếu WalkSpeed hoặc dịch chuyển (Position) của nhân vật vượt quá giới hạn hợp lý, người chơi sẽ tự động bị kick.
-   Có thể áp dụng tương tự cho MoveDirection, FireServer, InvokeServer để xác định hành vi hack.

---

### Một Vấn Đề Khi Chỉ Kiểm Tra Ở Client

Tuy nhiên, nếu kiểm tra ở client thì việc exploiter đọc script, decompile và patch lại cũng rất đơn giản bằng cách **disable script** hoặc **hook function**.

Ví dụ về hookfunction:

```lua
hookfunction(game.Players.LocalPlayer.Kick, function()
    return warn('KHO NHA BRO')
end)
```

Ví dụ về disable script từ game Grow A Garden:

[Xem video minh họa tại đây](https://drive.google.com/file/d/1cbCbkYRcrvaolj4oBVIXwxCoij4H2kYB/preview)

---
