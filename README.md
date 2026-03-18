--[[ 
    RT HUB | SUPREME EDITION (INTEGRATED)
    Funcionalidades: Movimento, Automação, Visual, Jogadores, Config e Item Spawner.
    Pasta de Itens: ReplicatedStorage.Items
    Senha: 9921
]]

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualUser = game:GetService("VirtualUser")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local StarterGui = game:GetService("StarterGui")

local Player = Players.LocalPlayer
local pGui = Player:WaitForChild("PlayerGui")

-- // CONFIGURAÇÃO CORES E ESTILO
local SENHA_CORRETA = "9921"
local COR_ROXA = Color3.fromRGB(140, 60, 255)
local COR_FUNDO = Color3.fromRGB(10, 10, 12)
local COR_SIDEBAR = Color3.fromRGB(16, 16, 18)
local COR_BT_OFF = Color3.fromRGB(30, 30, 35)
local COR_TEXTO = Color3.fromRGB(255, 255, 255)
local ITEMS_FOLDER_NAME = "Items"
local ITEMS_FOLDER = ReplicatedStorage:FindFirstChild(ITEMS_FOLDER_NAME)

-- // ESTADO GLOBAL E KEYBINDS
local Keybinds = {
    Fly = Enum.KeyCode.F,
    Noclip = Enum.KeyCode.N,
    Fling = Enum.KeyCode.X,
    ESP = Enum.KeyCode.P,
    Menu = Enum.KeyCode.K
}

local States = {
    Flying = false,
    Noclip = false,
    Fling = false,
    ESP = false,
    GodMode = false,
    FakeLag = false,
    AutoEquip = false,
    SavedPoint = nil
}

-- // LIMPEZA DE VERSÕES ANTERIORES
if pGui:FindFirstChild("RT_HUB_SUPREME") then pGui["RT_HUB_SUPREME"]:Destroy() end

local screenGui = Instance.new("ScreenGui", pGui)
screenGui.Name = "RT_HUB_SUPREME"
screenGui.ResetOnSpawn = false

-- // FUNÇÕES AUXILIARES DE UI
local function aplicarEstilo(obj, radius)
    local ui = Instance.new("UICorner", obj)
    ui.CornerRadius = UDim.new(0, radius or 6)
end

local function notify(t)
    StarterGui:SetCore("SendNotification", {Title = "RT HUB", Text = t, Duration = 2})
end

-- // --- TELA DE LOGIN ---
local loginFrame = Instance.new("Frame", screenGui)
loginFrame.Size = UDim2.new(0, 320, 0, 220)
loginFrame.Position = UDim2.new(0.5, -160, 0.5, -110)
loginFrame.BackgroundColor3 = COR_FUNDO
aplicarEstilo(loginFrame, 12)

local loginTitle = Instance.new("TextLabel", loginFrame)
loginTitle.Size = UDim2.new(1, 0, 0, 60)
loginTitle.Text = "RT HUB | SISTEMA DE ACESSO"
loginTitle.TextColor3 = COR_ROXA
loginTitle.Font = Enum.Font.GothamBold
loginTitle.TextSize = 18
loginTitle.BackgroundTransparency = 1

local keyBox = Instance.new("TextBox", loginFrame)
keyBox.Size = UDim2.new(0.8, 0, 0, 45)
keyBox.Position = UDim2.new(0.1, 0, 0.35, 0)
keyBox.BackgroundColor3 = COR_BT_OFF
keyBox.PlaceholderText = "Digite a Senha..."
keyBox.Text = ""
keyBox.TextColor3 = Color3.new(1,1,1)
keyBox.Font = Enum.Font.Gotham
aplicarEstilo(keyBox, 8)

local loginBtn = Instance.new("TextButton", loginFrame)
loginBtn.Size = UDim2.new(0.8, 0, 0, 45)
loginBtn.Position = UDim2.new(0.1, 0, 0.65, 0)
loginBtn.BackgroundColor3 = COR_ROXA
loginBtn.Text = "ENTRAR NO HUB"
loginBtn.TextColor3 = Color3.new(1,1,1)
loginBtn.Font = Enum.Font.GothamBold
aplicarEstilo(loginBtn, 8)

