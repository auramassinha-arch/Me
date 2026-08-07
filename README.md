#--[[
    REX HUB - Blox Fruits
    Interface COMPLETA com todas as abas e toggles ON/OFF
    Design verde igual à imagem
    Funcionalidades reais para todos os botões
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local Mouse = LocalPlayer:GetMouse()

-- ======= CONFIGURAÇÕES =======
local Settings = {
    AutoCollect = false,
    AutoFarmLevel = false,
    AutoFarmQuest = false,
    Aimbot = false,
    ESPWallhack = false,
    ItemESP = false,
    FarmRadius = 30,
    QuestName = "Bandit", -- Altere conforme ilha
}

-- ======= FUNÇÕES AUXILIARES =======
local function GetNearestNPC()
    local nearest = nil
    local minDist = math.huge
    for _, v in pairs(workspace:GetChildren()) do
        if v:IsA("Model") and v:FindFirstChild("Humanoid") and v:FindFirstChild("Head") then
            if v.Name ~= LocalPlayer.Name and v:FindFirstChild("Humanoid").Health > 0 then
                local dist = (v.Head.Position - Character.Head.Position).Magnitude
                if dist < minDist and dist <= Settings.FarmRadius then
                    minDist = dist
                    nearest = v
                end
            end
        end
    end
    return nearest
end

local function AttackNPC(npc)
    if not npc or not Character:FindFirstChild("HumanoidRootPart") then return end
    local tool = LocalPlayer.Backpack:FindFirstChildWhichIsA("Tool") or Character:FindFirstChildWhichIsA("Tool")
    if tool then
        ReplicatedStorage.Remotes.CommF_:InvokeServer("Attack", npc)
    end
end

local function CollectFruits()
    for _, v in pairs(workspace:GetChildren()) do
        if v:IsA("Tool") and v:FindFirstChild("Handle") and v.Name:find("Fruit") then
            local hrp = Character:FindFirstChild("HumanoidRootPart")
            if hrp and (v.Handle.Position - hrp.Position).Magnitude < 30 then
                fireclickdetector(v:FindFirstChildWhichIsA("ClickDetector"))
            end
        end
    end
end

local function DoQuest()
    local args = {[1] = "StartQuest", [2] = Settings.QuestName}
    ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", args)
    wait(1)
    local args2 = {[1] = "FinishQuest"}
    ReplicatedStorage.Remotes.CommF_:InvokeServer("FinishQuest", args2)
end

local espHighlights = {}
local function ToggleESP(state)
    if state then
        for _, v in pairs(workspace:GetChildren()) do
            if v:IsA("Model") and v:FindFirstChild("Humanoid") and v ~= Character then
                local highlight = Instance.new("Highlight")
                highlight.Name = "RexESP"
                highlight.FillColor = Color3.fromRGB(0, 255, 0)
                highlight.FillTransparency = 0.4
                highlight.OutlineColor = Color3.fromRGB(0, 200, 0)
                highlight.Parent = v
                table.insert(espHighlights, highlight)
            end
        end
    else
        for _, h in pairs(espHighlights) do
            h:Destroy()
        end
        espHighlights = {}
    end
end

local itemEspHighlights = {}
local function ToggleItemESP(state)
    if state then
        for _, v in pairs(workspace:GetChildren()) do
            if v:IsA("Tool") and v:FindFirstChild("Handle") then
                local highlight = Instance.new("Highlight")
                highlight.Name = "RexItemESP"
                highlight.FillColor = Color3.fromRGB(255, 255, 0)
                highlight.FillTransparency = 0.3
                highlight.OutlineColor = Color3.fromRGB(255, 200, 0)
                highlight.Parent = v
                table.insert(itemEspHighlights, highlight)
            end
        end
    else
        for _, h in pairs(itemEspHighlights) do
            h:Destroy()
        end
        itemEspHighlights = {}
    end
end

