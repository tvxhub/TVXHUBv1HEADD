--# TVX HUB V1

--# TVX-MENU-v1.2

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
    ESPBoxEnabled = false,
    TeamCheckEnabled = false, -- Adicionando Team Check
    AimLockKey = Enum.KeyCode.X -- Padrão para X
}

local FOVCircle = Drawing.new("Circle")
FOVCircle.Visible = false
FOVCircle.Radius = AimbotSettings.FOV
FOVCircle.Thickness = 1
FOVCircle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y / 2)
FOVCircle.Transparency = 1 -- Invisível inicialmente
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
print("FOV Circle created with Radius:", FOVCircle.Radius)

-- press key to activate lock
local function onKeyPress(input, gameProcessedEvent)
    if input.KeyCode == AimbotSettings.AimLockKey and not gameProcessedEvent and AimbotSettings.Enabled then
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

local function isTeammate(player)
    return game.Players.LocalPlayer.Team ~= nil and player.Team == game.Players.LocalPlayer.Team
end

local function updateESP()
    for _, player in pairs(game:GetService("Players"):GetPlayers()) do
        if player ~= game.Players.LocalPlayer and (not AimbotSettings.TeamCheckEnabled or not isTeammate(player)) then
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
                if Player ~= LocalPlayer and Player.Character and Player.Character:FindFirstChild("Head") and Player.Character.Humanoid.Health > 0 and (not AimbotSettings.TeamCheckEnabled or not isTeammate(Player)) then
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
            if not AimbotSettings.AimPlayer or AimbotSettings.AimPlayer.Character.Humanoid.Health <= 0 then
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
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, TargetPosition) -- Travar a câmera no inimigo com previsão
            print("Aiming at player: " .. AimbotSettings.AimPlayer.Name)
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

Aimingsec1:Slider{
    name = "Prediction Value",
    min = 1,
    max = 350,
    def = AimbotSettings.Prediction * 1000, -- Convertendo de segundos para ms
    callback = function(value)
        AimbotSettings.Prediction = value / 1000 -- Convertendo de ms para segundos
        print("Prediction Value set to:", value, "ms")
    end
}
print("Prediction Value slider added")

VisualsSec1:Toggle{
    name = "ESP Name",
    def = false,
    callback = function(value)
        AimbotSettings.ESPNamesEnabled = value
        if not value then
            for _, ESPName in pairs(ESPNames) do
                ESPName.Visible = false
            end
        end
        print("ESP Names Enabled: " .. tostring(value))
    end
}
print("Enable ESP Names toggle added")

VisualsSec1:Toggle{
    name = "ESP Box",
    def = false,
    callback = function(value)
        AimbotSettings.ESPBoxEnabled = value
        if not value then
            for _, ESPBox in pairs(ESPBoxes) do
                ESPBox.Visible = false
            end
        end
        print("ESP Box 2D Enabled: " .. tostring(value))
    end
}
print("Enable ESP Box 2D toggle added")

-- Adicionando a aba Misc
local MiscTab = Window:Tab("Misc")
print("Misc tab created")

local MiscSec = MiscTab:Section{name = "Misc Settings", column = 1}
print("Misc section created")

MiscSec:Toggle{
    name = "Team Check",
    def = false,
    callback = function(value)
        AimbotSettings.TeamCheckEnabled = value
        print("Team Check Enabled: " .. tostring(value))
    end
}
print("Team Check toggle added")

-- Adicionando a aba Keys
local KeysTab = Window:Tab("Keys")
print("Keys tab created")

local KeysSec = KeysTab:Section{name = "AimLock Key", column = 1}
print("AimLock Key section created")

-- Função para definir a tecla de Aimlock
local function setAimLockKey(key)
    AimbotSettings.AimLockKey = key
    print("AimLock Key set to:", key.Name)
end

-- Criando caixas de seleção para cada letra
local keys = {'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'}

for _, key in ipairs(keys) do
    KeysSec:Toggle{
        name = key,
        def = false,
        callback = function(value)
            if value then
                setAimLockKey(Enum.KeyCode[key])
                -- Desativar outras teclas
                for _, otherKey in ipairs(keys) do
                    if otherKey ~= key then
                        KeysSec.Items[otherKey]:SetState(false)
                    end
                end
            end
        end
    }
    print("Added key toggle for:", key)
end

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
