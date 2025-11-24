-- AnimatorPolo - LocalScript (versão melhorada)
-- Coloque em StarterPlayerScripts (ou onde você use para executar GUI)
-- Comentários em pt-BR para facilitar edição

local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Cria ScreenGui apenas se não existir
local screenGui = playerGui:FindFirstChild("AnimadorGui")
if not screenGui then
    screenGui = Instance.new("ScreenGui")
    screenGui.Name = "AnimadorGui"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = playerGui
end

-- Função utilitária para criar instâncias com propriedades
local function create(class, props)
    local inst = Instance.new(class)
    if props then
        for k,v in pairs(props) do
            inst[k] = v
        end
    end
    return inst
end

-- KeyframeManager modularizado
local KeyframeManager = {}
KeyframeManager.__index = KeyframeManager

function KeyframeManager.new()
    local self = setmetatable({}, KeyframeManager)
    self.keyframes = {}
    return self
end

function KeyframeManager:addKeyframe(time, data)
    if type(time) ~= "number" or time < 0 then return false, "tempo inválido" end
    table.insert(self.keyframes, { time = time, data = data })
    table.sort(self.keyframes, function(a,b) return a.time < b.time end)
    return true
end

function KeyframeManager:removeKeyframeAtIndex(i)
    if type(i) ~= "number" or not self.keyframes[i] then return false end
    table.remove(self.keyframes, i)
    return true
end

function KeyframeManager:getKeyframes()
    return self.keyframes
end

function KeyframeManager:exportToJSON(pretty)
    if pretty then
        return HttpService:JSONEncode(self.keyframes) -- Roblox não tem pretty nativo fácil; mantém compact
    else
        return HttpService:JSONEncode(self.keyframes)
    end
end

function KeyframeManager:importFromJSON(jsonText)
    local ok, t = pcall(HttpService.JSONDecode, HttpService, jsonText)
    if not ok then return false, "JSON inválido" end
    if type(t) ~= "table" then return false, "JSON não é uma lista de keyframes" end
    self.keyframes = {}
    for _, v in ipairs(t) do
        -- validação mínima
        if type(v.time) == "number" and type(v.data) == "table" then
            table.insert(self.keyframes, v)
        end
    end
    table.sort(self.keyframes, function(a,b) return a.time < b.time end)
    return true
end

function KeyframeManager:exportToKeyframeSequenceString()
    -- Gera uma string Lua formatada com a sequência para copiar/colar no Studio
    local out = {"-- KeyframeSequence gerado por AnimatorPolo\nlocal sequence = {\n"}
    for _, kf in ipairs(self.keyframes) do
        local dataJson = HttpService:JSONEncode(kf.data)
        table.insert(out, string.format("    { Time = %.4f, Value = %s },\n", kf.time, dataJson))
    end
    table.insert(out, "}\nreturn sequence")
    return table.concat(out, "")
end

-- instancia o gerenciador
local animador = KeyframeManager.new()

-- === GUI ===
local mainFrame = screenGui:FindFirstChild("MainFrame")
if not mainFrame then
    mainFrame = create("Frame", {
        Name = "MainFrame",
        Parent = screenGui,
        Size = UDim2.new(0, 380, 0, 460),
        Position = UDim2.new(0.5, -190, 0.5, -230),
        BackgroundColor3 = Color3.fromRGB(240,248,255),
        Active = true,
        Draggable = true,
        BorderSizePixel = 2,
        BorderColor3 = Color3.fromRGB(100,149,237)
    })
end

-- Título
local title = mainFrame:FindFirstChild("Title")
if not title then
    title = create("TextLabel", {
        Name = "Title", Parent = mainFrame,
        Size = UDim2.new(1,0,0,42), Position = UDim2.new(0,0,0,0),
        BackgroundColor3 = Color3.fromRGB(230,230,250), Text = "Animador ✨",
        Font = Enum.Font.GothamBold, TextSize = 24, TextColor3 = Color3.fromRGB(30,60,100)
    })
end

