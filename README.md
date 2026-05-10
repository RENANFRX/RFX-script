
-- ╔══════════════════════════════════════════════════╗
-- ║         RENAN FRX | Scripts  v2.0               ║
-- ║   Auto Farm Uber — Brazilian Mechanics           ║
-- ╚══════════════════════════════════════════════════╝

-- ============================================================
--  SERVIÇOS
-- ============================================================
local Players          = game:GetService("Players")
local TweenService     = game:GetService("TweenService")
local StarterGui       = game:GetService("StarterGui")
local RunService       = game:GetService("RunService")

local LocalPlayer      = Players.LocalPlayer
local Character        = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
local Humanoid         = Character:WaitForChild("Humanoid")

-- ============================================================
--  CONFIGURAÇÕES
-- ============================================================
local CONFIG = {
    TempoEspera   = 1.5,
    TempoEntrada  = 3,
    Debug         = true,
}

local farmAtivo  = false
local ciclos     = 0
local corridas   = 0
local farmThread = nil
local minimizado = false

-- ============================================================
--  UTILITÁRIOS
-- ============================================================
local function Log(msg)
    if CONFIG.Debug then
        print("[RENAN FRX] " .. tostring(msg))
    end
end

local function Notificar(titulo, mensagem, duracao)
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title    = titulo,
            Text     = mensagem,
            Duration = duracao or 5,
        })
    end)
end

LocalPlayer.CharacterAdded:Connect(function(char)
    Character        = char
    HumanoidRootPart = char:WaitForChild("HumanoidRootPart")
    Humanoid         = char:WaitForChild("Humanoid")
end)

-- ============================================================
--  CRIAÇÃO DA GUI
-- ============================================================
-- Remove GUI antiga se existir
pcall(function()
    local old = game:GetService("CoreGui"):FindFirstChild("RenanFRX_GUI")
    if old then old:Destroy() end
end)

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name            = "RenanFRX_GUI"
ScreenGui.ResetOnSpawn    = false
ScreenGui.ZIndexBehavior  = Enum.ZIndexBehavior.Sibling
ScreenGui.DisplayOrder    = 999

local ok = pcall(function() ScreenGui.Parent = game:GetService("CoreGui") end)
if not ok then ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

-- ── FRAME PRINCIPAL ─────────────────────────────────────────
local MENU_H   = 310   -- altura expandida
local TITLE_H  = 44    -- altura minimizada

local MainFrame = Instance.new("Frame")
MainFrame.Name              = "MainFrame"
MainFrame.Size              = UDim2.new(0, 290, 0, MENU_H)
MainFrame.Position          = UDim2.new(0.5, -145, 0.04, 0)
MainFrame.BackgroundColor3  = Color3.fromRGB(12, 10, 20)
MainFrame.BorderSizePixel   = 0
MainFrame.Active             = true
MainFrame.Draggable          = true
MainFrame.ClipsDescendants  = true
MainFrame.Parent            = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

-- Borda roxa
local Stroke = Instance.new("UIStroke", MainFrame)
Stroke.Color     = Color3.fromRGB(140, 0, 255)
Stroke.Thickness = 1.5

-- ── BARRA DE TÍTULO ─────────────────────────────────────────
local TitleBar = Instance.new("Frame")
TitleBar.Name             = "TitleBar"
TitleBar.Size             = UDim2.new(1, 0, 0, TITLE_H)
TitleBar.BackgroundColor3 = Color3.fromRGB(28, 0, 56)
TitleBar.BorderSizePixel  = 0
TitleBar.ZIndex           = 10
TitleBar.Parent           = MainFrame

Instance.new("UICorner", TitleBar).CornerRadius = UDim.new(0, 12)

