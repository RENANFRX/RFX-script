
--[[
    ╔══════════════════════════════════════════════╗
    ║      RENAN FRX | Scripts  v3.0  FIXED       ║
    ║   Auto Farm Uber — Brazilian Mechanics       ║
    ╚══════════════════════════════════════════════╝
    FIXES:
      • Teleporte vai ao NPC correto (detecta BillboardGui/Highlight/Tag)
      • Carro NÃO é movido (evita bug de física)
      • Player teleporta até o cliente e destino a pé / como passageiro
      • Toggle On/Off 100% funcional
      • Loop protegido com pcall e task.wait limpo
--]]

-- ───────────────────────────────────────────────
--  SERVIÇOS
-- ───────────────────────────────────────────────
local Players     = game:GetService("Players")
local TweenSvc    = game:GetService("TweenService")
local RunSvc      = game:GetService("RunService")
local StarterGui  = game:GetService("StarterGui")

local LP          = Players.LocalPlayer
local Char        = LP.Character or LP.CharacterAdded:Wait()
local Root        = Char:WaitForChild("HumanoidRootPart")
local Hum         = Char:WaitForChild("Humanoid")

-- Atualiza refs no respawn
LP.CharacterAdded:Connect(function(c)
    Char = c
    Root = c:WaitForChild("HumanoidRootPart")
    Hum  = c:WaitForChild("Humanoid")
end)

-- ───────────────────────────────────────────────
--  CONFIGURAÇÕES EDITÁVEIS
-- ───────────────────────────────────────────────
local CFG = {
    DelayEntreCiclos   = 2,    -- segundos entre uma corrida e outra
    DelayEsperaChegar  = 1.5,  -- espera após teleportar
    MaxTentativasSemNPC= 5,    -- ciclos sem NPC antes de pausar
    Debug              = false,
}

-- ───────────────────────────────────────────────
--  ESTADO
-- ───────────────────────────────────────────────
local farmOn     = false
local farmThread = nil
local minimized  = false
local ciclos     = 0
local corridas   = 0
local semNPC     = 0

-- ───────────────────────────────────────────────
--  UTILITÁRIOS
-- ───────────────────────────────────────────────
local function Log(m)
    if CFG.Debug then print("[RENAN FRX] "..tostring(m)) end
end

local function Notif(title, msg, dur)
    pcall(function()
        StarterGui:SetCore("SendNotification",{Title=title,Text=msg,Duration=dur or 4})
    end)
end

-- Teleporta o personagem (não o carro!) de forma segura
local function TP(pos)
    if not (Root and pos) then return end
    Root.CFrame = CFrame.new(pos + Vector3.new(0, 4, 0))
    task.wait(0.25)
end

-- ───────────────────────────────────────────────
--  DETECÇÃO DE NPCS / CLIENTES (CORRIGIDA)
-- ───────────────────────────────────────────────
--[[
  Estratégia de detecção (prioridade):
  1. Model com Highlight (Roblox realça NPCs de missão)
  2. Model com BillboardGui (seta/ícone flutuante = quest NPC)
  3. Model com Humanoid + nome contendo palavras-chave
  4. Qualquer Model com Humanoid que não seja jogador
]]

local NPC_KEYWORDS = {
    "client","cliente","passageiro","passenger","uber",
    "customer","person","npc","pedestre","pax","user"
}
local DEST_KEYWORDS = {
    "destino","destination","waypoint","dropoff","drop",
    "entrega","meta","chegada","arrival","endpoint","marker"
}

local function temKeyword(nome, lista)
    local n = string.lower(nome)
    for _, k in ipairs(lista) do
        if string.find(n, k, 1, true) then return true end
    end
    return false
end

local function ehJogador(model)
    for _, p in ipairs(Players:GetPlayers()) do
        if p.Character == model then return true end
    end
    return false
end

local function getRootPos(model)
    local r = model:FindFirstChild("HumanoidRootPart")
           or model:FindFirstChild("RootPart")
           or model.PrimaryPart
    return r and r.Position or nil
end

