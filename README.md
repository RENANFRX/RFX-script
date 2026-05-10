
-- ██████╗ ███████╗███╗   ██╗ █████╗ ███╗   ██╗    ███████╗██████╗ ██╗  ██╗
-- ██╔══██╗██╔════╝████╗  ██║██╔══██╗████╗  ██║    ██╔════╝██╔══██╗╚██╗██╔╝
-- ██████╔╝█████╗  ██╔██╗ ██║███████║██╔██╗ ██║    █████╗  ██████╔╝ ╚███╔╝ 
-- ██╔══██╗██╔══╝  ██║╚██╗██║██╔══██║██║╚██╗██║    ██╔══╝  ██╔══██╗ ██╔██╗ 
-- ██║  ██║███████╗██║ ╚████║██║  ██║██║ ╚████║    ██║     ██║  ██║██╔╝ ██╗
-- ╚═╝  ╚═╝╚══════╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝  ╚═══╝    ╚═╝     ╚═╝  ╚═╝╚═╝  ╚═╝
--            RENAN FRX | Scripts — Auto Farm Uber
--         Jogo: NEW CAR PHYSICS | Brazilian Mechanics

-- ============================================================
--  CONFIGURAÇÕES
-- ============================================================
local CONFIG = {
    FarmAtivo       = true,       -- Liga/desliga o auto farm
    Velocidade      = 16,         -- Velocidade de WalkSpeed temporária (para entrar no carro)
    TempoEspera     = 1.5,        -- Segundos entre cada ciclo
    TempoEntrada    = 3,          -- Segundos esperando cliente entrar
    Debug           = true,       -- Mostra logs no console
}

-- ============================================================
--  SERVIÇOS
-- ============================================================
local Players        = game:GetService("Players")
local RunService     = game:GetService("RunService")
local TweenService   = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local StarterGui     = game:GetService("StarterGui")

local LocalPlayer   = Players.LocalPlayer
local Character     = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
local Humanoid      = Character:WaitForChild("Humanoid")

-- ============================================================
--  NOTIFICAÇÃO
-- ============================================================
local function Notificar(titulo, mensagem, duracao)
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title    = titulo,
            Text     = mensagem,
            Duration = duracao or 5,
        })
    end)
end

local function Log(msg)
    if CONFIG.Debug then
        print("[RENAN FRX] " .. tostring(msg))
    end
end

-- ============================================================
--  GUI PRINCIPAL
-- ============================================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RenanFRX_GUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local ok, err = pcall(function()
    ScreenGui.Parent = game:GetService("CoreGui")
end)
if not ok then
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

-- Frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 280, 0, 320)
MainFrame.Position = UDim2.new(0.5, -140, 0.05, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 10)
UICorner.Parent = MainFrame

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 42)
TitleBar.BackgroundColor3 = Color3.fromRGB(30, 0, 60)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local UICornerTitle = Instance.new("UICorner")
UICornerTitle.CornerRadius = UDim.new(0, 10)
UICornerTitle.Parent = TitleBar

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, 0, 1, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "⚡ RENAN FRX | Scripts"
TitleLabel.TextColor3 = Color3.fromRGB(200, 80, 255)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 15
TitleLabel.Parent = TitleBar

-- Status Label
local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, -20, 0, 30)
StatusLabel.Position = UDim2.new(0, 10, 0, 50)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "🔴 Status: Desativado"
StatusLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
StatusLabel.Font = Enum.Font.GothamSemibold
StatusLabel.TextSize = 13
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left
StatusLabel.Parent = MainFrame

-- Ciclos Label
local CiclosLabel = Instance.new("TextLabel")
CiclosLabel.Size = UDim2.new(1, -20, 0, 25)
CiclosLabel.Position = UDim2.new(0, 10, 0, 82)
CiclosLabel.BackgroundTransparency = 1
CiclosLabel.Text = "🔄 Ciclos: 0"
CiclosLabel.TextColor3 = Color3.fromRGB(180, 180, 255)
CiclosLabel.Font = Enum.Font.Gotham
CiclosLabel.TextSize = 12
CiclosLabel.TextXAlignment = Enum.TextXAlignment.Left
CiclosLabel.Parent = MainFrame