-- Ícone / título
local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size               = UDim2.new(1, -100, 1, 0)
TitleLabel.Position           = UDim2.new(0, 12, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text               = "⚡ RENAN FRX | Scripts"
TitleLabel.TextColor3         = Color3.fromRGB(200, 80, 255)
TitleLabel.Font               = Enum.Font.GothamBold
TitleLabel.TextSize           = 14
TitleLabel.TextXAlignment     = Enum.TextXAlignment.Left
TitleLabel.ZIndex             = 11
TitleLabel.Parent             = TitleBar

-- ── BOTÃO MINIMIZAR ─────────────────────────────────────────
local MinBtn = Instance.new("TextButton")
MinBtn.Name              = "MinBtn"
MinBtn.Size              = UDim2.new(0, 32, 0, 26)
MinBtn.Position          = UDim2.new(1, -72, 0.5, -13)
MinBtn.BackgroundColor3  = Color3.fromRGB(50, 0, 100)
MinBtn.Text              = "—"
MinBtn.TextColor3        = Color3.fromRGB(200, 150, 255)
MinBtn.Font              = Enum.Font.GothamBold
MinBtn.TextSize          = 16
MinBtn.BorderSizePixel   = 0
MinBtn.ZIndex            = 12
MinBtn.Parent            = TitleBar

Instance.new("UICorner", MinBtn).CornerRadius = UDim.new(0, 6)

-- ── BOTÃO FECHAR ────────────────────────────────────────────
local CloseBtn = Instance.new("TextButton")
CloseBtn.Name              = "CloseBtn"
CloseBtn.Size              = UDim2.new(0, 32, 0, 26)
CloseBtn.Position          = UDim2.new(1, -34, 0.5, -13)
CloseBtn.BackgroundColor3  = Color3.fromRGB(130, 0, 20)
CloseBtn.Text              = "✕"
CloseBtn.TextColor3        = Color3.fromRGB(255, 100, 100)
CloseBtn.Font              = Enum.Font.GothamBold
CloseBtn.TextSize          = 14
CloseBtn.BorderSizePixel   = 0
CloseBtn.ZIndex            = 12
CloseBtn.Parent            = TitleBar

Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)

-- ── CONTEÚDO (esconde ao minimizar) ─────────────────────────
local Content = Instance.new("Frame")
Content.Name                 = "Content"
Content.Size                 = UDim2.new(1, 0, 1, -TITLE_H)
Content.Position             = UDim2.new(0, 0, 0, TITLE_H)
Content.BackgroundTransparency = 1
Content.Parent               = MainFrame

local function MkLabel(parent, text, posY, color, size, font)
    local lb = Instance.new("TextLabel")
    lb.Size               = UDim2.new(1, -20, 0, 26)
    lb.Position           = UDim2.new(0, 10, 0, posY)
    lb.BackgroundTransparency = 1
    lb.Text               = text
    lb.TextColor3         = color or Color3.fromRGB(210, 210, 255)
    lb.Font               = font or Enum.Font.Gotham
    lb.TextSize           = size or 12
    lb.TextXAlignment     = Enum.TextXAlignment.Left
    lb.Parent             = parent
    return lb
end

local function MkButton(parent, text, posY, bgColor, txtColor)
    local btn = Instance.new("TextButton")
    btn.Size              = UDim2.new(1, -20, 0, 38)
    btn.Position          = UDim2.new(0, 10, 0, posY)
    btn.BackgroundColor3  = bgColor  or Color3.fromRGB(40, 0, 80)
    btn.Text              = text
    btn.TextColor3        = txtColor or Color3.fromRGB(200, 80, 255)
    btn.Font              = Enum.Font.GothamBold
    btn.TextSize          = 13
    btn.BorderSizePixel   = 0
    btn.Parent            = parent
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    return btn
end

-- Labels de info
local StatusLabel  = MkLabel(Content, "🔴 Status: Desativado",  6,  Color3.fromRGB(255,80,80),   13, Enum.Font.GothamSemibold)
local CiclosLabel  = MkLabel(Content, "🔄 Ciclos: 0",           34, Color3.fromRGB(180,150,255), 12)
local CoridasLabel = MkLabel(Content, "💰 Corridas: 0",         57, Color3.fromRGB(80,255,150),  12)