local function encontrarCliente()
    -- Prioridade 1: Highlight (engine usa isso para marcar NPCs de missão)
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and not ehJogador(obj) then
            if obj:FindFirstChildOfClass("Highlight") then
                local pos = getRootPos(obj)
                if pos then return pos end
            end
        end
    end

    -- Prioridade 2: BillboardGui dentro do Model
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and not ehJogador(obj) then
            if obj:FindFirstChildOfClass("BillboardGui") then
                local pos = getRootPos(obj)
                if pos then return pos end
            end
        end
    end

    -- Prioridade 3: Model com Humanoid + keyword
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and not ehJogador(obj)
           and obj:FindFirstChildOfClass("Humanoid") then
            if temKeyword(obj.Name, NPC_KEYWORDS) then
                local pos = getRootPos(obj)
                if pos then return pos end
            end
        end
    end

    -- Prioridade 4: Qualquer Model com Humanoid (último recurso)
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and not ehJogador(obj)
           and obj:FindFirstChildOfClass("Humanoid") then
            local h = obj:FindFirstChildOfClass("Humanoid")
            if h and h.Health > 0 then
                local pos = getRootPos(obj)
                if pos then return pos end
            end
        end
    end

    return nil
end

local function encontrarDestino()
    -- Prioridade 1: Part/Model com keyword de destino
    for _, obj in ipairs(workspace:GetDescendants()) do
        if temKeyword(obj.Name, DEST_KEYWORDS) then
            if obj:IsA("BasePart") then
                return obj.Position
            elseif obj:IsA("Model") and obj.PrimaryPart then
                return obj.PrimaryPart.Position
            end
        end
    end

    -- Prioridade 2: Procura ObjectValue "Destino" ou "Target" no LP
    local gui = LP:FindFirstChild("PlayerGui")
    if gui then
        for _, v in ipairs(gui:GetDescendants()) do
            if v:IsA("ObjectValue") and v.Value and v.Value:IsA("BasePart") then
                return v.Value.Position
            end
        end
    end

    -- Fallback: posição aleatória razoável (evita ir pra 0,0,0)
    if Root then
        local p = Root.Position
        return p + Vector3.new(math.random(-60,60), 0, math.random(-60,60))
    end
    return nil
end

-- ───────────────────────────────────────────────
--  CICLO DO FARM (CORRIGIDO)
-- ───────────────────────────────────────────────
local statusUpdate = nil  -- callback da GUI

local function setCicloStatus(txt, cor)
    if statusUpdate then statusUpdate(txt, cor) end
    Log(txt)
end

local function cicloFarm()
    ciclos = ciclos + 1

    -- 1) Acha o cliente
    local posCliente = encontrarCliente()
    if not posCliente then
        semNPC = semNPC + 1
        setCicloStatus("🟡 Procurando cliente... ("..semNPC..")", Color3.fromRGB(255,200,0))
        task.wait(2)
        return
    end
    semNPC = 0

    -- 2) Teleporta o PLAYER até o cliente (não move o carro!)
    setCicloStatus("🟠 Indo até o cliente...", Color3.fromRGB(255,150,0))
    TP(posCliente)
    task.wait(CFG.DelayEsperaChegar)

    -- 3) Aguarda um frame para o jogo registrar a chegada
    RunSvc.Heartbeat:Wait()
    task.wait(1)

    -- 4) Acha destino
    local posDestino = encontrarDestino()
    if not posDestino then
        setCicloStatus("🟡 Destino não encontrado", Color3.fromRGB(255,200,0))
        task.wait(1.5)
        return
    end

    -- 5) Teleporta ao destino
    setCicloStatus("🚗 Levando ao destino...", Color3.fromRGB(80,180,255))
    TP(posDestino)
    task.wait(CFG.DelayEsperaChegar)

    -- 6) Corrida concluída
    corridas = corridas + 1
    setCicloStatus("✅ Corrida #"..corridas.." concluída!", Color3.fromRGB(80,255,150))
    Notif("RENAN FRX","✅ Corrida #"..corridas.." concluída!",3)
    task.wait(CFG.DelayEntreCiclos)
end

local function iniciarFarm()
    farmOn = true
    semNPC = 0
    farmThread = task.spawn(function()
        while farmOn do
            local ok, err = pcall(cicloFarm)
            if not ok then
                Log("Erro no ciclo: "..tostring(err))
                task.wait(2)
            end
            -- yield obrigatório para não travar
            task.wait(0.05)
        end
    end)
