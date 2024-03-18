-- // Main Variables // --

local RunService, UserInputService, TweenService = game:GetService("RunService"), game:GetService("UserInputService"), game:GetService("TweenService")
local Player, ReplicatedStorage, Debris = game.Players.LocalPlayer, game.ReplicatedStorage, game.Debris

local Remotes = {
    ThunderDash = ReplicatedStorage.Remotes.ThunderDash,
    Emote = ReplicatedStorage.Remotes.CustomEmote,
    Parry = ReplicatedStorage.Remotes.ParryButtonPress
}

local ParryCD = false
local ParryActive = false
local Visual = false

local HitboxPart = Instance.new('Part', workspace)
HitboxPart.Color = Color3.fromRGB(255, 29, 0)
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

-- // toggles and stuff // --

Combat:AddToggle({
    Name = 'Auto-parry',
    Default = false,
    Callback = function (Value)
        ParryActive = Value
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
local function PerformParry(ball)
    local Player = game.Players.LocalPlayer
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
    local Player = game.Players.LocalPlayer
    for i, ball in pairs(workspace.Balls:GetChildren()) do
        if ball:GetAttribute('realBall') then
            local ballVelocity = ball.Velocity
            local ballMagnitude = ballVelocity.Magnitude / 3
            if Visual then
                HitboxPart.Position = Player.Character.HumanoidRootPart.Position
                HitboxPart.Size = Vector3.new(ballMagnitude, ballMagnitude, ballMagnitude)
            else
                HitboxPart.Position = Vector3.new(0, 100000, 0)
            end
            if ball:GetAttribute('target') == Player.Name and not ParryCD then
                local distance = (Player.Character.HumanoidRootPart.Position - ball.Position).Magnitude
                if distance <= ballMagnitude or distance <= 15 then
                    PerformParry(ball)
                else
                    warn('If you skid, you bad :grin:')
                end
            end
        end
    end
end)

-- // Create UI Element for Ball Speed // --
local BallSpeedLabel = MainWindow:MakeLabel({
    Name = 'Ball Speed: N/A',
    Size = UDim2.new(0, 200, 0, 30),
    Position = UDim2.new(0.5, 0, 0.8, 0),
    Background = Color3.fromRGB(30, 30, 30),
    TextColor = Color3.fromRGB(255, 255, 255),
    BorderSizePixel = 2
})

-- // Update Ball Speed in UI Element // --
RunService.Heartbeat:Connect(function(Time, DeltaTime)
    for _, ball in pairs(workspace.Balls:GetChildren()) do
        if ball:GetAttribute('realBall') then
            local ballVelocity = ball.Velocity
            local ballMagnitude = ballVelocity.Magnitude / 3
            BallSpeedLabel:SetText('Ball Speed: ' .. math.floor(ballMagnitude))
        end
    end
end)

-- // Initiate UI // --
Library:Init()