-- // --- INTERFACE PRINCIPAL ---
local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 650, 0, 400)
mainFrame.Position = UDim2.new(0.5, -325, 0.5, -200)
mainFrame.BackgroundColor3 = COR_FUNDO
mainFrame.Visible = false
aplicarEstilo(mainFrame, 12)

-- SIDEBAR LATERAL
local sidebar = Instance.new("Frame", mainFrame)
sidebar.Size = UDim2.new(0, 180, 1, 0)
sidebar.BackgroundColor3 = COR_SIDEBAR
aplicarEstilo(sidebar, 12)

local logo = Instance.new("TextLabel", sidebar)
logo.Size = UDim2.new(1, 0, 0, 70)
logo.Text = "RT HUB"
logo.TextColor3 = COR_ROXA
logo.Font = Enum.Font.FredokaOne
logo.TextSize = 28
logo.BackgroundTransparency = 1

local pagesFolder = Instance.new("Frame", mainFrame)
pagesFolder.Size = UDim2.new(1, -200, 1, -30)
pagesFolder.Position = UDim2.new(0, 190, 0, 15)
pagesFolder.BackgroundTransparency = 1

local pages = {}
local activeTabBtn = nil

local function createPage(name)
    local pg = Instance.new("ScrollingFrame", pagesFolder)
    pg.Name = name
    pg.Size = UDim2.new(1, 0, 1, 0)
    pg.BackgroundTransparency = 1
    pg.Visible = false
    pg.ScrollBarThickness = 3
    pg.ScrollBarImageColor3 = COR_ROXA
    pg.CanvasSize = UDim2.new(0, 0, 0, 0)
    pg.AutomaticCanvasSize = Enum.AutomaticSize.Y
    local layout = Instance.new("UIListLayout", pg)
    layout.Padding = UDim.new(0, 12)
    pages[name] = pg
    return pg
end

-- Páginas
local homePage = createPage("Início")
local movePage = createPage("Movimento")
local spawnerPage = createPage("Spawner")
local autoPage = createPage("Automação")
local visualPage = createPage("Visual")
local playersPage = createPage("Jogadores")
local configPage = createPage("Config")
homePage.Visible = true

local function createSidebarBtn(name, yPos)
    local btn = Instance.new("TextButton", sidebar)
    btn.Size = UDim2.new(0.85, 0, 0, 38)
    btn.Position = UDim2.new(0.075, 0, 0, yPos)
    btn.BackgroundColor3 = COR_BT_OFF
    btn.Text = "  " .. name
    btn.TextColor3 = Color3.fromRGB(180, 180, 180)
    btn.Font = Enum.Font.GothamSemibold
    btn.TextSize = 13
    btn.TextXAlignment = Enum.TextXAlignment.Left
    aplicarEstilo(btn, 8)

    local selectionBar = Instance.new("Frame", btn)
    selectionBar.Name = "selectionBar"
    selectionBar.Size = UDim2.new(0, 4, 0.5, 0)
    selectionBar.Position = UDim2.new(0, 0, 0.25, 0)
    selectionBar.BackgroundColor3 = COR_ROXA
    selectionBar.Visible = false
    aplicarEstilo(selectionBar, 2)

    btn.MouseButton1Click:Connect(function()
        for n, p in pairs(pages) do p.Visible = (n == name) end
        if activeTabBtn then 
            activeTabBtn.BackgroundColor3 = COR_BT_OFF
            activeTabBtn.TextColor3 = Color3.fromRGB(180, 180, 180)
            activeTabBtn.selectionBar.Visible = false
        end
        btn.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
        btn.TextColor3 = Color3.new(1, 1, 1)
        selectionBar.Visible = true
        activeTabBtn = btn
    end)

    if name == "Início" then
        btn.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
        btn.TextColor3 = Color3.new(1, 1, 1)
        selectionBar.Visible = true
        activeTabBtn = btn
    end
end