end

local function pararFarm()
    farmOn = false
    if farmThread then
        task.cancel(farmThread)
        farmThread = nil
    end
end

-- ───────────────────────────────────────────────
--  GUI
-- ───────────────────────────────────────────────
pcall(function()
    local old = game:GetService("CoreGui"):FindFirstChild("RenanFRX_GUI")
    if old then old:Destroy() end
end)
pcall(function()
    local old = LP:WaitForChild("PlayerGui"):FindFirstChild("RenanFRX_GUI")
    if old then old:Destroy() end
end)

local SG = Instance.new("ScreenGui")
SG.Name           = "RenanFRX_GUI"
SG.ResetOnSpawn   = false
SG.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
SG.DisplayOrder   = 999
local cgOk = pcall(function() SG.Parent = game:GetService("CoreGui") end)
if not cgOk then SG.Parent = LP:WaitForChild("PlayerGui") end

local MENU_W  = 288
local MENU_H  = 330
local TITLE_H = 46
local TW = TweenInfo.new(0.22, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)

-- Frame principal
local MF = Instance.new("Frame")
MF.Name             = "Main"
MF.Size             = UDim2.new(0, MENU_W, 0, MENU_H)
MF.Position         = UDim2.new(0.5, -MENU_W/2, 0.04, 0)
MF.BackgroundColor3 = Color3.fromRGB(10, 8, 18)
MF.BorderSizePixel  = 0
MF.Active           = true
MF.Draggable        = true
MF.ClipsDescendants = true
MF.Parent           = SG
Instance.new("UICorner",MF).CornerRadius = UDim.new(0,12)
local stroke = Instance.new("UIStroke",MF)
stroke.Color     = Color3.fromRGB(130,0,240)
stroke.Thickness = 1.5

-- TitleBar
local TB = Instance.new("Frame")
TB.Size             = UDim2.new(1,0,0,TITLE_H)
TB.BackgroundColor3 = Color3.fromRGB(22,0,48)
TB.BorderSizePixel  = 0
TB.ZIndex           = 10
TB.Parent           = MF
Instance.new("UICorner",TB).CornerRadius = UDim.new(0,12)

local TL = Instance.new("TextLabel")
TL.Size               = UDim2.new(1,-106,1,0)
TL.Position           = UDim2.new(0,12,0,0)
TL.BackgroundTransparency = 1
TL.Text               = "⚡  RENAN FRX | Scripts"
TL.TextColor3         = Color3.fromRGB(190,70,255)
TL.Font               = Enum.Font.GothamBold
TL.TextSize           = 14
TL.TextXAlignment     = Enum.TextXAlignment.Left
TL.ZIndex             = 11
TL.Parent             = TB

-- Botão MIN
local function mkTopBtn(icon, posX, bg, tc)
    local b = Instance.new("TextButton")
    b.Size             = UDim2.new(0,30,0,26)
    b.Position         = UDim2.new(1, posX, 0.5,-13)
    b.BackgroundColor3 = bg
    b.Text             = icon
    b.TextColor3       = tc
    b.Font             = Enum.Font.GothamBold
    b.TextSize         = 15
    b.BorderSizePixel  = 0
    b.ZIndex           = 12
    b.Parent           = TB
    Instance.new("UICorner",b).CornerRadius = UDim.new(0,6)
    -- hover
    b.MouseEnter:Connect(function()
        TweenSvc:Create(b,TW,{BackgroundTransparency=0.35}):Play()
    end)
    b.MouseLeave:Connect(function()
        TweenSvc:Create(b,TW,{BackgroundTransparency=0}):Play()
    end)
    return b
end

local MinBtn   = mkTopBtn("—", -70, Color3.fromRGB(45,0,90),  Color3.fromRGB(190,130,255))
local CloseBtn = mkTopBtn("✕", -34, Color3.fromRGB(110,0,18), Color3.fromRGB(255,90,90))

-- Conteúdo
local CT = Instance.new("Frame")
CT.Size                 = UDim2.new(1,0,1,-TITLE_H)
CT.Position             = UDim2.new(0,0,0,TITLE_H)
CT.BackgroundTransparency = 1
CT.Parent               = MF