-- Separador
local Sep = Instance.new("Frame")
Sep.Size             = UDim2.new(1, -20, 0, 1)
Sep.Position         = UDim2.new(0, 10, 0, 88)
Sep.BackgroundColor3 = Color3.fromRGB(100, 0, 180)
Sep.BorderSizePixel  = 0
Sep.Parent           = Content

-- Botões de ação
local ToggleBtn = MkButton(Content, "▶  INICIAR AUTO FARM",   96,  Color3.fromRGB(40,0,80),   Color3.fromRGB(200,80,255))
local TpBtn     = MkButton(Content, "📍  Teleportar ao Cliente", 142, Color3.fromRGB(10,30,70),  Color3.fromRGB(100,180,255))

-- Segundo separador
local Sep2 = Instance.new("Frame")
Sep2.Size             = UDim2.new(1, -20, 0, 1)
Sep2.Position         = UDim2.new(0, 10, 0, 188)
Sep2.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
Sep2.BorderSizePixel  = 0
Sep2.Parent           = Content

-- Rodapé
local Footer = Instance.new("TextLabel")
Footer.Size               = UDim2.new(1, 0, 0, 22)
Footer.Position           = UDim2.new(0, 0, 0, 196)
Footer.BackgroundTransparency = 1
Footer.Text               = "RENAN FRX | Scripts  •  v2.0"
Footer.TextColor3         = Color3.fromRGB(70, 70, 90)
Footer.Font               = Enum.Font.Gotham
Footer.TextSize           = 10
Footer.Parent             = Content

-- ============================================================
--  MINIMIZE / CLOSE LOGIC
-- ============================================================
local tweenInfo = TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)

MinBtn.MouseButton1Click:Connect(function()
    minimizado = not minimizado
    if minimizado then
        -- Colapsa para só a TitleBar
        TweenService:Create(MainFrame, tweenInfo, {
            Size = UDim2.new(0, 290, 0, TITLE_H)
        }):Play()
        MinBtn.Text = "▢"  -- ícone expandir
        MinBtn.TextColor3 = Color3.fromRGB(255, 200, 80)
    else
        -- Expande de volta
        TweenService:Create(MainFrame, tweenInfo, {
            Size = UDim2.new(0, 290, 0, MENU_H)
        }):Play()
        MinBtn.Text = "—"
        MinBtn.TextColor3 = Color3.fromRGB(200, 150, 255)
    end
end)

CloseBtn.MouseButton1Click:Connect(function()
    -- Para o farm antes de fechar
    farmAtivo = false
    if farmThread then
        task.cancel(farmThread)
        farmThread = nil
    end
    -- Animação de saída
    TweenService:Create(MainFrame, TweenInfo.new(0.2), {
        Size     = UDim2.new(0, 0, 0, 0),
        Position = UDim2.new(MainFrame.Position.X.Scale, MainFrame.Position.X.Offset + 145,
                             MainFrame.Position.Y.Scale, MainFrame.Position.Y.Offset + MENU_H/2)
    }):Play()
    task.delay(0.25, function()
        ScreenGui:Destroy()
    end)
    Notificar("RENAN FRX | Scripts", "GUI fechada. Até mais! 👋", 3)
end)

-- Hover nos botões da TitleBar
for _, btn in ipairs({MinBtn, CloseBtn}) do
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.12), {
            BackgroundTransparency = 0.3
        }):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.12), {
            BackgroundTransparency = 0
        }):Play()
    end)
end

-- ============================================================
--  LÓGICA DO AUTO FARM
-- ============================================================
local function TeleportarPara(pos, offset)
    offset = offset or Vector3.new(0, 3, 0)
    if HumanoidRootPart and pos then
        HumanoidRootPart.CFrame = CFrame.new(pos + offset)
        task.wait(0.2)
    end
end

