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

Ví dụ về hook function:

```lua
hookfunction(game.Players.LocalPlayer.Kick, function()
    return warn('KHO NHA BRO')
end)
```

Ví dụ về disable script từ game Grow A Garden:

[Xem video minh họa tại đây](https://drive.google.com/file/d/1cbCbkYRcrvaolj4oBVIXwxCoij4H2kYB/preview)

---

### Các Biện Pháp Chống Cheat Hiệu Quả

Để ngăn chặn cheat một cách hiệu quả, cần kết hợp nhiều phương pháp:

#### 1. Sanity Check từ Client đến Server

Kiểm tra hợp lệ các giá trị gửi từ client như WalkSpeed, JumpPower, Position,... trước khi xử lý.

Ví dụ kiểm tra WalkSpeed trên server:

```lua
local Players = game:GetService('Players')

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        local humanoid = character:WaitForChild("Humanoid")
        humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
            if humanoid.WalkSpeed > 32 then -- Giới hạn tốc độ tối đa
                player:Kick("WalkSpeed bất thường.")
            end
        end)
    end)
end)
```

#### 2. Rate Limit

Giới hạn số lượng request FireServer/InvokeServer trong một khoảng thời gian nhất định để tránh spam hoặc abuse.

Ví dụ giới hạn FireServer:

```lua
local remote = game.ReplicatedStorage:WaitForChild("SomeRemoteEvent")
local playerCooldowns = {}

remote.OnServerEvent:Connect(function(player)
    local now = tick()
    if playerCooldowns[player] and (now - playerCooldowns[player]) < 1 then
        player:Kick("Spam FireServer detected!")
    else
        playerCooldowns[player] = now
        -- Thực hiện hành động hợp lệ ở đây
    end
end)
```

#### 3. Detect Teleport Từ Server

So sánh vị trí trước và sau của nhân vật trên server, nếu khoảng cách quá lớn trong thời gian ngắn thì đánh dấu khả nghi.

Ví dụ:

```lua
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        local hrp = character:WaitForChild("HumanoidRootPart")
        local lastPosition = hrp.Position
        while character.Parent do
            wait(1)
            local distance = (hrp.Position - lastPosition).Magnitude
            if distance > 50 then -- Giới hạn di chuyển bình thường
                player:Kick("Teleport bất hợp lệ.")
            end
            lastPosition = hrp.Position
        end
    end)
end)
```

#### 4. Server-Side Anti-Cheat

-   Mọi tính toán quan trọng như gây sát thương, cộng tiền, hoàn thành nhiệm vụ phải thực hiện **ở server**.
-   Client chỉ nên gửi yêu cầu, server phải xác minh tính hợp lệ.

#### 5. Client-Side Anti-Cheat

-   Phát hiện bất thường ngay tại client như WalkSpeed, JumpPower vượt giới hạn.
-   Có thể làm nhẹ nhàng để đẩy thông tin bất thường về server xử lý.
-   Tuy nhiên không nên đặt quá nhiều niềm tin vào client.

---

### Các Kỹ Thuật Phòng Chống Exploit Nâng Cao

Ngoài các phương pháp cơ bản, còn có thể sử dụng các kỹ thuật nâng cao để bảo vệ game:

#### 1. Ẩn Script Khỏi Stack và Tự Hủy

Ẩn script khỏi môi trường stack trace và xóa nó sau khi chạy:

```lua
for i = 0, 1 do
    xpcall(function()
        local currentScript = getfenv(i).script
        currentScript:Destroy()
        currentScript.Parent = nil
        currentScript = nil
        getfenv(i).script = nil
    end, function(err)
    end)
end
```

**Giải thích chi tiết:**

-   Lặp qua các stack 0 và 1 để lấy script đang chạy.
-   Xóa hoàn toàn script khỏi bộ nhớ và Parent.
-   Dọn sạch tham chiếu script trong môi trường `getfenv`.
-   Bao lỗi bằng `xpcall` để tránh crash.
-   Mục tiêu: **Ẩn dấu vết script**, chống bị trace hoặc decompile.

#### 2. Phát Hiện Hook `__index`

Kiểm tra việc hook `__index` bằng cách cố gắng truy cập thuộc tính ẩn và phát hiện lỗi bất thường:

```lua
while wait(1) do
    xpcall(function()
        return game.________________________
    end, function()
        local __index = debug.info(2, 'f')
        local thr = nil
        for i = 1, 198 do
            thr = coroutine.wrap(__index)
        end

        local s, e = pcall(thr, newproxy(true))
        if not s and string.find(e, 'Ugc') then
            warn('__index hooked')
        end
    end)
end
```

**Giải thích chi tiết:**

-   Truy cập 1 property không tồn tại nhằm trigger hook `__index`.
-   Nếu `__index` bị exploiter hook, quá trình sẽ gặp lỗi.
-   Dùng `debug.info` lấy function hook, kiểm tra lỗi qua coroutine.
-   Nếu lỗi chứa "Ugc", chứng minh `__index` đã bị hook.

#### 3. Phát Hiện Hook `__namecall`

Kiểm tra việc hook `__namecall` qua việc truy cập CoreGui:

```lua
while wait(1) do
    xpcall(function()
        return game:GetService('CoreGui'):GetChildren()
    end, function()
        local __namecall = debug.info(2, 'f')
        local thr = nil
        for i = 1, 198 do
            thr = coroutine.wrap(__namecall)
        end

        local s, e = pcall(thr, newproxy(true))
        if not s and string.find(e, 'CoreGui') then
            warn('__namecall hooked')
            error(e)
        end
    end)
end
```

**Giải thích chi tiết:**

-   Gọi `:GetChildren()` trên CoreGui.
-   Nếu `__namecall` bị hook, lỗi xảy ra.
-   `debug.info` lấy function hook để ép chạy kiểm tra.
-   Nếu lỗi liên quan đến CoreGui, thì chắc chắn đã bị hook `__namecall`.

#### 4. Phát Hiện Hook `__newindex`

Kiểm tra việc hook `__newindex` thông qua việc thử gán giá trị vào `game`:

```lua
while wait(1) do
    xpcall(function()
        game['_________'] = 1
        return
    end, function()
        local __newindex = debug.info(2, 'f')
        local thr = nil
        for i = 1, 198 do
            thr = coroutine.wrap(__newindex)
        end

        local s, e = pcall(thr, newproxy(true))
        if not s and string.find(e, 'Ugc') then
            warn('__newindex hooked')
        end
    end)
end
```

**Giải thích chi tiết:**

-   Gán key không hợp lệ vào `game` để kích hoạt `__newindex`.
-   Nếu `__newindex` bị hook thì hành động gán sẽ lỗi.
-   Kết hợp `debug.info` và kiểm tra lỗi để xác định hook.

#### 5. Phát Hiện Dex (Exploit GUI Detection)

Sử dụng cơ chế kiểm tra bộ thu gom rác (GC) để phát hiện hành vi bất thường:

```lua
if not game:IsLoaded() then
    game.Loaded:Wait()
end

task.wait(3)
local name = tostring(math.random())
local Chat = game:GetService("Chat")
Instance.new("BoolValue", Chat).Name = name

local t = setmetatable({}, {__mode="v"})
while task.wait() do
    t[1] = {}
    t[2] = Chat:FindFirstChild(name)
    while t[1] ~= nil do
        t[3] = string.rep("ab", 1024*2)
        t[3] = nil
        task.wait()
    end
    if t[2] ~= nil then
        warn("dex detected - invalid gc behaviour")
        break
    end
end
```

**Giải thích chi tiết:**

-   Tạo object nhỏ (BoolValue) trong Chat với tên ngẫu nhiên.
-   Theo dõi object này bằng weak reference (`__mode = "v"`).
-   Nếu bộ nhớ không thu hồi object sau khi GC, chứng tỏ bị giữ lại bởi exploit như Dex.

---
