local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

--////////////////////////// VARIÁVEIS /////////////////////////////////////
local plr = game.Players.LocalPlayer
local char = plr.Character or plr.CharacterAdded:Wait()
local aimbotEnabled = false
local aimSensitivity = 0.3
local fovSize = 100
local fovCircle = Drawing.new("Circle")
fovCircle.Color = Color3.fromRGB(255, 255, 255)
fovCircle.Thickness = 2
fovCircle.NumSides = 100
fovCircle.Radius = fovSize
fovCircle.Filled = false
fovCircle.Visible = false
local aimDistance = 100 -- Valor padrão
local BoxESPEnabled = false
local NameTagsEnabled = false
local AimbotEnabled = false
local flyEnabled = false
local noClipEnabled = false
local speedValue = 75 -- Velocidade padrão
local ESPObjects = {}
local NameTags = {}


-- Funções para Box ESP
local function CreateBox()
    local box = Drawing.new("Square")
    box.Thickness = 2.5
    box.Color = Color3.fromRGB(255, 255, 255)
    box.Filled = false
    box.Visible = false
    return box
end

local function UpdateBox(box, rootPart)
    local viewportPoint, onScreen = workspace.CurrentCamera:WorldToViewportPoint(rootPart.Position)
    if onScreen then
        local size = Vector2.new(2000 / viewportPoint.Z, 4000 / viewportPoint.Z)
        box.Size = size
        box.Position = Vector2.new(viewportPoint.X - size.X / 2, viewportPoint.Y - size.Y / 2)
        box.Visible = true
    else
        box.Visible = false
    end
end


local function IsFriendly(player)
    if player.Team == plr.Team then
        return true
    end
    if player:IsFriendsWith(plr.UserId) then
        return true
    end
    return false
end


-- Atualizar Box ESP
local function HandleExistingPlayersESP()
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= plr and not IsFriendly(player) and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local box = CreateBox()
            ESPObjects[player.Name] = box
            coroutine.wrap(function()
                while BoxESPEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") do
                    UpdateBox(box, player.Character.HumanoidRootPart)
                    wait(0.01)
                end
                if ESPObjects[player.Name] then
                    ESPObjects[player.Name]:Remove()
                    ESPObjects[player.Name] = nil
                end
            end)()
        end
    end
end

local function ToggleBoxESP(state)
    BoxESPEnabled = state
    if BoxESPEnabled then
        HandleExistingPlayersESP() -- Lida com jogadores já existentes
        game.Players.PlayerAdded:Connect(function(player)
            player.CharacterAdded:Connect(function(character)
                if character:FindFirstChild("HumanoidRootPart") then
                    local box = CreateBox()
                    ESPObjects[player.Name] = box
                    coroutine.wrap(function()
                        while BoxESPEnabled and character:FindFirstChild("HumanoidRootPart") do
                            UpdateBox(box, character.HumanoidRootPart)
                            wait(0.03)
                        end
                        if ESPObjects[player.Name] then
                            ESPObjects[player.Name]:Remove()
                            ESPObjects[player.Name] = nil
                        end
                    end)()
                end
            end)
        end)
    else
        for _, box in pairs(ESPObjects) do
            box:Remove()
        end
        ESPObjects = {}
    end
end

-- Funções para Nametags
local function CreateNameTag()
    local nametag = Drawing.new("Text")
    nametag.Size = 17.5
    nametag.Color = Color3.fromRGB(255, 255, 255)
    nametag.Visible = false
    return nametag
end

local function UpdateNameTag(nametag, rootPart)
    local viewportPoint, onScreen = workspace.CurrentCamera:WorldToViewportPoint(rootPart.Position)
    if onScreen then
        nametag.Position = Vector2.new(viewportPoint.X, viewportPoint.Y - 20)
        nametag.Visible = true
    else
        nametag.Visible = false
    end
end

