--# TVX HUB V1 HEAD

--# TVX-MENU-v1.1

local LibraryURL = 'https://raw.githubusercontent.com/drillygzzly/Roblox-UI-Libs/main/Yun%20V2%20Lib/Yun%20V2%20Lib%20Source.lua'
loadstring(game:HttpGet(LibraryURL))()
print("Library loaded")

local Library = initLibrary()
print("Library initialized")

local Window = Library:Load({
    name = "TVX HUB V1",
    sizeX = 425,
    sizeY = 512,
    color = Color3.fromRGB(255, 255, 255)
})
print("Window created")

local Tab = Window:Tab("Aim/ESP")
print("Tab created")

local Aimingsec1 = Tab:Section{name = "Aimbot", column = 1}
local VisualsSec1 = Tab:Section{name = "Visuals", column = 2}
print("Sections created")

local AimbotSettings = {
    AimLock = false,
    AimPlayer = nil,
    Enabled = false,
    FOV = 80,
    FOVEnabled = false,
    Prediction = 0.170, -- 170ms
    PredictionEnabled = false,
    ESPNamesEnabled = false,
    ESPBoxEnabled = false
}

local FOVCircle = Drawing.new("Circle")
FOVCircle.Visible = false
FOVCircle.Radius = AimbotSettings.FOV
FOVCircle.Thickness = 1
FOVCircle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y / 2)
FOVCircle.Transparency = 1 -- Invisível inicialmente
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
print("FOV Circle created with Radius:", FOVCircle.Radius)

-- press X to activate lock
local function onKeyPress(input, gameProcessedEvent)
    if input.KeyCode == Enum.KeyCode.X and not gameProcessedEvent and AimbotSettings.Enabled then
        AimbotSettings.AimLock = not AimbotSettings.AimLock
        print("AimLock: " .. tostring(AimbotSettings.AimLock))
    end
end

game:GetService("UserInputService").InputBegan:Connect(onKeyPress)

local ESPNames = {}
local ESPBoxes = {}

local function createESPName(player)
    local ESPName = Drawing.new("Text")
    ESPName.Text = player.Name
    ESPName.Size = 8
    ESPName.Center = true
    ESPName.Outline = true
    ESPName.Color = Color3.fromRGB(255, 255, 255)
    ESPNames[player.Name] = ESPName
end

local function createESPBox(player)
    local ESPBox = Drawing.new("Square")
    ESPBox.Thickness = 1
    ESPBox.Color = Color3.fromRGB(255, 255, 255) -- Caixa na cor branca
    ESPBox.Filled = false
    ESPBoxes[player.Name] = ESPBox
end

local function updateESP()
    for _, player in pairs(game:GetService("Players"):GetPlayers()) do
        if player ~= game.Players.LocalPlayer then
            if AimbotSettings.ESPNamesEnabled and not ESPNames[player.Name] then
                createESPName(player)
            end
            if AimbotSettings.ESPBoxEnabled and not ESPBoxes[player.Name] then
                createESPBox(player)
            end

            local character = player.Character
            if character and character:FindFirstChild("Head") then
                local head = character.Head
                local screenPosition, onScreen = workspace.CurrentCamera:WorldToViewportPoint(head.Position)

                if onScreen then
                    if AimbotSettings.ESPNamesEnabled then
                        local ESPName = ESPNames[player.Name]
                        ESPName.Position = Vector2.new(screenPosition.X, screenPosition.Y - 25) -- Ajusta a posição do texto
                        ESPName.Visible = true
                    end

                    if AimbotSettings.ESPBoxEnabled then
                        local ESPBox = ESPBoxes[player.Name]
                        local size = Vector2.new(25, 25) -- Tamanho fixo e reduzido
                        ESPBox.Position = Vector2.new(screenPosition.X - size.X / 2, screenPosition.Y - size.Y / 2)
                        ESPBox.Size = size
                        ESPBox.Visible = true
                    end
                else
                    if AimbotSettings.ESPNamesEnabled then
                        ESPNames[player.Name].Visible = false
                    end
                    if AimbotSettings.ESPBoxEnabled then
                        ESPBoxes[player.Name].Visible = false
                    end
                end
            else
                if AimbotSettings.ESPNamesEnabled then
                    ESPNames[player.Name].Visible = false
                end
                if AimbotSettings.ESPBoxEnabled then
                    ESPBoxes[player.Name].Visible = false
                end
            end
        end
    end
end

