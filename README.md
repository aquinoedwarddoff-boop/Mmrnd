-- LocalScript: Coloque em StarterPlayerScripts
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Criar a interface gráfica
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "EdwardFloatSystem"
screenGui.ResetOnSpawn = false

-- Frame principal
local mainFrame = Instance.new("Frame")
mainFrame.Name = "EdwardFrame"
mainFrame.Size = UDim2.new(0, 180, 0, 110)
mainFrame.Position = UDim2.new(0, 20, 0, 20)
mainFrame.BackgroundColor3 = Color3.fromRGB(0, 100, 200)
mainFrame.BorderSizePixel = 0
mainFrame.BackgroundTransparency = 0.1
mainFrame.Active = true

-- Arredondar cantos do frame principal
local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 12)
mainCorner.Parent = mainFrame

-- Sombra suave
local shadow = Instance.new("UIStroke")
shadow.Color = Color3.fromRGB(255, 255, 255)
shadow.Thickness = 1
shadow.Transparency = 0.8
shadow.Parent = mainFrame

-- Título Edward
local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "EdwardTitle"
titleLabel.Size = UDim2.new(0.8, 0, 0.2, 0)
titleLabel.Position = UDim2.new(0.1, 0, -0.25, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "EDWARD SYSTEM"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextScaled = true
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 12
titleLabel.Parent = mainFrame

-- Botão de salvar posição
local saveButton = Instance.new("TextButton")
saveButton.Name = "EdwardSaveButton"
saveButton.Size = UDim2.new(0.85, 0, 0.35, 0)
saveButton.Position = UDim2.new(0.075, 0, 0.15, 0)
saveButton.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
saveButton.BorderSizePixel = 0
saveButton.BackgroundTransparency = 0.1
saveButton.Text = "SALVAR"
saveButton.TextColor3 = Color3.fromRGB(255, 255, 255)
saveButton.TextScaled = true
saveButton.Font = Enum.Font.GothamMedium
saveButton.TextSize = 14
saveButton.ZIndex = 2

local saveCorner = Instance.new("UICorner")
saveCorner.CornerRadius = UDim.new(0, 8)
saveCorner.Parent = saveButton

local saveShadow = Instance.new("UIStroke")
saveShadow.Color = Color3.fromRGB(200, 230, 255)
saveShadow.Thickness = 1
saveShadow.Transparency = 0.7
saveShadow.Parent = saveButton

-- Botão de flutuar para posição salva
local floatButton = Instance.new("TextButton")
floatButton.Name = "EdwardFloatButton"
floatButton.Size = UDim2.new(0.85, 0, 0.35, 0)
floatButton.Position = UDim2.new(0.075, 0, 0.55, 0)
floatButton.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
floatButton.BorderSizePixel = 0
floatButton.BackgroundTransparency = 0.1
floatButton.Text = "FLUTUAR"
floatButton.TextColor3 = Color3.fromRGB(255, 255, 255)
floatButton.TextScaled = true
floatButton.Font = Enum.Font.GothamMedium
floatButton.TextSize = 14
floatButton.ZIndex = 2

local floatCorner = Instance.new("UICorner")
floatCorner.CornerRadius = UDim.new(0, 8)
floatCorner.Parent = floatButton

local floatShadow = Instance.new("UIStroke")
floatShadow.Color = Color3.fromRGB(200, 230, 255)
floatShadow.Thickness = 1
floatShadow.Transparency = 0.7
floatShadow.Parent = floatButton

-- Montar a interface
saveButton.Parent = mainFrame
floatButton.Parent = mainFrame
mainFrame.Parent = screenGui
screenGui.Parent = playerGui

-- Variáveis de controle
local isFloating = false
local isDragging = false
local dragStartPosition = nil
local frameStartPosition = nil
local floatConnection = nil
local savedPosition = nil

-- Função para mostrar notificação
local function showNotification(message, color)
    local notification = Instance.new("TextLabel")
    notification.Name = "EdwardNotification"
    notification.Size = UDim2.new(0, 200, 0, 35)
    notification.Position = UDim2.new(0.5, -100, 0.1, 0)
    notification.BackgroundColor3 = Color3.fromRGB(0, 100, 200)
    notification.BackgroundTransparency = 0.2
    notification.Text = "EDWARD: " .. message
    notification.TextColor3 = Color3.fromRGB(255, 255, 255)
    notification.TextScaled = true
    notification.Font = Enum.Font.GothamMedium
    notification.BorderSizePixel = 0
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = notification
    
    local notifShadow = Instance.new("UIStroke")
    notifShadow.Color = Color3.fromRGB(255, 255, 255)
    notifShadow.Thickness = 1
    notifShadow.Transparency = 0.8
    notifShadow.Parent = notification
    
    notification.Parent = screenGui
    
    -- Animação de fade out
    wait(2)
    
    local tweenInfo = TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local tween = TweenService:Create(notification, tweenInfo, {BackgroundTransparency = 1, TextTransparency = 1})
    tween:Play()
    
    tween.Completed:Connect(function()
        notification:Destroy()
    end)
end

-- Função para salvar posição
local function savePosition()
    local character = player.Character
    if not character then
        showNotification("Personagem não encontrado!")
        return
    end
    
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then
        showNotification("HumanoidRootPart não encontrado!")
        return
    end
    
    savedPosition = humanoidRootPart.Position
    showNotification("Posição salva!")
    
    -- Atualizar aparência do botão de flutuar
    floatButton.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
    floatButton.Text = "FLUTUAR ✓"
end

-- Função para iniciar a flutuação
local function startFloating()
    if isFloating then 
        stopFloating()
        return
    end
    
    if not savedPosition then
        showNotification("Nenhuma posição salva!")
        return
    end
    
    local character = player.Character
    if not character then
        showNotification("Personagem não encontrado!")
        return
    end
    
    local humanoid = character:FindFirstChild("Humanoid")
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    
    if not humanoid or not humanoidRootPart then
        showNotification("Humanoid ou HumanoidRootPart não encontrado!")
        return
    end
    
    isFloating = true
    floatButton.Text = "PARAR"
    floatButton.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
    
    -- Posição de destino com altura fixa de 5 unidades
    local targetPosition = Vector3.new(savedPosition.X, savedPosition.Y + 5, savedPosition.Z)
    
    showNotification("Flutuando...")
    
    humanoid.PlatformStand = true
    
    floatConnection = RunService.Heartbeat:Connect(function(deltaTime)
        if not character or not humanoidRootPart or not isFloating then
            if floatConnection then
                floatConnection:Disconnect()
            end
            return
        end
        
        local direction = (targetPosition - humanoidRootPart.Position)
        local distance = direction.Magnitude
        
        -- Se estiver muito perto, para a flutuação
        if distance < 3 then
            stopFloating()
            humanoid.PlatformStand = false
            showNotification("Chegou ao destino!")
            return
        end
        
        direction = direction.Unit
        
        -- Velocidade fixa de 26
        local speed = 26
        
        -- Aplica velocidade em todas as direções
        humanoidRootPart.Velocity = direction * speed
        
        -- Mantém altura constante de ~5 unidades com pequeno ajuste
        local currentHeight = humanoidRootPart.Position.Y
        local desiredHeight = savedPosition.Y + 5
        
        -- Pequeno ajuste para manter a altura consistente
        if currentHeight < desiredHeight - 1 then
            humanoidRootPart.Velocity = humanoidRootPart.Velocity + Vector3.new(0, 5, 0)
        elseif currentHeight > desiredHeight + 1 then
            humanoidRootPart.Velocity = humanoidRootPart.Velocity + Vector3.new(0, -3, 0)
        end
    end)
end

-- Função para parar a flutuação
local function stopFloating()
    if not isFloating then return end
    
    isFloating = false
    
    -- Resetar botão
    if savedPosition then
        floatButton.Text = "FLUTUAR ✓"
        floatButton.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
    else
        floatButton.Text = "FLUTUAR"
        floatButton.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    end
    
    if floatConnection then
        floatConnection:Disconnect()
        floatConnection = nil
    end
    
    local character = player.Character
    if character then
        local humanoid = character:FindFirstChild("Humanoid")
        local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
        
        if humanoid then
            humanoid.PlatformStand = false
        end
        
        if humanoidRootPart then
            humanoidRootPart.Velocity = Vector3.new(0, 0, 0)
        end
    end
end

-- Eventos dos botões
saveButton.MouseButton1Click:Connect(function()
    savePosition()
end)

floatButton.MouseButton1Click:Connect(function()
    if isFloating then
        stopFloating()
    else
        startFloating()
    end
end)

-- Efeitos visuais nos botões
local function setupButtonEffects(button)
    button.MouseEnter:Connect(function()
        button.BackgroundTransparency = 0
        if button == floatButton and not isFloating and savedPosition then
            button.BackgroundColor3 = Color3.fromRGB(0, 180, 120)
        end
    end)
    
    button.MouseLeave:Connect(function()
        button.BackgroundTransparency = 0.1
        if button == floatButton and not isFloating then
            if savedPosition then
                button.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
            else
                button.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
            end
        end
    end)
    
    button.MouseButton1Down:Connect(function()
        button.BackgroundTransparency = 0.2
    end)
    
    button.MouseButton1Up:Connect(function()
        button.BackgroundTransparency = 0.1
    end)
end

setupButtonEffects(saveButton)
setupButtonEffects(floatButton)

-- Sistema de arrastar para o frame principal
local function startDrag(input)
    isDragging = true
    dragStartPosition = Vector2.new(input.Position.X, input.Position.Y)
    frameStartPosition = mainFrame.Position
    
    mainFrame.BackgroundTransparency = 0.3
end

local function updateDrag(input)
    if not isDragging then return end
    
    local delta = Vector2.new(
        input.Position.X - dragStartPosition.X,
        input.Position.Y - dragStartPosition.Y
    )
    
    mainFrame.Position = UDim2.new(
        frameStartPosition.X.Scale,
        frameStartPosition.X.Offset + delta.X,
        frameStartPosition.Y.Scale,
        frameStartPosition.Y.Offset + delta.Y
    )
end

local function endDrag()
    isDragging = false
    mainFrame.BackgroundTransparency = 0.1
end

-- Conectar eventos de drag
mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        startDrag(input)
    end
end)

mainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and isDragging then
        updateDrag(input)
    end
end)

mainFrame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        endDrag()
    end
end)

-- Limpar quando o personagem morrer
player.CharacterAdded:Connect(function(character)
    stopFloating()
    character:WaitForChild("HumanoidRootPart")
end)

player.CharacterRemoving:Connect(function()
    stopFloating()
end)

-- Função para mostrar/ocultar a interface
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.H then
        mainFrame.Visible = not mainFrame.Visible
    end
end)

-- Mensagem inicial
print("=== EDWARD FLOAT SYSTEM ===")
print("Sistema personalizado para Edward carregado!")
print("Instruções:")
print("- Clique em 'SALVAR' para salvar onde você está")
print("- Clique em 'FLUTUAR' para flutuar até lá")
print("- Clique novamente para parar")
print("- Arraste a janela para mover")
print("- Pressione H para mostrar/ocultar")
print("===========================")
