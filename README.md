local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "Mec BR - FREEHUB V1",
    Icon = 0,
    LoadingTitle = "Carregando Menu",
    LoadingSubtitle = "by OfficialPatozoid",
    ShowText = "Rayfield",
    Theme = "Default",
    ToggleUIKeybind = "K",
    DisableRayfieldPrompts = false,
    DisableBuildWarnings = false,
    ConfigurationSaving = {
        Enabled = true,
        FolderName = nil,
        FileName = "Big Hub"
    },
    Discord = {
        Enabled = false,
        Invite = "noinvitelink",
        RememberJoins = true
    },
    KeySystem = false,
    KeySettings = {
        Title = "PEGUE A KEY",
        Subtitle = "Key System",
        Note = "Passe pelo rekonise para pegar a key",
        FileName = "Key",
        SaveKey = true,
        GrabKeyFromSite = false,
        Key = {"https://pastebin.com/raw/B9ytkGHC"}
    }
})

-- ===================== PLAYER TAB =====================
local PlayerTab = Window:CreateTab("Player", 4483362458)

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local humanoid
local infiniteJumpEnabled = false

local function onCharacterAdded(character)
    humanoid = character:WaitForChild("Humanoid")
end
player.CharacterAdded:Connect(onCharacterAdded)
if player.Character then onCharacterAdded(player.Character) end

-- Infinite Jump
local InfiniteJumpToggle = PlayerTab:CreateToggle({
    Name = "Infinite Jump",
    CurrentValue = false,
    Flag = "InfiniteJumpToggle",
    Callback = function(Value)
        infiniteJumpEnabled = Value
    end,
})
UserInputService.JumpRequest:Connect(function()
    if infiniteJumpEnabled and humanoid then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-- WalkSpeed Slider
local WalkSpeedSlider = PlayerTab:CreateSlider({
    Name = "WalkSpeed",
    Range = {0, 150},
    Increment = 1,
    Suffix = "WS",
    CurrentValue = 16,
    Flag = "WalkSpeedSlider",
    Callback = function(Value)
        if humanoid then humanoid.WalkSpeed = Value end
    end,
})

-- JumpPower Slider
local currentJumpPower = 50
local JumpPowerSlider = PlayerTab:CreateSlider({
    Name = "JumpPower",
    Range = {0, 250},
    Increment = 1,
    Suffix = "JP",
    CurrentValue = currentJumpPower,
    Flag = "JumpPowerSlider",
    Callback = function(Value)
        currentJumpPower = Value
        if humanoid then humanoid.JumpPower = currentJumpPower end
    end,
})

-- Noclip
local noclipEnabled = false
local NoclipToggle = PlayerTab:CreateToggle({
    Name = "Noclip",
    CurrentValue = false,
    Flag = "NoclipToggle",
    Callback = function(Value)
        noclipEnabled = Value
    end,
})
RunService.Stepped:Connect(function()
    local char = player.Character
    if noclipEnabled and char then
        for _, part in ipairs(char:GetChildren()) do
            if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                part.CanCollide = false
            end
        end
    elseif char then
        for _, part in ipairs(char:GetChildren()) do
            if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                part.CanCollide = true
            end
        end
    end
end)

-- Fly
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local flying = false
local flySpeed = 50

player.CharacterAdded:Connect(function(char)
    character = char
    humanoidRootPart = char:WaitForChild("HumanoidRootPart")
end)

local FlyToggle = PlayerTab:CreateToggle({
    Name = "Fly",
    CurrentValue = false,
    Flag = "FlyToggle",
    Callback = function(Value)
        flying = Value
        if not flying and humanoidRootPart then
            humanoidRootPart.Velocity = Vector3.new(0,0,0)
        end
    end,
})
local FlySpeedSlider = PlayerTab:CreateSlider({
    Name = "Fly Speed",
    Range = {10, 300},
    Increment = 5,
    Suffix = "Studs/s",
    CurrentValue = flySpeed,
    Flag = "FlySpeedSlider",
    Callback = function(Value)
        flySpeed = Value
    end,
})

local keysPressed = {W=false,A=false,S=false,D=false,Space=false,LeftShift=false}
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.W then keysPressed.W = true end
    if input.KeyCode == Enum.KeyCode.A then keysPressed.A = true end
    if input.KeyCode == Enum.KeyCode.S then keysPressed.S = true end
    if input.KeyCode == Enum.KeyCode.D then keysPressed.D = true end
    if input.KeyCode == Enum.KeyCode.Space then keysPressed.Space = true end
    if input.KeyCode == Enum.KeyCode.LeftShift then keysPressed.LeftShift = true end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.W then keysPressed.W = false end
    if input.KeyCode == Enum.KeyCode.A then keysPressed.A = false end
    if input.KeyCode == Enum.KeyCode.S then keysPressed.S = false end
    if input.KeyCode == Enum.KeyCode.D then keysPressed.D = false end
    if input.KeyCode == Enum.KeyCode.Space then keysPressed.Space = false end
    if input.KeyCode == Enum.KeyCode.LeftShift then keysPressed.LeftShift = false end
end)
RunService.RenderStepped:Connect(function()
    if flying and humanoidRootPart then
        local dir = Vector3.new(0,0,0)
        if keysPressed.W then dir = dir + workspace.CurrentCamera.CFrame.LookVector end
        if keysPressed.S then dir = dir - workspace.CurrentCamera.CFrame.LookVector end
        if keysPressed.A then dir = dir - workspace.CurrentCamera.CFrame.RightVector end
        if keysPressed.D then dir = dir + workspace.CurrentCamera.CFrame.RightVector end
        if keysPressed.Space then dir = dir + Vector3.new(0,1,0) end
        if keysPressed.LeftShift then dir = dir - Vector3.new(0,1,0) end
        if dir.Magnitude > 0 then
            humanoidRootPart.Velocity = dir.Unit * flySpeed
        else
            humanoidRootPart.Velocity = Vector3.new(0,0,0)
        end
    end
end)

