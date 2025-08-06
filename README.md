--[[
    Blox Fruits Open Source Script
    Autor: Comunidade Open Source
    Versão: 1.0
    
    AVISO: Use por sua própria conta e risco.
    Este script é apenas para fins educacionais.
]]--

-- Serviços necessários
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")

-- Variáveis principais
local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")

-- Configurações do script
local Config = {
    AutoFarm = false,
    AutoQuest = false,
    WalkSpeed = 16,
    JumpPower = 50,
    NoClip = false,
    InfiniteJump = false,
    TeleportSpeed = 50
}

-- GUI Principal
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local TitleLabel = Instance.new("TextLabel")
local CloseButton = Instance.new("TextButton")

-- Configuração da GUI
ScreenGui.Name = "BloxFruitsGUI"
ScreenGui.Parent = game.CoreGui
ScreenGui.ResetOnSpawn = false

MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Position = UDim2.new(0.1, 0, 0.1, 0)
MainFrame.Size = UDim2.new(0, 400, 0, 500)
MainFrame.Active = true
MainFrame.Draggable = true

-- Título
TitleLabel.Name = "TitleLabel"
TitleLabel.Parent = MainFrame
TitleLabel.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
TitleLabel.BorderSizePixel = 0
TitleLabel.Size = UDim2.new(1, 0, 0, 40)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.Text = "Blox Fruits Open Source"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.TextSize = 16

-- Botão fechar
CloseButton.Name = "CloseButton"
CloseButton.Parent = TitleLabel
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
CloseButton.BorderSizePixel = 0
CloseButton.Position = UDim2.new(1, -30, 0, 5)
CloseButton.Size = UDim2.new(0, 25, 0, 25)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.Text = "X"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.TextSize = 14

CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

-- Função para criar botões
local function createButton(name, text, position, callback)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Parent = MainFrame
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    button.BorderSizePixel = 0
    button.Position = position
    button.Size = UDim2.new(0, 180, 0, 30)
    button.Font = Enum.Font.Gotham
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 12
    
    button.MouseButton1Click:Connect(callback)
    
    -- Efeito hover
    button.MouseEnter:Connect(function()
        button.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    end)
    
    button.MouseLeave:Connect(function()
        button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    end)
    
    return button
end

-- Função para criar sliders
local function createSlider(name, text, position, min, max, default, callback)
    local frame = Instance.new("Frame")
    frame.Name = name .. "Frame"
    frame.Parent = MainFrame
    frame.BackgroundTransparency = 1
    frame.Position = position
    frame.Size = UDim2.new(0, 180, 0, 50)
    
    local label = Instance.new("TextLabel")
    label.Parent = frame
    label.BackgroundTransparency = 1
    label.Size = UDim2.new(1, 0, 0, 20)
    label.Font = Enum.Font.Gotham
    label.Text = text .. ": " .. default
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.TextSize = 12
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local slider = Instance.new("Frame")
    slider.Parent = frame
    slider.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    slider.BorderSizePixel = 0
    slider.Position = UDim2.new(0, 0, 0, 25)
    slider.Size = UDim2.new(1, 0, 0, 5)
    
    local fill = Instance.new("Frame")
    fill.Parent = slider
    fill.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
    fill.BorderSizePixel = 0
    fill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    
    local dragging = false
    
    slider.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
        end
    end)
    
    slider.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local mouse = UserInputService:GetMouseLocation()
            local sliderPos = slider.AbsolutePosition
            local sliderSize = slider.AbsoluteSize
            
            local percentage = math.clamp((mouse.X - sliderPos.X) / sliderSize.X, 0, 1)
            local value = math.floor(min + (max - min) * percentage)
            
            fill.Size = UDim2.new(percentage, 0, 1, 0)
            label.Text = text .. ": " .. value
            
            callback(value)
        end
    end)
end

-- Funções principais
local function toggleAutoFarm()
    Config.AutoFarm = not Config.AutoFarm
    print("Auto Farm:", Config.AutoFarm and "Ativado" or "Desativado")
