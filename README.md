-- Aguarda o jogo carregar completamente
repeat wait() until game:IsLoaded()

-- Variáveis principais
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart, humanoid

-- Configuração de voo
local flying = false
local speed = 100  -- Velocidade do voo
local heightLock = 0  -- Mantém a altura ao voar

-- Criando objetos de voo
local bodyVelocity = Instance.new("BodyVelocity")
local bodyGyro = Instance.new("BodyGyro")

-- Criar botão de voo na tela
local ScreenGui = Instance.new("ScreenGui")
local FlyButton = Instance.new("TextButton")

ScreenGui.Parent = game.CoreGui
FlyButton.Parent = ScreenGui
FlyButton.Size = UDim2.new(0, 200, 0, 50)
FlyButton.Position = UDim2.new(0.4, 0, 0.85, 0)  -- Ajuste a posição como quiser
FlyButton.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
FlyButton.TextSize = 20
FlyButton.Text = "Ativar Voo"
FlyButton.TextColor3 = Color3.fromRGB(255, 255, 255)

-- Função para configurar o personagem após renascer
local function setupCharacter()
    character = player.Character or player.CharacterAdded:Wait()
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoid = character:FindFirstChildOfClass("Humanoid")

    -- Evitar a morte ao cair ou se mover durante o voo
    humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
    humanoid:SetStateEnabled(Enum.HumanoidStateType.Freefall, false)

    -- Se estava voando, ativa novamente o voo
    if flying then
        toggleFly(true)
    end
end

-- Configurar personagem inicial ou após morrer
player.CharacterAdded:Connect(setupCharacter)

-- Função para ativar/desativar o voo
local function toggleFly(forceEnable)
    if forceEnable then
        flying = true
    else
        flying = not flying
    end

    if flying then
        -- Define altura fixa ao ativar o voo
        heightLock = humanoidRootPart.Position.Y

        -- Configuração de voo
        bodyVelocity.Parent = humanoidRootPart
        bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)

        bodyGyro.Parent = humanoidRootPart
        bodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)

        -- Torna o personagem imortal enquanto voa
        humanoid.Health = math.huge
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)

        -- Desativa animações de movimento (braços e pernas)
        humanoid:ChangeState(Enum.HumanoidStateType.Physics)

        FlyButton.Text = "Desativar Voo"
        FlyButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    else
        -- Desativa o voo
        bodyVelocity:Destroy()
        bodyGyro:Destroy()

        -- Restaura a capacidade de morrer
        humanoid.Health = 100
        humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)

        FlyButton.Text = "Ativar Voo"
        FlyButton.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    end
end

-- Atualiza a movimentação do voo conforme a direção do analógico
game:GetService("RunService").RenderStepped:Connect(function()
    if flying then
        local moveDirection = humanoid.MoveDirection
        local cameraDirection = workspace.CurrentCamera.CFrame.LookVector

        -- Voar apenas se o jogador mover o analógico
        if moveDirection.Magnitude > 0 then
            local flyDirection = (cameraDirection * 1.5 + moveDirection).unit
            bodyVelocity.Velocity = flyDirection * speed
            bodyGyro.CFrame = CFrame.new(humanoidRootPart.Position, humanoidRootPart.Position + flyDirection)
        else
            -- Se parado, mantém a altura constante
            bodyVelocity.Velocity = Vector3.new(0, (heightLock - humanoidRootPart.Position.Y) * 2, 0)
        end
    end
end)

-- Quando o botão for pressionado, ativa ou desativa o voo
FlyButton.MouseButton1Click:Connect(toggleFly)

-- Mensagem para confirmar a ativação do script
game.StarterGui:SetCore("SendNotification", {
    Title = "Script de Voo Ativado!";
    Text = "Use o analógico e a câmera para voar.";
    Duration = 5;
})

-- Configura o personagem quando o script é executado pela primeira vez
setupCharacter()
