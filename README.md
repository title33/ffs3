
-- // Main Variables // --

local RunService, UserInputService, TweenService = game:GetService("RunService"), game:GetService("UserInputService"), game:GetService("TweenService")
local Players, ReplicatedStorage, Debris = game:GetService("Players"), game:GetService("ReplicatedStorage"), game:GetService("Debris")

local Remotes = {
    ThunderDash = ReplicatedStorage.Remotes.ThunderDash,
    Emote = ReplicatedStorage.Remotes.CustomEmote,
    Parry = ReplicatedStorage.Remotes.ParryButtonPress
}

local ParryCD = false
local ParryEnabled = false
local VisualEnabled = false

local HitboxPart = Instance.new('Part')
HitboxPart.Color = Color3.fromHex('#f51d00')
HitboxPart.Material = Enum.Material.ForceField 
HitboxPart.Shape = Enum.PartType.Ball
HitboxPart.Transparency = 0.75
HitboxPart.Anchored = true
HitboxPart.CanCollide = false
HitboxPart.CastShadow = false

-- // Start of script // --
local Library = loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Orion/main/source'))()

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
local function PerformParry(ball)
    if not ParryCD then
        Remotes.Parry:Fire()
        ParryCD = true
        ball:SetAttribute('target', '')
        spawn(function() 
            wait(0.1)
            ParryCD = false
        end)
    end
end

RunService.Heartbeat:Connect(function(Time, DeltaTime)
    local LocalPlayer = Players.LocalPlayer
    for _, ball in ipairs(workspace.Balls:GetChildren()) do
        if ball:GetAttribute('realBall') then
            local distance = (LocalPlayer.Character.HumanoidRootPart.Position - ball.Position).Magnitude
            local ballVelocity = ball.Velocity
            local ballMagnitude = ballVelocity.Magnitude / 3
            local ballVolume = math.abs(ballVelocity.X + ballVelocity.Y + ballVelocity.Z)
            if VisualEnabled then
                HitboxPart.Position = LocalPlayer.Character.HumanoidRootPart.Position
                HitboxPart.Size = Vector3.new(ballVolume, ballVolume, ballVolume)
            else
                HitboxPart.Position = Vector3.new(0, 100000, 0)
            end
            if ball:GetAttribute('target') == LocalPlayer.Name and not ParryCD then
                if distance <= ballMagnitude or distance <= 15 then
                    PerformParry(ball)
                else
                    warn('If you skid, you bad :grin:')
                end
            end
        end
    end
end)

-- // extra // --

Library:Init()