-- Ganhos Label
local GanhosLabel = Instance.new("TextLabel")
GanhosLabel.Size = UDim2.new(1, -20, 0, 25)
GanhosLabel.Position = UDim2.new(0, 10, 0, 107)
GanhosLabel.BackgroundTransparency = 1
GanhosLabel.Text = "💰 Corridas: 0"
GanhosLabel.TextColor3 = Color3.fromRGB(80, 255, 150)
GanhosLabel.Font = Enum.Font.Gotham
GanhosLabel.TextSize = 12
GanhosLabel.TextXAlignment = Enum.TextXAlignment.Left
GanhosLabel.Parent = MainFrame

-- Separador
local Sep = Instance.new("Frame")
Sep.Size = UDim2.new(1, -20, 0, 1)
Sep.Position = UDim2.new(0, 10, 0, 140)
Sep.BackgroundColor3 = Color3.fromRGB(80, 0, 150)
Sep.BorderSizePixel = 0
Sep.Parent = MainFrame

-- Botão Toggle Farm
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(1, -20, 0, 40)
ToggleBtn.Position = UDim2.new(0, 10, 0, 150)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(40, 0, 80)
ToggleBtn.Text = "▶ INICIAR AUTO FARM"
ToggleBtn.TextColor3 = Color3.fromRGB(200, 80, 255)
ToggleBtn.Font = Enum.Font.GothamBold
ToggleBtn.TextSize = 13
ToggleBtn.BorderSizePixel = 0
ToggleBtn.Parent = MainFrame

local UICornerBtn = Instance.new("UICorner")
UICornerBtn.CornerRadius = UDim.new(0, 8)
UICornerBtn.Parent = ToggleBtn

-- Botão Teleporte Manual
local TpBtn = Instance.new("TextButton")
TpBtn.Size = UDim2.new(1, -20, 0, 36)
TpBtn.Position = UDim2.new(0, 10, 0, 200)
TpBtn.BackgroundColor3 = Color3.fromRGB(20, 40, 80)
TpBtn.Text = "📍 Teleportar ao Cliente"
TpBtn.TextColor3 = Color3.fromRGB(100, 180, 255)
TpBtn.Font = Enum.Font.GothamSemibold
TpBtn.TextSize = 12
TpBtn.BorderSizePixel = 0
TpBtn.Parent = MainFrame

local UICornerTp = Instance.new("UICorner")
UICornerTp.CornerRadius = UDim.new(0, 8)
UICornerTp.Parent = TpBtn

-- Botão Fechar
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(1, -20, 0, 36)
CloseBtn.Position = UDim2.new(0, 10, 0, 245)
CloseBtn.BackgroundColor3 = Color3.fromRGB(80, 10, 10)
CloseBtn.Text = "✖ Fechar GUI"
CloseBtn.TextColor3 = Color3.fromRGB(255, 80, 80)
CloseBtn.Font = Enum.Font.GothamSemibold
CloseBtn.TextSize = 12
CloseBtn.BorderSizePixel = 0
CloseBtn.Parent = MainFrame

local UICornerClose = Instance.new("UICorner")
UICornerClose.CornerRadius = UDim.new(0, 8)
UICornerClose.Parent = CloseBtn

-- Rodapé
local Footer = Instance.new("TextLabel")
Footer.Size = UDim2.new(1, 0, 0, 24)
Footer.Position = UDim2.new(0, 0, 1, -24)
Footer.BackgroundTransparency = 1
Footer.Text = "github/RenanFRX  •  v1.0"
Footer.TextColor3 = Color3.fromRGB(80, 80, 100)
Footer.Font = Enum.Font.Gotham
Footer.TextSize = 10
Footer.Parent = MainFrame

-- ============================================================
--  LÓGICA DO AUTO FARM
-- ============================================================
local farmAtivo   = false
local ciclos      = 0
local corridas    = 0
local farmThread  = nil

-- Atualiza o character caso respawn
LocalPlayer.CharacterAdded:Connect(function(char)
    Character        = char
    HumanoidRootPart = char:WaitForChild("HumanoidRootPart")
    Humanoid         = char:WaitForChild("Humanoid")
    Log("Character atualizado após respawn.")
end)

-- Função segura de teleporte
local function TeleportarPara(posicao, offset)
    offset = offset or Vector3.new(0, 3, 0)
    if HumanoidRootPart and posicao then
        HumanoidRootPart.CFrame = CFrame.new(posicao + offset)
        task.wait(0.2)
    end
end