createSidebarBtn("Início", 80)
createSidebarBtn("Movimento", 125)
createSidebarBtn("Spawner", 170)
createSidebarBtn("Automação", 215)
createSidebarBtn("Visual", 260)
createSidebarBtn("Jogadores", 305)
createSidebarBtn("Config", 350)

-- // --- COMPONENTES DE UI ---

local function createToggle(parent, text, initial, callback)
    local holder = Instance.new("Frame", parent)
    holder.Size = UDim2.new(1, -10, 0, 48)
    holder.BackgroundColor3 = COR_BT_OFF
    aplicarEstilo(holder, 8)

    local label = Instance.new("TextLabel", holder)
    label.Size = UDim2.new(0.7, 0, 1, 0)
    label.Position = UDim2.new(0, 15, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = COR_TEXTO
    label.Font = Enum.Font.GothamSemibold
    label.TextSize = 14
    label.TextXAlignment = Enum.TextXAlignment.Left

    local btn = Instance.new("TextButton", holder)
    btn.Size = UDim2.new(0, 44, 0, 24)
    btn.Position = UDim2.new(1, -60, 0.5, -12)
    btn.BackgroundColor3 = initial and COR_ROXA or Color3.fromRGB(50, 50, 55)
    btn.Text = ""
    aplicarEstilo(btn, 12)

    local indicator = Instance.new("Frame", btn)
    indicator.Size = UDim2.new(0, 18, 0, 18)
    indicator.Position = initial and UDim2.new(1, -22, 0.5, -9) or UDim2.new(0, 4, 0.5, -9)
    indicator.BackgroundColor3 = Color3.new(1,1,1)
    aplicarEstilo(indicator, 10)

    local state = initial
    local function updateUI()
        local targetPos = state and UDim2.new(1, -22, 0.5, -9) or UDim2.new(0, 4, 0.5, -9)
        local targetColor = state and COR_ROXA or Color3.fromRGB(50, 50, 55)
        TweenService:Create(indicator, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {Position = targetPos}):Play()
        TweenService:Create(btn, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {BackgroundColor3 = targetColor}):Play()
    end

    btn.MouseButton1Click:Connect(function()
        state = not state
        updateUI()
        callback(state)
    end)

    return function(ext) state = ext; updateUI() end
end

local function createButton(parent, text, callback)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1, -10, 0, 42)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    btn.Text = text
    btn.TextColor3 = COR_TEXTO
    btn.Font = Enum.Font.GothamSemibold
    btn.TextSize = 14
    aplicarEstilo(btn, 8)
    btn.MouseButton1Click:Connect(callback)
    return btn
end

local function createKeybindSetting(parent, text, keyType)
    local holder = Instance.new("Frame", parent)
    holder.Size = UDim2.new(1, -10, 0, 48)
    holder.BackgroundColor3 = COR_BT_OFF
    aplicarEstilo(holder, 8)

    local label = Instance.new("TextLabel", holder)
    label.Size = UDim2.new(0.6, 0, 1, 0)
    label.Position = UDim2.new(0, 15, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = "Tecla: " .. text
    label.TextColor3 = COR_TEXTO
    label.Font = Enum.Font.GothamSemibold
    label.TextSize = 14
    label.TextXAlignment = Enum.TextXAlignment.Left

    local btn = Instance.new("TextButton", holder)
    btn.Size = UDim2.new(0, 90, 0, 32)
    btn.Position = UDim2.new(1, -105, 0.5, -16)
    btn.BackgroundColor3 = Color3.fromRGB(45, 45, 50)
    btn.Text = Keybinds[keyType].Name
    btn.TextColor3 = COR_ROXA
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 12
    aplicarEstilo(btn, 6)

    btn.MouseButton1Click:Connect(function()
        btn.Text = "..."
        local conn
        conn = UserInputService.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.Keyboard then
                Keybinds[keyType] = input.KeyCode
                btn.Text = input.KeyCode.Name
                conn:Disconnect()
            end
        end)
    end)
end