-- Atualizar Nametags
local function HandleExistingPlayersNameTags()
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= plr and not IsFriendly(player) and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local nametag = CreateNameTag()
            NameTags[player.Name] = nametag
            coroutine.wrap(function()
                while NameTagsEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") do
                    nametag.Text = player.Name
                    UpdateNameTag(nametag, player.Character.HumanoidRootPart)
                    wait(0.01)
                end
                if NameTags[player.Name] then
                    NameTags[player.Name]:Remove()
                    NameTags[player.Name] = nil
                end
            end)()
        end
    end
end

local function ToggleNameTags(state)
    NameTagsEnabled = state
    if NameTagsEnabled then
        HandleExistingPlayersNameTags() -- Lida com jogadores já existentes
        game.Players.PlayerAdded:Connect(function(player)
            player.CharacterAdded:Connect(function(character)
                if character:FindFirstChild("HumanoidRootPart") then
                    local nametag = CreateNameTag()
                    NameTags[player.Name] = nametag
                    coroutine.wrap(function()
                        while NameTagsEnabled and character:FindFirstChild("HumanoidRootPart") do
                            nametag.Text = player.Name
                            UpdateNameTag(nametag, character.HumanoidRootPart)
                            wait(0.03)
                        end
                        if NameTags[player.Name] then
                            NameTags[player.Name]:Remove()
                            NameTags[player.Name] = nil
                        end
                    end)()
                end
            end)
        end)
    else
        for _, nametag in pairs(NameTags) do
            nametag:Remove()
        end
        NameTags = {}
    end
end


-- Funções para atualizar o FOV
function updateFovCircle()
    fovCircle.Radius = fovSize
    fovCircle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y / 2)
end

-- Atualizar Aimbot
function enableAimbot(state)
    aimbotEnabled = state
    fovCircle.Visible = state
    if aimbotEnabled then
        game:GetService("RunService").RenderStepped:Connect(function()
            if aimbotEnabled then
                local camera = workspace.CurrentCamera
                local closestTarget = nil
                local shortestDistance = math.huge

                for _, player in pairs(game.Players:GetPlayers()) do
                    if player ~= plr and not IsFriendly(player) and player.Character and player.Character:FindFirstChild("Head") then
                        local head = player.Character.Head
                        local screenPoint, onScreen = camera:WorldToScreenPoint(head.Position)
                        local mousePosition = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
                        local distanceFromMouse = (Vector2.new(screenPoint.X, screenPoint.Y) - mousePosition).Magnitude

                        -- Verifica se o jogador está dentro do FOV e dentro da distância máxima
                        local playerDistance = (head.Position - camera.CFrame.Position).Magnitude
                        if onScreen and distanceFromMouse < fovSize and distanceFromMouse < shortestDistance and playerDistance <= aimDistance then
                            shortestDistance = distanceFromMouse
                            closestTarget = head
                        end
                    end
                end

                if closestTarget then
                    local targetPosition = closestTarget.Position
                    local aimDirection = (targetPosition - camera.CFrame.Position).unit
                    camera.CFrame = CFrame.new(camera.CFrame.Position, camera.CFrame.Position + aimDirection * aimSensitivity)
                end
            end
        end)
    end
end

-- Funções para atualizar o FOV
function updateflyspeed()
    speedValue = Speed 
end

-- Função para ativar/desativar o voo
function toggleFly(state)
    flyEnabled = state
    local player = game.Players.LocalPlayer

    if flyEnabled then
        local humanoid = player.Character:WaitForChild("Humanoid")
        humanoid.PlatformStand = true

        bodyGyro = Instance.new("BodyGyro")
        bodyGyro.CFrame = player.Character.HumanoidRootPart.CFrame
        bodyGyro.P = 9e4
        bodyGyro.MaxTorque = Vector3.new(9e4, 9e4, 9e4)
        bodyGyro.Parent = player.Character.HumanoidRootPart

        bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.MaxForce = Vector3.new(9e4, 9e4, 9e4)
        bodyVelocity.Parent = player.Character.HumanoidRootPart

        -- Controlando o movimento durante o voo
        game:GetService("RunService").RenderStepped:Connect(function()
            if flyEnabled then
                local camera = workspace.CurrentCamera
                local moveDirection = Vector3.new(0, 0, 0)

                if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.W) then
                    moveDirection = moveDirection + camera.CFrame.LookVector
                end
                if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.S) then
                    moveDirection = moveDirection - camera.CFrame.LookVector
                end
                if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.A) then
                    moveDirection = moveDirection - camera.CFrame.RightVector
                end
                if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.D) then
                    moveDirection = moveDirection + camera.CFrame.RightVector
                end

                bodyVelocity.Velocity = moveDirection * speedValue -- Ajusta a velocidade do voo aqui
                bodyGyro.CFrame = camera.CFrame
            end
        end)
    else
        if bodyGyro then bodyGyro:Destroy() end
        if bodyVelocity then bodyVelocity:Destroy() end
        player.Character.Humanoid.PlatformStand = false
    end
