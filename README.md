local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "Mec BR - FREEHUB V1",
    LoadingTitle = "Carregando Menu",
    LoadingSubtitle = "by OfficialPatozoid",
    Theme = "Default",
    ToggleUIKeybind = "K",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = nil,
        FileName = "MecBR_Hub"
    }
})

-- ==================== TABS ====================
local PlayerTab = Window:CreateTab("Player", 4483362458)
local TPTab = Window:CreateTab("🧿Teleports", 4483362458)
local VeiculoTab = Window:CreateTab("🚗Veiculos", 4483362458)
local UberTab = Window:CreateTab("🚕 Uber Farm", 4483362458)

-- ==================== PLAYER TAB ====================
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
PlayerTab:CreateToggle({
    Name = "Infinite Jump",
    CurrentValue = false,
    Callback = function(Value)
        infiniteJumpEnabled = Value
    end,
})

UserInputService.JumpRequest:Connect(function()
    if infiniteJumpEnabled and humanoid then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-- WalkSpeed
PlayerTab:CreateSlider({
    Name = "WalkSpeed",
    Range = {0, 150},
    Increment = 1,
    Suffix = "WS",
    CurrentValue = 16,
    Callback = function(Value)
        if humanoid then humanoid.WalkSpeed = Value end
    end,
})

-- JumpPower
local currentJumpPower = 50
PlayerTab:CreateSlider({
    Name = "JumpPower",
    Range = {0, 250},
    Increment = 1,
    Suffix = "JP",
    CurrentValue = 50,
    Callback = function(Value)
        currentJumpPower = Value
        if humanoid then humanoid.JumpPower = Value end
    end,
})

-- Noclip
local noclipEnabled = false
local character = player.Character or player.CharacterAdded:Wait()

PlayerTab:CreateToggle({
    Name = "Noclip",
    CurrentValue = false,
    Callback = function(Value)
        noclipEnabled = Value
    end,
})

RunService.Stepped:Connect(function()
    if noclipEnabled and character then
        for _, part in ipairs(character:GetChildren()) do
            if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                part.CanCollide = false
            end
        end
    end
end)

-- Fly
local flying = false
local flySpeed = 50
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

PlayerTab:CreateToggle({
    Name = "Fly",
    CurrentValue = false,
    Callback = function(Value)
        flying = Value
        if not flying and humanoidRootPart then
            humanoidRootPart.Velocity = Vector3.new(0,0,0)
        end
    end,
})

PlayerTab:CreateSlider({
    Name = "Fly Speed",
    Range = {10, 300},
    Increment = 5,
    Suffix = "Studs/s",
    CurrentValue = 50,
    Callback = function(Value)
        flySpeed = Value
    end,
})

-- Fly Controls
local keysPressed = {W=false, A=false, S=false, D=false, Space=false, LeftShift=false}

UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    local k = input.KeyCode
    if k == Enum.KeyCode.W then keysPressed.W = true end
    if k == Enum.KeyCode.A then keysPressed.A = true end
    if k == Enum.KeyCode.S then keysPressed.S = true end
    if k == Enum.KeyCode.D then keysPressed.D = true end
    if k == Enum.KeyCode.Space then keysPressed.Space = true end
    if k == Enum.KeyCode.LeftShift then keysPressed.LeftShift = true end
end)

UserInputService.InputEnded:Connect(function(input)
    local k = input.KeyCode
    if k == Enum.KeyCode.W then keysPressed.W = false end
    if k == Enum.KeyCode.A then keysPressed.A = false end
    if k == Enum.KeyCode.S then keysPressed.S = false end
    if k == Enum.KeyCode.D then keysPressed.D = false end
    if k == Enum.KeyCode.Space then keysPressed.Space = false end
    if k == Enum.KeyCode.LeftShift then keysPressed.LeftShift = false end
end)

RunService.RenderStepped:Connect(function()
    if flying and humanoidRootPart then
        local direction = Vector3.new(0,0,0)
        local cam = workspace.CurrentCamera

        if keysPressed.W then direction += cam.CFrame.LookVector end
        if keysPressed.S then direction -= cam.CFrame.LookVector end
        if keysPressed.A then direction -= cam.CFrame.RightVector end
        if keysPressed.D then direction += cam.CFrame.RightVector end
        if keysPressed.Space then direction += Vector3.new(0,1,0) end
        if keysPressed.LeftShift then direction -= Vector3.new(0,1,0) end

        if direction.Magnitude > 0 then
            humanoidRootPart.Velocity = direction.Unit * flySpeed
        else
            humanoidRootPart.Velocity = Vector3.new(0,0,0)
        end
    end
end)

