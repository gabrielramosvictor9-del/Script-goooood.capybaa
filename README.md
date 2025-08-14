-- Referências principais
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer

-- Usa PlayerGui ao invés de CoreGui
local gui = Instance.new("ScreenGui")
gui.Name = "SideMenuGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

-- Interface Lateral centralizada
local menu = Instance.new("Frame")
menu.Size = UDim2.new(0, 0, 0, 260) -- altura menor
menu.Position = UDim2.new(0.5, -90, 0.5, -130) -- centralizado
menu.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
menu.BorderSizePixel = 0
menu.ClipsDescendants = true
menu.Visible = true
menu.Parent = gui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 25) -- menor
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
title.Text = "goooood.capybara"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 16
title.Parent = menu

-- Botão toggle no topo da tela
local toggle = Instance.new("TextButton")
toggle.Size = UDim2.new(0, 35, 0, 35)
toggle.Position = UDim2.new(0.5, -17, 0.5, -180) -- acima do menu
toggle.Text = "≡"
toggle.Font = Enum.Font.SourceSansBold
toggle.TextSize = 24
toggle.TextColor3 = Color3.new(1, 1, 1)
toggle.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
toggle.Parent = gui

local openSize = UDim2.new(0, 180, 0, 260)
local closedSize = UDim2.new(0, 0, 0, 260)
local open = false
local buttons = {}

local function createButton(name, yOffset, color)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 150, 0, 28) -- botão menor
    btn.Position = UDim2.new(0, 15, 0, yOffset)
    btn.BackgroundColor3 = color or Color3.fromRGB(0, 120, 215)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 14
    btn.Text = name
    btn.Visible = false
    btn.Parent = menu
    table.insert(buttons, btn)
    return btn
end

toggle.MouseButton1Click:Connect(function()
    open = not open
    TweenService:Create(menu, TweenInfo.new(0.25), { Size = open and openSize or closedSize }):Play()
    title.Visible = open
    for _, btn in pairs(buttons) do btn.Visible = open end
end)

-- Função ESP
local espEnabled = false
local espList = {}
local espConnections = {}

local function applyESP(plr)
    if not plr.Character or not plr.Character:FindFirstChild("HumanoidRootPart") then return end
    if plr.Character:FindFirstChild("ESP_GUI") then return end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP_GUI"
    billboard.Adornee = plr.Character:FindFirstChild("HumanoidRootPart")
    billboard.Size = UDim2.new(0, 150, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 2, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = plr.Character

    local label = Instance.new("TextLabel", billboard)
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.new(1, 1, 1)
    label.TextStrokeTransparency = 0
    label.TextScaled = true

    table.insert(espList, billboard)

    local conn = RunService.RenderStepped:Connect(function()
        if espEnabled and plr.Character and plr.Character:FindFirstChild("Humanoid") then
            local hum = plr.Character:FindFirstChild("Humanoid")
            label.Text = string.format("%s\n%.0f/%.0f", plr.Name, hum.Health, hum.MaxHealth)
        end
    end)
    table.insert(espConnections, conn)
end

-- Botões compactos
local tp = createButton("Teleport Tool", 30, Color3.fromRGB(0, 140, 200))
tp.MouseButton1Click:Connect(function()
    pcall(function()
        loadstring(game:HttpGet("https://rawscripts.net/raw/Universal-Script-Teleport-Tools-34066"))()
    end)
end)

local speed = createButton("Velocidade", 65, Color3.fromRGB(0, 170, 0))
local speedLevels = {16, 50, 100}
local speedIdx = 1
speed.MouseButton1Click:Connect(function()
    local hum = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
    if hum then
        speedIdx = speedIdx % #speedLevels + 1
        hum.WalkSpeed = speedLevels[speedIdx]
        speed.Text = "Velocidade: " .. speedLevels[speedIdx]
    end
end)

local jump = createButton("Super Pulo", 100, Color3.fromRGB(200, 120, 0))
local jumping = false
jump.MouseButton1Click:Connect(function()
    local hum = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
    if hum then
        jumping = not jumping
        hum.JumpPower = jumping and 150 or 50
        jump.Text = jumping and "Super Pulo: ON" or "Super Pulo: OFF"
    end
end)

local noclip = createButton("Noclip", 135, Color3.fromRGB(170, 0, 170))
local clipOn = false
local connNoclip
noclip.MouseButton1Click:Connect(function()
    local char = player.Character
    if not char then return end
    clipOn = not clipOn
    noclip.Text = clipOn and "Noclip: ON" or "Noclip: OFF"
    if clipOn then
        connNoclip = RunService.Stepped:Connect(function()
            for _, v in pairs(char:GetDescendants()) do
                if v:IsA("BasePart") then
                    v.CanCollide = false
                end
            end
        end)
    else
        if connNoclip then connNoclip:Disconnect() connNoclip = nil end
        for _, v in pairs(char:GetDescendants()) do
            if v:IsA("BasePart") then v.CanCollide = true end
        end
    end
end)

local espButton = createButton("ESP (Health)", 170, Color3.fromRGB(0, 170, 170))
espButton.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    espButton.Text = espEnabled and "ESP (Health): ON" or "ESP (Health): OFF"

    if espEnabled then
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= player then
                applyESP(plr)
                plr.CharacterAdded:Connect(function()
                    wait(1)
                    if espEnabled then applyESP(plr) end
                end)
            end
        end
    else
        for _, gui in ipairs(espList) do
            if gui and gui.Parent then
                gui:Destroy()
            end
        end
        espList = {}
        for _, conn in ipairs(espConnections) do
            if conn then conn:Disconnect() end
        end
        espConnections = {}
    end
end)

-- Fly
local flyButton = createButton("Fly", 205, Color3.fromRGB(0, 120, 215))
flyButton.MouseButton1Click:Connect(function()
    local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
    pcall(function()
        if isMobile then
            loadstring(game:HttpGet("https://raw.githubusercontent.com/396abc/Script/refs/heads/main/MobileFly.lua", true))()
        else
            loadstring(game:HttpGet("https://raw.githubusercontent.com/396abc/Script/refs/heads/main/FlyR15.lua", true))()
        end
    end)
end)