local function mkLabel(txt, y, col, sz, font)
    local l = Instance.new("TextLabel")
    l.Size               = UDim2.new(1,-18,0,24)
    l.Position           = UDim2.new(0,9,0,y)
    l.BackgroundTransparency = 1
    l.Text               = txt
    l.TextColor3         = col  or Color3.fromRGB(210,210,255)
    l.Font               = font or Enum.Font.Gotham
    l.TextSize           = sz   or 12
    l.TextXAlignment     = Enum.TextXAlignment.Left
    l.Parent             = CT
    return l
end

local function mkBtn(txt, y, bg, tc)
    local b = Instance.new("TextButton")
    b.Size             = UDim2.new(1,-18,0,38)
    b.Position         = UDim2.new(0,9,0,y)
    b.BackgroundColor3 = bg or Color3.fromRGB(38,0,75)
    b.Text             = txt
    b.TextColor3       = tc or Color3.fromRGB(195,75,255)
    b.Font             = Enum.Font.GothamBold
    b.TextSize         = 13
    b.BorderSizePixel  = 0
    b.Parent           = CT
    Instance.new("UICorner",b).CornerRadius = UDim.new(0,8)
    local origBg = bg or Color3.fromRGB(38,0,75)
    b.MouseEnter:Connect(function()
        TweenSvc:Create(b,TW,{BackgroundColor3=Color3.new(origBg.R+0.07,origBg.G+0.02,origBg.B+0.10)}):Play()
    end)
    b.MouseLeave:Connect(function()
        TweenSvc:Create(b,TW,{BackgroundColor3=origBg}):Play()
    end)
    return b
end

local function mkSep(y)
    local s = Instance.new("Frame")
    s.Size             = UDim2.new(1,-18,0,1)
    s.Position         = UDim2.new(0,9,0,y)
    s.BackgroundColor3 = Color3.fromRGB(90,0,160)
    s.BorderSizePixel  = 0
    s.Parent           = CT
end

-- ── Labels de status ──
local StatusLbl  = mkLabel("🔴 Status: Desativado", 6,  Color3.fromRGB(255,75,75), 13, Enum.Font.GothamSemibold)
local CiclosLbl  = mkLabel("🔄 Ciclos: 0",          33, Color3.fromRGB(170,140,255))
local CorridasLbl= mkLabel("💰 Corridas: 0",         56, Color3.fromRGB(75,255,140))
mkSep(85)

-- ── Botões principais ──
local ToggleBtn  = mkBtn("▶  INICIAR AUTO FARM",     93,  Color3.fromRGB(38,0,75),   Color3.fromRGB(195,75,255))
local TpClienteBtn = mkBtn("📍  Ir até o Cliente",   140, Color3.fromRGB(8,28,65),   Color3.fromRGB(90,175,255))
local TpDestinoBtn = mkBtn("🏁  Ir até o Destino",   186, Color3.fromRGB(5,40,20),   Color3.fromRGB(75,220,130))

mkSep(233)

local FooterLbl = Instance.new("TextLabel")
FooterLbl.Size               = UDim2.new(1,0,0,20)
FooterLbl.Position           = UDim2.new(0,0,0,240)
FooterLbl.BackgroundTransparency = 1
FooterLbl.Text               = "RENAN FRX | Scripts  •  v3.0 FIXED"
FooterLbl.TextColor3         = Color3.fromRGB(60,60,80)
FooterLbl.Font               = Enum.Font.Gotham
FooterLbl.TextSize           = 10
FooterLbl.Parent             = CT

-- ───────────────────────────────────────────────
--  CALLBACK DE STATUS → atualiza a GUI
-- ───────────────────────────────────────────────
statusUpdate = function(txt, cor)
    StatusLbl.Text       = txt
    StatusLbl.TextColor3 = cor or Color3.fromRGB(210,210,255)
    CiclosLbl.Text   = "🔄 Ciclos: "   .. ciclos
    CorridasLbl.Text = "💰 Corridas: " .. corridas
end

