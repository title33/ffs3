-- // Main Variables // --
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Player = game.Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Debris = game:GetService("Debris")

local Remotes = {
    ThunderDash = ReplicatedStorage.Remotes.ThunderDash,
    Emote = ReplicatedStorage.Remotes.CustomEmote,
    Parry = ReplicatedStorage.Remotes.ParryButtonPress
}

local ParryEnabled = false
local VisualEnabled = false

local HitboxPart = Instance.new('Part')
HitboxPart.Color = Color3.fromHex('#f51d00')
HitboxPart.Anchored = true
HitboxPart.Material = Enum.Material.ForceField 
HitboxPart.Shape = Enum.PartType.Ball
HitboxPart.CanCollide = false
HitboxPart.CastShadow = false
HitboxPart.Transparency = 0.75
HitboxPart.Parent = workspace

-- // Start of script // --
local Library = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()

local MainWindow = Library:MakeWindow({
    Name = 'Blade',
    HidePremium = true,
    SaveConfig = true,
    ConfigFolder = 'Blade'
})

-- // tabs // --
local Combat = MainWindow:MakeTab({
    Name = 'Combat',
    Icon = 'rbxassetid://11385161113',
    PremiumOnly = false
})

-- // toggles n shit // --

Combat:AddToggle({
    Name = 'Auto-parry',
    Default = false,
    Callback = function (Value)
        ParryEnabled = Value
    end
})

Combat:AddToggle({
    Name = 'Visual',
    Default = false,
    Callback = function (Value)
        VisualEnabled = Value
    end
})

-- // main stuff // --
local function Parry(ball)
    Remotes.Parry:Fire()
end

RunService.Heartbeat:Connect(function()
    local character = Player.Character
    if not character then return end

    local rootPart = character:WaitForChild("HumanoidRootPart")
    for _, ball in ipairs(workspace.Balls:GetChildren()) do
        if ball:GetAttribute('realBall') then
            local distance = (rootPart.Position - ball.Position).Magnitude
            local ballVelocity = ball.Velocity
            local ballMagnitude = ballVelocity.Magnitude / 3
            local ballVolume = math.abs(ballVelocity.X + ballVelocity.Y + ballVelocity.Z)

            if VisualEnabled then
                HitboxPart.Position = rootPart.Position
                HitboxPart.Size = Vector3.new(ballVolume, ballVolume, ballVolume)
            else
                HitboxPart.Position = Vector3.new(0, 100000, 0)
            end

            if ball:GetAttribute('target') == Player.Name then
                if distance <= ballMagnitude or distance <= 17 then
                    if ParryEnabled then
                        Parry(ball)
                    end
                end
            end
        end
    end
end)

-- // extra // --
Library:Init()
