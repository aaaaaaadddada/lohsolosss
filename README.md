local player = game.Players.LocalPlayer
local mouse = player:GetMouse()

-- Função para criar o botão
local function createButton(name, color, position, onClick)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Size = UDim2.new(0, 200, 0, 50)
    button.Position = UDim2.new(0, position.X, 0, position.Y)
    button.BackgroundColor3 = color
    button.Text = name
    button.Font = Enum.Font.SourceSans
    button.TextSize = 24
    button.TextColor3 = Color3.new(1, 1, 1)
    button.Parent = player.PlayerGui:WaitForChild("ScreenGui")

    button.MouseButton1Click:Connect(onClick)
    
    return button
end

-- Variáveis globais
local isAutoFarming = false
local stopFarming = false
local currentMeshPartIndex = 1
local debrisFolder = game.Workspace:WaitForChild("Debris") -- A pasta Debris no Workspace
local meshParts = {}

-- Função para procurar MeshParts na pasta Debris
local function findMeshParts()
    meshParts = {}
    for _, part in ipairs(debrisFolder:GetChildren()) do
        if part:IsA("MeshPart") and part:FindFirstChild("Audio") and part:FindFirstChild("Audio").Name == "Coins" then
            table.insert(meshParts, part)
        end
    end
end

-- Função para teletransportar e coletar MeshPart
local function teleportAndCollect()
    if #meshParts == 0 then
        findMeshParts()
    end

    if currentMeshPartIndex <= #meshParts then
        local meshPart = meshParts[currentMeshPartIndex]
        local targetPos = meshPart.Position + Vector3.new(0, 5, 0)

        -- Teleporta o jogador para a posição do MeshPart
        player.Character:SetPrimaryPartCFrame(CFrame.new(targetPos))

        -- Espera 1 segundo para garantir que o jogador chegou
        wait(1)
        
        -- Simula o pressionamento da tecla E
        local UserInputService = game:GetService("UserInputService")
        local inputObject = Instance.new("InputObject")
        inputObject.UserInputType = Enum.UserInputType.Keyboard
        inputObject.KeyCode = Enum.KeyCode.E
        UserInputService.InputBegan:Fire(inputObject)
        
        -- Vai para o próximo MeshPart após 1 segundo
        wait(1)
        currentMeshPartIndex = currentMeshPartIndex + 1
    else
        -- Recomeça o loop
        currentMeshPartIndex = 1
    end
end

-- Função para iniciar ou parar o AutoFarm
local function toggleAutoFarm()
    if isAutoFarming then
        isAutoFarming = false
        stopFarming = true
    else
        isAutoFarming = true
        stopFarming = false
        currentMeshPartIndex = 1
        while isAutoFarming do
            if stopFarming then
                break
            end
            teleportAndCollect()
            wait(1) -- Espera 1 segundo antes de teleportar novamente
        end
    end
end

-- Função para parar o AutoFarm
local function stopAutoFarm()
    stopFarming = true
    isAutoFarming = false
end

-- Criação dos botões na tela
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player.PlayerGui

createButton("AutoFarm", Color3.fromRGB(0, 255, 0), Vector2.new(10, 10), toggleAutoFarm)
createButton("Stop", Color3.fromRGB(255, 0, 0), Vector2.new(10, 70), stopAutoFarm)

-- Inicializa a busca pelos MeshParts
findMeshParts()