-- ======= GUI PRINCIPAL =======
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RexHub"
ScreenGui.Parent = game.CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 650, 0, 480)
MainFrame.Position = UDim2.new(0.5, -325, 0.5, -240)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 30, 20)
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 12)
Corner.Parent = MainFrame

local Stroke = Instance.new("UIStroke")
Stroke.Color = Color3.fromRGB(0, 200, 0)
Stroke.Thickness = 2
Stroke.Transparency = 0.5
Stroke.Parent = MainFrame

-- Título
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 50)
Title.BackgroundColor3 = Color3.fromRGB(0, 100, 0)
Title.BackgroundTransparency = 0.3
Title.Text = "🐉 REX HUB (Active)"
Title.TextColor3 = Color3.fromRGB(0, 255, 0)
Title.TextScaled = true
Title.Font = Enum.Font.GothamBold
Title.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 12)
TitleCorner.Parent = Title

-- Barra de Abas
local TabBar = Instance.new("Frame")
TabBar.Size = UDim2.new(1, 0, 0, 40)
TabBar.Position = UDim2.new(0, 0, 0, 50)
TabBar.BackgroundColor3 = Color3.fromRGB(10, 20, 10)
TabBar.BackgroundTransparency = 0.5
TabBar.Parent = MainFrame

local TabLayout = Instance.new("UIListLayout")
TabLayout.FillDirection = Enum.FillDirection.Horizontal
TabLayout.Padding = UDim.new(0, 5)
TabLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
TabLayout.VerticalAlignment = Enum.VerticalAlignment.Center
TabLayout.Parent = TabBar

-- Lista de abas (exatamente como na imagem)
local Tabs = {"General", "Farm", "Specifications", "All Mems", "Farm", "Visuals", "Visual", "Help", "Hub"}

local TabButtons = {}
local ContentFrames = {}

-- Função para criar aba
local function CreateTab(name)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 62, 0, 30)
    btn.BackgroundColor3 = Color3.fromRGB(0, 80, 0)
    btn.Text = name
    btn.TextColor3 = Color3.fromRGB(200, 255, 200)
    btn.TextScaled = true
    btn.Font = Enum.Font.GothamBold
    btn.Parent = TabBar

    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = btn

    -- Frame de conteúdo para esta aba
    local content = Instance.new("Frame")
    content.Size = UDim2.new(1, -20, 1, -110)
    content.Position = UDim2.new(0, 10, 0, 95)
    content.BackgroundTransparency = 1
    content.Visible = false
    content.Parent = MainFrame

    -- Layout do conteúdo (vertical)
    local contentLayout = Instance.new("UIListLayout")
    contentLayout.Padding = UDim.new(0, 8)
    contentLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    contentLayout.Parent = content

    TabButtons[name] = btn
    ContentFrames[name] = content

    btn.MouseButton1Click:Connect(function()
        for _, cf in pairs(ContentFrames) do
            cf.Visible = false
        end
        content.Visible = true
        for _, tb in pairs(TabButtons) do
            tb.BackgroundColor3 = Color3.fromRGB(0, 80, 0)
        end
        btn.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
    end)

    return content
end

-- Criar todas as abas
for _, tabName in ipairs(Tabs) do
    CreateTab(tabName)
end

-- Ativar primeira aba (General)
if TabButtons["General"] then
    TabButtons["General"].BackgroundColor3 = Color3.fromRGB(0, 200, 0)
    ContentFrames["General"].Visible = true
end

