local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "Mec BR - FREEHUB V1",
    LoadingTitle = "Carregando Menu",
    LoadingSubtitle = "by OfficialPatozoid",
    Theme = "Default",
    ToggleUIKeybind = "K",
    ConfigurationSaving = {
        Enabled = true,
        FileName = "MecBR_Hub"
    }
})

-- ==================== TABS ====================
local PlayerTab = Window:CreateTab("Player", 4483362458)
local TPTab = Window:CreateTab("🧿Teleports", 4483362458)
local VeiculoTab = Window:CreateTab("🚗Veiculos", 4483362458)
local UberTab = Window:CreateTab("🚕 Uber Farm", 4483362458)
local CreditsTab = Window:CreateTab("Credits", 4483362458)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local humanoid
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- ==================== PLAYER TAB ====================

local function onCharacterAdded(char)
    character = char
    humanoid = char:WaitForChild("Humanoid")
    humanoidRootPart = char:WaitForChild("HumanoidRootPart")
end
player.CharacterAdded:Connect(onCharacterAdded)

-- Infinite Jump
local infiniteJumpEnabled = false
PlayerTab:CreateToggle({
    Name = "Infinite Jump",
    CurrentValue = false,
    Callback = function(Value) infiniteJumpEnabled = Value end
})
UserInputService.JumpRequest:Connect(function()
    if infiniteJumpEnabled and humanoid then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-- WalkSpeed
PlayerTab:CreateSlider({
    Name = "WalkSpeed",
    Range = {16, 150},
    Increment = 1,
    Suffix = "WS",
    CurrentValue = 16,
    Callback = function(Value)
        if humanoid then humanoid.WalkSpeed = Value end
    end
})

-- JumpPower
PlayerTab:CreateSlider({
    Name = "JumpPower",
    Range = {0, 250},
    Increment = 1,
    Suffix = "JP",
    CurrentValue = 50,
    Callback = function(Value)
        if humanoid then humanoid.JumpPower = Value end
    end
})

-- Noclip
local noclipEnabled = false
PlayerTab:CreateToggle({
    Name = "Noclip",
    CurrentValue = false,
    Callback = function(Value) noclipEnabled = Value end
})
RunService.Stepped:Connect(function()
    if noclipEnabled and character then
        for _, part in pairs(character:GetChildren()) do
            if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                part.CanCollide = false
            end
        end
    end
end)

-- Fly
local flying = false
local flySpeed = 50
PlayerTab:CreateToggle({
    Name = "Fly",
    CurrentValue = false,
    Callback = function(Value)
        flying = Value
        if not flying and humanoidRootPart then
            humanoidRootPart.Velocity = Vector3.new(0,0,0)
        end
    end
})
PlayerTab:CreateSlider({
    Name = "Fly Speed",
    Range = {10, 300},
    Increment = 5,
    Suffix = "Studs/s",
    CurrentValue = 50,
    Callback = function(Value) flySpeed = Value end
})

-- Fly Controls
local keys = {W=false,A=false,S=false,D=false,Space=false,LeftShift=false}
UserInputService.InputBegan:Connect(function(i,gp) if gp then return end
    if i.KeyCode == Enum.KeyCode.W then keys.W = true end
    if i.KeyCode == Enum.KeyCode.A then keys.A = true end
    if i.KeyCode == Enum.KeyCode.S then keys.S = true end
    if i.KeyCode == Enum.KeyCode.D then keys.D = true end
    if i.KeyCode == Enum.KeyCode.Space then keys.Space = true end
    if i.KeyCode == Enum.KeyCode.LeftShift then keys.LeftShift = true end
end)
UserInputService.InputEnded:Connect(function(i)
    if i.KeyCode == Enum.KeyCode.W then keys.W = false end
    if i.KeyCode == Enum.KeyCode.A then keys.A = false end
    if i.KeyCode == Enum.KeyCode.S then keys.S = false end
    if i.KeyCode == Enum.KeyCode.D then keys.D = false end
    if i.KeyCode == Enum.KeyCode.Space then keys.Space = false end
    if i.KeyCode == Enum.KeyCode.LeftShift then keys.LeftShift = false end
end)

RunService.RenderStepped:Connect(function()
    if flying and humanoidRootPart then
        local dir = Vector3.new(0,0,0)
        local cam = workspace.CurrentCamera.CFrame
        if keys.W then dir += cam.LookVector end
        if keys.S then dir -= cam.LookVector end
        if keys.A then dir -= cam.RightVector end
        if keys.D then dir += cam.RightVector end
        if keys.Space then dir += Vector3.new(0,1,0) end
        if keys.LeftShift then dir -= Vector3.new(0,1,0) end
        humanoidRootPart.Velocity = dir.Unit * flySpeed
    end
end)

-- ==================== TELEPORTS ====================
local function TP(cf)
    if character and character:FindFirstChild("HumanoidRootPart") then
        character.HumanoidRootPart.CFrame = cf
    end
end

TPTab:CreateButton({Name = "🚗 FERRO-VELHO", Callback = function() TP(CFrame.new(-3418.64502, 148.252899, -458.328003)) end})
TPTab:CreateButton({Name = "🚙 Concessionaria", Callback = function() TP(CFrame.new(-3167.74683, 147.764023, -75.9954834)) end})
TPTab:CreateButton({Name = "🔩 Auto-Peças", Callback = function() TP(CFrame.new(-3571.43774, 149.245956, 377.565308)) end})
TPTab:CreateButton({Name = "🚧 Garagem", Callback = function() TP(CFrame.new(-3556.26294, 148.45932, 907.711731)) end})
TPTab:CreateButton({Name = "Dinametros", Callback = function() TP(CFrame.new(-4079.77246, 160.567535, 1490.77039)) end})

-- ==================== AUTO UBER FARM (TELEPORTE) ====================
local autoUberEnabled = false
local uberConn = nil

local function getDestination()
    for _, v in ipairs(workspace:GetDescendants()) do
        local n = v.Name:lower()
        if n:find("destination") or n:find("destino") or n:find("marker") or n:find("target") or n:find("goal") then
            if v:IsA("BasePart") then return v
            elseif v:IsA("Model") and v.PrimaryPart then return v.PrimaryPart end
        end
    end
    return nil
end

local function teleportToDestination()
    local dest = getDestination()
    if not dest then return end

    local char = player.Character
    if not char then return end

    -- Tenta encontrar o VehicleSeat
    local seat = char:FindFirstChildWhichIsA("VehicleSeat")
    local root = seat or char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local targetCFrame = dest.CFrame * CFrame.new(0, 6, 0) -- 6 studs acima

    pcall(function()
        root.CFrame = targetCFrame
        if root.Velocity then root.Velocity = Vector3.new(0,0,0) end
    end)
end

UberTab:CreateToggle({
    Name = "🚕 Auto Farm Uber (Teleporte)",
    CurrentValue = false,
    Callback = function(Value)
        autoUberEnabled = Value
        if Value then
            print("🚕 Auto Uber Farm (Teleporte) ATIVADO")
            
            uberConn = RunService.Heartbeat:Connect(teleportToDestination)

            -- Auto aceitar corrida
            task.spawn(function()
                while autoUberEnabled do
                    for _, obj in ipairs(workspace:GetDescendants()) do
                        if obj:FindFirstChild("ProximityPrompt") then
                            local name = obj.Name:lower()
                            if name:find("uber") or name:find("taxi") or name:find("passageiro") then
                                pcall(function()
                                    fireproximityprompt(obj.ProximityPrompt, 0, true)
                                end)
                            end
                        end
                    end
                    task.wait(3)
                end
            end)
        else
            if uberConn then
                uberConn:Disconnect()
                uberConn = nil
            end
            print("🚕 Auto Uber Farm DESATIVADO")
        end
    end
})

UberTab:CreateLabel("💡 Deixe ligado enquanto estiver em uma corrida", Color3.fromRGB(255, 215, 0))
UberTab:CreateLabel("O carro vai teleportar até o destino automaticamente", Color3.fromRGB(255, 100, 100))

-- ==================== CREDITS ====================
CreditsTab:CreateLabel("HUB BY: OfficialPatozoid")
CreditsTab:CreateLabel("Auto Uber Farm (Teleporte) adicionado por Grok", nil, Color3.fromRGB(0, 255, 100))
CreditsTab:CreateButton({
    Name = "Copiar Discord",
    Callback = function()
        setclipboard("https://discord.gg/7dkp6uhYNb")
        game:GetService("StarterGui"):SetCore("SendNotification", {Title="Sucesso", Text="Link copiado!", Duration=3})
    end
})

print("✅ Mec BR Hub carregado com Auto Uber Teleporte!")