-- ===================== TELEPORTS TAB =====================
local TPTab = Window:CreateTab("🧿Teleports", 4483362458)

TPTab:CreateButton({
    Name = "🚗FERRO-VELHO",
    Callback = function()
        local char = player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = CFrame.new(-3418.64502,148.252899,-458.328003,-1,8.74227766e-08,0,8.74227766e-08,1,8.74227766e-08,7.64274186e-15,8.74227766e-08,-1)
        end
    end,
})
TPTab:CreateButton({
    Name = "🚙Concessionaria",
    Callback = function()
        local char = player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = CFrame.new(-3167.74683,147.764023,-75.9954834,0,0,-1,0,1,0,1,0,0)
        end
    end,
})
TPTab:CreateButton({
    Name = "🔩Auto-Peças",
    Callback = function()
        local char = player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = CFrame.new(-3571.43774,149.245956,377.565308,-1,0,0,0,1,0,0,0,-1)
        end
    end,
})
TPTab:CreateButton({
    Name = "🚧Garagem",
    Callback = function()
        local char = player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = CFrame.new(-3556.26294,148.45932,907.711731,1,0,0,0,1,0,0,0,1)
        end
    end,
})

local selectedTeleport = nil
local Dropdown = TPTab:CreateDropdown({
    Name = "🔧Oficinas Publicas",
    Options = {"🔧Oficina1","🔧Oficina2"},
    CurrentOption = {},
    MultipleOptions = false,
    Flag = "TeleportDropdown",
    Callback = function(Options)
        selectedTeleport = Options[1]
    end,
})
TPTab:CreateButton({
    Name = "Teleportar",
    Callback = function()
        if not selectedTeleport then return end
        local char = player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            if selectedTeleport == "🔧Oficina1" then
                char.HumanoidRootPart.CFrame = CFrame.new(-2995.61377,161.869431,1483.03894,1,0,0,0,1,0,0,0,1)
            elseif selectedTeleport == "🔧Oficina2" then
                char.HumanoidRootPart.CFrame = CFrame.new(-4094.27637,148.055481,48.6451569,0,0,-1,0,1,0,1,0,0)
            end
        end
    end,
})
TPTab:CreateButton({
    Name = "Dinametros",
    Callback = function()
        local char = player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = CFrame.new(-4079.77246,160.567535,1490.77039,1,0,0,0,1,0,0,0,1)
        end
    end,
})

