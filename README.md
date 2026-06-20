-- ====================================================================
--                  🦊 FOXNAME SZA HUB — AREA 51 🦊
-- ====================================================================

-- 1. ПОДКЛЮЧЕНИЕ СТАБИЛЬНОЙ UI-БИБЛИОТЕКИ RAYFIELD
local Rayfield = loadstring(game:HttpGet('https://sirius.menu'))()

-- 2. СОЗДАНИЕ ГЛАВНОГО ОКНА ХАБА
local Window = Rayfield:CreateWindow({
   Name = "🦊 Area 51: Foxname SZA Hub",
   LoadingTitle = "Загрузка Foxname SZA...",
   LoadingSubtitle = "by Foxname Developer",
   ConfigurationSaving = {
      Enabled = false -- Отключено для стабильности при первом запуске
   },
   KeySystem = false -- Без ключа, быстрый старт для пользователей
})

-- 3. СОЗДАНИЕ ВКЛАДОК МЕНЮ
local MainTab = Window:CreateTab("Главные Читы", 4483362458)
local TeleportTab = Window:CreateTab("Телепортация", 4483362458)
local VisualsTab = Window:CreateTab("Визуалы (ESP)", 4483362458)

-- Глобальные переменные для удобства
local player = game.Players.LocalPlayer
local TweenService = game:GetService("TweenService")
local folders = game.Workspace:FindFirstChild("Killers")

------------------------------------------------------------------------
-- ВКЛАДКА: ГЛАВНЫЕ ЧИТЫ (БЕССМЕРТИЕ, ПАТРОНЫ, СПАВН ПРЕДМЕТОВ)
------------------------------------------------------------------------

-- Функция Бессмертия (Мгновенный авто-хил при получении урона)
local GodModeEnabled = false
MainTab:CreateToggle({
   Name = "Бессмертие (Auto-Regen)",
   CurrentValue = false,
   Flag = "GodModeFlag", 
   Callback = function(Value)
       GodModeEnabled = Value
       if GodModeEnabled then
           task.spawn(function()
               while GodModeEnabled do
                   if player.Character and player.Character:FindFirstChild("Humanoid") then
                       local hum = player.Character.Humanoid
                       if hum.Health < hum.MaxHealth then 
                           hum.Health = hum.MaxHealth 
                       end
                   end
                   task.wait(0.1) -- Безопасный интервал проверки
               end
           end)
       end
   end,
})

-- Функция Бесконечных Патронов (Авто-запись патронов в конфигурацию пушки)
local InfAmmoEnabled = false
MainTab:CreateToggle({
   Name = "Бесконечные патроны",
   CurrentValue = false,
   Flag = "InfAmmoFlag",
   Callback = function(Value)
       InfAmmoEnabled = Value
       if InfAmmoEnabled then
           task.spawn(function()
               while InfAmmoEnabled do
                   local tool = player.Character:FindFirstChildOfClass("Tool")
                   if tool and tool:FindFirstChild("Configuration") then
                       local ammo = tool.Configuration:FindFirstChild("Ammo")
                       local maxAmmo = tool.Configuration:FindFirstChild("MaxAmmo")
                       if ammo then ammo.Value = 999 end
                       if maxAmmo then maxAmmo.Value = 999 end
                   end
                   task.wait(0.2)
               end
           end)
       end
   end,
})

-- Функция Безопасного Спавна Предметов ( firetouchinterest без детекта)
local function spawnItem(itemName)
    local itemModel = game.Workspace:FindFirstChild(itemName, true)
    if itemModel and itemModel:FindFirstChild("TouchInterest") then
        local hrp = player.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            firetouchinterest(hrp, itemModel, 0) -- Симуляция касания
            task.wait(0.05)
            firetouchinterest(hrp, itemModel, 1) -- Симуляция отпускания
            Rayfield:Notify({Title = "Foxname SZA", Content = "Предмет " .. itemName .. " получен!", Duration = 3})
        end
    else
        Rayfield:Notify({Title = "Ошибка", Content = "Предмет на карте не найден или уже поднят", Duration = 3})
    end
end

