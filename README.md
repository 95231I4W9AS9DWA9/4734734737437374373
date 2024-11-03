local UIS = game:GetService("UserInputService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer

local flickEnabled = false  -- Controla se o efeito de flick está ativo
local guiVisible = true      -- Controla se a GUI está visível
local toggleKey = Enum.KeyCode.Q  -- Tecla padrão para alternar o efeito de flick
local flickAngle = 90  -- Ângulo padrão do flick

-- Função para criar a GUI principal
local function createMainGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "FlickControlGUI"
    screenGui.Parent = player:WaitForChild("PlayerGui")

    -- Main Frame
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 300, 0, 260)
    mainFrame.Position = UDim2.new(0.5, -150, 0.2, -130)
    mainFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    mainFrame.BorderSizePixel = 0
    mainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
    mainFrame.Parent = screenGui
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Visible = guiVisible  -- A GUI é visível por padrão

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 16)
    corner.Parent = mainFrame

    -- Título
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Name = "TitleLabel"
    titleLabel.Size = UDim2.new(1, 0, 0, 30)
    titleLabel.Position = UDim2.new(0, 0, 0, 0)
    titleLabel.Text = "FlickSpecter Pro"
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextSize = 20
    titleLabel.TextXAlignment = Enum.TextXAlignment.Center
    titleLabel.Parent = mainFrame

    local statusLabel = Instance.new("TextLabel")
    statusLabel.Name = "StatusLabel"
    statusLabel.Size = UDim2.new(1, -20, 0, 30)
    statusLabel.Position = UDim2.new(0, 10, 0, 40)
    statusLabel.Text = "Flick Effect: Disabled"
    statusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Font = Enum.Font.GothamBold
    statusLabel.TextSize = 18
    statusLabel.TextXAlignment = Enum.TextXAlignment.Left
    statusLabel.Parent = mainFrame

    local toggleButton = Instance.new("TextButton")
    toggleButton.Name = "ToggleButton"
    toggleButton.Size = UDim2.new(0, 24, 0, 24)
    toggleButton.Position = UDim2.new(1, -30, 0, 6)
    toggleButton.Text = "-"
    toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleButton.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
    toggleButton.BorderSizePixel = 0
    toggleButton.Font = Enum.Font.GothamBold
    toggleButton.TextSize = 16
    toggleButton.Parent = mainFrame

    toggleButton.MouseButton1Click:Connect(function()
        guiVisible = not guiVisible
        mainFrame.Visible = guiVisible
        toggleButton.Text = guiVisible and "-" or "+"
    end)

    -- Label para mudar a tecla
    local changeKeyLabel = Instance.new("TextLabel")
    changeKeyLabel.Size = UDim2.new(1, -20, 0, 30)
    changeKeyLabel.Position = UDim2.new(0, 10, 0, 70)
    changeKeyLabel.Text = "Toggle Key: " .. toggleKey.Name
    changeKeyLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    changeKeyLabel.BackgroundTransparency = 1
    changeKeyLabel.Font = Enum.Font.Gotham
    changeKeyLabel.TextSize = 16
    changeKeyLabel.Parent = mainFrame

    -- Caixa de texto para nova tecla
    local keyInputBox = Instance.new("TextBox")
    keyInputBox.Size = UDim2.new(1, -20, 0, 30)
    keyInputBox.Position = UDim2.new(0, 10, 0, 110)
    keyInputBox.PlaceholderText = "Pressione nova tecla"
    keyInputBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    keyInputBox.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
    keyInputBox.BorderSizePixel = 0
    keyInputBox.Font = Enum.Font.Gotham
    keyInputBox.TextSize = 16
    keyInputBox.ClearTextOnFocus = true
    keyInputBox.Parent = mainFrame

    -- Label para mostrar angulação do flick
    local flickAngleLabel = Instance.new("TextLabel")
    flickAngleLabel.Size = UDim2.new(1, -20, 0, 30)
    flickAngleLabel.Position = UDim2.new(0, 10, 0, 150)
    flickAngleLabel.Text = "Flick Angle: " .. flickAngle .. "°"
    flickAngleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    flickAngleLabel.BackgroundTransparency = 1
    flickAngleLabel.Font = Enum.Font.Gotham
    flickAngleLabel.TextSize = 16
    flickAngleLabel.Parent = mainFrame

    -- Caixa de texto para configurar o ângulo de flick
    local angleInputBox = Instance.new("TextBox")
    angleInputBox.Size = UDim2.new(1, -20, 0, 30)
    angleInputBox.Position = UDim2.new(0, 10, 0, 190)
    angleInputBox.PlaceholderText = "Defina o ângulo (graus)"
    angleInputBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    angleInputBox.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
    angleInputBox.BorderSizePixel = 0
    angleInputBox.Font = Enum.Font.Gotham
    angleInputBox.TextSize = 16
    angleInputBox.ClearTextOnFocus = true
    angleInputBox.Parent = mainFrame

    return mainFrame, statusLabel, changeKeyLabel, keyInputBox, angleInputBox