-- ==================== TELEPORTS ====================
local function TP(pos)
    local char = player.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = pos
    end
end

TPTab:CreateButton({Name = "🚗 FERRO-VELHO", Callback = function() TP(CFrame.new(-3418.64502, 148.252899, -458.328003)) end})
TPTab:CreateButton({Name = "🚙 Concessionaria", Callback = function() TP(CFrame.new(-3167.74683, 147.764023, -75.9954834)) end})
TPTab:CreateButton({Name = "🔩 Auto-Peças", Callback = function() TP(CFrame.new(-3571.43774, 149.245956, 377.565308)) end})
TPTab:CreateButton({Name = "🚧 Garagem", Callback = function() TP(CFrame.new(-3556.26294, 148.45932, 907.711731)) end})
TPTab:CreateButton({Name = "Dinametros", Callback = function() TP(CFrame.new(-4079.77246, 160.567535, 1490.77039)) end})

-- ==================== AUTO UBER FARM ====================

local autoUberEnabled = false
local uberConnection = nil

local function findUberDestination()
    for _, v in ipairs(workspace:GetDescendants()) do
        if v.Name:lower():find("destination") or v.Name:lower():find("destino") or 
           v.Name:lower():find("marker") or v.Name:lower():find("target") then
            if v:IsA("BasePart") then
                return v
            end
        end
    end
    return nil
end

UberTab:CreateToggle({
    Name = "🚕 Ativar Auto Farm Uber",
    CurrentValue = false,
    Callback = function(Value)
        autoUberEnabled = Value
        
        if Value then
            print("🚕 Auto Uber Farm LIGADO")
            
            uberConnection = RunService.Heartbeat:Connect(function()
                if not autoUberEnabled then return end
                
                local char = player.Character
                if not char or not char:FindFirstChild("HumanoidRootPart") then return end
                
                local root = char.HumanoidRootPart
                local dest = findUberDestination()
                
                if dest then
                    local distance = (dest.Position - root.Position).Magnitude
                    
                    if distance > 25 then
                        local direction = (dest.Position - root.Position).Unit
                        root.Velocity = direction * 140
                        root.CFrame = CFrame.lookAt(root.Position, dest.Position)
                    end
                end
            end)
            
            -- Auto aceitar jobs
            task.spawn(function()
                while autoUberEnabled do
                    for _, v in ipairs(workspace:GetDescendants()) do
                        if v:FindFirstChild("ProximityPrompt") and 
                           (v.Name:lower():find("uber") or v.Name:lower():find("taxi")) then
                            pcall(function()
                                fireproximityprompt(v.ProximityPrompt)
                            end)
                        end
                    end
                    task.wait(4)
                end
            end)
        else
            if uberConnection then
                uberConnection:Disconnect()
                uberConnection = nil
            end
            print("🚕 Auto Uber Farm DESLIGADO")
        end
    end,
})

UberTab:CreateSlider({
    Name = "Velocidade do Uber",
    Range = {80, 200},
    Increment = 5,
    CurrentValue = 140,
    Suffix = "Studs/s",
    Callback = function(v) -- velocidade já está hardcoded, pode melhorar depois
    end,
})

UberTab:CreateLabel("Dica: Use com um carro bom e deixe o Auto Farm ligado", Color3.fromRGB(255, 215, 0))

-- ==================== CREDITS ====================
local CreditsTab = Window:CreateTab("Credits", 4483362458)
CreditsTab:CreateLabel("HUB BY: OfficialPatozoid")
CreditsTab:CreateLabel("Auto Uber Farm adicionado por Grok", nil, Color3.fromRGB(0, 255, 100))
CreditsTab:CreateButton({
    Name = "Copy Discord Link",
    Callback = function()
        setclipboard("https://discord.gg/7dkp6uhYNb")
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Discord",
            Text = "Link copiado!",
            Duration = 3
        })
    end,
})