local function AcharCliente()
    local nomesNPC = {"uberclient","cliente","passageiro","npc","passenger",
                       "uberpassenger","clienteuber","customer","person"}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj ~= Character then
            local nLow = string.lower(obj.Name)
            for _, n in ipairs(nomesNPC) do
                if string.find(nLow, n) then
                    local root = obj:FindFirstChild("HumanoidRootPart")
                        or obj:FindFirstChild("RootPart") or obj.PrimaryPart
                    if root then return obj, root.Position end
                end
            end
        end
    end
    -- fallback: qualquer humanoid
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChildOfClass("Humanoid") and obj ~= Character then
            local root = obj:FindFirstChild("HumanoidRootPart") or obj.PrimaryPart
            if root then return obj, root.Position end
        end
    end
    return nil, nil
end

local function AcharDestino()
    local nomesDestino = {"destino","destination","waypoint","uberdestino",
                           "uberdestination","dropoff","droppoint","meta"}
    for _, obj in ipairs(workspace:GetDescendants()) do
        local nLow = string.lower(obj.Name)
        for _, nd in ipairs(nomesDestino) do
            if string.find(nLow, nd) then
                if obj:IsA("BasePart") then
                    return obj.Position
                elseif obj:IsA("Model") and obj.PrimaryPart then
                    return obj.PrimaryPart.Position
                end
            end
        end
    end
    -- fallback aleatório
    if HumanoidRootPart then
        local p = HumanoidRootPart.Position
        return p + Vector3.new(math.random(-100,100), 0, math.random(-100,100))
    end
end

local function AcharCarro()
    local keywords = {"car","carro","veiculo","taxi","uber","vehicle","auto"}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") then
            local nLow = string.lower(obj.Name)
            for _, kw in ipairs(keywords) do
                if string.find(nLow, kw) then
                    local seat = obj:FindFirstChildOfClass("VehicleSeat")
                        or obj:FindFirstChild("DriveSeat")
                    if seat then return obj, seat end
                end
            end
        end
    end
    return nil, nil
end

local function EsperarClienteEntrar(assento, timeout)
    local t = 0
    while t < (timeout or CONFIG.TempoEntrada) do
        task.wait(0.4)
        t = t + 0.4
        if assento and assento.Occupant then return true end
    end
    return false
end

local function AtualizarStatus(texto, cor)
    StatusLabel.Text       = texto
    StatusLabel.TextColor3 = cor or Color3.fromRGB(210,210,255)
end

local function CicloFarm()
    ciclos = ciclos + 1
    CiclosLabel.Text = "🔄 Ciclos: " .. ciclos
    Log("Ciclo #" .. ciclos)

    local carro, assento = AcharCarro()
    if not assento then
        AtualizarStatus("🟡 Status: Procurando carro...", Color3.fromRGB(255,220,0))
        task.wait(2)
        return
    end

    -- Teleporta ao carro
    local posAssento = assento.WorldCFrame.Position
    TeleportarPara(posAssento, Vector3.new(0, 2, 0))
    task.wait(0.4)
    pcall(function() assento:Sit(Humanoid) end)
    task.wait(0.6)

    -- Procura cliente
    local _, posCliente = AcharCliente()
    if not posCliente then
        AtualizarStatus("🟡 Status: Procurando cliente...", Color3.fromRGB(255,200,0))
        task.wait(2)
        return
    end

    AtualizarStatus("🟠 Status: Indo ao cliente...", Color3.fromRGB(255,150,0))
    Log("Teleportando ao cliente: " .. tostring(posCliente))

    -- Move carro ao cliente
    if carro and carro.PrimaryPart then
        carro:SetPrimaryPartCFrame(CFrame.new(posCliente + Vector3.new(0, 3, 5)))
    else
        TeleportarPara(posCliente)
    end
    task.wait(0.5)

    AtualizarStatus("🟡 Status: Aguardando cliente entrar...", Color3.fromRGB(255,220,0))
    local assentoPass = carro and (
        carro:FindFirstChild("PassengerSeat")
        or carro:FindFirstChild("Seat")
        or carro:FindFirstChild("Passageiro")
    )
    EsperarClienteEntrar(assentoPass, CONFIG.TempoEntrada)

    -- Destino
    local posDestino = AcharDestino()
    AtualizarStatus("🚗 Status: Levando ao destino...", Color3.fromRGB(80,200,255))
    Log("Teleportando ao destino: " .. tostring(posDestino))

    if carro and carro.PrimaryPart then
        carro:SetPrimaryPartCFrame(CFrame.new(posDestino + Vector3.new(0, 3, 0)))
    else
        TeleportarPara(posDestino)
    end

    task.wait(0.8)
    corridas = corridas + 1
    CoridasLabel.Text = "💰 Corridas: " .. corridas
    AtualizarStatus("✅ Status: Corrida " .. corridas .. " concluída!", Color3.fromRGB(80,255,150))
    Notificar("RENAN FRX", "✅ Corrida #" .. corridas .. " finalizada!", 3)

    task.wait(CONFIG.TempoEspera)