-- ===================== VEICULOS TAB =====================
local VeiculoTab = Window:CreateTab("🚗Veiculos", 4483362458)

local StarterGui = game:GetService("StarterGui")
local notifyEnabled = false
local teleportEnabled = false
local processed = setmetatable({}, {__mode = "k"})

local function showNotification(title, text, duration)
    duration = duration or 4
    local ok = pcall(function()
        StarterGui:SetCore("SendNotification", {Title=title,Text=text,Duration=duration})
    end)
    if not ok then
        local sg = Instance.new("ScreenGui")
        sg.ResetOnSpawn = false
        sg.Parent = player:WaitForChild("PlayerGui")
        local f = Instance.new("Frame")
        f.Size = UDim2.new(0,360,0,64)
        f.Position = UDim2.new(0.5,-180,0.08,0)
        f.BackgroundTransparency = 0.25
        f.Parent = sg
        local l = Instance.new("TextLabel")
        l.Size = UDim2.new(1,-10,1,-10)
        l.Position = UDim2.new(0,5,0,5)
        l.BackgroundTransparency = 1
        l.Text = (title and (title.."\n") or "")..(text or "")
        l.TextWrapped = true
        l.TextScaled = true
        l.Parent = f
        task.delay(duration, function() if sg and sg.Parent then sg:Destroy() end end)
    end
end

VeiculoTab:CreateToggle({
    Name = "Notificar Veículos",
    CurrentValue = false,
    Flag = "NotifyCarToggle",
    Callback = function(v) notifyEnabled = v end,
})
VeiculoTab:CreateToggle({
    Name = "Teleporte Automático p/ Veículo",
    CurrentValue = false,
    Flag = "AutoTeleportCarToggle",
    Callback = function(v) teleportEnabled = v end,
})

local purchaseableFolderRef = nil

local function findPurchaseFolder()
    local ok, folder = pcall(function()
        if workspace:FindFirstChild("cidade") and workspace.cidade:FindFirstChild("active_locations") then
            local al = workspace.cidade.active_locations
            if al:FindFirstChild("chatarera") and al.chatarera:FindFirstChild("funcionalidad") then
                return al.chatarera.funcionalidad:FindFirstChild("purchaseblecars")
            end
        end
        return nil
    end)
    if ok and folder then return folder end
    local tries = {
        {"cidade","active_locations","chatarera","funcionalidad","purchaseblecars"},
        {"cidade","active_locations","chatarrera","funcionalidad","purchaseblecars"},
        {"cidade","active_locations","chatarera","funcionalidad","purchaseablecars"},
    }
    for _, path in ipairs(tries) do
        local cur = workspace
        local okpath = true
        for _, name in ipairs(path) do
            cur = cur:FindFirstChild(name)
            if not cur then okpath = false; break end
        end
        if okpath and cur then return cur end
    end
    for _, inst in ipairs(workspace:GetDescendants()) do
        if inst:IsA("Folder") or inst:IsA("Model") then
            local lname = string.lower(inst.Name or "")
            for _, cand in ipairs({"purchaseblecars","purchasablecars","purchaseablecars","purchasecars"}) do
                if lname == cand then return inst end
            end
        end
    end
    return nil
end

local function getRepresentativePart(model)
    if not model then return nil end
    if model.PrimaryPart and model.PrimaryPart:IsA("BasePart") then return model.PrimaryPart end
    for _, v in ipairs(model:GetDescendants()) do
        if v:IsA("BasePart") then return v end
    end
    return nil
