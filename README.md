local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local Remotes = {
    ThunderDash = ReplicatedStorage.Remotes.ThunderDash,
    Emote = ReplicatedStorage.Remotes.CustomEmote,
    Parry = ReplicatedStorage.Remotes.ParryButtonPress
}

local ParryEnabled = false
local VisualEnabled = false

local HitboxPart = Instance.new('Part')
HitboxPart.Color = Color3.fromRGB(245, 29, 0)
HitboxPart.Anchored = true
HitboxPart.Material = Enum.Material.ForceField 
HitboxPart.Shape = Enum.PartType.Ball
HitboxPart.CanCollide = false
HitboxPart.CastShadow = false
HitboxPart.Transparency = 0.75
HitboxPart.Parent = Workspace

local function PerformParry(ball)
    Remotes.Parry:Fire()
end

RunService.Heartbeat:Connect(function(deltaTime)
    local Character = LocalPlayer.Character
    if not Character then return end
    
    local HumanoidRootPart = Character:FindFirstChild("HumanoidRootPart")
    if not HumanoidRootPart then return end
    
    local CharacterPosition = HumanoidRootPart.Position
    
    for _, ball in ipairs(Workspace.Balls:GetChildren()) do
        if ball:GetAttribute('realBall') then
            local distance = (CharacterPosition - ball.Position).Magnitude
            local ballVelocity = ball.Velocity
            local ballMagnitude = ballVelocity.Magnitude / 3
            local ballVolume = math.abs(ballVelocity.X + ballVelocity.Y + ballVelocity.Z)
            
            if VisualEnabled then
                HitboxPart.Position = CharacterPosition
                HitboxPart.Size = Vector3.new(ballVolume, ballVolume, ballVolume)
            else
                HitboxPart.Position = Vector3.new(0, 100000, 0)
            end
            
            if ball:GetAttribute('target') == LocalPlayer.Name then
                if distance <= ballMagnitude or distance <= 15 then
                    PerformParry(ball)
                end
            end
        end
    end
end)

-- GUI Creation (using the provided library)
local Library = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()
local MainWindow = Library:MakeWindow({
    Name = 'Blade',
    HidePremium = true,
    SaveConfig = true,
    ConfigFolder = 'Blade'
})

local CombatTab = MainWindow:MakeTab({
    Name = 'Combat',
    Icon = 'rbxassetid://11385161113',
    PremiumOnly = false
})

CombatTab:AddToggle({
    Name = 'Auto-parry',
    Default = false,
    Callback = function(value)
        ParryEnabled = value
    end
})

CombatTab:AddToggle({
    Name = 'Visual',
    Default = false,
    Callback = function(value)
        VisualEnabled = value
    end
})

Library:Init()
