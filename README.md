-- // Main Variables // --

local RunService, UserInputService, TweenService = game.RunService, game.UserInputService, game.TweenService
local Player, ReplicatedStorage, Debris = game.Players.LocalPlayer, game.ReplicatedStorage, game.Debris

local Remotes = {
    ThunderDash = ReplicatedStorage.Remotes.ThunderDash,
    Emote = ReplicatedStorage.Remotes.CustomEmote,
    Parry = ReplicatedStorage.Remotes.ParryButtonPress
}

local Parry = false
local Visual = false

local HitboxPart = Instance.new('Part', workspace)
HitboxPart.Color = Color3.fromHex('#f51d00')
HitboxPart.Anchored = true
HitboxPart.Material = Enum.Material.ForceField 
HitboxPart.Shape = Enum.PartType.Ball
HitboxPart.CanCollide = false
HitboxPart.CastShadow = false
HitboxPart.Transparency = 0.75

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
        Parry = Value
    end
})

Combat:AddToggle({
    Name = 'Visual',
    Default = false,
    Callback = function (Value)
        Visual = Value
    end
})


-- // main stuff // --
local function Parry(OBJ)
    local Player = game.Players.LocalPlayer
    Remotes.Parry:Fire()
end

RunService.Heartbeat:Connect(function(Time, DeltaTime)
    local Player = game.Players.LocalPlayer
    for i, ball in pairs(workspace.Balls:GetChildren()) do
        if ball:GetAttribute('realBall') then
            local distance = (Player.Character.HumanoidRootPart.Position - ball.Position).Magnitude
            local ballVelocity = ball.Velocity
            local ballMagnitude = ballVelocity.Magnitude / 3
            local ballVolume = math.abs(ballVelocity.X + ballVelocity.Y + ballVelocity.Z)
            if Visual then
                HitboxPart.Position = Player.Character.HumanoidRootPart.Position
                HitboxPart.Size = Vector3.new(ballVolume, ballVolume, ballVolume)
            else
                HitboxPart.Position = Vector3.new(0, 100000, 0)
            end
            if ball:GetAttribute('target') == Player.Name then
                if distance <= ballMagnitude or distance <= 15 then
                    Parry(ball)
                end
            end
        end
    end
end)

-- // extra // --

Library:Init()