end

local mainFrame, statusLabel, changeKeyLabel, keyInputBox, angleInputBox = createMainGUI()

local function updateStatusLabel()
    statusLabel.Text = "Flick Effect: " .. (flickEnabled and "Enabled" or "Disabled")
    statusLabel.TextColor3 = flickEnabled and Color3.fromRGB(85, 170, 255) or Color3.fromRGB(255, 255, 255)
end

local function flickCamera()
    local currentCam = workspace.CurrentCamera
    local initialPos = currentCam.CFrame.Position
    local initialRot = currentCam.CFrame - initialPos
    currentCam.CFrame = CFrame.new(initialPos) * CFrame.Angles(0, math.rad(flickAngle), 0) * initialRot
end

local function flickCameraBack()
    local currentCam = workspace.CurrentCamera
    local initialPos = currentCam.CFrame.Position
    local initialRot = currentCam.CFrame - initialPos
    currentCam.CFrame = CFrame.new(initialPos) * CFrame.Angles(0, math.rad(-flickAngle), 0) * initialRot
end

local function flickCameraSequence()
    flickCamera()
    wait(0.03)
    flickCameraBack()
end

-- Alternar efeito de flick com a tecla configurada
UIS.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == toggleKey then
        flickEnabled = not flickEnabled
        updateStatusLabel()
    end

    -- Acionar efeito de flick com clique do mouse se habilitado
    if not processed and flickEnabled and input.UserInputType == Enum.UserInputType.MouseButton1 then
        flickCameraSequence()
    end

    -- Minimizar/Restaurar a GUI com a tecla Insert
    if not processed and input.KeyCode == Enum.KeyCode.Insert then
        guiVisible = not guiVisible
        mainFrame.Visible = guiVisible
    end
end)

-- Atualizar a tecla de atalho
keyInputBox.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        local newKey = Enum[keyInputBox.Text]
        if newKey then
            toggleKey = newKey
            changeKeyLabel.Text = "Toggle Key: " .. toggleKey.Name
            keyInputBox.Text = ""  -- Limpa a caixa de texto
        else
            keyInputBox.Text = "Tecla Inválida"  -- Mensagem de erro
        end
    end
end)

-- Atualizar o ângulo do flick
angleInputBox.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        local newAngle = tonumber(angleInputBox.Text)
        if newAngle then
            flickAngle = newAngle
            flickAngleLabel.Text = "Flick Angle: " .. flickAngle .. "°"
            angleInputBox.Text = ""  -- Limpa a caixa de texto
        else
            angleInputBox.Text = "Ângulo Inválido"  -- Mensagem de erro
        end
    end
end)

-- A GUI é visível ao iniciar
mainFrame.Visible = guiVisible