game:GetService("RunService").RenderStepped:Connect(function()
    if AimbotSettings.FOVEnabled then
        FOVCircle.Visible = true
        FOVCircle.Transparency = 1 -- Visível quando ativado
        FOVCircle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y / 2)
        print("FOV Circle is visible at Position:", FOVCircle.Position)
    else
        FOVCircle.Visible = false
        FOVCircle.Transparency = 0 -- Invisível quando desativado
    end

    if AimbotSettings.Enabled then
        local Players = game:GetService("Players")
        local LocalPlayer = Players.LocalPlayer
        local Camera = workspace.CurrentCamera

        local function IsPlayerInFOV(Player)
            local ScreenPoint = Camera:WorldToViewportPoint(Player.Character.Head.Position)
            local DistanceFromCenter = (Vector2.new(ScreenPoint.X, ScreenPoint.Y) - Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)).Magnitude
            print("Distance from center:", DistanceFromCenter, "FOV:", AimbotSettings.FOV)
            return DistanceFromCenter <= AimbotSettings.FOV
        end

        local function GetClosestPlayer()
            local ClosestPlayer = nil
            local ShortestDistance = math.huge

            for _, Player in ipairs(Players:GetPlayers()) do
                if Player ~= LocalPlayer and Player.Character and Player.Character:FindFirstChild("Head") and Player.Character.Humanoid.Health > 0 then
                    local Distance = (Player.Character.Head.Position - LocalPlayer.Character.Head.Position).magnitude
                    if Distance < ShortestDistance and IsPlayerInFOV(Player) then
                        ShortestDistance = Distance
                        ClosestPlayer = Player
                    end
                end
            end

            return ClosestPlayer
        end

        if AimbotSettings.AimLock then
            if not AimbotSettings.AimPlayer or AimbotSettings.AimPlayer.Character.Humanoid.Health <= 0 or not IsPlayerInFOV(AimbotSettings.AimPlayer) then
                AimbotSettings.AimPlayer = GetClosestPlayer()
            end
        else
            AimbotSettings.AimPlayer = nil
        end

        if AimbotSettings.AimPlayer and AimbotSettings.AimPlayer.Character and AimbotSettings.AimPlayer.Character:FindFirstChild("Head") then
            local TargetPosition = AimbotSettings.AimPlayer.Character.Head.Position
            if AimbotSettings.PredictionEnabled then
                TargetPosition = TargetPosition + (AimbotSettings.AimPlayer.Character.Head.Velocity * AimbotSettings.Prediction)
            end
            if IsPlayerInFOV(AimbotSettings.AimPlayer) then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, TargetPosition) -- Travar a câmera no inimigo com previsão
                print("Aiming at player: " .. AimbotSettings.AimPlayer.Name)
            end
        end
    end

    updateESP()
end)

Aimingsec1:Toggle{
    name = "AimLock",
    def = false,
    callback = function(value)
        AimbotSettings.Enabled = value
        print("Aimbot Enabled: " .. tostring(value))
    end
}
print("Enable AimLock toggle added")

Aimingsec1:Toggle{
    name = "Active FOV",
    def = false,
    callback = function(value)
        AimbotSettings.FOVEnabled = value
        FOVCircle.Visible = value
        FOVCircle.Transparency = value and 0 or 1 -- Certificando-se de que é visível quando ativado
        print("FOV Circle Enabled: " .. tostring(value))
    end
}
print("Enable FOV Circle toggle added")

Aimingsec1:Slider{
    name = "FOV Value",
    min = 1,
    max = 200,
    def = AimbotSettings.FOV,
    callback = function(value)
        AimbotSettings.FOV = value
        FOVCircle.Radius = value
        print("FOV Value set to:", value)
    end
}
print("FOV Value slider added")

Aimingsec1:Toggle{
    name = "Active Prediction",
    def = false,
    callback = function(value)
        AimbotSettings.PredictionEnabled = value
        print("Prediction Enabled: " .. tostring(value))
    end
}
print("Enable Prediction toggle added")

VisualsSec1:Toggle{
    name = "ESP Name",
    def = false,
    callback = function(value)
        AimbotSettings.ESPNamesEnabled = value
        print("ESP Names Enabled: " .. tostring(value))
    end
}
print("Enable ESP Names toggle added")

VisualsSec1:Toggle{
    name = "ESP Box",
    def = false,
    callback = function(value)
        AimbotSettings.ESPBoxEnabled = value
        print("ESP Box 2D Enabled: " .. tostring(value))
    end
}
print("Enable ESP Box 2D toggle added")

local function cleanESP()
    for playerName, ESPName in pairs(ESPNames) do
        if not game:GetService("Players"):FindFirstChild(playerName) then
            ESPName:Remove()
            ESPNames[playerName] = nil
        end
    end

    for playerName, ESPBox in pairs(ESPBoxes) do
        if not game:GetService("Players"):FindFirstChild(playerName) then
            ESPBox:Remove()
            ESPBoxes[playerName] = nil
        end
    end
end

game:GetService("Players").PlayerRemoving:Connect(function(player)
    cleanESP()
end)

game:GetService("Players").PlayerAdded:Connect(function(player)
    cleanESP()
end)