end

local function teleportToModel(model)
    if not model then return end
    local rep = getRepresentativePart(model)
    if not rep then return end
    local char = player.Character or player.CharacterAdded:Wait()
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    pcall(function()
        char.HumanoidRootPart.CFrame = rep.CFrame + Vector3.new(0,5,0)
        char.HumanoidRootPart.Velocity = Vector3.new(0,0,0)
    end)
end

local function processCarModel(model)
    if not model then return end
    local rootModel = model
    if purchaseableFolderRef then
        while rootModel.Parent and rootModel.Parent ~= purchaseableFolderRef and rootModel.Parent:IsA("Model") do
            rootModel = rootModel.Parent
        end
    else
        while rootModel.Parent and rootModel.Parent:IsA("Model") do
            rootModel = rootModel.Parent
        end
    end
    if not rootModel:IsA("Model") then return end
    if processed[rootModel] then return end
    local rep = getRepresentativePart(rootModel)
    local tries = 0
    while not rep and tries < 20 do task.wait(0.05); rep = getRepresentativePart(rootModel); tries = tries + 1 end
    if not rep then processed[rootModel] = true; return end
    processed[rootModel] = true
    local carName = rootModel.Name or "<Veículo sem nome>"
    if notifyEnabled then showNotification("🚗 Novo Veículo!", carName.." spawnou!", 5) end
    if teleportEnabled then task.delay(0.06, function() teleportToModel(rootModel) end) end
end

local function monitorFolder(folder)
    if not folder then
        task.spawn(function()
            while not purchaseableFolderRef do
                local f = findPurchaseFolder()
                if f then purchaseableFolderRef = f; monitorFolder(f); return end
                task.wait(1)
            end
        end)
        return
    end
    purchaseableFolderRef = folder
    for _, child in ipairs(folder:GetChildren()) do
        task.delay(0.04, function()
            if child:IsA("Model") then processCarModel(child) end
        end)
    end
    folder.ChildAdded:Connect(function(child)
        task.delay(0.04, function()
            if child:IsA("Model") then processCarModel(child) end
        end)
    end)
    folder.DescendantAdded:Connect(function(desc)
        task.delay(0.04, function()
            local ancestor = desc:FindFirstAncestorOfClass("Model")
            if ancestor and ancestor:IsDescendantOf(folder) then processCarModel(ancestor) end
        end)
    end)
end
monitorFolder(findPurchaseFolder())

-- ===================== UBER FARM TAB =====================
local UberTab = Window:CreateTab("🚖Uber Farm", 4483362458)

local uberFarmEnabled = false
local uberFarmStatus = "Aguardando..."
local uberDelay = 1.5 -- segundos entre cada ciclo

-- Label de status
local StatusLabel = UberTab:CreateLabel("Status: Desativado", 4483362458, Color3.fromRGB(255,200,0), false)

-- Toggle para ativar/desativar
UberTab:CreateToggle({
    Name = "Auto Farm Uber (TP)",
    CurrentValue = false,
    Flag = "UberFarmToggle",
    Callback = function(v)
        uberFarmEnabled = v
        if v then
            StatusLabel:Set("Status: 🟢 Farmando...")
        else
            StatusLabel:Set("Status: 🔴 Desativado")
        end
    end,
})

-- Slider de delay entre ciclos
UberTab:CreateSlider({
    Name = "Delay entre corridas (s)",
    Range = {0.5, 5},
    Increment = 0.5,
    Suffix = "s",
    CurrentValue = uberDelay,
    Flag = "UberDelaySlider",
    Callback = function(v)
        uberDelay = v
    end,
})

-- Funções auxiliares para Uber
local function findUberNPC()
    for _, obj in ipairs(workspace:GetDescendants()) do
        local name = string.lower(obj.Name or "")
        if obj:IsA("Model") or obj:IsA("BasePart") then
            if name:find("uber") or name:find("passageiro") or name:find("cliente") or name:find("passenger") then
                return obj
            end
        end
    end
    return nil
