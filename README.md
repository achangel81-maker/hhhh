

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Bake or Die FIX",
   ConfigurationSaving = { Enabled = true, FolderName = "BakeOrDieFix", FileName = "Config" },
   Discord = { Enabled = false },
   KeySystem = false,
})


_G.Settings = {
    KillAura = { Enabled = false, Distance = 30 },
    BringMobs = { Enabled = false, Distance = 50 }, -- ЖИВІ МОБИ
    BringBodies = { Enabled = false, Distance = 50 }, -- МЕРТВІ ТІЛА
    BringItems = { Enabled = false, Distance = 50 },
    Speed = 16, Jump = 50, WeaponSlot = 2,
    ESP = { Monsters = false, Bodies = true, Items = false }
}


local Players, LocalPlayer, Remotes = game:GetService("Players"), game.Players.LocalPlayer,
    require(game:GetService("ReplicatedStorage").Client.ClientRemotes)


local function getChar() return LocalPlayer.Character end
local function getRoot(char) return char and char:FindFirstChild("HumanoidRootPart") end
local function isBody(m) -- Перевірка на мертве тіло
    local h = m:FindFirstChild("Humanoid")
    return h and (h.Health <= 0 or h:GetState() == Enum.HumanoidStateType.Dead)
end


local BodiesTab = Window:CreateTab("👻 Тіла", "")
BodiesTab:CreateSection("⚰️ МЕРТВІ ТІЛА")
BodiesTab:CreateToggle({ Name = "👻 Притягувати мертві тіла", CurrentValue = false,
    Callback = function(v) _G.Settings.BringBodies.Enabled = v end })
BodiesTab:CreateSlider({ Name = "Радіус", Range = {20,200}, CurrentValue = 50,
    Callback = function(v) _G.Settings.BringBodies.Distance = v end })
BodiesTab:CreateButton({ Name = "⚰️ Притягнути всі тіла (разово)", Callback = function()
    local r = getRoot(getChar()); if not r then return end
    for _, m in pairs(workspace.Monsters:GetChildren()) do
        if isBody(m) and m:FindFirstChild("HumanoidRootPart") then
            m.HumanoidRootPart.CFrame = r.CFrame * CFrame.new(0,0,-3)
            task.wait()
        end
    end
end })

local MobsTab = Window:CreateTab("👹 Моби", "")
MobsTab:CreateSection("👹 ЖИВІ МОБИ")
MobsTab:CreateToggle({ Name = "👹 Притягувати ЖИВИХ мобів", CurrentValue = false,
    Callback = function(v) _G.Settings.BringMobs.Enabled = v end })
MobsTab:CreateButton({ Name = "🔪 Вбити всіх", Callback = function()
    for _, m in pairs(workspace.Monsters:GetChildren()) do
        if m:FindFirstChild("Humanoid") and m.Humanoid.Health > 0 then
            pcall(function() Remotes.meleeAttack:Fire({monsters={m}, civilians={}, activeSlot=_G.Settings.WeaponSlot}) end)
            task.wait()
        end
    end
end })

local ItemsTab = Window:CreateTab("📦 Предмети", "")
ItemsTab:CreateToggle({ Name = "📦 Притягувати предмети", CurrentValue = false,
    Callback = function(v) _G.Settings.BringItems.Enabled = v end })
ItemsTab:CreateButton({ Name = "📦 Притягнути всі", Callback = function()
    local r = getRoot(getChar()); if not r then return end
    for _, i in pairs(workspace.Interactables:GetChildren()) do
        if i:IsA("Model") and not i:FindFirstChild("ProductPriceTag") and i.PrimaryPart then
            i.PrimaryPart.CFrame = r.CFrame * CFrame.new(0,0,-3)
            task.wait()
        end
    end
end })


task.spawn(function() while task.wait(0.3) do
    if _G.Settings.BringBodies.Enabled then
        local r = getRoot(getChar()); if not r then return end
        for _, m in pairs(workspace.Monsters:GetChildren()) do
            if isBody(m) and m:FindFirstChild("HumanoidRootPart") then
                local dist = (r.Position - m.HumanoidRootPart.Position).Magnitude
                if dist < _G.Settings.BringBodies.Distance and dist > 5 then
                    m.HumanoidRootPart.CFrame = r.CFrame * CFrame.new(0,0,-3)
                end
            end
        end
    end
end end)

task.spawn(function() while task.wait(0.3) do
    if _G.Settings.BringMobs.Enabled then
        local r = getRoot(getChar()); if not r then return end
        for _, m in pairs(workspace.Monsters:GetChildren()) do
            if m:FindFirstChild("Humanoid") and m.Humanoid.Health > 0 and m:FindFirstChild("HumanoidRootPart") then
                local dist = (r.Position - m.HumanoidRootPart.Position).Magnitude
                if dist < _G.Settings.BringMobs.Distance and dist > 5 then
                    m.HumanoidRootPart.CFrame = r.CFrame * CFrame.new(0,0,-5)
                end
            end
        end
    end
end end)


task.spawn(function() while task.wait(0.2) do
    if _G.Settings.KillAura.Enabled then
        local r = getRoot(getChar()); if not r then return end
        for _, m in pairs(workspace.Monsters:GetChildren()) do
            if m:FindFirstChild("Humanoid") and m.Humanoid.Health > 0 and m:FindFirstChild("HumanoidRootPart") then
                if (r.Position - m.HumanoidRootPart.Position).Magnitude < _G.Settings.KillAura.Distance then
                    pcall(function() Remotes.meleeAttack:Fire({monsters={m}, civilians={}, activeSlot=_G.Settings.WeaponSlot}) end)
                    break
                end
            end
        end
    end
end end)


task.spawn(function() while task.wait(0.3) do
    if _G.Settings.BringItems.Enabled then
        local r = getRoot(getChar()); if not r then return end
        for _, i in pairs(workspace.Interactables:GetChildren()) do
            if i:IsA("Model") and not i:FindFirstChild("ProductPriceTag") and i.PrimaryPart then
                local dist = (r.Position - i.PrimaryPart.Position).Magnitude
                if dist < _G.Settings.BringItems.Distance and dist > 5 then
                    i.PrimaryPart.CFrame = r.CFrame * CFrame.new(0,0,-3)
                end
            end
        end
    end
end end)


task.spawn(function() while task.wait(1) do
    local c = getChar(); if c and c:FindFirstChild("Humanoid") then
        c.Humanoid.WalkSpeed, c.Humanoid.JumpPower = _G.Settings.Speed, _G.Settings.Jump
    end
end end)

Rayfield:Notify({ Title = "Bake or Die FIX", Content = "✅ Притягування тіл та мобів активне!", Duration = 5 })