MainTab:CreateButton({
   Name = "Получить Автомат M4A1",
   Callback = function() spawnItem("M4A1") end
})

MainTab:CreateButton({
   Name = "Получить Бластер RayGun",
   Callback = function() spawnItem("RayGun") end
})

------------------------------------------------------------------------
-- ВКЛАДКА: ТЕЛЕПОРТАЦИЯ (ПЛАВНЫЙ СЕРВЕРНЫЙ ОБХОД ЧЕРЕЗ TWEEN)
------------------------------------------------------------------------

local function safeTeleport(targetCoords)
    local hrp = player.Character:FindFirstChild("HumanoidRootPart")
    if hrp then
        local distance = (hrp.Position - targetCoords).Magnitude
        local speed = 65 -- Оптимальная скорость, чтобы серверный античит не кикнул
        local duration = distance / speed
        local tweenInfo = TweenInfo.new(duration, Enum.EasingStyle.Linear)
        local tween = TweenService:Create(hrp, tweenInfo, {CFrame = CFrame.new(targetCoords)})
        
        Rayfield:Notify({Title = "Телепорт", Content = "Перемещение началось...", Duration = 2})
        tween:Play()
    end
end

TeleportTab:CreateButton({
   Name = "Телепорт на Спавн (Безопасная зона)",
   Callback = function() safeTeleport(Vector3.new(15, 3, 20)) end
})

TeleportTab:CreateButton({
   Name = "Телепорт в Оружейную Комнату",
   Callback = function() safeTeleport(Vector3.new(-45, -10, 110)) end
})

TeleportTab:CreateButton({
   Name = "Телепорт в Комнату Управления",
   Callback = function() safeTeleport(Vector3.new(115, 5, -80)) end
})

------------------------------------------------------------------------
-- ВКЛАДКА: ВИЗУАЛЫ (БЕЗОПАСНЫЙ HIGHLIGHT ESP НА ИГРОКОВ И УБИЙЦ)
------------------------------------------------------------------------

local function applyESP(object, color)
    if not object:FindFirstChild("SZA_ESP") then
        local highlight = Instance.new("Highlight")
        highlight.Name = "SZA_ESP"
        highlight.FillColor = color
        highlight.FillTransparency = 0.5
        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
        highlight.OutlineTransparency = 0
        highlight.Adornee = object
        highlight.Parent = object
    end
end

local function removeESP(object)
    local esp = object:FindFirstChild("SZA_ESP")
    if esp then esp:Destroy() end
end

-- ESP на Монстров / Убийц (Красный цвет)
local KillersESPEnabled = false
VisualsTab:CreateToggle({
   Name = "Подсветка Убийц (Красный)",
   CurrentValue = false,
   Flag = "KillersESP",
   Callback = function(Value)
       KillersESPEnabled = Value
       if KillersESPEnabled then
           task.spawn(function()
               while KillersESPEnabled do
                   for _, monster in pairs(game.Workspace:GetChildren()) do
                       if monster:IsA("Model") and monster:FindFirstChild("Humanoid") and not game.Players:GetPlayerFromCharacter(monster) then
                           applyESP(monster, Color3.fromRGB(255, 0, 0))
                       end
                   end
                   task.wait(1)
               end
           end)
       else
           for _, monster in pairs(game.Workspace:GetChildren()) do removeESP(monster) end
       end
   end,
})

-- ESP на Других Игроков (Зеленый цвет)
local PlayersESPEnabled = false
VisualsTab:CreateToggle({
   Name = "Подсветка Игроков (Зеленый)",
   CurrentValue = false,
   Flag = "PlayersESP",
   Callback = function(Value)
       PlayersESPEnabled = Value
       if PlayersESPEnabled then
           task.spawn(function()
               while PlayersESPEnabled do
                   for _, p in pairs(game.Players:GetPlayers()) do
                       if p ~= player and p.Character then 
                           applyESP(p.Character, Color3.fromRGB(0, 255, 0)) 
                       end
                   end
                   task.wait(1)
               end
           end)
       else
           for _, p in pairs(game.Players:GetPlayers()) do 
               if p.Character then removeESP(p.Character) end 
           end
       end
   end,
})