-- Close button
local closeButton = mainFrame:FindFirstChild("CloseButton")
if not closeButton then
    closeButton = create("TextButton", {
        Name = "CloseButton", Parent = mainFrame,
        Size = UDim2.new(0,36,0,36), Position = UDim2.new(1,-44,0,4),
        Text = "X", Font = Enum.Font.GothamBold, TextSize = 20, AutoButtonColor = false,
        BackgroundColor3 = Color3.fromRGB(255,100,100), TextColor3 = Color3.new(1,1,1)
    })
end

closeButton.MouseButton1Click:Connect(function()
    mainFrame.Visible = false
end)

-- Inputs (tempo e dados)
local inputTime = mainFrame:FindFirstChild("InputTime")
if not inputTime then
    inputTime = create("TextBox", {
        Name = "InputTime", Parent = mainFrame,
        Size = UDim2.new(0.46,0,0,32), Position = UDim2.new(0.03,0,0.78,0),
        PlaceholderText = "Tempo (segundos)", ClearTextOnFocus = false, Font = Enum.Font.Gotham, TextSize = 14
    })
end

local inputData = mainFrame:FindFirstChild("InputData")
if not inputData then
    inputData = create("TextBox", {
        Name = "InputData", Parent = mainFrame,
        Size = UDim2.new(0.46,0,0,32), Position = UDim2.new(0.51,0,0.78,0),
        PlaceholderText = 'Dados (ex: {"pos":{x:0,y:0,z:0}})', ClearTextOnFocus = false, MultiLine = false, Font = Enum.Font.Gotham, TextSize = 14
    })
end

-- Botões
local addBtn = mainFrame:FindFirstChild("AddBtn")
if not addBtn then
    addBtn = create("TextButton", {
        Name = "AddBtn", Parent = mainFrame,
        Size = UDim2.new(0.94,0,0,34), Position = UDim2.new(0.03,0,0.85,0),
        Text = "Adicionar Keyframe", Font = Enum.Font.GothamBold, TextSize = 16, TextColor3 = Color3.new(1,1,1),
        BackgroundColor3 = Color3.fromRGB(100,149,237), AutoButtonColor = false
    })
end

local exportJSONBtn = mainFrame:FindFirstChild("ExportJSON")
if not exportJSONBtn then
    exportJSONBtn = create("TextButton", {
        Name = "ExportJSON", Parent = mainFrame,
        Size = UDim2.new(0.46,0,0,34), Position = UDim2.new(0.03,0,0.65,0),
        Text = "Exportar JSON", Font = Enum.Font.GothamBold, TextSize = 14, AutoButtonColor = false
    })
end

local importJSONBtn = mainFrame:FindFirstChild("ImportJSON")
if not importJSONBtn then
    importJSONBtn = create("TextButton", {
        Name = "ImportJSON", Parent = mainFrame,
        Size = UDim2.new(0.46,0,0,34), Position = UDim2.new(0.51,0,0.65,0),
        Text = "Importar JSON", Font = Enum.Font.GothamBold, TextSize = 14, AutoButtonColor = false
    })
end

local genScriptBtn = mainFrame:FindFirstChild("GenScript")
if not genScriptBtn then
    genScriptBtn = create("TextButton", {
        Name = "GenScript", Parent = mainFrame,
        Size = UDim2.new(0.94,0,0,30), Position = UDim2.new(0.03,0,0.58,0),
        Text = "Gerar Script (copiar/colar)", Font = Enum.Font.GothamBold, TextSize = 14
    })
end

-- Área rolável para lista de keyframes
local scroll = mainFrame:FindFirstChild("KFScroll")
if not scroll then
    scroll = create("ScrollingFrame", {
        Name = "KFScroll", Parent = mainFrame,
        Size = UDim2.new(0.94,0,0.52,0), Position = UDim2.new(0.03,0,0.06,0),
        CanvasSize = UDim2.new(0,0,0,0), BackgroundColor3 = Color3.fromRGB(255,255,255),
        BorderColor3 = Color3.fromRGB(200,200,200)
    })
    local uiList = create("UIListLayout", {Parent = scroll, Padding = UDim.new(0,6)})
    uiList.SortOrder = Enum.SortOrder.LayoutOrder
end

