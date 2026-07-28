-- โหลด Rayfield Library
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- สร้าง หน้าต่างหลัก (Window)
local Window = Rayfield:CreateWindow({
   Name = "Player Script | Rayfield UI ⚡",
   LoadingTitle = "กำลังโหลดสคริปต์...",
   LoadingSubtitle = "by Gemini",
   ConfigurationSaving = {
      Enabled = false,
   },
   KeySystem = false -- ปิดระบบใส่คีย์
})

-- สร้าง Tab หลัก
local MainTab = Window:CreateTab("Main", 4483362458) -- ไอคอนระบบ

-- ตัวแปรตั้งค่าระบบ
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ESPEnabled = false
local Highlights = {}

-- 1. ระบบปรับความเร็ว (Speed)
MainTab:CreateSlider({
   Name = "WalkSpeed (ความเร็วในการเดิน)",
   Range = {16, 500},
   Increment = 1,
   Suffix = " Speed",
   CurrentValue = 16,
   Flag = "SpeedSlider",
   Callback = function(Value)
      if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
         LocalPlayer.Character.Humanoid.WalkSpeed = Value
      end
   end,
})

-- 2. ระบบปรับความสูงการกระโดด (Jump Power)
MainTab:CreateSlider({
   Name = "JumpPower (แรงกระโดด)",
   Range = {50, 500},
   Increment = 1,
   Suffix = " Power",
   CurrentValue = 50,
   Flag = "JumpSlider",
   Callback = function(Value)
      if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
         LocalPlayer.Character.Humanoid.UseJumpPower = true
         LocalPlayer.Character.Humanoid.JumpPower = Value
      end
   end,
})

-- ฟังก์ชันจัดการ ESP สีฟ้า
local function ApplyESP(player)
   if player == LocalPlayer then return end
   
   local function CharacterAdded(char)
      local highlight = Instance.new("Highlight")
      highlight.Name = "BlueESP"
      highlight.Adornee = char
      highlight.FillColor = Color3.fromRGB(0, 170, 255) -- สีฟ้า
      highlight.FillTransparency = 0.5 -- ความโปร่งแสง
      highlight.OutlineColor = Color3.fromRGB(255, 255, 255) -- เส้นขอบสีขาว
      highlight.OutlineTransparency = 0
      highlight.Enabled = ESPEnabled
      highlight.Parent = char
      
      Highlights[player] = highlight
   end

   if player.Character then
      CharacterAdded(player.Character)
   end
   player.CharacterAdded:Connect(CharacterAdded)
end

-- สแกนผู้เล่นเดิมและผู้เล่นใหม่ที่เข้ามาในห้อง
for _, player in pairs(Players:GetPlayers()) do
   ApplyESP(player)
end
Players.PlayerAdded:Connect(ApplyESP)

-- ลบ ESP เมื่อมีคนออกจากเกม
Players.PlayerRemoving:Connect(function(player)
   if Highlights[player] then
      Highlights[player]:Destroy()
      Highlights[player] = nil
   end
end)

-- 3. สวิตช์ เปิด-ปิด มองทะลุสีฟ้า (ESP Blue)
MainTab:CreateToggle({
   Name = "ESP มองทะลุ (สีฟ้า) 👀",
   CurrentValue = false,
   Flag = "ESPToggle",
   Callback = function(Value)
      ESPEnabled = Value
      for _, highlight in pairs(Highlights) do
         if highlight then
            highlight.Enabled = ESPEnabled
         end
      end
   end,
})

-- คืนค่า Speed / Jump เวลาเกิดใหม่ (Respawn)
LocalPlayer.CharacterAdded:Connect(function(char)
   local humanoid = char:WaitForChild("Humanoid")
   task.wait(0.5)
   humanoid.WalkSpeed = Rayfield.Flags["SpeedSlider"].CurrentValue
   humanoid.UseJumpPower = true
   humanoid.JumpPower = Rayfield.Flags["JumpSlider"].CurrentValue
end)