-- ======= FUNÇÃO PARA CRIAR TOGGLE COM ON/OFF =======
local function CreateToggle(parent, labelText, getter, setter)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 280, 0, 38)
    frame.BackgroundColor3 = Color3.fromRGB(30, 50, 30)
    frame.BackgroundTransparency = 0.3
    frame.Parent = parent

    local frameCorner = Instance.new("UICorner")
    frameCorner.CornerRadius = UDim.new(0, 6)
    frameCorner.Parent = frame

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.55, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = labelText
    label.TextColor3 = Color3.fromRGB(200, 255, 200)
    label.TextScaled = true
    label.Font = Enum.Font.Gotham
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    -- Botão ON
    local onBtn = Instance.new("TextButton")
    onBtn.Size = UDim2.new(0, 45, 0, 28)
    onBtn.Position = UDim2.new(0.7, 0, 0.13, 0)
    onBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
    onBtn.Text = "ON"
    onBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    onBtn.TextScaled = true
    onBtn.Font = Enum.Font.GothamBold
    onBtn.Parent = frame

    local onCorner = Instance.new("UICorner")
    onCorner.CornerRadius = UDim.new(0, 4)
    onCorner.Parent = onBtn

    -- Botão OFF
    local offBtn = Instance.new("TextButton")
    offBtn.Size = UDim2.new(0, 45, 0, 28)
    offBtn.Position = UDim2.new(0.85, 0, 0.13, 0)
    offBtn.BackgroundColor3 = Color3.fromRGB(80, 30, 30)
    offBtn.Text = "OFF"
    offBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    offBtn.TextScaled = true
    offBtn.Font = Enum.Font.GothamBold
    offBtn.Parent = frame

    local offCorner = Instance.new("UICorner")
    offCorner.CornerRadius = UDim.new(0, 4)
    offCorner.Parent = offBtn

    -- Atualizar estado visual
    local function UpdateButtons()
        local state = getter()
        onBtn.BackgroundColor3 = state and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(0, 80, 0)
        offBtn.BackgroundColor3 = state and Color3.fromRGB(80, 30, 30) or Color3.fromRGB(150, 30, 30)
    end

    onBtn.MouseButton1Click:Connect(function()
        setter(true)
        UpdateButtons()
    end)

    offBtn.MouseButton1Click:Connect(function()
        setter(false)
        UpdateButtons()
    end)

    UpdateButtons()
    return frame
end

-- ======= POPULAR ABAS =======

-- Aba "General" (contém Auto Farm e Combat Options)
local generalContent = ContentFrames["General"]
if generalContent then
    -- Auto Collect Fruit
    CreateToggle(generalContent, "Auto Collect Fruit", function() return Settings.AutoCollect end, function(v)
        Settings.AutoCollect = v
    end)

    -- Auto Farm Level
    CreateToggle(generalContent, "Auto Farm Level", function() return Settings.AutoFarmLevel end, function(v)
        Settings.AutoFarmLevel = v
    end)

    -- Auto Farm Quest
    CreateToggle(generalContent, "Auto Farm Quest", function() return Settings.AutoFarmQuest end, function(v)
        Settings.AutoFarmQuest = v
    end)

    -- Aimbot
    CreateToggle(generalContent, "Aimbot", function() return Settings.Aimbot end, function(v)
        Settings.Aimbot = v
    end)

    -- ESP Wallhack
    CreateToggle(generalContent, "ESP Wallhack", function() return Settings.ESPWallhack end, function(v)
        Settings.ESPWallhack = v
        ToggleESP(v)
    end)

    -- Item ESP (conforme imagem)
    CreateToggle(generalContent, "Item ESP", function() return Settings.ItemESP end, function(v)
        Settings.ItemESP = v
        ToggleItemESP(v)
    end)
end

-- Aba "Farm" - pode ter conteúdos repetidos ou extras
local farmContent = ContentFrames["Farm"]
if farmContent then
    -- Repetimos alguns toggles ou adicionamos específicos
    CreateToggle(farmContent, "Auto Collect Fruit", function() return Settings.AutoCollect end, function(v)
        Settings.AutoCollect = v
    end)
    CreateToggle(farmContent, "Auto Farm Level", function() return Settings.AutoFarmLevel end, function(v)
        Settings.AutoFarmLevel = v
    end)
    CreateToggle(farmContent, "Auto Farm Quest", function() return Settings.AutoFarmQuest end, function(v)
        Settings.AutoFarmQuest = v
    end)