local function refreshUI()
    -- limpa
    for _, child in ipairs(scroll:GetChildren()) do
        if child:IsA("TextLabel") or child:IsA("TextButton") then child:Destroy() end
    end

    local count = 0
    for i, kf in ipairs(animador:getKeyframes()) do
        count = count + 1
        local line = create("Frame", {Parent = scroll, Size = UDim2.new(1,0,0,36)})
        local lbl = create("TextLabel", {
            Parent = line, Size = UDim2.new(0.8,0,1,0), Position = UDim2.new(0,4,0,0),
            BackgroundTransparency = 1, Text = string.format("Tempo: %.3f  |  Dados: %s", kf.time, HttpService:JSONEncode(kf.data)),
            TextXAlignment = Enum.TextXAlignment.Left, Font = Enum.Font.Gotham, TextSize = 14
        })
        local del = create("TextButton", {
            Parent = line, Size = UDim2.new(0.18, -8, 1, -4), Position = UDim2.new(0.82, 4, 0, 2),
            Text = "Rem", Font = Enum.Font.GothamBold, TextSize = 14, BackgroundColor3 = Color3.fromRGB(230,80,80)
        })
        del.MouseButton1Click:Connect(function()
            animador:removeKeyframeAtIndex(i)
            refreshUI()
        end)
    end
    scroll.CanvasSize = UDim2.new(0,0,0, 40 * count)
end

-- botão adicionar
addBtn.MouseButton1Click:Connect(function()
    local t = tonumber(inputTime.Text)
    local ok, data = pcall(function() return HttpService:JSONDecode(inputData.Text) end)
    if not t or not ok then
        -- mensagem rápida no título
        title.Text = "Erro: tempo inválido ou JSON dos dados incorreto"
        wait(1.2)
        title.Text = "Animador ✨"
        return
    end
    animador:addKeyframe(t, data)
    inputTime.Text = ""
    inputData.Text = ""
    refreshUI()
end)

-- exportar json (mostra num TextBox modal)
exportJSONBtn.MouseButton1Click:Connect(function()
    local json = animador:exportToJSON()
    -- abrir uma pequena janela com o texto para copiar
    local win = create("Frame", {Parent = screenGui, Size = UDim2.new(0.6,0,0.5,0), Position = UDim2.new(0.2,0,0.25,0), BackgroundColor3 = Color3.fromRGB(245,245,250), BorderSizePixel = 2})
    local tb = create("TextBox", {Parent = win, Size = UDim2.new(1,-10,1,-40), Position = UDim2.new(0,5,0,35), MultiLine = true, Text = json, ClearTextOnFocus = false, Font = Enum.Font.Gotham})
    local ok = create("TextButton", {Parent = win, Size = UDim2.new(0.2,0,0,30), Position = UDim2.new(0.8,0,0,4), Text = "Fechar"})
    ok.MouseButton1Click:Connect(function() win:Destroy() end)
end)

-- importar json (abre prompt e tenta importar)
importJSONBtn.MouseButton1Click:Connect(function()
    local win = create("Frame", {Parent = screenGui, Size = UDim2.new(0.6,0,0.5,0), Position = UDim2.new(0.2,0,0.25,0), BackgroundColor3 = Color3.fromRGB(255,255,255), BorderSizePixel = 2})
    local tb = create("TextBox", {Parent = win, Size = UDim2.new(1,-10,1,-40), Position = UDim2.new(0,5,0,35), MultiLine = true, PlaceholderText = "Cole JSON aqui", ClearTextOnFocus = false, Font = Enum.Font.Gotham})
    local ok = create("TextButton", {Parent = win, Size = UDim2.new(0.2,0,0,30), Position = UDim2.new(0.6,0,0,4), Text = "Importar"})
    local close = create("TextButton", {Parent = win, Size = UDim2.new(0.2,0,0,30), Position = UDim2.new(0.8,0,0,4), Text = "Fechar"})
    ok.MouseButton1Click:Connect(function()
        local success, err = animador:importFromJSON(tb.Text)
        if not success then
            title.Text = "Erro: "..(err or "import")
            wait(1.4)
            title.Text = "Animador ✨"
        else
            refreshUI()
            win:Destroy()
        end
    end)
    close.MouseButton1Click:Connect(function() win:Destroy() end)
end)