-- // --- LÓGICA DO SPAWNER (INTEGRADA) ---
local spawnerInput = Instance.new("TextBox", spawnerPage)
spawnerInput.Size = UDim2.new(1, -10, 0, 45)
spawnerInput.BackgroundColor3 = COR_BT_OFF
spawnerInput.PlaceholderText = "Nome do item (Ex: Fruit, Sword...)"
spawnerInput.Text = ""
spawnerInput.TextColor3 = Color3.new(1,1,1)
spawnerInput.Font = Enum.Font.Gotham
aplicarEstilo(spawnerInput, 8)

local spawnerButtons = Instance.new("Frame", spawnerPage)
spawnerButtons.Size = UDim2.new(1, -10, 0, 45)
spawnerButtons.BackgroundTransparency = 1

local spawnBtn = Instance.new("TextButton", spawnerButtons)
spawnBtn.Size = UDim2.new(0.48, 0, 1, 0)
spawnBtn.BackgroundColor3 = COR_ROXA
spawnBtn.Text = "SPAWN ITEM"
spawnBtn.TextColor3 = Color3.new(1,1,1)
spawnBtn.Font = Enum.Font.GothamBold
aplicarEstilo(spawnBtn, 8)

local refreshSpawner = Instance.new("TextButton", spawnerButtons)
refreshSpawner.Size = UDim2.new(0.48, 0, 1, 0)
refreshSpawner.Position = UDim2.new(0.52, 0, 0, 0)
refreshSpawner.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
refreshSpawner.Text = "ATUALIZAR (R)"
refreshSpawner.TextColor3 = Color3.new(1,1,1)
refreshSpawner.Font = Enum.Font.GothamBold
aplicarEstilo(refreshSpawner, 8)

createToggle(spawnerPage, "Auto-Equip Item", States.AutoEquip, function(s) States.AutoEquip = s end)

local listContainer = Instance.new("Frame", spawnerPage)
listContainer.Size = UDim2.new(1, -10, 0, 180)
listContainer.BackgroundColor3 = Color3.fromRGB(15, 15, 18)
aplicarEstilo(listContainer, 10)

local itemScroll = Instance.new("ScrollingFrame", listContainer)
itemScroll.Size = UDim2.new(1, -10, 1, -10)
itemScroll.Position = UDim2.new(0, 5, 0, 5)
itemScroll.BackgroundTransparency = 1
itemScroll.ScrollBarThickness = 2
itemScroll.ScrollBarImageColor3 = COR_ROXA
itemScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
local itemLayout = Instance.new("UIListLayout", itemScroll)
itemLayout.Padding = UDim.new(0, 5)

local function spawnItem(name)
    if not ITEMS_FOLDER then notify("ERRO: Pasta 'Items' não encontrada em ReplicatedStorage.") return end
    local item = ITEMS_FOLDER:FindFirstChild(name)
    if item then
        local tool = item:IsA("Tool") and item or item:FindFirstChildOfClass("Tool") or item:FindFirstChildWhichIsA("Tool", true)
        if tool then
            local c = tool:Clone()
            c.Parent = Player.Backpack
            notify("Sucesso: " .. name .. " spawnado!")
            if States.AutoEquip and Player.Character then
                task.wait(0.1)
                Player.Character.Humanoid:EquipTool(c)
            end
        else notify("Objeto não é uma Tool válida!") end
    else notify("Item '" .. name .. "' não existe na pasta.") end
end

local function updateSpawnerList()
    for _, v in pairs(itemScroll:GetChildren()) do if v:IsA("TextButton") then v:Destroy() end end
    if not ITEMS_FOLDER then return end
    for _, item in pairs(ITEMS_FOLDER:GetChildren()) do
        local b = Instance.new("TextButton", itemScroll)
        b.Size = UDim2.new(1, 0, 0, 32)
        b.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
        b.Text = item.Name
        b.TextColor3 = Color3.fromRGB(200, 200, 200)
        b.Font = Enum.Font.Gotham
        b.TextSize = 12
        aplicarEstilo(b, 6)
        b.MouseButton1Click:Connect(function() spawnerInput.Text = item.Name end)
    end
end

spawnBtn.MouseButton1Click:Connect(function() spawnItem(spawnerInput.Text) end)
refreshSpawner.MouseButton1Click:Connect(updateSpawnerList)