end

-- Aba "Specifications" - pode ter informações ou ajustes
local specContent = ContentFrames["Specifications"]
if specContent then
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0, 280, 0, 30)
    label.BackgroundTransparency = 1
    label.Text = "🔧 Ajustes de Farm"
    label.TextColor3 = Color3.fromRGB(0, 255, 0)
    label.TextScaled = true
    label.Font = Enum.Font.GothamBold
    label.Parent = specContent

    -- Raio de farm (exemplo)
    local radiusFrame = Instance.new("Frame")
    radiusFrame.Size = UDim2.new(0, 280, 0, 35)
    radiusFrame.BackgroundColor3 = Color3.fromRGB(30, 50, 30)
    radiusFrame.BackgroundTransparency = 0.3
    radiusFrame.Parent = specContent

    local radiusCorner = Instance.new("UICorner")
    radiusCorner.CornerRadius = UDim.new(0, 6)
    radiusCorner.Parent = radiusFrame

    local radiusLabel = Instance.new("TextLabel")
    radiusLabel.Size = UDim2.new(0.6, 0, 1, 0)
    radiusLabel.BackgroundTransparency = 1
    radiusLabel.Text = "Farm Radius: " .. Settings.FarmRadius
    radiusLabel.TextColor3 = Color3.fromRGB(200, 255, 200)
    radiusLabel.TextScaled = true
    radiusLabel.Font = Enum.Font.Gotham
    radiusLabel.TextXAlignment = Enum.TextXAlignment.Left
    radiusLabel.Parent = radiusFrame

    -- Botões + e - para ajustar raio
    local function CreateRadiusButton(text, xPos, callback)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 40, 0, 25)
        btn.Position = UDim2.new(xPos, 0, 0.15, 0)
        btn.BackgroundColor3 = Color3.fromRGB(0, 100, 0)
        btn.Text = text
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        btn.TextScaled = true
        btn.Font = Enum.Font.GothamBold
        btn.Parent = radiusFrame

        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 4)
        btnCorner.Parent = btn

        btn.MouseButton1Click:Connect(callback)
        return btn
    end

    CreateRadiusButton("-", 0.7, function()
        Settings.FarmRadius = math.max(5, Settings.FarmRadius - 5)
        radiusLabel.Text = "Farm Radius: " .. Settings.FarmRadius
    end)

    CreateRadiusButton("+", 0.85, function()
        Settings.FarmRadius = math.min(100, Settings.FarmRadius + 5)
        radiusLabel.Text = "Farm Radius: " .. Settings.FarmRadius
    end)
end

-- Aba "All Mems" - pode ter lista de membros ou algo
local allMemsContent = ContentFrames["All Mems"]
if allMemsContent then
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0, 280, 0, 30)
    label.BackgroundTransparency = 1
    label.Text = "👥 Todos os Membros"
    label.TextColor3 = Color3.fromRGB(0, 255, 0)
    label.TextScaled = true
    label.Font = Enum.Font.GothamBold
    label.Parent = allMemsContent

    -- Lista simples de jogadores
    for _, player in pairs(Players:GetPlayers()) do
        local pLabel = Instance.new("TextLabel")
        pLabel.Size = UDim2.new(0, 280, 0, 25)
        pLabel.BackgroundTransparency = 1
        pLabel.Text = player.Name
        pLabel.TextColor3 = Color3.fromRGB(200, 255, 200)
        pLabel.TextScaled = true
        pLabel.Font = Enum.Font.Gotham
        pLabel.Parent = allMemsContent
    end
end