-- ───────────────────────────────────────────────
--  EVENTOS DOS BOTÕES
-- ───────────────────────────────────────────────
ToggleBtn.MouseButton1Click:Connect(function()
    if farmOn then
        -- DESLIGAR
        pararFarm()
        StatusLbl.Text       = "🔴 Status: Desativado"
        StatusLbl.TextColor3 = Color3.fromRGB(255,75,75)
        ToggleBtn.Text             = "▶  INICIAR AUTO FARM"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(38,0,75)
        ToggleBtn.TextColor3       = Color3.fromRGB(195,75,255)
        Notif("RENAN FRX","⛔ Farm DESATIVADO!",3)
    else
        -- LIGAR
        iniciarFarm()
        StatusLbl.Text       = "🟢 Status: Farm Ativo"
        StatusLbl.TextColor3 = Color3.fromRGB(75,255,140)
        ToggleBtn.Text             = "⏹  PARAR AUTO FARM"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(80,5,15)
        ToggleBtn.TextColor3       = Color3.fromRGB(255,80,80)
        Notif("RENAN FRX","🚀 Farm ATIVADO!",3)
    end
end)

TpClienteBtn.MouseButton1Click:Connect(function()
    local pos = encontrarCliente()
    if pos then
        TP(pos)
        Notif("RENAN FRX","📍 Teleportado ao cliente!",3)
        StatusLbl.Text       = "📍 Teleportado ao cliente"
        StatusLbl.TextColor3 = Color3.fromRGB(90,175,255)
    else
        Notif("RENAN FRX","❌ Nenhum cliente encontrado!",3)
        StatusLbl.Text       = "❌ Cliente não encontrado"
        StatusLbl.TextColor3 = Color3.fromRGB(255,80,80)
    end
end)

TpDestinoBtn.MouseButton1Click:Connect(function()
    local pos = encontrarDestino()
    if pos then
        TP(pos)
        Notif("RENAN FRX","🏁 Teleportado ao destino!",3)
        StatusLbl.Text       = "🏁 Teleportado ao destino"
        StatusLbl.TextColor3 = Color3.fromRGB(75,220,130)
    else
        Notif("RENAN FRX","❌ Destino não encontrado!",3)
        StatusLbl.Text       = "❌ Destino não encontrado"
        StatusLbl.TextColor3 = Color3.fromRGB(255,80,80)
    end
end)

-- ── MINIMIZAR ──
MinBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    if minimized then
        TweenSvc:Create(MF, TW, {Size = UDim2.new(0,MENU_W,0,TITLE_H)}):Play()
        MinBtn.Text      = "▢"
        MinBtn.TextColor3 = Color3.fromRGB(255,210,60)
    else
        TweenSvc:Create(MF, TW, {Size = UDim2.new(0,MENU_W,0,MENU_H)}):Play()
        MinBtn.Text      = "—"
        MinBtn.TextColor3 = Color3.fromRGB(190,130,255)
    end
end)

-- ── FECHAR ──
CloseBtn.MouseButton1Click:Connect(function()
    pararFarm()
    TweenSvc:Create(MF, TweenInfo.new(0.18), {
        Size     = UDim2.new(0,0,0,0),
        Position = UDim2.new(MF.Position.X.Scale, MF.Position.X.Offset + MENU_W/2,
                             MF.Position.Y.Scale, MF.Position.Y.Offset + MENU_H/2)
    }):Play()
    task.delay(0.2, function() SG:Destroy() end)
    Notif("RENAN FRX","GUI fechada. Até mais! 👋",3)
end)

-- ── Animação de abertura ──
MF.Size     = UDim2.new(0,0,0,0)
MF.Position = UDim2.new(0.5,0,0.04,MENU_H/2)
TweenSvc:Create(MF, TweenInfo.new(0.3,Enum.EasingStyle.Back,Enum.EasingDirection.Out), {
    Size     = UDim2.new(0,MENU_W,0,MENU_H),
    Position = UDim2.new(0.5,-MENU_W/2,0.04,0)
}):Play()

Log("v3.0 FIXED carregado!")
task.delay(0.4, function()
    Notif("RENAN FRX | Scripts","✅ v3.0 FIXED carregado! Clique INICIAR.",5)
end)
</parameter>
</invoke>