end

-- Função para ativar/desativar o NoClip
function toggleNoClip(state)
    noClipEnabled = state
    local character = plr.Character or plr.CharacterAdded:Wait()

    if noClipEnabled then
        for _, part in pairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false -- Desativa a colisão das partes do personagem
            end
        end
    else
        for _, part in pairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = true -- Reativa a colisão das partes do personagem
            end
        end
    end
end

-- Criar janela Fluent
local Window = Fluent:CreateWindow({
    Title = "Japa Cheat",
    SubTitle = "Powered by Japa",
    TabWidth = 160,
    Size = UDim2.fromOffset(1000, 500),
    Acrylic = false,
    transparency = false,
    Theme = "Light",
    MinimizeKey = Enum.KeyCode.RightShift -- Used when theres no MinimizeKeybind
})

-- Criar Tabs
local Tabs = {
    AimBot = Window:AddTab({ Title = "Aimbot", Icon = "crosshair" }),
    Self = Window:AddTab({ Title = "Self", Icon = "user" }),
    Esp = Window:AddTab({ Title = "Esp", Icon = "box" }),
}

-- Criar Toggle para Box ESP
local ToggleBox = Tabs.Esp:AddToggle("BoxESP_toggle", { Title = "Box ESP" })
ToggleBox:OnChanged(function(state)
    ToggleBoxESP(state)
end)

-- Criar Toggle para Nametags
local ToggleName = Tabs.Esp:AddToggle("NameTags_toggle", { Title = "Nametags" })
ToggleName:OnChanged(function(state)
    ToggleNameTags(state)
end)

-- Criar Toggle para Aimbot
local ToggleAimbot = Tabs.AimBot:AddToggle("Aimbot_toggle", { Title = "Aimbot" })
ToggleAimbot:OnChanged(function(state)
    enableAimbot(state)
end)

-- Inputs para ajustar a distância do Aimbot e o tamanho do FOV
local DistanceInput = Tabs.AimBot:AddInput("Aimbot Distance", {
    Title = "Aimbot Distance",
    Default = aimDistance,
    Numeric = true,
    Min = 50,
    Max = 1000
})
DistanceInput:OnChanged(function(value)
    aimDistance = tonumber(value)
end)

local FovInput = Tabs.AimBot:AddInput("FOV Size", {
    Title = "FOV Size",
    Default = fovSize,
    Numeric = true,
    Min = 10,
    Max = 500
})
FovInput:OnChanged(function(value)
    fovSize = tonumber(value)
    updateFovCircle()  -- Atualizar o FOV Circle quando o valor for alterado
end)

-- Criar Toggle para Fly
local ToggleAimbot = Tabs.Self:AddToggle("Fly_toggle", { Title = "Fly" })
ToggleAimbot:OnChanged(function(state)
    toggleFly(state)
end)

-- Criar Toggle para Nc
local ToggleAimbot = Tabs.Self:AddToggle("Nc_toggle", { Title = "Nc" })
ToggleAimbot:OnChanged(function(state)
    toggleNoClip(state)
end)



updateFovCircle()


-- Notificação inicial
Fluent:Notify({
    Title = "Japa Cheat",
    Content = "Japa Cheat Injected Successfully!",
    Duration = 15
})

Window:SelectTab(AimBot)