end

local function findUberMission()
    -- Tenta encontrar RemoteEvent/RemoteFunction de uber
    for _, obj in ipairs(game:GetDescendants()) do
        local name = string.lower(obj.Name or "")
        if (obj:IsA("RemoteEvent") or obj:IsA("RemoteFunction")) then
            if name:find("uber") or name:find("corrida") or name:find("taxi") or name:find("ride") then
                return obj
            end
        end
    end
    return nil
end

local function getUberPickupAndDropoff()
    -- Busca objetos de pickup e destino no workspace
    local pickup, dropoff = nil, nil
    for _, obj in ipairs(workspace:GetDescendants()) do
        local name = string.lower(obj.Name or "")
        if obj:IsA("BasePart") or obj:IsA("Model") then
            if name:find("pickup") or name:find("embarque") or name:find("ponto") then
                pickup = obj
            end
            if name:find("dropoff") or name:find("destino") or name:find("desembarque") or name:find("entrega") then
                dropoff = obj
            end
        end
    end
    return pickup, dropoff
end

local function tpTo(target)
    local char = player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    local pos
    if target:IsA("BasePart") then
        pos = target.CFrame + Vector3.new(0, 4, 0)
    elseif target:IsA("Model") then
        local part = target.PrimaryPart or getRepresentativePart(target)
        if part then
            pos = part.CFrame + Vector3.new(0, 4, 0)
        end
    end

    if pos then
        pcall(function()
            hrp.CFrame = pos
            hrp.Velocity = Vector3.new(0, 0, 0)
        end)
    end
end

local function fireUberEvents()
    -- Dispara RemoteEvents relacionados ao Uber automaticamente
    for _, obj in ipairs(game:GetDescendants()) do
        local name = string.lower(obj.Name or "")
        if obj:IsA("RemoteEvent") then
            if name:find("aceitar") or name:find("accept") or name:find("uber") or name:find("corrida") or name:find("startride") or name:find("endride") or name:find("finish") then
                pcall(function() obj:FireServer() end)
            end
        end
    end
end

-- Loop principal do Auto Farm Uber
task.spawn(function()
    while true do
        task.wait(0.5)
        if uberFarmEnabled then
            -- Tenta disparar eventos de aceitar corrida
            pcall(fireUberEvents)

            -- Tenta teleportar para pickup
            local pickup, dropoff = getUberPickupAndDropoff()

            if pickup then
                StatusLabel:Set("Status: 🟡 TP → Passageiro")
                tpTo(pickup)
                task.wait(uberDelay)
                pcall(fireUberEvents) -- confirma embarque
            end

            if dropoff then
                StatusLabel:Set("Status: 🟡 TP → Destino")
                tpTo(dropoff)
                task.wait(uberDelay)
                pcall(fireUberEvents) -- confirma entrega
            end

            -- Caso encontre NPC diretamente
            local npc = findUberNPC()
            if npc then
                StatusLabel:Set("Status: 🟡 TP → NPC Uber")
                tpTo(npc)
                task.wait(uberDelay)
                pcall(fireUberEvents)
            end

            StatusLabel:Set("Status: 🟢 Farmando... (ciclo completo)")
            task.wait(uberDelay)
        end
    end
end)

-- ===================== CREDITS TAB =====================
local CreditsTab = Window:CreateTab("Credits", 4483362458)

CreditsTab:CreateLabel("HUB BY: OfficialPatozoid", 4483362458, Color3.fromRGB(255,255,255), false)
CreditsTab:CreateLabel("Channel On YT:@OfficialPatozoid4", 1275974022, Color3.fromRGB(235,11,7), false)
CreditsTab:CreateButton({
    Name = "Copy Discord Link",
    Callback = function()
        local discordLink = "https://discord.gg/7dkp6uhYNb"
        pcall(function() setclipboard(discordLink) end)
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Discord",
            Text = "Link copiado para a área de transferência!",
            Duration = 3
        })
    end,
})