-- Procura NPCs / clientes de uber no workspace
local function AcharCliente()
    local workspace = game:GetService("Workspace")

    -- Tenta encontrar NPCs com nomes comuns do jogo
    local nomesNPC = {
        "UberClient", "Cliente", "Passageiro", "NPC", "Passenger",
        "UberPassenger", "ClienteUber", "Customer"
    }

    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj ~= Character then
            for _, nome in ipairs(nomesNPC) do
                if string.find(string.lower(obj.Name), string.lower(nome)) then
                    local root = obj:FindFirstChild("HumanoidRootPart")
                        or obj:FindFirstChild("RootPart")
                        or obj.PrimaryPart
                    if root then
                        return obj, root.Position
                    end
                end
            end
        end
    end

    -- Fallback: procura qualquer Model com Humanoid (NPC)
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChildOfClass("Humanoid") and obj ~= Character then
            local root = obj:FindFirstChild("HumanoidRootPart")
                or obj.PrimaryPart
            if root then
                return obj, root.Position
            end
        end
    end

    return nil, nil
end

-- Procura o destino do cliente (waypoint/destino marcado no jogo)
local function AcharDestino()
    local workspace = game:GetService("Workspace")

    local nomesDestino = {
        "Destino", "Destination", "Waypoint", "UberDestino",
        "UberDestination", "DropOff", "DropPoint", "Meta"
    }

    for _, obj in ipairs(workspace:GetDescendants()) do
        local nLower = string.lower(obj.Name)
        for _, nd in ipairs(nomesDestino) do
            if string.find(nLower, string.lower(nd)) then
                if obj:IsA("BasePart") or obj:IsA("Model") then
                    local pos = obj:IsA("BasePart") and obj.Position
                        or (obj.PrimaryPart and obj.PrimaryPart.Position)
                    if pos then
                        return pos
                    end
                end
            end
        end
    end

    -- Fallback: gera destino aleatório próximo (caso não ache)
    if HumanoidRootPart then
        local pos = HumanoidRootPart.Position
        return pos + Vector3.new(math.random(-80, 80), 0, math.random(-80, 80))
    end

    return nil
end

-- Procura o carro do jogador
local function AcharCarro()
    local workspace = game:GetService("Workspace")
    local nomeJogador = string.lower(LocalPlayer.Name)

    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") then
            local nLower = string.lower(obj.Name)
            -- Procura modelos de carro associados ao jogador
            if string.find(nLower, "car") or string.find(nLower, "veiculo")
                or string.find(nLower, "carro") or string.find(nLower, "taxi")
                or string.find(nLower, "uber") then
                local seat = obj:FindFirstChildOfClass("VehicleSeat")
                    or obj:FindFirstChild("DriveSeat")
                if seat then
                    return obj, seat
                end
            end
        end
    end
    return nil, nil
end

-- Espera o cliente entrar no carro
local function EsperarClienteEntrar(assentoCliente, timeout)
    timeout = timeout or CONFIG.TempoEntrada
    local t = 0
    while t < timeout do
        task.wait(0.5)
        t = t + 0.5
        -- Verifica se algum humanoid está sentado no assento de passageiro
        if assentoCliente and assentoCliente.Occupant then
            return true
        end
    end
    return false
end