-- // --- LÓGICA DE MOVIMENTO (COMPLETA) ---
local updateFlyUI, updateNoclipUI, updateFlingUI, updateESPUI

-- FLY LOGIC
local function toggleFly(state)
    States.Flying = state
    local char = Player.Character
    if States.Flying and char then
        local hrp = char:WaitForChild("HumanoidRootPart")
        local bv = Instance.new("BodyVelocity", hrp); bv.Name = "RT_FlyVel"
        bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        local bg = Instance.new("BodyGyro", hrp); bg.Name = "RT_FlyGyro"
        bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
        char.Humanoid.PlatformStand = true
        task.spawn(function()
            while States.Flying and char.Parent do
                local cam = workspace.CurrentCamera
                local dir = Vector3.new(0,0,0)
                if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir = dir + cam.CFrame.LookVector end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir = dir - cam.CFrame.LookVector end
                if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir = dir - cam.CFrame.RightVector end
                if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir = dir + cam.CFrame.RightVector end
                bv.Velocity = dir.Unit * 80
                if dir == Vector3.new(0,0,0) then bv.Velocity = Vector3.new(0,0,0) end
                bg.CFrame = cam.CFrame
                task.wait()
            end
            if bv then bv:Destroy() end
            if bg then bg:Destroy() end
            if char:FindFirstChild("Humanoid") then char.Humanoid.PlatformStand = false end
        end)
    end
    if updateFlyUI then updateFlyUI(state) end
end

-- NOCLIP LOGIC
RunService.Stepped:Connect(function()
    if States.Noclip and Player.Character then
        for _, v in pairs(Player.Character:GetDescendants()) do
            if v:IsA("BasePart") then v.CanCollide = false end
        end
    end
end)

-- FLING LOGIC (MATAR JOGADORES AO TOCAR)
task.spawn(function()
    local movel = 0.1
    while true do
        RunService.Heartbeat:Wait()
        if States.Fling then
            local char = Player.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            if hrp then
                local oldVel = hrp.Velocity
                hrp.Velocity = oldVel * 10000 + Vector3.new(0, 10000, 0)
                RunService.RenderStepped:Wait()
                if hrp.Parent then hrp.Velocity = oldVel end
                RunService.Stepped:Wait()
                if hrp.Parent then
                    hrp.Velocity = oldVel + Vector3.new(0, movel, 0)
                    movel = movel * -1
                end
            end
        end
    end
end)

-- ESP LOGIC
RunService.RenderStepped:Connect(function()
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= Player and p.Character then
            local high = p.Character:FindFirstChild("RT_ESP")
            if States.ESP then
                if not high then
                    high = Instance.new("Highlight", p.Character)
                    high.Name = "RT_ESP"
                    high.FillColor = COR_ROXA
                    high.OutlineColor = Color3.new(1,1,1)
                    high.FillTransparency = 0.5
                end
            elseif high then high:Destroy() end
        end
    end
end)

-- FAKE LAG LOGIC
task.spawn(function()
    while true do
        task.wait()
        if States.FakeLag then
            local hrp = Player.Character and Player.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                hrp.Anchored = true
                task.wait(0.25)
                hrp.Anchored = false
                task.wait(0.05)
            end
        end
    end
end)

-- // --- POPULANDO PÁGINAS ---

-- Home
local hTitle = Instance.new("TextLabel", homePage)
hTitle.Size = UDim2.new(1, 0, 0, 30); hTitle.BackgroundTransparency = 1
hTitle.Text = "Bem-vindo ao RT HUB SUPREME"; hTitle.TextColor3 = COR_ROXA; hTitle.Font = Enum.Font.GothamBold; hTitle.TextSize = 16

local hSub = Instance.new("TextLabel", homePage)
hSub.Size = UDim2.new(1, 0, 0, 60); hSub.BackgroundTransparency = 1
hSub.Text = "Use [" .. Keybinds.Menu.Name .. "] para abrir/fechar o menu.\nTodas as funções foram unificadas aqui."; hSub.TextColor3 = Color3.new(0.8,0.8,0.8); hSub.Font = Enum.Font.Gotham; hSub.TextSize = 13