end

-- ============================================================
--  TOGGLE FARM
-- ============================================================
local function IniciarFarm()
    farmAtivo = true
    AtualizarStatus("🟢 Status: Farm Ativo", Color3.fromRGB(80,255,150))
    ToggleBtn.Text             = "⏹  PARAR AUTO FARM"
    ToggleBtn.BackgroundColor3 = Color3.fromRGB(80, 0, 20)
    ToggleBtn.TextColor3       = Color3.fromRGB(255, 80, 80)
    Notificar("RENAN FRX | Scripts", "🚀 Auto Farm Uber INICIADO!", 4)

    farmThread = task.spawn(function()
        while farmAtivo do
            local s, e = pcall(CicloFarm)
            if not s then
                Log("Erro: " .. tostring(e))
                task.wait(2)
            end
            task.wait(0.05)
        end
    end)
end

local function PararFarm()
    farmAtivo = false
    if farmThread then
        task.cancel(farmThread)
        farmThread = nil
    end
    AtualizarStatus("🔴 Status: Desativado", Color3.fromRGB(255,80,80))
    ToggleBtn.Text             = "▶  INICIAR AUTO FARM"
    ToggleBtn.BackgroundColor3 = Color3.fromRGB(40, 0, 80)
    ToggleBtn.TextColor3       = Color3.fromRGB(200, 80, 255)
    Notificar("RENAN FRX | Scripts", "⛔ Auto Farm PARADO!", 3)
end

ToggleBtn.MouseButton1Click:Connect(function()
    if farmAtivo then PararFarm() else IniciarFarm() end
end)

TpBtn.MouseButton1Click:Connect(function()
    local _, posCliente = AcharCliente()
    if posCliente then
        TeleportarPara(posCliente)
        Notificar("RENAN FRX", "📍 Teleportado ao cliente!", 3)
    else
        Notificar("RENAN FRX", "❌ Nenhum cliente encontrado!", 3)
    end
end)

-- Hover nos botões de ação
for _, btn in ipairs({ToggleBtn, TpBtn}) do
    local origColor = btn.BackgroundColor3
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.12), {
            BackgroundColor3 = Color3.new(
                origColor.R + 0.08,
                origColor.G + 0.03,
                origColor.B + 0.12
            )
        }):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.12), {
            BackgroundColor3 = origColor
        }):Play()
    end)
end

-- ============================================================
--  ANIMAÇÃO DE ABERTURA
-- ============================================================
MainFrame.Size = UDim2.new(0, 0, 0, 0)
MainFrame.Position = UDim2.new(0.5, 0, 0.04, MENU_H/2)
TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
    Size     = UDim2.new(0, 290, 0, MENU_H),
    Position = UDim2.new(0.5, -145, 0.04, 0)
}):Play()

Log("RENAN FRX | Scripts v2.0 carregado!")
task.delay(0.4, function()
    Notificar("RENAN FRX | Scripts", "✅ Menu carregado! Clique em INICIAR.", 5)
end)