-- gerar script pronto pra copiar/colar
genScriptBtn.MouseButton1Click:Connect(function()
    local scriptText = animador:exportToKeyframeSequenceString()
    local win = create("Frame", {Parent = screenGui, Size = UDim2.new(0.6,0,0.5,0), Position = UDim2.new(0.2,0,0.25,0), BackgroundColor3 = Color3.fromRGB(245,245,250), BorderSizePixel = 2})
    local tb = create("TextBox", {Parent = win, Size = UDim2.new(1,-10,1,-40), Position = UDim2.new(0,5,0,35), MultiLine = true, Text = scriptText, ClearTextOnFocus = false, Font = Enum.Font.Gotham})
    local ok = create("TextButton", {Parent = win, Size = UDim2.new(0.2,0,0,30), Position = UDim2.new(0.8,0,0,4), Text = "Fechar"})
    ok.MouseButton1Click:Connect(function() win:Destroy() end)
end)

-- inicializa UI vazia
refreshUI()f.keyframes, {
        time = time,
        data = data
    })
    table.sort(self.keyframes, function(a,b) return a.time < b.time end)
end

function KeyframeManager:getKeyframes()
    return self.keyframes
end

function KeyframeManager:exportToJSON()
    return HttpService:JSONEncode(self.keyframes)
end

function KeyframeManager:exportToKeyframeSequence()
    local sequence = {}
    for _, kf in ipairs(self.keyframes) do
        table.insert(sequence, {
            Time = kf.time,
            Value = kf.data
        })
    end
    return sequence
end

local animador = KeyframeManager.new()