end

local function toggleNoClip()
    Config.NoClip = not Config.NoClip
    print("No Clip:", Config.NoClip and "Ativado" or "Desativado")
    
    if Config.NoClip then
        for _, part in pairs(Character:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    else
        for _, part in pairs(Character:GetChildren()) do
            if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                part.CanCollide = true
            end
        end
    end
end

local function toggleInfiniteJump()
    Config.InfiniteJump = not Config.InfiniteJump
    print("Infinite Jump:", Config.InfiniteJump and "Ativado" or "Desativado")
end

local function teleportToSpawn()
    if RootPart then
        RootPart.CFrame = CFrame.new(0, 50, 0)
        print("Teleportado para o spawn!")
    end
end

-- Criar interface
createButton("AutoFarmBtn", "Toggle Auto Farm", UDim2.new(0, 10, 0, 60), toggleAutoFarm)
createButton("NoClipBtn", "Toggle No Clip", UDim2.new(0, 210, 0, 60), toggleNoClip)
createButton("InfiniteJumpBtn", "Toggle Infinite Jump", UDim2.new(0, 10, 0, 100), toggleInfiniteJump)
createButton("TeleportSpawnBtn", "Teleport to Spawn", UDim2.new(0, 210, 0, 100), teleportToSpawn)

-- Sliders
createSlider("WalkSpeed", "Walk Speed", UDim2.new(0, 10, 0, 150), 16, 200, 16, function(value)
    Config.WalkSpeed = value
    if Humanoid then
        Humanoid.WalkSpeed = value
    end
end)

createSlider("JumpPower", "Jump Power", UDim2.new(0, 210, 0, 150), 50, 300, 50, function(value)
    Config.JumpPower = value
    if Humanoid then
        Humanoid.JumpPower = value
    end
end)

-- Sistema de infinite jump
UserInputService.JumpRequest:Connect(function()
    if Config.InfiniteJump and Humanoid then
        Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-- Sistema de no clip
RunService.Stepped:Connect(function()
    if Config.NoClip and Character then
        for _, part in pairs(Character:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end)

-- Auto Farm básico (exemplo)
spawn(function()
    while wait(1) do
        if Config.AutoFarm then
            -- Aqui você pode adicionar lógica de auto farm
            -- Por exemplo, atacar inimigos próximos
            local enemies = Workspace.Enemies:GetChildren()
            for _, enemy in pairs(enemies) do
                if enemy:FindFirstChild("HumanoidRootPart") and enemy:FindFirstChild("Humanoid") then
                    if enemy.Humanoid.Health > 0 then
                        local distance = (RootPart.Position - enemy.HumanoidRootPart.Position).Magnitude
                        if distance < 50 then
                            -- Teleportar para o inimigo
                            RootPart.CFrame = enemy.HumanoidRootPart.CFrame * CFrame.new(0, 0, 5)
                            
                            -- Atacar (você precisa adaptar isso para as mechanics do jogo)
                            -- game:GetService("Players").LocalPlayer.Character.Combat.Attack:FireServer()
                            break
                        end
                    end
                end
            end
        end
    end
end)

-- Atualizar referências do personagem
LocalPlayer.CharacterAdded:Connect(function(newCharacter)
    Character = newCharacter
    Humanoid = Character:WaitForChild("Humanoid")
    RootPart = Character:WaitForChild("HumanoidRootPart")
    
    -- Aplicar configurações
    Humanoid.WalkSpeed = Config.WalkSpeed
    Humanoid.JumpPower = Config.JumpPower
end)

print("Blox Fruits Open Source Script carregado!")
print("Pressione F1 para mostrar/ocultar a GUI")

-- Toggle GUI com F1
UserInputService.InputBegan:Connect(function(key, gameProcessed)
    if not gameProcessed and key.KeyCode == Enum.KeyCode.F1 then
        MainFrame.Visible = not MainFrame.Visible
    end
end)