-- Movimento
updateFlyUI = createToggle(movePage, "Fly / Voar", States.Flying, toggleFly)
updateNoclipUI = createToggle(movePage, "Noclip (Paredes)", States.Noclip, function(s) States.Noclip = s end)
updateFlingUI = createToggle(movePage, "Touch Fling (Matar)", States.Fling, function(s) States.Fling = s end)
createToggle(movePage, "Fake Lag", States.FakeLag, function(s) States.FakeLag = s end)
createButton(movePage, "DEFINIR PONTO DE TELEPORT", function() if Player.Character then States.SavedPoint = Player.Character.HumanoidRootPart.CFrame; notify("Ponto salvo!") end end)
createButton(movePage, "TELEPORTAR PARA PONTO", function() if States.SavedPoint then Player.Character.HumanoidRootPart.CFrame = States.SavedPoint end end)

-- Automação
createToggle(autoPage, "God Mode (Cura Infinita)", States.GodMode, function(s) States.GodMode = s end)
RunService.Heartbeat:Connect(function() 
    if States.GodMode then 
        local h = Player.Character:FindFirstChild("Humanoid") 
        if h and h.Health < h.MaxHealth then h.Health = h.MaxHealth end 
    end 
end)

-- Visual
updateESPUI = createToggle(visualPage, "ESP Players (Highlight)", States.ESP, function(s) States.ESP = s end)

-- Jogadores (Exemplo de botões rápidos)
createButton(playersPage, "REDEFINIR PERSONAGEM", function() Player.Character:BreakJoints() end)
createButton(playersPage, "LIMPAR FERRAMENTAS", function() Player.Backpack:ClearAllChildren() end)

-- Configurações
createKeybindSetting(configPage, "Menu Principal", "Menu")
createKeybindSetting(configPage, "Fly / Voo", "Fly")
createKeybindSetting(configPage, "Noclip", "Noclip")
createKeybindSetting(configPage, "Fling", "Fling")
createKeybindSetting(configPage, "ESP", "ESP")

-- // --- SISTEMA DE LOGIN E CONTROLE ---
loginBtn.MouseButton1Click:Connect(function()
    if keyBox.Text == SENHA_CORRETA then
        loginFrame:Destroy()
        mainFrame.Visible = true
        updateSpawnerList() -- Carrega a lista de itens
        notify("RT HUB Ativado com Sucesso!")
    else
        keyBox.Text = ""; keyBox.PlaceholderText = "SENHA INCORRETA!"; keyBox.PlaceholderColor3 = Color3.new(1,0,0)
    end
end)

-- DRAG SYSTEM
local function setupDrag(frame)
    local drag, dStart, sPos
    frame.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then drag = true; dStart = i.Position; sPos = frame.Position end end)
    UserInputService.InputChanged:Connect(function(i) if drag and i.UserInputType == Enum.UserInputType.MouseMovement then local delta = i.Position - dStart; frame.Position = UDim2.new(sPos.X.Scale, sPos.X.Offset + delta.X, sPos.Y.Scale, sPos.Y.Offset + delta.Y) end end)
    UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then drag = false end end)
end
setupDrag(mainFrame); setupDrag(loginFrame)

-- INPUT SYSTEM
UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == Keybinds.Menu then
        mainFrame.Visible = not mainFrame.Visible
    elseif input.KeyCode == Keybinds.Fly then
        toggleFly(not States.Flying)
    elseif input.KeyCode == Keybinds.Noclip then
        States.Noclip = not States.Noclip
        if updateNoclipUI then updateNoclipUI(States.Noclip) end
    elseif input.KeyCode == Keybinds.Fling then
        States.Fling = not States.Fling
        if updateFlingUI then updateFlingUI(States.Fling) end
    elseif input.KeyCode == Keybinds.ESP then
        States.ESP = not States.ESP
        if updateESPUI then updateESPUI(States.ESP) end
    elseif input.KeyCode == Enum.KeyCode.R and spawnerPage.Visible then
        updateSpawnerList()
    end
end)

notify("RT HUB Carregado. Insira a senha.")