local function refreshKeyframeList()
    local lines = {"Lista de Keyframes:"}
    for _, kf in ipairs(animador:getKeyframes()) do
        table.insert(lines, string.format("Tempo: %.2f, Dados: %s", kf.time, HttpService:JSONEncode(kf.data)))
    end
    keyframeList.Text = table.concat(lines, "
")
end

addButton.MouseButton1Click:Connect(function()
    playClick()
    local time = tonumber(inputTime.Text)
    local success, data = pcall(function() return HttpService:JSONDecode(inputData.Text) end)
    if not time or not success then
        keyframeList.Text = "Erro: Tempo inválido ou dados JSON incorretos."
        return
    end
    animador:addKeyframe(time, data)
    refreshKeyframeList()
    inputTime.Text = ""
    inputData.Text = ""
end)

jsonButton.MouseButton1Click:Connect(function()
    playClick()
    local jsonExport = animador:exportToJSON()
    keyframeList.Text = "Exportação JSON:
" .. jsonExport
end)

keyframeButton.MouseButton1Click:Connect(function()
    playClick()
    local seqExport = animador:exportToKeyframeSequence()
    local text = "Exportação KeyframeSequence:
"
    for _, entry in ipairs(seqExport) do
        text = text .. string.format("Time: %.2f, Value: %s
", entry.Time, HttpService:JSONEncode(entry.Value))
    end
    keyframeList.Text = text
end)

closeButton.MouseButton1Click:Connect(function()
    playClick()
    screenGui.Enabled = false
end)

-- Inicializa com refresh para mostrar vazio
refreshKeyframeList()

local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local screenGui = playerGui:FindFirstChild("AnimadorGui")
if not screenGui then
    warn("GUI não encontrada. Execute a Parte 3 antes.")
    return
end

local mainFrame = screenGui:WaitForChild("MainFrame")

-- Criar frame para seleção de scripts animados e área de exibição
local scriptFrame = Instance.new("Frame")
scriptFrame.Name = "ScriptFrame"
scriptFrame.Size = UDim2.new(0, 340, 0, 220)
scriptFrame.Position = UDim2.new(0.02, 0, 0.6, 0)
scriptFrame.BackgroundColor3 = Color3.fromRGB(230, 240, 255)
scriptFrame.BorderColor3 = Color3.fromRGB(100, 149, 237)
scriptFrame.Visible = false
scriptFrame.Parent = mainFrame

local scriptClose = Instance.new("TextButton")
scriptClose.Name = "ScriptClose"
scriptClose.Size = UDim2.new(0, 30, 0, 30)
scriptClose.Position = UDim2.new(1, -35, 0, 5)
scriptClose.Text = "X"
scriptClose.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
scriptClose.Font = Enum.Font.GothamBold
scriptClose.TextSize = 20
scriptClose.TextColor3 = Color3.new(1,1,1)
scriptClose.Parent = scriptFrame
scriptClose.AutoButtonColor = false

local scriptTitle = Instance.new("TextLabel")
scriptTitle.Name = "ScriptTitle"
scriptTitle.Size = UDim2.new(1, -40, 0, 30)
scriptTitle.Position = UDim2.new(0, 5, 0, 5)
scriptTitle.BackgroundTransparency = 1
scriptTitle.Font = Enum.Font.GothamBold
scriptTitle.TextSize = 20
scriptTitle.TextColor3 = Color3.fromRGB(30, 60, 100)
scriptTitle.TextXAlignment = Enum.TextXAlignment.Left
scriptTitle.Parent = scriptFrame

local scriptTextBox = Instance.new("TextBox")
scriptTextBox.Name = "ScriptTextBox"
scriptTextBox.Size = UDim2.new(1, -10, 1, -45)
scriptTextBox.Position = UDim2.new(0, 5, 0, 35)
scriptTextBox.MultiLine = true
scriptTextBox.ClearTextOnFocus = false
scriptTextBox.TextWrapped = true
scriptTextBox.Font = Enum.Font.Gotham
scriptTextBox.TextSize = 14
scriptTextBox.TextColor3 = Color3.new(0,0,0)
scriptTextBox.BackgroundColor3 = Color3.new(1,1,1)
scriptTextBox.Parent = scriptFrame

-- Criação dos botões para scripts animados
local animations = {"Idle", "Walk", "Run", "Jump", "Fall"}

local animButtons = {}
for i, animName in ipairs(animations) do
    local btn = Instance.new("TextButton")
    btn.Name = animName .. "Button"
    btn.Size = UDim2.new(0, 60, 0, 30)
    btn.Position = UDim2.new(0, 10 + (i-1)*65, 0, 10)
    btn.BackgroundColor3 = Color3.fromRGB(100, 149, 237)
    btn.Text = animName
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 16
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Parent = mainFrame
    btn.AutoButtonColor = false
    animButtons[animName] = btn
end

local clickSound = mainFrame:FindFirstChildOfClass("Sound") or Instance.new("Sound", mainFrame)
clickSound.SoundId = "rbxassetid://2101148"
clickSound.Volume = 0.5

local function playClick()
    clickSound:Play()
end

local animador = require(script.Parent:WaitForChild("AnimadorModule")) -- Assumindo que a Parte 2 está em AnimadorModule

-- Função geradora padrão de script animado por tipo
local function generateAnimationScript(animType)
    local keyframes = animador:getKeyframes()
    -- Script exemplo simples: define estados de animação baseados no tipo
    local scriptLines = {
        "-- Script de animação: " .. animType,
        "local Animation = {}",
        "Animation.Name = '" .. animType .. "'",
        "Animation.Keyframes = {}"
    }
    for _, kf in ipairs(keyframes) do
        table.insert(scriptLines, string.format("Animation.Keyframes[%d] = {Time = %.2f, Value = %s}", _, kf.time, HttpService:JSONEncode(kf.data)))
    end
    table.insert(scriptLines, "return Animation")
    return table.concat(scriptLines, "
")
end

-- Mostrar script na tela, atualizar título e exibir o frame
local function showScript(animType)
    playClick()
    scriptTitle.Text = "Script: " .. animType
    scriptTextBox.Text = generateAnimationScript(animType)
    scriptFrame.Visible = true
end

for animName, btn in pairs(animButtons) do
    btn.MouseButton1Click:Connect(function()
        showScript(animName)
    end)
end

scriptClose.MouseButton1Click:Connect(function()
    playClick()
    scriptFrame.Visible = false
end)