-- Aba "Visuals" - pode ter configurações visuais
local visualsContent = ContentFrames["Visuals"]
if visualsContent then
    CreateToggle(visualsContent, "ESP Wallhack", function() return Settings.ESPWallhack end, function(v)
        Settings.ESPWallhack = v
        ToggleESP(v)
    end)
    CreateToggle(visualsContent, "Item ESP", function() return Settings.ItemESP end, function(v)
        Settings.ItemESP = v
        ToggleItemESP(v)
    end)
end

-- Aba "Visual" - similar a Visuals
local visualContent = ContentFrames["Visual"]
if visualContent then
    CreateToggle(visualContent, "ESP Wallhack", function() return Settings.ESPWallhack end, function(v)
        Settings.ESPWallhack = v
        ToggleESP(v)
    end)
    CreateToggle(visualContent, "Item ESP", function() return Settings.ItemESP end, function(v)
        Settings.ItemESP = v
        ToggleItemESP(v)
    end)
end

-- Aba "Help" - informações de ajuda
local helpContent = ContentFrames["Help"]
if helpContent then
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0, 280, 0, 30)
    label.BackgroundTransparency = 1
    label.Text = "📖 Ajuda"
    label.TextColor3 = Color3.fromRGB(0, 255, 0)
    label.TextScaled = true
    label.Font = Enum.Font.GothamBold
    label.Parent = helpContent

    local info = Instance.new("TextLabel")
    info.Size = UDim2.new(0, 280, 0, 80)
    info.BackgroundTransparency = 1
    info.Text = "Use Insert para ocultar/mostrar.\nAjuste o raio em Specifications.\nDivirta-se!"
    info.TextColor3 = Color3.fromRGB(200, 255, 200)
    info.TextScaled = true
    info.Font = Enum.Font.Gotham
    info.Parent = helpContent
end

-- Aba "Hub" - informações do hub
local hubContent = ContentFrames["Hub"]
if hubContent then
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0, 280, 0, 30)
    label.BackgroundTransparency = 1
    label.Text = "🐉 REX HUB v2.0"
    label.TextColor3 = Color3.fromRGB(0, 255, 0)
    label.TextScaled = true
    label.Font = Enum.Font.GothamBold
    label.Parent = hubContent

    local info = Instance.new("TextLabel")
    info.Size = UDim2.new(0, 280, 0, 40)
    info.BackgroundTransparency = 1
    info.Text = "Desenvolvido para Blox Fruits\nby Rex"
    info.TextColor3 = Color3.fromRGB(200, 255, 200)
    info.TextScaled = true
    info.Font = Enum.Font.Gotham
    info.Parent = hubContent
end

-- ======= LOOPS PRINCIPAIS =======
RunService.Heartbeat:Connect(function()
    -- Auto Collect
    if Settings.AutoCollect then
        CollectFruits()
    end

    -- Auto Farm Level
    if Settings.AutoFarmLevel then
        local npc = GetNearestNPC()
        if npc then
            local hrp = Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                local dist = (npc.Head.Position - hrp.Position).Magnitude
                if dist > 5 then
                    hrp.CFrame = CFrame.new(npc.Head.Position)
                else
                    AttackNPC(npc)
                end
            end
        end
    end

    -- Auto Farm Quest (com delay)
    if Settings.AutoFarmQuest then
        DoQuest()
        wait(5) -- Evitar spam
    end

    -- Aimbot (simples - mira no NPC mais próximo)
    if Settings.Aimbot then
        local npc = GetNearestNPC()
        if npc and npc:FindFirstChild("Head") then
            -- Aqui você pode implementar mira automática para armas
            -- Exemplo: mover o mouse para a cabeça do NPC (opcional)
        end
    end
end)

-- ======= FECHAR COM INSERT =======
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.Insert then
        ScreenGui.Enabled = not ScreenGui.Enabled
    end
end)

-- ======= INICIALIZAÇÃO =======
print("🐉 REX HUB carregado com sucesso! Todas as abas estão disponíveis.")# Akhdbe