-- Ciclo principal do auto farm
local function CicloFarm()
    ciclos = ciclos + 1
    CiclosLabel.Text = "🔄 Ciclos: " .. ciclos
    Log("Iniciando ciclo #" .. ciclos)

    -- 1) Achar carro
    local carro, assento = AcharCarro()
    if not assento then
        Log("Carro não encontrado, aguardando...")
        task.wait(2)
        return
    end

    -- 2) Teleportar ao assento do carro (motorista)
    local posAssento = assento.WorldCFrame.Position
    TeleportarPara(posAssento, Vector3.new(0, 2, 0))
    Log("Teleportado ao carro.")
    task.wait(0.5)

    -- Sentar no carro
    pcall(function()
        assento:Sit(Humanoid)
    end)
    task.wait(0.8)

    -- 3) Achar cliente
    local cliente, posCliente = AcharCliente()
    if not posCliente then
        Log("Nenhum cliente encontrado, aguardando...")
        StatusLabel.Text = "🟡 Status: Procurando cliente..."
        StatusLabel.TextColor3 = Color3.fromRGB(255, 200, 0)
        task.wait(2)
        return
    end

    Log("Cliente encontrado em: " .. tostring(posCliente))
    StatusLabel.Text = "🟠 Status: Indo ao cliente..."
    StatusLabel.TextColor3 = Color3.fromRGB(255, 150, 0)

    -- 4) Teleportar carro até o cliente
    if carro and carro.PrimaryPart then
        carro:SetPrimaryPartCFrame(CFrame.new(posCliente + Vector3.new(0, 3, 5)))
        task.wait(0.5)
    else
        TeleportarPara(posCliente)
    end

    Log("Aguardando cliente entrar no carro...")
    StatusLabel.Text = "🟡 Status: Aguardando cliente..."
    StatusLabel.TextColor3 = Color3.fromRGB(255, 220, 0)

    -- Assento de passageiro
    local assentoPassageiro = carro and (
        carro:FindFirstChild("PassengerSeat")
        or carro:FindFirstChild("Seat")
        or carro:FindFirstChild("Passageiro")
    )

    -- Espera cliente entrar
    local clienteEntrou = EsperarClienteEntrar(assentoPassageiro, CONFIG.TempoEntrada)

    -- 5) Teleportar para o destino
    local posDestino = AcharDestino()
    if not posDestino then
        Log("Destino não encontrado, usando posição aleatória.")
        posDestino = HumanoidRootPart.Position + Vector3.new(
            math.random(-100, 100), 0, math.random(-100, 100)
        )
    end

    StatusLabel.Text = "🟢 Status: Indo ao destino..."
    StatusLabel.TextColor3 = Color3.fromRGB(80, 255, 150)
    Log("Teleportando ao destino: " .. tostring(posDestino))

    if carro and carro.PrimaryPart then
        carro:SetPrimaryPartCFrame(CFrame.new(posDestino + Vector3.new(0, 3, 0)))
    else
        TeleportarPara(posDestino)
    end

    task.wait(1)
    corridas = corridas + 1
    GanhosLabel.Text = "💰 Corridas: " .. corridas
    Log("Corrida #" .. corridas .. " concluída!")

    Notificar("RENAN FRX", "✅ Corrida " .. corridas .. " concluída!", 3)
    StatusLabel.Text = "✅ Status: Corrida concluída!"
    StatusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)

    task.wait(CONFIG.TempoEspera)
end

-- Loop principal sem travar
local function IniciarFarm()
    farmAtivo = true
    StatusLabel.Text = "🟢 Status: Farm Ativo"
    StatusLabel.TextColor3 = Color3.fromRGB(80, 255, 150)
    ToggleBtn.Text = "⏹ PARAR AUTO FARM"
    ToggleBtn.BackgroundColor3 = Color3.fromRGB(80, 0, 20)

    Notificar("RENAN FRX | Scripts", "🚀 Auto Farm Uber INICIADO!", 4)

    farmThread = task.spawn(function()
        while farmAtivo do
            local ok, err = pcall(CicloFarm)
            if not ok then
                Log("Erro no ciclo: " .. tostring(err))
                task.wait(2)
            end
            task.wait(0.1) -- yield para não travar
        end
    end)
end

local function PararFarm()
    farmAtivo = false
    if farmThread then
        task.cancel(farmThread)
        farmThread = nil
    end
    StatusLabel.Text = "🔴 Status: Desativado"
    StatusLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
    ToggleBtn.Text = "▶ INICIAR AUTO FARM"
    ToggleBtn.BackgroundColor3 = Color3.fromRGB(40, 0, 80)
    Notificar("RENAN FRX | Scripts", "⛔ Auto Farm Uber PARADO!", 3)
end

-- ============================================================
--  EVENTOS DOS BOTÕES
-- ============================================================
ToggleBtn.MouseButton1Click:Connect(function()
    if farmAtivo then
        PararFarm()
    else
        IniciarFarm()
    end
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

CloseBtn.MouseButton1Click:Connect(function()
    PararFarm()
    ScreenGui:Destroy()
end)

-- Hover effects
ToggleBtn.MouseEnter:Connect(function()
    TweenService:Create(ToggleBtn, TweenInfo.new(0.15), {
        BackgroundColor3 = Color3.fromRGB(90, 0, 160)
    }):Play()
end)
ToggleBtn.MouseLeave:Connect(function()
    TweenService:Create(ToggleBtn, TweenInfo.new(0.15), {
        BackgroundColor3 = farmAtivo and Color3.fromRGB(80, 0, 20) or Color3.fromRGB(40, 0, 80)
    }):Play()
end)

-- ============================================================
--  INICIALIZAÇÃO
-- ============================================================
Log("RENAN FRX | Scripts carregado com sucesso!")
Notificar("RENAN FRX | Scripts", "✅ Script carregado! Clique em INICIAR.", 5)
</parameter>
</invoke>
